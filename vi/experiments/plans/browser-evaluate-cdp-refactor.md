---
summary: >-
  Kế hoạch: cách ly browser act:evaluate khỏi hàng đợi Playwright bằng CDP, với
  các deadline end-to-end và giải quyết ref an toàn hơn
read_when:
  - >-
    Đang làm việc trên các vấn đề về timeout, abort hoặc queue blocking của
    `act:evaluate` trong trình duyệt
  - Lập kế hoạch cách ly dựa trên CDP để đánh giá thực thi
owner: openclaw
status: draft
last_updated: '2026-02-10'
title: Browser Evaluate CDP Refactor
x-i18n:
  source_path: experiments\plans\browser-evaluate-cdp-refactor.md
  source_hash: 7176b8e2d41c3114657e57262938089635c7dbb3617c20ac796c2f8957e9dac2
  provider: anthropic
  model: claude-sonnet-4-20250514
  generated_at: '2026-02-27T19:53:37.687Z'
---

# Kế hoạch Tái cấu trúc Browser Evaluate CDP

## Bối cảnh

`act:evaluate` thực thi JavaScript do người dùng cung cấp trong trang. Hiện tại nó chạy qua Playwright
(`page.evaluate` hoặc `locator.evaluate`). Playwright tuần tự hóa các lệnh CDP cho mỗi trang, vì vậy một evaluate bị kẹt hoặc chạy lâu có thể chặn hàng đợi lệnh trang và làm cho mọi hành động sau đó
trên tab đó trông "bị kẹt".

PR #13498 thêm một lưới an toàn thực dụng (evaluate có giới hạn, lan truyền hủy bỏ và phục hồi tốt nhất). Tài liệu này mô tả một tái cấu trúc lớn hơn làm cho `act:evaluate` vốn dĩ
bị cô lập khỏi Playwright để một evaluate bị kẹt không thể làm tê liệt các hoạt động Playwright bình thường.
## Mục tiêu

- `act:evaluate` không thể vĩnh viễn chặn các hành động trình duyệt sau này trên cùng một tab.
- Timeouts là nguồn sự thật duy nhất từ đầu đến cuối để người gọi có thể dựa vào một ngân sách.
- Abort và timeout được xử lý theo cách tương tự trên HTTP và trong quá trình gửi trong quy trình.
- Hỗ trợ nhắm mục tiêu phần tử cho evaluate mà không cần tắt mọi thứ trên Playwright.
- Duy trì khả năng tương thích ngược cho những người gọi và tải trọng hiện có.
## Các mục tiêu không phải

- Thay thế tất cả các hành động trình duyệt (click, type, wait, v.v.) bằng các triển khai CDP.
- Loại bỏ mạng lưới an toàn hiện có được giới thiệu trong PR #13498 (nó vẫn là một fallback hữu ích).
- Giới thiệu các khả năng không an toàn mới ngoài cổng `browser.evaluateEnabled` hiện có.
- Thêm cách ly quy trình (worker process/thread) cho evaluate. Nếu chúng tôi vẫn thấy các trạng thái bị kẹt khó khôi phục sau lần refactor này, đó là một ý tưởng tiếp theo.
## Kiến trúc hiện tại (Tại sao nó bị treo)

Ở mức cao:

- Những người gọi gửi `act:evaluate` đến dịch vụ điều khiển trình duyệt.
- Trình xử lý tuyến đường gọi vào Playwright để thực thi JavaScript.
- Playwright tuần tự hóa các lệnh trang, vì vậy một evaluate không bao giờ kết thúc sẽ chặn hàng đợi.
- Một hàng đợi bị treo có nghĩa là các hoạt động click/type/wait sau này trên tab có thể dường như bị treo.
## Kiến trúc Đề xuất

### 1. Truyền Deadline

Giới thiệu một khái niệm ngân sách duy nhất và lấy mọi thứ từ nó:

- Người gọi đặt `timeoutMs` (hoặc deadline trong tương lai).
- Timeout yêu cầu bên ngoài, logic xử lý route và ngân sách thực thi bên trong trang
  đều sử dụng cùng một ngân sách, với một chút dư địa nơi cần thiết cho chi phí tuần tự hóa.
- Abort được truyền dưới dạng `AbortSignal` ở mọi nơi để hủy bỏ nhất quán.

Hướng triển khai:

- Thêm một trình trợ giúp nhỏ (ví dụ `createBudget({ timeoutMs, signal })`) trả về:
  - `signal`: AbortSignal được liên kết
  - `deadlineAtMs`: deadline tuyệt đối
  - `remainingMs()`: ngân sách còn lại cho các hoạt động con
- Sử dụng trình trợ giúp này trong:
  - `src/browser/client-fetch.ts` (gửi HTTP và trong quá trình xử lý)
  - `src/node-host/runner.ts` (đường dẫn proxy)
  - triển khai hành động trình duyệt (Playwright và CDP)

### 2. Công cụ Evaluate Riêng biệt (Đường dẫn CDP)

