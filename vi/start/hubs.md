---
summary: Các Hub kết nối đến mọi tài liệu OpenClaw
read_when:
  - Bạn muốn có một bản đồ hoàn chỉnh của tài liệu
title: Trung tâm Tài liệu
x-i18n:
  source_path: start\hubs.md
  source_hash: a89ce9e262cea864e2e7caa8dd512321f77c449d11669e0e69ba304fa0fbe81f
  provider: anthropic
  model: claude-sonnet-4-20250514
  generated_at: '2026-02-27T20:24:37.958Z'
---

# Trung tâm tài liệu

<Note>
Nếu bạn mới sử dụng OpenClaw, hãy bắt đầu với [Bắt đầu](/start/getting-started).
</Note>

Sử dụng các trung tâm này để khám phá mọi trang, bao gồm các bài viết chuyên sâu và tài liệu tham khảo không xuất hiện trong thanh điều hướng bên trái.
## Bắt đầu ở đây

- [Chỉ mục](/)
- [Bắt đầu](/start/getting-started)
- [Bắt đầu nhanh](/start/quickstart)
- [Thiết lập ban đầu](/start/onboarding)
- [Trình hướng dẫn](/start/wizard)
- [Cài đặt](/start/setup)
- [Bảng điều khiển](http://127.0.0.1:18789/)
- [Trợ giúp](/help)
- [Thư mục tài liệu](/start/docs-directory)
- [Cấu hình](/gateway/configuration)
- [Ví dụ cấu hình](/gateway/configuration-examples)
- [Trợ lý OpenClaw](/start/openclaw)
- [Trưng bày](/start/showcase)
- [Lore](/start/lore)
## Cài đặt + cập nhật

- [Docker](/install/docker)
- [Nix](/install/nix)
- [Cập nhật / quay lại phiên bản trước](/install/updating)
- [Quy trình Bun (/install/bun)
## Các khái niệm cốt lõi

- [Kiến trúc](/concepts/architecture)
- [Tính năng](/concepts/features)
- [Trung tâm mạng](/network)
- [Thời gian chạy agent](/concepts/agent)
- [Không gian làm việc agent](/concepts/agent-workspace)
- [Bộ nhớ](/concepts/memory)
- [Vòng lặp agent](/concepts/agent-loop)
- [Truyền phát + chia nhỏ](/concepts/streaming)
- [Định tuyến đa agent](/concepts/multi-agent)
- [Nén dữ liệu](/concepts/compaction)
- [Phiên](/concepts/session)
- [Phiên (/concepts/sessions)
- [Dọn dẹp phiên](/concepts/session-pruning)
- [Công cụ phiên](/concepts/session-tool)
- [Hàng đợi](/concepts/queue)
- [Lệnh gạch chéo](/tools/slash-commands)
- [Bộ điều hợp RPC](/reference/rpc)
- [Lược đồ TypeBox](/concepts/typebox)
- [Xử lý múi giờ](/concepts/timezone)
- [Trạng thái hiện diện](/concepts/presence)
- [Khám phá + giao thức truyền tải](/gateway/discovery)
- [Bonjour](/gateway/bonjour)
- [Định tuyến kênh](/channels/channel-routing)
- [Nhóm](/channels/groups)
- [Tin nhắn nhóm](/channels/group-messages)
- [Chuyển đổi dự phòng mô hình](/concepts/model-failover)
- [OAuth](/concepts/oauth)
## Nhà cung cấp + đầu vào

- [Trung tâm kênh trò chuyện](/channels)
- [Trung tâm nhà cung cấp mô hình](/providers/models)
- [WhatsApp](/channels/whatsapp)
- [Telegram](/channels/telegram)
- [Slack](/channels/slack)
- [Discord](/channels/discord)
- [Mattermost](/channels/mattermost) (plugin)
- [Signal](/channels/signal)
- [BlueBubbles (/channels/bluebubbles)
- [iMessage (/channels/imessage)
- [Phân tích vị trí](/channels/location)
- [WebChat](/web/webchat)
- [Webhooks](/automation/webhook)
- [Gmail Pub/Sub](/automation/gmail-pubsub)
## Gateway + hoạt động

- [Hướng dẫn Gateway](/gateway)
- [Mô hình mạng](/gateway/network-model)
- [Ghép nối Gateway](/gateway/pairing)
- [Khóa Gateway](/gateway/gateway-lock)
- [Quy trình nền](/gateway/background-process)
- [Sức khỏe](/gateway/health)
- [Nhịp tim](/gateway/heartbeat)
- [Doctor](/gateway/doctor)
- [Ghi nhật ký](/gateway/logging)
- [Sandboxing](/gateway/sandboxing)
- [Bảng điều khiển](/web/dashboard)
- [Giao diện điều khiển](/web/control-ui)
- [Truy cập từ xa](/gateway/remote)
- [README Gateway từ xa](/gateway/remote-gateway-readme)
- [Tailscale](/gateway/tailscale)
- [Bảo mật](/gateway/security)
- [Khắc phục sự cố](/gateway/troubleshooting)
## Công cụ + tự động hóa

- [Bề mặt công cụ](/tools)
- [OpenProse](/prose)
- [Tham chiếu CLI](/cli)
- [Công cụ Exec](/tools/exec)
- [Chế độ nâng cao](/tools/elevated)
- [Cron jobs](/automation/cron-jobs)
- [Cron vs Heartbeat](/automation/cron-vs-heartbeat)
- [Thinking + verbose](/tools/thinking)
- [Mô hình](/concepts/models)
- [Sub-agents](/tools/subagents)
- [Agent send CLI](/tools/agent-send)
- [Giao diện Terminal](/web/tui)
- [Điều khiển trình duyệt](/tools/browser)
- [Trình duyệt (/tools/browser-linux-troubleshooting)
- [Bình chọn](/automation/poll)
## Nodes, phương tiện, giọng nói

- [Tổng quan về Nodes](/nodes)
- [Camera](/nodes/camera)
- [Hình ảnh](/nodes/images)
- [Âm thanh](/nodes/audio)
- [Lệnh vị trí](/nodes/location-command)
- [Kích hoạt bằng giọng nói](/nodes/voicewake)
- [Chế độ nói chuyện](/nodes/talk)
## Các nền tảng

- [Tổng quan về các nền tảng](/platforms)
- [macOS](/platforms/macos)
- [iOS](/platforms/ios)
- [Android](/platforms/android)
- [Windows](/platforms/windows)
- [Linux](/platforms/linux)
- [Giao diện web](/web)
## Ứng dụng đi kèm macOS (nâng cao)

- [Thiết lập phát triển macOS](/platforms/mac/dev-setup)
- [Thanh menu macOS](/platforms/mac/menu-bar)
- [Kích hoạt bằng giọng nói macOS](/platforms/mac/voicewake)
- [Lớp phủ giọng nói macOS](/platforms/mac/voice-overlay)
- [WebChat macOS](/platforms/mac/webchat)
- [Canvas macOS](/platforms/mac/canvas)
- [Tiến trình con macOS](/platforms/mac/child-process)
- [Sức khỏe macOS](/platforms/mac/health)
- [Biểu tượng macOS](/platforms/mac/icon)
- [Ghi nhật ký macOS](/platforms/mac/logging)
- [Quyền macOS](/platforms/mac/permissions)
- [Điều khiển từ xa macOS](/platforms/mac/remote)
- [Ký macOS](/platforms/mac/signing)
- [Phát hành macOS](/platforms/mac/release)
- [Gateway macOS](/platforms/mac/bundled-gateway)
- [XPC macOS](/platforms/mac/xpc)
- [Skills macOS](/platforms/mac/skills)
- [Peekaboo macOS](/platforms/mac/peekaboo)
## Không gian làm việc + mẫu

- [Skills](/tools/skills)
- [ClawHub](/tools/clawhub)
- [Cấu hình Skills](/tools/skills-config)
- [AGENTS mặc định](/reference/AGENTS.default)
- [Mẫu: AGENTS](/reference/templates/AGENTS)
- [Mẫu: BOOTSTRAP](/reference/templates/BOOTSTRAP)
- [Mẫu: HEARTBEAT](/reference/templates/HEARTBEAT)
- [Mẫu: IDENTITY](/reference/templates/IDENTITY)
- [Mẫu: SOUL](/reference/templates/SOUL)
- [Mẫu: TOOLS](/reference/templates/TOOLS)
- [Mẫu: USER](/reference/templates/USER)
## Thử nghiệm (khám phá)

- [Giao thức cấu hình thiết lập ban đầu](/experiments/onboarding-config-protocol)
- [Nghiên cứu: bộ nhớ](/experiments/research/memory)
- [Khám phá cấu hình mô hình](/experiments/proposals/model-config)
## Dự án

- [Ghi nhận](/reference/credits)
## Kiểm tra + phát hành

- [Kiểm tra](/reference/test)
- [Danh sách kiểm tra phát hành](/reference/RELEASING)
- [Mô hình thiết bị](/reference/device-models)