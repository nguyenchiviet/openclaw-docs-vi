---
title: Các tác nhân mặc định.md
summary: >-
  Hướng dẫn tác nhân OpenClaw mặc định và danh sách Skills cho thiết lập trợ lý
  cá nhân
read_when:
  - Bắt đầu một phiên tác nhân OpenClaw mới
  - Kích hoạt hoặc kiểm tra Skills mặc định
x-i18n:
  source_path: reference\AGENTS.default.md
  source_hash: 7d544c51781ee5b635f36a5e393ffbc92652769bd296e2c63ea3445db518a0a2
  provider: anthropic
  model: claude-sonnet-4-20250514
  generated_at: '2026-03-01T05:17:07.516Z'
---

# AGENTS.md — Trợ lý cá nhân OpenClaw (mặc định)

## Chạy lần đầu (khuyến nghị)

OpenClaw sử dụng một thư mục không gian làm việc chuyên dụng cho agent. Mặc định: __OC_I19N_0000__ (có thể cấu hình qua __OC_I19N_0001__).

1. Tạo không gian làm việc (nếu chưa tồn tại):

``__OC_I19N_0002__`__OC_I19N_0003__`__OC_I19N_0004__`__OC_I19N_0005__`__OC_I19N_0006__`__OC_I19N_0007__agents.defaults.workspace__OC_I19N_0008__~__OC_I19N_0009__`__OC_I19N_0010__``
## Mặc định an toàn

- Không đổ thư mục hoặc bí mật vào cuộc trò chuyện.
- Không chạy các lệnh phá hoại trừ khi được yêu cầu rõ ràng.
- Không gửi các phản hồi một phần/truyền phát đến các nền tảng nhắn tin bên ngoài (chỉ gửi các phản hồi cuối cùng).

## Bắt đầu phiên (bắt buộc)

- Đọc `SOUL.md`, `USER.md`, `memory.md`, và hôm nay+hôm qua trong `memory/`.
- Thực hiện trước khi phản hồi.

## Linh hồn (bắt buộc)

- `SOUL.md` định nghĩa danh tính, giọng điệu và ranh giới. Hãy giữ nó luôn cập nhật.
- Nếu bạn thay đổi `SOUL.md`, hãy thông báo cho người dùng.
- Bạn là một phiên bản mới trong mỗi phiên; tính liên tục nằm trong các tệp này.

## Không gian chia sẻ (được khuyến nghị)

- Bạn không phải là tiếng nói của người dùng; hãy cẩn thận trong các cuộc trò chuyện nhóm hoặc kênh công khai.
- Không chia sẻ dữ liệu riêng tư, thông tin liên hệ hoặc ghi chú nội bộ.

## Hệ thống bộ nhớ (khuyến nghị)

- Nhật ký hàng ngày: `memory/YYYY-MM-DD.md` (tạo `memory/` nếu cần).
- Bộ nhớ dài hạn: `memory.md` cho các sự kiện, tùy chọn và quyết định bền vững.
- Khi bắt đầu phiên, đọc hôm nay + hôm qua + `memory.md` nếu có.
- Ghi lại: các quyết định, tùy chọn, ràng buộc, các vòng lặp mở.
- Tránh các bí mật trừ khi được yêu cầu rõ ràng.

## Công cụ & Skills

- Các công cụ nằm trong Skills; hãy làm theo `SKILL.md` của từng skill khi bạn cần.
- Giữ các ghi chú dành riêng cho môi trường trong `TOOLS.md` (Ghi chú cho Skills).

## Mẹo sao lưu (khuyên dùng)

Nếu bạn coi không gian làm việc này là “bộ nhớ” của Clawd, hãy biến nó thành một kho lưu trữ git (lý tưởng nhất là riêng tư) để __OC_I19N_0000__ và các tệp bộ nhớ của bạn được sao lưu.

``__OC_I19N_0001__``
## OpenClaw làm gì