Thêm triển khai evaluate dựa trên CDP không chia sẻ hàng đợi lệnh mỗi trang của Playwright. Thuộc tính chính là transport evaluate là một kết nối WebSocket riêng biệt và một phiên CDP riêng biệt được đính kèm vào mục tiêu.

Hướng triển khai:

- Mô-đun mới, ví dụ `src/browser/cdp-evaluate.ts`, cái mà:
  - Kết nối với điểm cuối CDP được cấu hình (socket cấp trình duyệt).
  - Sử dụng `Target.attachToTarget({ targetId, flatten: true })` để lấy `sessionId`.
  - Chạy một trong những điều sau:
    - `Runtime.evaluate` cho evaluate cấp trang, hoặc
    - `DOM.resolveNode` cộng với `Runtime.callFunctionOn` cho evaluate phần tử.
  - Khi timeout hoặc abort:
    - Gửi `Runtime.terminateExecution` theo cách tốt nhất cho phiên.
    - Đóng WebSocket và trả về lỗi rõ ràng.

Ghi chú:

- Điều này vẫn thực thi JavaScript trong trang, vì vậy chấm dứt có thể có tác dụng phụ. Lợi ích là nó không làm tắc hàng đợi Playwright và nó có thể hủy bỏ ở lớp transport bằng cách giết phiên CDP.

### 3. Câu chuyện Ref (Nhắm mục tiêu phần tử mà không cần viết lại toàn bộ)

Phần khó là nhắm mục tiêu phần tử. CDP cần một DOM handle hoặc `backendDOMNodeId`, trong khi ngày nay hầu hết các hành động trình duyệt sử dụng locator Playwright dựa trên ref từ snapshot.

Cách tiếp cận được đề xuất: giữ ref hiện có, nhưng đính kèm một id CDP có thể giải quyết được tùy chọn.

#### 3.1 Mở rộng Thông tin Ref được lưu trữ

Mở rộng siêu dữ liệu ref vai trò được lưu trữ để tùy chọn bao gồm một id CDP:

- Ngày nay: `{ role, name, nth }`
- Đề xuất: `{ role, name, nth, backendDOMNodeId?: number }`
Điều này giữ cho tất cả các hành động dựa trên Playwright hiện có hoạt động và cho phép CDP evaluate chấp nhận
giá trị `ref` tương tự khi `backendDOMNodeId` có sẵn.

#### 3.2 Điền backendDOMNodeId Tại Thời điểm Snapshot

Khi tạo role snapshot:

1. Tạo bản đồ role ref hiện có như hôm nay (role, name, nth).
2. Lấy AX tree thông qua CDP (`Accessibility.getFullAXTree`) và tính toán bản đồ song song của
   `(role, name, nth) -> backendDOMNodeId` sử dụng các quy tắc xử lý trùng lặp tương tự.
3. Hợp nhất id trở lại thông tin ref được lưu trữ cho tab hiện tại.

Nếu ánh xạ không thành công cho một ref, để `backendDOMNodeId` không xác định. Điều này làm cho tính năng
là best-effort và an toàn để triển khai.

#### 3.3 Đánh giá Hành vi Với Ref

Trong `act:evaluate`:

- Nếu `ref` có mặt và có `backendDOMNodeId`, chạy element evaluate thông qua CDP.
- Nếu `ref` có mặt nhưng không có `backendDOMNodeId`, quay lại đường dẫn Playwright (với
  lưới an toàn).

Cách thoát tùy chọn:

- Mở rộng hình dạng yêu cầu để chấp nhận `backendDOMNodeId` trực tiếp cho những người gọi nâng cao (và
  để gỡ lỗi), trong khi giữ `ref` làm giao diện chính.

### 4. Giữ Đường dẫn Phục hồi Cuối cùng

Ngay cả với CDP evaluate, có những cách khác để khóa một tab hoặc kết nối. Giữ
các cơ chế phục hồi hiện có (chấm dứt thực thi + ngắt kết nối Playwright) làm giải pháp cuối cùng
cho:

- những người gọi cũ
- các môi trường nơi CDP attach bị chặn
- các trường hợp cạnh Playwright không mong đợi
## Kế hoạch triển khai (Một lần lặp)

### Các sản phẩm giao

- Một CDP based evaluate engine chạy bên ngoài hàng đợi lệnh per-page của Playwright.
- Một ngân sách timeout/abort end-to-end duy nhất được sử dụng nhất quán bởi các caller và handler.
- Ref metadata có thể tùy chọn mang `backendDOMNodeId` cho element evaluate.
- `act:evaluate` ưu tiên CDP engine khi có thể và quay lại Playwright khi không.
- Các bài kiểm tra chứng minh rằng một evaluate bị kẹt không làm cản trở các hành động sau.
- Nhật ký/số liệu giúp làm cho các lỗi và fallback trở nên rõ ràng.

### Danh sách kiểm tra triển khai

1. Thêm một helper "budget" được chia sẻ để liên kết `timeoutMs` + upstream `AbortSignal` thành:
   - một `AbortSignal` duy nhất
   - một deadline tuyệt đối
   - một helper `remainingMs()` cho các hoạt động hạ lưu
