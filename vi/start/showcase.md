---
title: Trưng bày
description: Real-world OpenClaw projects from the community
summary: Các dự án và tích hợp do cộng đồng xây dựng được hỗ trợ bởi OpenClaw
read_when:
  - Tìm kiếm các ví dụ sử dụng OpenClaw thực tế
  - Cập nhật các điểm nổi bật của dự án cộng đồng
x-i18n:
  source_path: start\showcase.md
  source_hash: 998b827ad2ae62beec663ce7cf2a6255beb5fac122bb657caaf63fe847b1129e
  provider: anthropic
  model: claude-sonnet-4-20250514
  generated_at: '2026-02-27T20:26:21.516Z'
---

# Trưng bày

Các dự án thực tế từ cộng đồng. Xem mọi người đang xây dựng gì với OpenClaw.

<Info>
**Muốn được giới thiệu?** Chia sẻ dự án của bạn trong [#showcase trên Discord](https://discord.gg/clawd) hoặc [tag @openclaw trên X](https://x.com/openclaw).
</Info>
## 🎥 OpenClaw in Action

Hướng dẫn thiết lập đầy đủ (28m) của VelvetShark.

<div
  style={{
    position: "relative",
    paddingBottom: "56.25%",
    height: 0,
    overflow: "hidden",
    borderRadius: 16,
  }}
>
  <iframe
    src="https://www.youtube-nocookie.com/embed/SaWSPZoPX34"
    title="OpenClaw: AI tự lưu trữ mà Siri đáng lẽ phải có (Thiết lập đầy đủ)"
    style={{ position: "absolute", top: 0, left: 0, width: "100%", height: "100%" }}
    frameBorder="0"
    loading="lazy"
    allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share"
    allowFullScreen
  />
</div>

[Xem trên YouTube](https://www.youtube.com/watch?v=SaWSPZoPX34)

<div
  style={{
    position: "relative",
    paddingBottom: "56.25%",
    height: 0,
    overflow: "hidden",
    borderRadius: 16,
  }}
>
  <iframe
    src="https://www.youtube-nocookie.com/embed/mMSKQvlmFuQ"
    title="Video giới thiệu OpenClaw"
    style={{ position: "absolute", top: 0, left: 0, width: "100%", height: "100%" }}
    frameBorder="0"
    loading="lazy"
    allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share"
    allowFullScreen
  />
</div>

[Xem trên YouTube](https://www.youtube.com/watch?v=mMSKQvlmFuQ)

<div
  style={{
    position: "relative",
    paddingBottom: "56.25%",
    height: 0,
    overflow: "hidden",
    borderRadius: 16,
  }}
>
  <iframe
    src="https://www.youtube-nocookie.com/embed/5kkIJNUGFho"
    title="Giới thiệu cộng đồng OpenClaw"
style={{ position: "absolute", top: 0, left: 0, width: "100%", height: "100%" }}
    frameBorder="0"
    loading="lazy"
    allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share"
    allowFullScreen
  />
</div>

[Xem trên YouTube](https://www.youtube.com/watch?v=5kkIJNUGFho)
## 🆕 Mới từ Discord

<CardGroup cols={2}>

<Card title="PR Review → Phản hồi Telegram" icon="code-pull-request" href="https://x.com/i/status/2010878524543131691">
  **@bangnokia** • `review` `github` `telegram`

OpenCode hoàn thành thay đổi → mở PR → OpenClaw xem xét diff và trả lời trong Telegram với "những gợi ý nhỏ" cộng với một quyết định merge rõ ràng (bao gồm các bản sửa lỗi quan trọng cần áp dụng trước).

  <img src="/assets/showcase/pr-review-telegram.jpg" alt="Phản hồi xem xét PR của OpenClaw được gửi trong Telegram" />
</Card>

<Card title="Wine Cellar Skill trong Vài Phút" icon="wine-glass" href="https://x.com/i/status/2010916352454791216">
  **@prades_maxime** • `skills` `local` `csv`

Yêu cầu "Robby" (@openclaw) tạo một skill wine cellar cục bộ. Nó yêu cầu một mẫu xuất CSV + nơi lưu trữ, sau đó xây dựng/kiểm tra skill nhanh chóng (962 chai trong ví dụ).

  <img src="/assets/showcase/wine-cellar-skill.jpg" alt="OpenClaw xây dựng skill wine cellar cục bộ từ CSV" />
</Card>

<Card title="Tesco Shop Autopilot" icon="cart-shopping" href="https://x.com/i/status/2009724862470689131">
  **@marchattonhere** • `automation` `browser` `shopping`

Kế hoạch bữa ăn hàng tuần → những mặt hàng thường xuyên → đặt lịch giao hàng → xác nhận đơn hàng. Không có API, chỉ kiểm soát trình duyệt.

  <img src="/assets/showcase/tesco-shop.jpg" alt="Tự động hóa mua sắm Tesco qua chat" />
</Card>

<Card title="SNAG Screenshot-to-Markdown" icon="scissors" href="https://github.com/am-will/snag">
  **@am-will** • `devtools` `screenshots` `markdown`

Phím tắt một vùng màn hình → Gemini vision → Markdown tức thì trong clipboard của bạn.

  <img src="/assets/showcase/snag.png" alt="Công cụ SNAG screenshot-to-markdown" />
</Card>

<Card title="Agents UI" icon="window-maximize" href="https://releaseflow.net/kitze/agents-ui">
  **@kitze** • `ui` `skills` `sync`

Ứng dụng desktop để quản lý skills/commands trên Agents, Claude, Codex và OpenClaw.

  <img src="/assets/showcase/agents-ui.jpg" alt="Ứng dụng Agents UI" />
</Card>

<Card title="Telegram Voice Notes (papla.media)" icon="microphone" href="https://papla.media/docs">
  **Community** • `voice` `tts` `telegram`

Bao bọc TTS papla.media và gửi kết quả dưới dạng ghi âm thoại Telegram (không có tự động phát bực bội).

  <img src="/assets/showcase/papla-tts.jpg" alt="Đầu ra ghi âm thoại Telegram từ TTS" />
</Card>

<Card title="CodexMonitor" icon="eye" href="https://clawhub.com/odrobnik/codexmonitor">
  **@odrobnik** • `devtools` `codex` `brew`

Trợ giúp được cài đặt qua Homebrew để liệt kê/kiểm tra/theo dõi các phiên OpenAI Codex cục bộ (CLI + VS Code).

  <img src="/assets/showcase/codexmonitor.png" alt="CodexMonitor trên ClawHub" />
</Card>
<Card title="Điều khiển Máy in 3D Bambu" icon="print" href="https://clawhub.com/tobiasbischoff/bambu-cli">
  **@tobiasbischoff** • `hardware` `3d-printing` `skill`

Điều khiển và khắc phục sự cố máy in BambuLab: trạng thái, công việc, camera, AMS, hiệu chỉnh và nhiều hơn nữa.

  <img src="/assets/showcase/bambu-cli.png" alt="Bambu CLI skill on ClawHub" />
</Card>

<Card title="Giao thông Vienna (Wiener Linien)" icon="train" href="https://clawhub.com/hjanuschka/wienerlinien">
  **@hjanuschka** • `travel` `transport` `skill`

Thời gian khởi hành thực tế, gián đoạn, trạng thái thang máy và định tuyến cho giao thông công cộng Vienna.

  <img src="/assets/showcase/wienerlinien.png" alt="Wiener Linien skill on ClawHub" />
</Card>

<Card title="ParentPay Bữa ăn Trường học" icon="utensils" href="#">
  **@George5562** • `automation` `browser` `parenting`

Đặt bữa ăn trường học Anh tự động qua ParentPay. Sử dụng tọa độ chuột để nhấp vào ô bảng một cách đáng tin cậy.
</Card>

<Card title="R2 Upload (Gửi cho tôi các tệp của tôi)" icon="cloud-arrow-up" href="https://clawhub.com/skills/r2-upload">
  **@julianengel** • `files` `r2` `presigned-urls`

Tải lên Cloudflare R2/S3 và tạo liên kết tải xuống được ký trước an toàn. Hoàn hảo cho các instance OpenClaw từ xa.
</Card>

<Card title="Ứng dụng iOS qua Telegram" icon="mobile" href="#">
  **@coard** • `ios` `xcode` `testflight`

Xây dựng một ứng dụng iOS hoàn chỉnh với bản đồ và ghi âm giọng nói, triển khai lên TestFlight hoàn toàn qua trò chuyện Telegram.

  <img src="/assets/showcase/ios-testflight.jpg" alt="iOS app on TestFlight" />
</Card>

<Card title="Trợ lý Sức khỏe Oura Ring" icon="heart-pulse" href="#">
  **@AS** • `health` `oura` `calendar`

Trợ lý sức khỏe AI cá nhân tích hợp dữ liệu vòng Oura với lịch, cuộc hẹn và lịch tập gym.

  <img src="/assets/showcase/oura-health.png" alt="Oura ring health assistant" />
</Card>
<Card title="Kev's Dream Team (14+ Agents)" icon="robot" href="https://github.com/adam91holt/orchestrated-ai-articles">
  **@adam91holt** • `multi-agent` `orchestration` `architecture` `manifesto`

14+ agent dưới một Gateway với orchestrator Opus 4.5 phân công cho các worker Codex. Bài viết kỹ thuật toàn diện [technical write-up](https://github.com/adam91holt/orchestrated-ai-articles) bao gồm danh sách Dream Team, lựa chọn mô hình, sandboxing, webhook, heartbeat và luồng phân công. [Clawdspace](https://github.com/adam91holt/clawdspace) cho sandboxing agent. [Bài viết blog](https://adams-ai-journey.ghost.io/2026-the-year-of-the-orchestrator/).
</Card>

<Card title="Linear CLI" icon="terminal" href="https://github.com/Finesssee/linear-cli">
  **@NessZerra** • `devtools` `linear` `cli` `issues`

CLI cho Linear tích hợp với quy trình làm việc agent (Claude Code, OpenClaw). Quản lý các vấn đề, dự án và quy trình từ terminal. PR bên ngoài đầu tiên đã được hợp nhất!
</Card>

<Card title="Beeper CLI" icon="message" href="https://github.com/blqke/beepcli">
  **@jules** • `messaging` `beeper` `cli` `automation`

Đọc, gửi và lưu trữ tin nhắn qua Beeper Desktop. Sử dụng Beeper local MCP API để các agent có thể quản lý tất cả các cuộc trò chuyện của bạn (iMessage, WhatsApp, v.v.) ở một nơi.
</Card>
</CardGroup>

## 🤖 Tự động hóa & Quy trình làm việc

<CardGroup cols={2}>

<Card title="Điều khiển Máy lọc không khí Winix" icon="wind" href="https://x.com/antonplex/status/2010518442471006253">
  **@antonplex** • `automation` `hardware` `air-quality`

Claude Code phát hiện và xác nhận các điều khiển máy lọc, sau đó OpenClaw tiếp quản để quản lý chất lượng không khí trong phòng.

  <img src="/assets/showcase/winix-air-purifier.jpg" alt="Điều khiển máy lọc không khí Winix qua OpenClaw" />
</Card>

<Card title="Ảnh Bầu trời Đẹp từ Camera" icon="camera" href="https://x.com/signalgaining/status/2010523120604746151">
  **@signalgaining** • `automation` `camera` `skill` `images`

Được kích hoạt bởi camera trên mái nhà: yêu cầu OpenClaw chụp ảnh bầu trời bất cứ khi nào trông đẹp — nó đã thiết kế một skill và chụp ảnh.

  <img src="/assets/showcase/roof-camera-sky.jpg" alt="Ảnh chụp bầu trời từ camera trên mái nhà được OpenClaw ghi lại" />
</Card>

<Card title="Cảnh Tóm tắt Buổi sáng Trực quan" icon="robot" href="https://x.com/buddyhadry/status/2010005331925954739">
  **@buddyhadry** • `automation` `briefing` `images` `telegram`

Một lời nhắc được lên lịch tạo ra một hình ảnh "cảnh" duy nhất mỗi sáng (thời tiết, nhiệm vụ, ngày tháng, bài viết/trích dẫn yêu thích) thông qua một nhân vật OpenClaw.
</Card>

<Card title="Đặt sân Padel" icon="calendar-check" href="https://github.com/joshp123/padel-cli">
  **@joshp123** • `automation` `booking` `cli`
  
  Trình kiểm tra tính khả dụng Playtomic + CLI đặt chỗ. Không bao giờ bỏ lỡ một sân trống nữa.
  
  <img src="/assets/showcase/padel-screenshot.jpg" alt="ảnh chụp màn hình padel-cli" />
</Card>

<Card title="Tiếp nhận Kế toán" icon="file-invoice-dollar">
  **Cộng đồng** • `automation` `email` `pdf`
  
  Thu thập PDF từ email, chuẩn bị tài liệu cho cố vấn thuế. Kế toán hàng tháng tự động.
</Card>

<Card title="Chế độ Nhà phát triển Couch Potato" icon="couch" href="https://davekiss.com">
  **@davekiss** • `telegram` `website` `migration` `astro`

Xây dựng lại toàn bộ trang web cá nhân qua Telegram trong khi xem Netflix — Notion → Astro, 18 bài viết được di chuyển, DNS sang Cloudflare. Không bao giờ mở laptop.
</Card>

<Card title="Agent Tìm kiếm Việc làm" icon="briefcase">
  **@attol8** • `automation` `api` `skill`

Tìm kiếm danh sách việc làm, so khớp với các từ khóa CV, và trả về các cơ hội liên quan kèm liên kết. Được xây dựng trong 30 phút sử dụng JSearch API.
</Card>

<Card title="Trình xây dựng Skill Jira" icon="diagram-project" href="https://x.com/jdrhyne/status/2008336434827002232">
  **@jdrhyne** • `automation` `jira` `skill` `devtools`

OpenClaw kết nối với Jira, sau đó tạo một skill mới ngay lập tức (trước khi nó tồn tại trên ClawHub).
</Card>
<Card title="Todoist Skill via Telegram" icon="list-check" href="https://x.com/iamsubhrajyoti/status/2009949389884920153">
  **@iamsubhrajyoti** • `automation` `todoist` `skill` `telegram`

Tự động hóa các tác vụ Todoist và OpenClaw tạo ra Skill trực tiếp trong cuộc trò chuyện Telegram.
</Card>

<Card title="Phân tích TradingView" icon="chart-line">
  **@bheem1798** • `finance` `browser` `automation`

Đăng nhập vào TradingView thông qua tự động hóa trình duyệt, chụp ảnh biểu đồ và thực hiện phân tích kỹ thuật theo yêu cầu. Không cần API—chỉ cần kiểm soát trình duyệt.
</Card>

<Card title="Slack Auto-Support" icon="slack">
  **@henrymascot** • `slack` `automation` `support`

Theo dõi kênh Slack công ty, phản hồi hữu ích và chuyển tiếp thông báo đến Telegram. Tự động sửa lỗi sản xuất trong ứng dụng được triển khai mà không cần được yêu cầu.
</Card>
## 🧠 Kiến thức & Bộ nhớ

<CardGroup cols={2}>

<Card title="xuezh Chinese Learning" icon="language" href="https://github.com/joshp123/xuezh">
  **@joshp123** • `learning` `voice` `skill`
  
  Công cụ học tiếng Trung với phản hồi phát âm và luồng học tập qua OpenClaw.
  
  <img src="/assets/showcase/xuezh-pronunciation.jpeg" alt="xuezh pronunciation feedback" />
</Card>

<Card title="WhatsApp Memory Vault" icon="vault">
  **Community** • `memory` `transcription` `indexing`
  
  Nhập toàn bộ xuất khẩu WhatsApp, chuyển đổi 1k+ ghi chú thoại, kiểm tra chéo với nhật ký git, xuất báo cáo markdown được liên kết.
</Card>

<Card title="Karakeep Semantic Search" icon="magnifying-glass" href="https://github.com/jamesbrooksco/karakeep-semantic-search">
  **@jamesbrooksco** • `search` `vector` `bookmarks`
  
  Thêm tìm kiếm vector vào dấu trang Karakeep bằng cách sử dụng nhúng Qdrant + OpenAI/Ollama.
</Card>

<Card title="Inside-Out-2 Memory" icon="brain">
  **Community** • `memory` `beliefs` `self-model`
  
  Trình quản lý bộ nhớ riêng biệt chuyển đổi tệp phiên thành bộ nhớ → niềm tin → mô hình tự thân phát triển.
</Card>

</CardGroup>
## 🎙️ Giọng nói & Điện thoại

<CardGroup cols={2}>

<Card title="Clawdia Phone Bridge" icon="phone" href="https://github.com/alejandroOPI/clawdia-bridge">
  **@alejandroOPI** • `voice` `vapi` `bridge`
  
  Cầu nối HTTP Vapi voice assistant ↔ OpenClaw. Các cuộc gọi điện thoại gần như thời gian thực với agent của bạn.
</Card>

<Card title="OpenRouter Transcription" icon="microphone" href="https://clawhub.com/obviyus/openrouter-transcribe">
  **@obviyus** • `transcription` `multilingual` `skill`

Chuyển đổi âm thanh đa ngôn ngữ qua OpenRouter (Gemini, v.v.). Có sẵn trên ClawHub.
</Card>

</CardGroup>
## 🏗️ Cơ sở hạ tầng & Triển khai

<CardGroup cols={2}>

<Card title="Tiện ích bổ sung Home Assistant" icon="home" href="https://github.com/ngutman/openclaw-ha-addon">
  **@ngutman** • `homeassistant` `docker` `raspberry-pi`
  
  Gateway OpenClaw chạy trên Home Assistant OS với hỗ trợ SSH tunnel và trạng thái bền vững.
</Card>

<Card title="Skill Home Assistant" icon="toggle-on" href="https://clawhub.com/skills/homeassistant">
  **ClawHub** • `homeassistant` `skill` `automation`
  
  Kiểm soát và tự động hóa các thiết bị Home Assistant thông qua ngôn ngữ tự nhiên.
</Card>

<Card title="Đóng gói Nix" icon="snowflake" href="https://github.com/openclaw/nix-openclaw">
  **@openclaw** • `nix` `packaging` `deployment`
  
  Cấu hình OpenClaw nixified đầy đủ tính năng cho các triển khai có thể tái tạo.
</Card>

<Card title="Lịch CalDAV" icon="calendar" href="https://clawhub.com/skills/caldav-calendar">
  **ClawHub** • `calendar` `caldav` `skill`
  
  Skill lịch sử dụng khal/vdirsyncer. Tích hợp lịch tự lưu trữ.
</Card>

</CardGroup>
## 🏠 Nhà & Phần cứng

<CardGroup cols={2}>

<Card title="GoHome Automation" icon="house-signal" href="https://github.com/joshp123/gohome">
  **@joshp123** • `home` `nix` `grafana`
  
  Tự động hóa nhà thông minh hỗ trợ Nix với OpenClaw làm giao diện, cùng với các bảng điều khiển Grafana đẹp mắt.
  
  <img src="/assets/showcase/gohome-grafana.png" alt="Bảng điều khiển GoHome Grafana" />
</Card>

<Card title="Roborock Vacuum" icon="robot" href="https://github.com/joshp123/gohome/tree/main/plugins/roborock">
  **@joshp123** • `vacuum` `iot` `plugin`
  
  Điều khiển robot hút bụi Roborock của bạn thông qua cuộc trò chuyện tự nhiên.
  
  <img src="/assets/showcase/roborock-screenshot.jpg" alt="Trạng thái Roborock" />
</Card>

</CardGroup>
## 🌟 Dự án cộng đồng

<CardGroup cols={2}>

<Card title="StarSwap Marketplace" icon="star" href="https://star-swap.com/">
  **Cộng đồng** • `marketplace` `astronomy` `webapp`
  
  Thị trường thiết bị thiên văn đầy đủ. Được xây dựng với/xung quanh hệ sinh thái OpenClaw.
</Card>

</CardGroup>

---
## Gửi Dự Án Của Bạn

Có gì muốn chia sẻ? Chúng tôi rất muốn giới thiệu nó!

<Steps>
  <Step title="Chia Sẻ">
    Đăng bài trong [#showcase trên Discord](https://discord.gg/clawd) hoặc [tweet @openclaw](https://x.com/openclaw)
  </Step>
  <Step title="Bao Gồm Chi Tiết">
    Cho chúng tôi biết nó làm gì, liên kết đến repo/demo, chia sẻ ảnh chụp màn hình nếu bạn có
  </Step>
  <Step title="Được Giới Thiệu">
    Chúng tôi sẽ thêm các dự án nổi bật vào trang này
  </Step>
</Steps>