- Chạy Gateway WhatsApp + agent lập trình Pi để trợ lý có thể đọc/ghi tin nhắn, lấy ngữ cảnh và chạy Skills thông qua máy Mac chủ.
- Ứng dụng macOS quản lý quyền (ghi màn hình, thông báo, micrô) và cung cấp CLI __OC_I19N_0000__ thông qua tệp nhị phân đi kèm.
- Tin nhắn riêng theo mặc định gộp vào phiên __OC_I19N_0001__ của agent; các nhóm vẫn được cách ly dưới dạng __OC_I19N_0002__ (phòng/kênh: __OC_I19N_0003__); nhịp tim duy trì các tác vụ nền hoạt động.

## Skills cốt lõi (bật trong Cài đặt → Skills)

- **mcporter** — Thời gian chạy máy chủ công cụ/CLI để quản lý các backend Skills bên ngoài.
- **Peekaboo** — Chụp ảnh màn hình macOS nhanh chóng với phân tích thị giác AI tùy chọn.
- **camsnap** — Chụp khung hình, clip hoặc cảnh báo chuyển động từ camera an ninh RTSP/ONVIF.
- **oracle** — CLI agent sẵn sàng cho OpenAI với tính năng phát lại phiên và điều khiển trình duyệt.
- **eightctl** — Kiểm soát giấc ngủ của bạn, từ terminal.
- **imsg** — Gửi, đọc, truyền phát iMessage & SMS.
- **wacli** — WhatsApp CLI: đồng bộ hóa, tìm kiếm, gửi.
- **discord** — Các hành động Discord: phản ứng, nhãn dán, thăm dò ý kiến. Sử dụng các mục tiêu `user:<id>` hoặc `channel:<id>` (ID số trần có thể gây nhầm lẫn).
- **gog** — Google Suite CLI: Gmail, Lịch, Drive, Danh bạ.
- **spotify-player** — Ứng dụng khách Spotify trên terminal để tìm kiếm/xếp hàng/điều khiển phát lại.
- **sag** — Lời nói ElevenLabs với trải nghiệm người dùng nói kiểu Mac; truyền phát đến loa theo mặc định.
- **Sonos CLI** — Điều khiển loa Sonos (khám phá/trạng thái/phát lại/âm lượng/nhóm) từ các tập lệnh.
- **blucli** — Phát, nhóm và tự động hóa trình phát BluOS từ các tập lệnh.
- **OpenHue CLI** — Điều khiển đèn Philips Hue cho các cảnh và tự động hóa.
- **OpenAI Whisper** — Chuyển giọng nói thành văn bản cục bộ để đọc chính tả nhanh và bản ghi thư thoại.
- **Gemini CLI** — Các mô hình Google Gemini từ terminal để hỏi đáp nhanh.
- **agent-tools** — Bộ công cụ tiện ích cho các tự động hóa và tập lệnh trợ giúp.
## Ghi chú sử dụng

- Ưu tiên sử dụng CLI `openclaw` để viết script; ứng dụng macOS xử lý quyền.
- Chạy cài đặt từ tab Skills; nó sẽ ẩn nút nếu tệp nhị phân đã có sẵn.
- Giữ nhịp tim (heartbeats) được bật để trợ lý có thể lên lịch nhắc nhở, giám sát hộp thư đến và kích hoạt chụp ảnh.
- Giao diện người dùng Canvas chạy toàn màn hình với các lớp phủ gốc. Tránh đặt các điều khiển quan trọng ở các cạnh trên cùng bên trái/trên cùng bên phải/dưới cùng; thêm các khoảng trống rõ ràng trong bố cục và không dựa vào các vùng an toàn (safe-area insets).
- Để xác minh dựa trên trình duyệt, hãy sử dụng `openclaw browser` (tab/trạng thái/ảnh chụp màn hình) với hồ sơ Chrome do OpenClaw quản lý.
- Để kiểm tra DOM, hãy sử dụng `openclaw browser eval|query|dom|snapshot` (và `--json`/`--out` khi bạn cần đầu ra máy).
- Để tương tác, hãy sử dụng `openclaw browser click|type|hover|drag|select|upload|press|wait|navigate|back|evaluate|run` (nhấp/gõ yêu cầu tham chiếu ảnh chụp nhanh; sử dụng `evaluate` cho bộ chọn CSS).