2. Cập nhật tất cả các đường dẫn caller để sử dụng helper đó sao cho `timeoutMs` có nghĩa giống nhau ở mọi nơi:
   - `src/browser/client-fetch.ts` (HTTP và in-process dispatch)
   - `src/node-host/runner.ts` (node proxy path)
   - CLI wrappers gọi `/act` (thêm `--timeout-ms` vào `browser evaluate`)
3. Triển khai `src/browser/cdp-evaluate.ts`:
   - kết nối với socket CDP cấp trình duyệt
   - `Target.attachToTarget` để lấy một `sessionId`
   - chạy `Runtime.evaluate` cho page evaluate
   - chạy `DOM.resolveNode` + `Runtime.callFunctionOn` cho element evaluate
   - khi timeout/abort: best-effort `Runtime.terminateExecution` rồi đóng socket
4. Mở rộng stored role ref metadata để tùy chọn bao gồm `backendDOMNodeId`:
   - giữ hành vi `{ role, name, nth }` hiện tại cho các hành động Playwright
   - thêm `backendDOMNodeId?: number` cho CDP element targeting
5. Điền `backendDOMNodeId` trong quá trình tạo snapshot (best-effort):
   - tìm nạp AX tree qua CDP (`Accessibility.getFullAXTree`)
   - tính toán `(role, name, nth) -> backendDOMNodeId` và hợp nhất vào stored ref map
   - nếu mapping không rõ ràng hoặc bị thiếu, để id không xác định
6. Cập nhật `act:evaluate` routing:
   - nếu không có `ref`: luôn sử dụng CDP evaluate
   - nếu `ref` phân giải thành một `backendDOMNodeId`: sử dụng CDP element evaluate
   - nếu không: quay lại Playwright evaluate (vẫn bị giới hạn và có thể hủy)
7. Giữ đường dẫn phục hồi "last resort" hiện tại như một fallback, không phải đường dẫn mặc định.
8. Thêm các bài kiểm tra:
   - stuck evaluate hết thời gian trong ngân sách và click/type tiếp theo thành công
   - abort hủy evaluate (client disconnect hoặc timeout) và mở khóa các hành động tiếp theo
   - mapping failures sạch sẽ quay lại Playwright
9. Thêm khả năng quan sát:
   - evaluate duration và timeout counters
   - terminateExecution usage
   - fallback rate (CDP -> Playwright) và lý do

### Tiêu chí chấp nhận

- Một `act:evaluate` bị treo cố ý trả về trong ngân sách caller và không làm cản trở tab cho các hành động sau.
- `timeoutMs` hoạt động nhất quán trên CLI, agent tool, node proxy, và in-process calls.
- Nếu `ref` có thể được ánh xạ tới `backendDOMNodeId`, element evaluate sử dụng CDP; nếu không đường dẫn fallback vẫn bị giới hạn và có thể phục hồi.
## Kế hoạch Kiểm thử

- Kiểm thử đơn vị:
  - `(role, name, nth)` logic khớp giữa các tham chiếu vai trò và các nút cây AX.
  - Hành vi trợ giúp ngân sách (không gian trống, toán học thời gian còn lại).
- Kiểm thử tích hợp:
  - CDP evaluate timeout trả về trong ngân sách và không chặn hành động tiếp theo.
  - Abort hủy evaluate và kích hoạt kết thúc nỗ lực tốt nhất.
- Kiểm thử hợp đồng:
  - Đảm bảo `BrowserActRequest` và `BrowserActResponse` vẫn tương thích.
## Rủi Ro Và Biện Pháp Giảm Thiểu

- Ánh xạ không hoàn hảo:
  - Biện pháp giảm thiểu: ánh xạ tốt nhất, quay lại Playwright evaluate, và thêm công cụ gỡ lỗi.
- `Runtime.terminateExecution` có tác dụng phụ:
  - Biện pháp giảm thiểu: chỉ sử dụng khi hết thời gian chờ/hủy bỏ và ghi lại hành vi trong lỗi.
- Overhead bổ sung:
  - Biện pháp giảm thiểu: chỉ tìm nạp cây AX khi yêu cầu ảnh chụp, lưu vào bộ nhớ đệm theo mục tiêu, và giữ phiên CDP ngắn hạn.
- Hạn chế của tiếp sức mở rộng:
  - Biện pháp giảm thiểu: sử dụng API đính kèm cấp trình duyệt khi các socket trên mỗi trang không khả dụng, và giữ đường dẫn Playwright hiện tại làm dự phòng.
## Các Câu Hỏi Mở

- Công cụ mới có nên được cấu hình dưới dạng `playwright`, `cdp`, hoặc `auto`?
- Chúng ta có muốn công khai định dạng "nodeRef" mới cho người dùng nâng cao hay chỉ giữ `ref`?
- Các bản chụp khung hình và bản chụp có phạm vi bộ chọn nên tham gia vào ánh xạ AX như thế nào?