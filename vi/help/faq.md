---
summary: 'Các câu hỏi thường gặp về thiết lập, cấu hình và sử dụng OpenClaw'
read_when:
  - >-
    Trả lời các câu hỏi thường gặp về cài đặt, cài đặt, onboarding hoặc hỗ trợ
    thời gian chạy
  - >-
    Phân loại các vấn đề do người dùng báo cáo trước khi tiến hành gỡ lỗi sâu
    hơn
title: Câu hỏi thường gặp
x-i18n:
  source_path: help\faq.md
  source_hash: 41caa7ac69c67e5dff4de4c61483ee9ac18cf71b05f66a726675707d8c0ff242
  provider: anthropic
  model: claude-sonnet-4-20250514
  generated_at: '2026-02-27T20:05:40.241Z'
---

# Câu hỏi thường gặp

Câu trả lời nhanh cộng với khắc phục sự cố sâu hơn cho các thiết lập thực tế (phát triển cục bộ, VPS, đa agent, khóa OAuth/API, chuyển đổi dự phòng mô hình). Để chẩn đoán thời gian chạy, xem [Khắc phục sự cố](/gateway/troubleshooting). Để tham khảo cấu hình đầy đủ, xem [Cấu hình](/gateway/configuration).
## Mục lục

- [Bắt đầu nhanh và thiết lập lần đầu]
  - [Tôi bị mắc kẹt, cách nhanh nhất để giải quyết là gì?](#im-stuck-whats-the-fastest-way-to-get-unstuck)
  - [Cách được khuyến nghị để cài đặt và thiết lập OpenClaw là gì?](#whats-the-recommended-way-to-install-and-set-up-openclaw)
  - [Làm cách nào để mở bảng điều khiển sau khi thiết lập ban đầu?](#how-do-i-open-the-dashboard-after-onboarding)
  - [Làm cách nào để xác thực bảng điều khiển (#how-do-i-authenticate-the-dashboard-token-on-localhost-vs-remote)
  - [Tôi cần runtime nào?](#what-runtime-do-i-need)
  - [Nó có chạy trên Raspberry Pi không?](#does-it-run-on-raspberry-pi)
  - [Có mẹo nào cho cài đặt Raspberry Pi không?](#any-tips-for-raspberry-pi-installs)
  - [Nó bị mắc kẹt ở "wake up my friend" / thiết lập ban đầu sẽ không hoàn thành. Bây giờ sao?](#it-is-stuck-on-wake-up-my-friend-onboarding-will-not-hatch-what-now)
  - [Tôi có thể di chuyển thiết lập của mình sang máy mới không (#can-i-migrate-my-setup-to-a-new-machine-mac-mini-without-redoing-onboarding)
  - [Tôi có thể xem những gì mới trong phiên bản mới nhất ở đâu?](#where-do-i-see-what-is-new-in-the-latest-version)
  - [Tôi không thể truy cập docs.openclaw.ai (#i-cant-access-docsopenclawai-ssl-error-what-now)
  - [Sự khác biệt giữa phiên bản ổn định và beta là gì?](#whats-the-difference-between-stable-and-beta)
  - [Làm cách nào để cài đặt phiên bản beta, và sự khác biệt giữa beta và dev là gì?](#how-do-i-install-the-beta-version-and-whats-the-difference-between-beta-and-dev)
  - [Làm cách nào để thử các bit mới nhất?](#how-do-i-try-the-latest-bits)
  - [Cài đặt và thiết lập ban đầu thường mất bao lâu?](#how-long-does-install-and-onboarding-usually-take)
  - [Trình cài đặt bị mắc kẹt? Làm cách nào để tôi nhận được phản hồi chi tiết hơn?](#installer-stuck-how-do-i-get-more-feedback)
  - [Cài đặt Windows nói git không tìm thấy hoặc openclaw không được nhận dạng](#windows-install-says-git-not-found-or-openclaw-not-recognized)
  - [Tài liệu không trả lời câu hỏi của tôi - làm cách nào để tôi nhận được câu trả lời tốt hơn?](#the-docs-didnt-answer-my-question-how-do-i-get-a-better-answer)
  - [Làm cách nào để cài đặt OpenClaw trên Linux?](#how-do-i-install-openclaw-on-linux)
  - [Làm cách nào để cài đặt OpenClaw trên VPS?](#how-do-i-install-openclaw-on-a-vps)
  - [Hướng dẫn cài đặt cloud/VPS ở đâu?](#where-are-the-cloudvps-install-guides)
  - [Tôi có thể yêu cầu OpenClaw tự cập nhật không?](#can-i-ask-openclaw-to-update-itself)
  - [Trình hướng dẫn thiết lập ban đầu thực sự làm gì?](#what-does-the-onboarding-wizard-actually-do)
  - [Tôi có cần đăng ký Claude hoặc OpenAI để chạy cái này không?](#do-i-need-a-claude-or-openai-subscription-to-run-this)
  - [Tôi có thể sử dụng đăng ký Claude Max mà không cần khóa API không](#can-i-use-claude-max-subscription-without-an-api-key)
  - [Xác thực "setup-token" của Anthropic hoạt động như thế nào?](#how-does-anthropic-setuptoken-auth-work)
  - [Tôi tìm setup-token của Anthropic ở đâu?](#where-do-i-find-an-anthropic-setuptoken)
  - [Bạn có hỗ trợ xác thực đăng ký Claude không (#do-you-support-claude-subscription-auth-claude-pro-or-max)
  - [Tại sao tôi lại thấy `HTTP 429: rate_limit_error` từ Anthropic?](#why-am-i-seeing-http-429-ratelimiterror-from-anthropic)
  - [AWS Bedrock có được hỗ trợ không?](#is-aws-bedrock-supported)
  - [Xác thực Codex hoạt động như thế nào?](#how-does-codex-auth-work)
  - [Bạn có hỗ trợ xác thực đăng ký OpenAI không (#do-you-support-openai-subscription-auth-codex-oauth)
  - [Làm cách nào để thiết lập Gemini CLI OAuth](#how-do-i-set-up-gemini-cli-oauth)
  - [Mô hình cục bộ có được sử dụng cho các cuộc trò chuyện bình thường không?](#is-a-local-model-ok-for-casual-chats)
  - [Làm cách nào để giữ lưu lượng mô hình được lưu trữ trong một khu vực cụ thể?](#how-do-i-keep-hosted-model-traffic-in-a-specific-region)
  - [Tôi có phải mua Mac Mini để cài đặt cái này không?](#do-i-have-to-buy-a-mac-mini-to-install-this)
  - [Tôi có cần Mac mini để hỗ trợ iMessage không?](#do-i-need-a-mac-mini-for-imessage-support)
  - [Nếu tôi mua Mac mini để chạy OpenClaw, tôi có thể kết nối nó với MacBook Pro của mình không?](#if-i-buy-a-mac-mini-to-run-openclaw-can-i-connect-it-to-my-macbook-pro)
  - [Tôi có thể sử dụng Bun không?](#can-i-use-bun)
  - [Telegram: cái gì vào `allowFrom`?](#telegram-what-goes-in-allowfrom)
  - [Nhiều người có thể sử dụng một số WhatsApp với các phiên bản OpenClaw khác nhau không?](#can-multiple-people-use-one-whatsapp-number-with-different-openclaw-instances)
  - [Tôi có thể chạy agent "trò chuyện nhanh" và agent "Opus để viết mã" không?](#can-i-run-a-fast-chat-agent-and-an-opus-for-coding-agent)
  - [Homebrew có hoạt động trên Linux không?](#does-homebrew-work-on-linux)
  - [Sự khác biệt giữa phiên bản có thể hack (#whats-the-difference-between-the-hackable-git-install-and-npm-install)
  - [Tôi có thể chuyển đổi giữa cài đặt npm và git sau này không?](#can-i-switch-between-npm-and-git-installs-later)
  - [Tôi nên chạy Gateway trên laptop hay VPS?](#should-i-run-the-gateway-on-my-laptop-or-a-vps)
  - [Việc chạy OpenClaw trên một máy chuyên dụng quan trọng đến mức nào?](#how-important-is-it-to-run-openclaw-on-a-dedicated-machine)
  - [Yêu cầu VPS tối thiểu và hệ điều hành được khuyến nghị là gì?](#what-are-the-minimum-vps-requirements-and-recommended-os)
  - [Tôi có thể chạy OpenClaw trong VM và yêu cầu là gì](#can-i-run-openclaw-in-a-vm-and-what-are-the-requirements)
- [OpenClaw là gì?](#what-is-openclaw)
  - [OpenClaw là gì, trong một đoạn?](#what-is-openclaw-in-one-paragraph)
  - [Đề xuất giá trị là gì?](#whats-the-value-proposition)
  - [Tôi vừa thiết lập xong, tôi nên làm gì trước tiên](#i-just-set-it-up-what-should-i-do-first)
  - [Năm trường hợp sử dụng hàng ngày hàng đầu cho OpenClaw là gì](#what-are-the-top-five-everyday-use-cases-for-openclaw)
  - [OpenClaw có thể giúp với tiếp cận lead gen, quảng cáo và blog cho SaaS không](#can-openclaw-help-with-lead-gen-outreach-ads-and-blogs-for-a-saas)
  - [Những ưu điểm so với Claude Code để phát triển web là gì?](#what-are-the-advantages-vs-claude-code-for-web-development)
- [Skills và tự động hóa](#skills-and-automation)
- [Làm cách nào để tùy chỉnh Skills mà không làm bẩn repo?](#how-do-i-customize-skills-without-keeping-the-repo-dirty)
  - [Tôi có thể tải Skills từ một thư mục tùy chỉnh không?](#can-i-load-skills-from-a-custom-folder)
  - [Làm cách nào để sử dụng các mô hình khác nhau cho các tác vụ khác nhau?](#how-can-i-use-different-models-for-different-tasks)
  - [Bot bị đóng băng khi thực hiện công việc nặng. Làm cách nào để tôi chuyển tải công việc đó?](#the-bot-freezes-while-doing-heavy-work-how-do-i-offload-that)
  - [Cron hoặc nhắc nhở không kích hoạt. Tôi nên kiểm tra cái gì?](#cron-or-reminders-do-not-fire-what-should-i-check)
  - [Làm cách nào để cài đặt Skills trên Linux?](#how-do-i-install-skills-on-linux)
  - [OpenClaw có thể chạy các tác vụ theo lịch trình hoặc liên tục trong nền không?](#can-openclaw-run-tasks-on-a-schedule-or-continuously-in-the-background)
  - [Tôi có thể chạy các Skills chỉ dành cho Apple macOS từ Linux không?](#can-i-run-apple-macos-only-skills-from-linux)
  - [Bạn có tích hợp Notion hoặc HeyGen không?](#do-you-have-a-notion-or-heygen-integration)
  - [Làm cách nào để cài đặt tiện ích mở rộng Chrome để kiểm soát trình duyệt?](#how-do-i-install-the-chrome-extension-for-browser-takeover)
- [Sandboxing và bộ nhớ](#sandboxing-and-memory)
  - [Có tài liệu sandboxing chuyên dụng không?](#is-there-a-dedicated-sandboxing-doc)
  - [Làm cách nào để liên kết một thư mục máy chủ vào sandbox?](#how-do-i-bind-a-host-folder-into-the-sandbox)
  - [Bộ nhớ hoạt động như thế nào?](#how-does-memory-work)
  - [Bộ nhớ liên tục quên những thứ. Làm cách nào để tôi làm cho nó lưu lại?](#memory-keeps-forgetting-things-how-do-i-make-it-stick)
  - [Bộ nhớ có tồn tại mãi mãi không? Giới hạn là gì?](#does-memory-persist-forever-what-are-the-limits)
  - [Tìm kiếm bộ nhớ ngữ nghĩa có yêu cầu khóa API OpenAI không?](#does-semantic-memory-search-require-an-openai-api-key)
- [Nơi các thứ được lưu trữ trên đĩa](#where-things-live-on-disk)
  - [Tất cả dữ liệu được sử dụng với OpenClaw có được lưu cục bộ không?](#is-all-data-used-with-openclaw-saved-locally)
  - [OpenClaw lưu trữ dữ liệu của nó ở đâu?](#where-does-openclaw-store-its-data)
  - [AGENTS.md / SOUL.md / USER.md / MEMORY.md nên được lưu ở đâu?](#where-should-agentsmd-soulmd-usermd-memorymd-live)
  - [Chiến lược sao lưu được khuyến nghị là gì?](#whats-the-recommended-backup-strategy)
  - [Làm cách nào để hoàn toàn gỡ cài đặt OpenClaw?](#how-do-i-completely-uninstall-openclaw)
  - [Các agent có thể hoạt động bên ngoài không gian làm việc không?](#can-agents-work-outside-the-workspace)
  - [Tôi ở chế độ từ xa - kho lưu trữ phiên ở đâu?](#im-in-remote-mode-where-is-the-session-store)
- [Cơ bản về cấu hình](#config-basics)
  - [Định dạng cấu hình là gì? Nó ở đâu?](#what-format-is-the-config-where-is-it)
  - [Tôi đặt `gateway.bind: "lan"` (#i-set-gatewaybind-lan-or-tailnet-and-now-nothing-listens-the-ui-says-unauthorized)
  - [Tại sao tôi cần một token trên localhost bây giờ?](#why-do-i-need-a-token-on-localhost-now)
  - [Tôi có phải khởi động lại sau khi thay đổi cấu hình không?](#do-i-have-to-restart-after-changing-config)
  - [Làm cách nào để bật tìm kiếm web (#how-do-i-enable-web-search-and-web-fetch)
  - [config.apply đã xóa cấu hình của tôi. Làm cách nào để tôi khôi phục và tránh điều này?](#configapply-wiped-my-config-how-do-i-recover-and-avoid-this)
  - [Làm cách nào để chạy một Gateway trung tâm với các worker chuyên biệt trên các thiết bị?](#how-do-i-run-a-central-gateway-with-specialized-workers-across-devices)
  - [Trình duyệt OpenClaw có thể chạy ở chế độ headless không?](#can-the-openclaw-browser-run-headless)
  - [Làm cách nào để sử dụng Brave để kiểm soát trình duyệt?](#how-do-i-use-brave-for-browser-control)
- [Gateway từ xa và các node](#remote-gateways-and-nodes)
  - [Các lệnh được truyền như thế nào giữa Telegram, gateway và các node?](#how-do-commands-propagate-between-telegram-the-gateway-and-nodes)
  - [Agent của tôi có thể truy cập máy tính của tôi nếu Gateway được lưu trữ từ xa như thế nào?](#how-can-my-agent-access-my-computer-if-the-gateway-is-hosted-remotely)
  - [Tailscale được kết nối nhưng tôi không nhận được phản hồi. Bây giờ sao?](#tailscale-is-connected-but-i-get-no-replies-what-now)
  - [Hai instance OpenClaw có thể nói chuyện với nhau không (#can-two-openclaw-instances-talk-to-each-other-local-vps)
  - [Tôi có cần các VPS riêng biệt cho nhiều agent không](#do-i-need-separate-vpses-for-multiple-agents)
  - [Có lợi ích gì khi sử dụng một node trên laptop cá nhân của tôi thay vì SSH từ VPS?](#is-there-a-benefit-to-using-a-node-on-my-personal-laptop-instead-of-ssh-from-a-vps)
  - [Các node có chạy dịch vụ gateway không?](#do-nodes-run-a-gateway-service)
  - [Có cách API / RPC để áp dụng cấu hình không?](#is-there-an-api-rpc-way-to-apply-config)
  - [Cấu hình "hợp lý" tối thiểu cho lần cài đặt đầu tiên là gì?](#whats-a-minimal-sane-config-for-a-first-install)
  - [Làm cách nào để thiết lập Tailscale trên VPS và kết nối từ Mac của tôi?](#how-do-i-set-up-tailscale-on-a-vps-and-connect-from-my-mac)
  - [Làm cách nào để kết nối một node Mac với một Gateway từ xa (#how-do-i-connect-a-mac-node-to-a-remote-gateway-tailscale-serve)
  - [Tôi nên cài đặt trên một laptop thứ hai hay chỉ thêm một node?](#should-i-install-on-a-second-laptop-or-just-add-a-node)
- [Biến môi trường và tải .env](#env-vars-and-env-loading)
  - [OpenClaw tải biến môi trường như thế nào?](#how-does-openclaw-load-environment-variables)
  - ["Tôi bắt đầu Gateway thông qua dịch vụ và biến môi trường của tôi biến mất." Bây giờ sao?](#i-started-the-gateway-via-the-service-and-my-env-vars-disappeared-what-now)
  - [Tôi đặt `COPILOT_GITHUB_TOKEN`, nhưng trạng thái mô hình hiển thị "Shell env: off." Tại sao?](#i-set-copilotgithubtoken-but-models-status-shows-shell-env-off-why)
- [Phiên và nhiều cuộc trò chuyện](#sessions-and-multiple-chats)
  - [Làm cách nào để bắt đầu một cuộc trò chuyện mới?](#how-do-i-start-a-fresh-conversation)
  - [Các phiên có tự động đặt lại nếu tôi không bao giờ gửi `/new` không?](#do-sessions-reset-automatically-if-i-never-send-new)
  - [Có cách nào để tạo một nhóm các instance OpenClaw một CEO và nhiều agent không](#is-there-a-way-to-make-a-team-of-openclaw-instances-one-ceo-and-many-agents)
  - [Tại sao ngữ cảnh bị cắt ngắn giữa tác vụ? Làm cách nào để tôi ngăn chặn điều này?](#why-did-context-get-truncated-midtask-how-do-i-prevent-it)
  - [Làm cách nào để hoàn toàn đặt lại OpenClaw nhưng giữ nó được cài đặt?](#how-do-i-completely-reset-openclaw-but-keep-it-installed)
  - [Tôi nhận được lỗi "context too large" - làm cách nào để tôi đặt lại hoặc nén?](#im-getting-context-too-large-errors-how-do-i-reset-or-compact)
  - [Tại sao tôi lại thấy "LLM request rejected: messages.content.tool_use.input field required"?](#why-am-i-seeing-llm-request-rejected-messagescontenttool_useinput-field-required)
- [Tại sao tôi nhận được tin nhắn heartbeat cứ 30 phút?](#why-am-i-getting-heartbeat-messages-every-30-minutes)
  - [Tôi có cần thêm "tài khoản bot" vào nhóm WhatsApp không?](#do-i-need-to-add-a-bot-account-to-a-whatsapp-group)
  - [Làm cách nào để lấy JID của nhóm WhatsApp?](#how-do-i-get-the-jid-of-a-whatsapp-group)
  - [Tại sao OpenClaw không trả lời trong nhóm?](#why-doesnt-openclaw-reply-in-a-group)
  - [Các nhóm/luồng có chia sẻ ngữ cảnh với tin nhắn riêng không?](#do-groupsthreads-share-context-with-dms)
  - [Tôi có thể tạo bao nhiêu không gian làm việc và agent?](#how-many-workspaces-and-agents-can-i-create)
  - [Tôi có thể chạy nhiều bot hoặc chat cùng lúc không (#can-i-run-multiple-bots-or-chats-at-the-same-time-slack-and-how-should-i-set-that-up)
- [Mô hình: mặc định, lựa chọn, bí danh, chuyển đổi](#models-defaults-selection-aliases-switching)
  - [Mô hình "mặc định" là gì?](#what-is-the-default-model)
  - [Bạn khuyên dùng mô hình nào?](#what-model-do-you-recommend)
  - [Làm cách nào để chuyển đổi mô hình mà không xóa cấu hình của tôi?](#how-do-i-switch-models-without-wiping-my-config)
  - [Tôi có thể sử dụng mô hình tự lưu trữ không (#can-i-use-selfhosted-models-llamacpp-vllm-ollama)
  - [OpenClaw, Flawd và Krill sử dụng mô hình nào?](#what-do-openclaw-flawd-and-krill-use-for-models)
  - [Làm cách nào để chuyển đổi mô hình ngay lập tức (#how-do-i-switch-models-on-the-fly-without-restarting)
  - [Tôi có thể sử dụng GPT 5.2 cho các tác vụ hàng ngày và Codex 5.3 để viết mã không](#can-i-use-gpt-52-for-daily-tasks-and-codex-53-for-coding)
  - [Tại sao tôi thấy "Mô hình … không được phép" rồi không có trả lời?](#why-do-i-see-model-is-not-allowed-and-then-no-reply)
  - [Tại sao tôi thấy "Mô hình không xác định: minimax/MiniMax-M2.1"?](#why-do-i-see-unknown-model-minimaxminimaxm21)
  - [Tôi có thể sử dụng MiniMax làm mặc định và OpenAI cho các tác vụ phức tạp không?](#can-i-use-minimax-as-my-default-and-openai-for-complex-tasks)
  - [opus / sonnet / gpt có phải là các phím tắt tích hợp không?](#are-opus-sonnet-gpt-builtin-shortcuts)
  - [Làm cách nào để xác định/ghi đè các phím tắt mô hình (#how-do-i-defineoverride-model-shortcuts-aliases)
  - [Làm cách nào để thêm mô hình từ các nhà cung cấp khác như OpenRouter hoặc Z.AI?](#how-do-i-add-models-from-other-providers-like-openrouter-or-zai)
- [Chuyển đổi dự phòng mô hình và "Tất cả mô hình đã thất bại"](#model-failover-and-all-models-failed)
  - [Chuyển đổi dự phòng hoạt động như thế nào?](#how-does-failover-work)
  - [Lỗi này có ý nghĩa gì?](#what-does-this-error-mean)
  - [Danh sách kiểm tra sửa chữa cho `No credentials found for profile "anthropic:default"`](#fix-checklist-for-no-credentials-found-for-profile-anthropicdefault)
  - [Tại sao nó cũng thử Google Gemini và thất bại?](#why-did-it-also-try-google-gemini-and-fail)
- [Hồ sơ xác thực: chúng là gì và cách quản lý](#auth-profiles-what-they-are-and-how-to-manage-them)
  - [Hồ sơ xác thực là gì?](#what-is-an-auth-profile)
  - [ID hồ sơ điển hình là gì?](#what-are-typical-profile-ids)
  - [Tôi có thể kiểm soát hồ sơ xác thực nào được thử trước tiên không?](#can-i-control-which-auth-profile-is-tried-first)
  - [OAuth so với khóa API: sự khác biệt là gì?](#oauth-vs-api-key-whats-the-difference)
- [Gateway: cổng, "đã chạy", và chế độ từ xa](#gateway-ports-already-running-and-remote-mode)
  - [Gateway sử dụng cổng nào?](#what-port-does-the-gateway-use)
  - [Tại sao `openclaw gateway status` nói `Runtime: running` nhưng `RPC probe: failed`?](#why-does-openclaw-gateway-status-say-runtime-running-but-rpc-probe-failed)
  - [Tại sao `openclaw gateway status` hiển thị `Config (cli)` và `Config (service)` khác nhau?](#why-does-openclaw-gateway-status-show-config-cli-and-config-service-different)
  - [Điều gì có ý nghĩa là "một phiên bản gateway khác đã đang lắng nghe"?](#what-does-another-gateway-instance-is-already-listening-mean)
  - [Làm cách nào để chạy OpenClaw ở chế độ từ xa (#how-do-i-run-openclaw-in-remote-mode-client-connects-to-a-gateway-elsewhere)
  - [Control UI nói "không được phép" (#the-control-ui-says-unauthorized-or-keeps-reconnecting-what-now)
  - [Tôi đặt `gateway.bind: "tailnet"` nhưng nó không thể liên kết / không có gì lắng nghe](#i-set-gatewaybind-tailnet-but-it-cant-bind-nothing-listens)
  - [Tôi có thể chạy nhiều Gateway trên cùng một máy chủ không?](#can-i-run-multiple-gateways-on-the-same-host)
  - [Điều gì có ý nghĩa là "bắt tay không hợp lệ" / mã 1008?](#what-does-invalid-handshake-code-1008-mean)
- [Ghi nhật ký và gỡ lỗi](#logging-and-debugging)
  - [Nhật ký ở đâu?](#where-are-logs)
  - [Làm cách nào để bắt đầu/dừng/khởi động lại dịch vụ Gateway?](#how-do-i-startstoprestart-the-gateway-service)
  - [Tôi đã đóng terminal của mình trên Windows - làm cách nào để khởi động lại OpenClaw?](#i-closed-my-terminal-on-windows-how-do-i-restart-openclaw)
  - [Gateway đang chạy nhưng các trả lời không bao giờ đến. Tôi nên kiểm tra cái gì?](#the-gateway-is-up-but-replies-never-arrive-what-should-i-check)
  - ["Ngắt kết nối khỏi gateway: không có lý do" - bây giờ sao?](#disconnected-from-gateway-no-reason-what-now)
  - [Telegram setMyCommands thất bại với lỗi mạng. Tôi nên kiểm tra cái gì?](#telegram-setmycommands-fails-with-network-errors-what-should-i-check)
  - [TUI không hiển thị đầu ra. Tôi nên kiểm tra cái gì?](#tui-shows-no-output-what-should-i-check)
  - [Làm cách nào để hoàn toàn dừng rồi bắt đầu Gateway?](#how-do-i-completely-stop-then-start-the-gateway)
  - [ELI5: `openclaw gateway restart` so với `openclaw gateway`](#eli5-openclaw-gateway-restart-vs-openclaw-gateway)
  - [Cách nhanh nhất để có được thêm chi tiết khi có sự cố là gì?](#whats-the-fastest-way-to-get-more-details-when-something-fails)
- [Phương tiện và tệp đính kèm](#media-and-attachments)
  - [Skill của tôi đã tạo một hình ảnh/PDF, nhưng không có gì được gửi](#my-skill-generated-an-imagepdf-but-nothing-was-sent)
- [Bảo mật và kiểm soát truy cập](#security-and-access-control)
  - [Có an toàn khi để OpenClaw tiếp nhận tin nhắn riêng không?](#is-it-safe-to-expose-openclaw-to-inbound-dms)
  - [Prompt injection chỉ là mối quan tâm đối với bot công khai không?](#is-prompt-injection-only-a-concern-for-public-bots)
  - [Bot của tôi có nên có email GitHub hoặc số điện thoại riêng không](#should-my-bot-have-its-own-email-github-account-or-phone-number)
  - [Tôi có thể cho nó quyền tự chủ trên tin nhắn văn bản của tôi và điều đó có an toàn không](#can-i-give-it-autonomy-over-my-text-messages-and-is-that-safe)
  - [Tôi có thể sử dụng mô hình rẻ hơn cho các tác vụ trợ lý cá nhân không?](#can-i-use-cheaper-models-for-personal-assistant-tasks)
- [Tôi đã chạy `/start` trong Telegram nhưng không nhận được mã ghép nối](#i-ran-start-in-telegram-but-didnt-get-a-pairing-code)
- [WhatsApp: nó sẽ nhắn tin cho các liên hệ của tôi không? Ghép nối hoạt động như thế nào?](#whatsapp-will-it-message-my-contacts-how-does-pairing-work)
- [Lệnh trò chuyện, hủy bỏ tác vụ và "nó sẽ không dừng lại"](#chat-commands-aborting-tasks-and-it-wont-stop)
  - [Làm cách nào để ngăn các tin nhắn hệ thống nội bộ hiển thị trong trò chuyện](#how-do-i-stop-internal-system-messages-from-showing-in-chat)
  - [Làm cách nào để dừng/hủy bỏ một tác vụ đang chạy?](#how-do-i-stopcancel-a-running-task)
  - [Làm cách nào để gửi tin nhắn Discord từ Telegram? (#how-do-i-send-a-discord-message-from-telegram-crosscontext-messaging-denied)
  - [Tại sao có vẻ như bot "bỏ qua" các tin nhắn liên tiếp?](#why-does-it-feel-like-the-bot-ignores-rapidfire-messages)
## 60 giây đầu tiên nếu có sự cố

1. **Kiểm tra trạng thái nhanh (kiểm tra đầu tiên)**

   ```bash
   openclaw status
   ```

   Fast local summary: OS + update, gateway/service reachability, agents/sessions, provider config + runtime issues (when gateway is reachable).

2. **Pasteable report (safe to share)**

   ```bash
   openclaw status --all
   ```

   Read-only diagnosis with log tail (tokens redacted).

3. **Daemon + port state**

   ```bash
   openclaw gateway status
   ```

   Shows supervisor runtime vs RPC reachability, the probe target URL, and which config the service likely used.

4. **Deep probes**

   ```bash
   openclaw status --deep
   ```

   Runs gateway health checks + provider probes (requires a reachable gateway). See [Health](/gateway/health).

5. **Tail the latest log**

   ```bash
   openclaw logs --follow
   ```

   If RPC is down, fall back to:

   ```bash
   tail -f "$(ls -t /tmp/openclaw/openclaw-*.log | head -1)"
   ```

   File logs are separate from service logs; see [Logging](/logging) and [Troubleshooting](/gateway/troubleshooting).

6. **Run the doctor (repairs)**

   ```bash
   openclaw doctor
   ```

   Repairs/migrates config/state + runs health checks. See [Doctor](/gateway/doctor).

7. **Gateway snapshot**

   ```bash
   openclaw health --json
   openclaw health --verbose   # shows the target URL + config path on errors
   ```
Yêu cầu gateway đang chạy để lấy một bản chụp đầy đủ (chỉ WS). Xem [Health](/gateway/health).

## Bắt đầu nhanh và thiết lập lần đầu chạy

### Tôi bị kẹt, cách nhanh nhất để giải quyết vấn đề là gì

Sử dụng một agent AI cục bộ có thể **nhìn thấy máy của bạn**. Điều đó hiệu quả hơn nhiều so với việc hỏi
trên Discord, vì hầu hết các trường hợp "tôi bị kẹt" là **các vấn đề cấu hình cục bộ hoặc môi trường** mà
những người trợ giúp từ xa không thể kiểm tra.

- **Claude Code**: [https://www.anthropic.com/claude-code/](https://www.anthropic.com/claude-code/)
- **OpenAI Codex**: [https://openai.com/codex/](https://openai.com/codex/)

Những công cụ này có thể đọc repo, chạy lệnh, kiểm tra nhật ký và giúp sửa thiết lập cấp máy của bạn
(PATH, dịch vụ, quyền, tệp xác thực). Cung cấp cho chúng **toàn bộ checkout mã nguồn** thông qua
cài đặt có thể hack (git):

```bash
curl -fsSL https://openclaw.ai/install.sh | bash -s -- --install-method git
```

This installs OpenClaw **from a git checkout**, so the agent can read the code + docs and
reason about the exact version you are running. You can always switch back to stable later
by re-running the installer without `--install-method git`.

Tip: ask the agent to **plan and supervise** the fix (step-by-step), then execute only the
necessary commands. That keeps changes small and easier to audit.

If you discover a real bug or fix, please file a GitHub issue or send a PR:
[https://github.com/openclaw/openclaw/issues](https://github.com/openclaw/openclaw/issues)
[https://github.com/openclaw/openclaw/pulls](https://github.com/openclaw/openclaw/pulls)

Start with these commands (share outputs when asking for help):

```bash
openclaw status
openclaw models status
openclaw doctor
```

What they do:

- `openclaw status`: quick snapshot of gateway/agent health + basic config.
- `openclaw models status`: checks provider auth + model availability.
- `openclaw doctor`: validates and repairs common config/state issues.

Other useful CLI checks: `openclaw status --all`, `openclaw logs --follow`,
`openclaw gateway status`, `openclaw health --verbose`.

Quick debug loop: [First 60 seconds if something's broken](#first-60-seconds-if-somethings-broken).
Install docs: [Install](/install), [Installer flags](/install/installer), [Updating](/install/updating).

### What's the recommended way to install and set up OpenClaw

The repo recommends running from source and using the onboarding wizard:

```bash
curl -fsSL https://openclaw.ai/install.sh | bash
openclaw onboard --install-daemon
```
Trình hướng dẫn cũng có thể xây dựng tài sản giao diện người dùng tự động. Sau khi thiết lập ban đầu, bạn thường chạy Gateway trên cổng **18789**.

Từ nguồn (contributors/dev):

```bash
git clone https://github.com/openclaw/openclaw.git
cd openclaw
pnpm install
pnpm build
pnpm ui:build # auto-installs UI deps on first run
openclaw onboard
```

If you don't have a global install yet, run it via `pnpm openclaw onboard`.

### How do I open the dashboard after onboarding

The wizard opens your browser with a clean (non-tokenized) dashboard URL right after onboarding and also prints the link in the summary. Keep that tab open; if it didn't launch, copy/paste the printed URL on the same machine.

### How do I authenticate the dashboard token on localhost vs remote

**Localhost (same machine):**

- Open `http://127.0.0.1:18789/`.
- If it asks for auth, paste the token from `gateway.auth.token` (or `OPENCLAW_GATEWAY_TOKEN`) into Control UI settings.
- Retrieve it from the gateway host: `openclaw config get gateway.auth.token` (or generate one: `openclaw doctor --generate-gateway-token`).

**Not on localhost:**

- **Tailscale Serve** (recommended): keep bind loopback, run `openclaw gateway --tailscale serve`, open `https://<magicdns>/`. If `gateway.auth.allowTailscale` is `true`, identity headers satisfy Control UI/WebSocket auth (no token, assumes trusted gateway host); HTTP APIs still require token/password.
- **Tailnet bind**: run `openclaw gateway --bind tailnet --token "<token>"`, open `http://<tailscale-ip>:18789/`, paste token in dashboard settings.
- **SSH tunnel**: `ssh -N -L 18789:127.0.0.1:18789 user@host` then open `http://127.0.0.1:18789/` and paste the token in Control UI settings.

See [Dashboard](/web/dashboard) and [Web surfaces](/web) for bind modes and auth details.

### What runtime do I need

Node **>= 22** is required. `pnpm` được khuyến nghị. Bun **không được khuyến nghị** cho Gateway.

### Nó có chạy trên Raspberry Pi không

Có. Gateway rất nhẹ - tài liệu liệt kê **512MB-1GB RAM**, **1 lõi**, và khoảng **500MB**
dung lượng đĩa là đủ cho mục đích sử dụng cá nhân, và lưu ý rằng **Raspberry Pi 4 có thể chạy nó**.

Nếu bạn muốn có thêm không gian (nhật ký, phương tiện, các dịch vụ khác), **khuyến nghị 2GB**, nhưng đó không phải là yêu cầu tối thiểu cứng.

Mẹo: một Pi/VPS nhỏ có thể lưu trữ Gateway, và bạn có thể ghép **nodes** trên máy tính xách tay/điện thoại của mình để thực hiện màn hình/camera/canvas cục bộ hoặc thực thi lệnh. Xem [Nodes](/nodes).

### Có mẹo nào cho cài đặt Raspberry Pi không

Phiên bản ngắn: nó hoạt động, nhưng hãy mong đợi những điểm không hoàn hảo.

- Sử dụng **64-bit** OS và giữ Node >= 22.
- Ưu tiên **cài đặt có thể hack (git)** để bạn có thể xem nhật ký và cập nhật nhanh.
- Bắt đầu mà không có kênh/Skills, sau đó thêm chúng từng cái một.
- Nếu bạn gặp phải các vấn đề nhị phân kỳ lạ, thường là vấn đề **tương thích ARM**.
Docs: [Linux](/platforms/linux), [Cài đặt](/install).

### Nó bị kẹt ở màn hình "Wake up my friend" - thiết lập ban đầu sẽ không khởi động. Bây giờ sao?

Màn hình đó phụ thuộc vào Gateway có thể truy cập được và được xác thực. TUI cũng gửi
"Wake up, my friend!" tự động khi khởi động lần đầu. Nếu bạn thấy dòng đó với **không có phản hồi**
và các token vẫn ở 0, agent chưa bao giờ chạy.

1. Khởi động lại Gateway:

```bash
openclaw gateway restart
```

2. Check status + auth:

```bash
openclaw status
openclaw models status
openclaw logs --follow
```

3. If it still hangs, run:

```bash
openclaw doctor
```

If the Gateway is remote, ensure the tunnel/Tailscale connection is up and that the UI
is pointed at the right Gateway. See [Remote access](/gateway/remote).

### Can I migrate my setup to a new machine Mac mini without redoing onboarding

Yes. Copy the **state directory** and **workspace**, then run Doctor once. This
keeps your bot "exactly the same" (memory, session history, auth, and channel
state) as long as you copy **both** locations:

1. Install OpenClaw on the new machine.
2. Copy `$OPENCLAW_STATE_DIR` (default: `~/.openclaw`) from the old machine.
3. Copy your workspace (default: `~/.openclaw/workspace`).
4. Run `openclaw doctor` and restart the Gateway service.

That preserves config, auth profiles, WhatsApp creds, sessions, and memory. If you're in
remote mode, remember the gateway host owns the session store and workspace.

**Important:** if you only commit/push your workspace to GitHub, you're backing
up **memory + bootstrap files**, but **not** session history or auth. Those live
under `~/.openclaw/` (for example `~/.openclaw/agents/<agentId>/sessions/`).

Liên quan: [Di chuyển](/install/migrating), [Vị trí lưu trữ các thứ trên đĩa](/help/faq#where-does-openclaw-store-its-data),
[Không gian làm việc của agent](/concepts/agent-workspace), [Doctor](/gateway/doctor),
[Chế độ từ xa](/gateway/remote).

### Tôi có thể xem những gì mới trong phiên bản mới nhất ở đâu

Kiểm tra changelog trên GitHub:
[https://github.com/openclaw/openclaw/blob/main/CHANGELOG.md](https://github.com/openclaw/openclaw/blob/main/CHANGELOG.md)

Các mục nhập mới nhất ở trên cùng. Nếu phần trên cùng được đánh dấu **Unreleased**, phần có ngày tháng tiếp theo
là phiên bản được phát hành mới nhất. Các mục nhập được nhóm theo **Highlights**, **Changes**, và
**Sửa lỗi** (cộng với tài liệu/các phần khác khi cần).

### Tôi không thể truy cập docs.openclaw.ai Lỗi SSL Bây giờ sao

Một số kết nối Comcast/Xfinity không chính xác chặn `docs.openclaw.ai` qua Xfinity
Advanced Security. Vô hiệu hóa nó hoặc cho phép `docs.openclaw.ai`, sau đó thử lại. Chi tiết thêm: [Khắc phục sự cố](/help/troubleshooting#docsopenclawai-shows-an-ssl-error-comcastxfinity).
Vui lòng giúp chúng tôi mở khóa nó bằng cách báo cáo tại đây: [https://spa.xfinity.com/check_url_status](https://spa.xfinity.com/check_url_status).

Nếu bạn vẫn không thể truy cập trang web, tài liệu được sao chép trên GitHub:
[https://github.com/openclaw/openclaw/tree/main/docs](https://github.com/openclaw/openclaw/tree/main/docs)

### Sự khác biệt giữa stable và beta là gì

**Stable** và **beta** là **npm dist-tags**, không phải các dòng mã riêng biệt:

- `latest` = stable
- `beta` = bản dựng sớm để kiểm tra

Chúng tôi gửi các bản dựng đến **beta**, kiểm tra chúng, và khi một bản dựng ổn định chúng tôi **nâng cấp phiên bản đó lên `latest`**. Đó là lý do tại sao beta và stable có thể trỏ đến **cùng một phiên bản**.

Xem những gì đã thay đổi:
[https://github.com/openclaw/openclaw/blob/main/CHANGELOG.md](https://github.com/openclaw/openclaw/blob/main/CHANGELOG.md)

### Làm cách nào để cài đặt phiên bản beta và sự khác biệt giữa beta và dev là gì

**Beta** là npm dist-tag `beta` (có thể khớp với `latest`).
**Dev** là đầu di chuyển của `main` (git); khi được xuất bản, nó sử dụng npm dist-tag `dev`.

Lệnh một dòng (macOS/Linux):

```bash
curl -fsSL --proto '=https' --tlsv1.2 https://openclaw.ai/install.sh | bash -s -- --beta
```

```bash
curl -fsSL --proto '=https' --tlsv1.2 https://openclaw.ai/install.sh | bash -s -- --install-method git
```

Trình cài đặt Windows (PowerShell):
[https://openclaw.ai/install.ps1](https://openclaw.ai/install.ps1)

Chi tiết thêm: [Kênh phát triển](/install/development-channels) và [Cờ trình cài đặt](/install/installer).

### Cài đặt và thiết lập ban đầu thường mất bao lâu

Hướng dẫn sơ bộ:

- **Cài đặt:** 2-5 phút
- **Thiết lập ban đầu:** 5-15 phút tùy thuộc vào số lượng kênh/mô hình bạn cấu hình

Nếu nó bị treo, hãy sử dụng [Trình cài đặt bị treo](/help/faq#installer-stuck-how-do-i-get-more-feedback)
và vòng lặp gỡ lỗi nhanh trong [Tôi bị kẹt](/help/faq#im-stuck--whats-the-fastest-way-to-get-unstuck).

### Làm cách nào để thử các bản mới nhất

Hai tùy chọn:
1. **Dev channel (git checkout):**

```bash
openclaw update --channel dev
```

This switches to the `main` branch and updates from source.

2. **Hackable install (from the installer site):**

```bash
curl -fsSL https://openclaw.ai/install.sh | bash -s -- --install-method git
```

That gives you a local repo you can edit, then update via git.

If you prefer a clean clone manually, use:

```bash
git clone https://github.com/openclaw/openclaw.git
cd openclaw
pnpm install
pnpm build
```

Docs: [Update](/cli/update), [Development channels](/install/development-channels),
[Install](/install).

### Installer stuck How do I get more feedback

Re-run the installer with **verbose output**:

```bash
curl -fsSL https://openclaw.ai/install.sh | bash -s -- --verbose
```

Beta install with verbose:

```bash
curl -fsSL https://openclaw.ai/install.sh | bash -s -- --beta --verbose
```

For a hackable (git) install:

```bash
curl -fsSL https://openclaw.ai/install.sh | bash -s -- --install-method git --verbose
```

Windows (PowerShell) equivalent:

```powershell
# install.ps1 has no dedicated -Verbose flag yet.
Set-PSDebug -Trace 1
& ([scriptblock]::Create((iwr -useb https://openclaw.ai/install.ps1))) -NoOnboard
Set-PSDebug -Trace 0
```

Thêm tùy chọn: [Cờ trình cài đặt](/install/installer).

### Cài đặt Windows báo git không tìm thấy hoặc openclaw không được nhận dạng
Hai vấn đề Windows phổ biến:

**1) npm error spawn git / git not found**

- Cài đặt **Git for Windows** và đảm bảo `git` nằm trên PATH của bạn.
- Đóng và mở lại PowerShell, sau đó chạy lại trình cài đặt.

**2) openclaw is not recognized after install**

- Thư mục npm global bin của bạn không nằm trên PATH.
- Kiểm tra đường dẫn:

  ```powershell
  npm config get prefix
  ```

- Ensure `<prefix>\\bin` is on PATH (on most systems it is `%AppData%\\npm`).
- Close and reopen PowerShell after updating PATH.

If you want the smoothest Windows setup, use **WSL2** instead of native Windows.
Docs: [Windows](/platforms/windows).

### The docs didn't answer my question how do I get a better answer

Use the **hackable (git) install** so you have the full source and docs locally, then ask
your bot (or Claude/Codex) _from that folder_ so it can read the repo and answer precisely.

```bash
curl -fsSL https://openclaw.ai/install.sh | bash -s -- --install-method git
```

Chi tiết hơn: [Cài đặt](/install) và [Cờ trình cài đặt](/install/installer).

### Làm cách nào để cài đặt OpenClaw trên Linux

Câu trả lời ngắn: làm theo hướng dẫn Linux, sau đó chạy trình hướng dẫn thiết lập ban đầu.

- Đường dẫn nhanh Linux + cài đặt dịch vụ: [Linux](/platforms/linux).
- Hướng dẫn đầy đủ: [Bắt đầu](/start/getting-started).
- Trình cài đặt + cập nhật: [Cài đặt & cập nhật](/install/updating).

### Làm cách nào để cài đặt OpenClaw trên VPS

Bất kỳ VPS Linux nào cũng hoạt động. Cài đặt trên máy chủ, sau đó sử dụng SSH/Tailscale để truy cập Gateway.

Hướng dẫn: [exe.dev](/install/exe-dev), [Hetzner](/install/hetzner), [Fly.io](/install/fly).
Truy cập từ xa: [Gateway remote](/gateway/remote).

### Hướng dẫn cài đặt cloudVPS ở đâu

Chúng tôi duy trì một **trung tâm lưu trữ** với các nhà cung cấp phổ biến. Chọn một và làm theo hướng dẫn:

- [Lưu trữ VPS](/vps) (tất cả các nhà cung cấp ở một nơi)
- [Fly.io](/install/fly)
- [Hetzner](/install/hetzner)
- [exe.dev](/install/exe-dev)

Cách hoạt động trong cloud: **Gateway chạy trên máy chủ**, và bạn truy cập nó
từ laptop/điện thoại của bạn thông qua Control UI (hoặc Tailscale/SSH). Trạng thái + không gian làm việc của bạn
tồn tại trên máy chủ, vì vậy hãy coi máy chủ là nguồn sự thật và sao lưu nó.

Bạn có thể ghép nối **nodes** (Mac/iOS/Android/headless) với Gateway đám mây đó để truy cập
màn hình/camera/canvas cục bộ hoặc chạy lệnh trên máy tính xách tay của bạn trong khi giữ
Gateway ở đám mây.

Hub: [Platforms](/platforms). Truy cập từ xa: [Gateway remote](/gateway/remote).
Nodes: [Nodes](/nodes), [Nodes CLI](/cli/nodes).

### Tôi có thể yêu cầu OpenClaw tự cập nhật không

Câu trả lời ngắn: **có thể, không được khuyến khích**. Quy trình cập nhật có thể khởi động lại
Gateway (làm gián đoạn phiên hoạt động), có thể cần kiểm tra git sạch, và
có thể yêu cầu xác nhận. An toàn hơn: chạy cập nhật từ shell với tư cách là người điều hành.

Sử dụng CLI:

```bash
openclaw update
openclaw update status
openclaw update --channel stable|beta|dev
openclaw update --tag <dist-tag|version>
openclaw update --no-restart
```

If you must automate from an agent:

```bash
openclaw update --yes --no-restart
openclaw gateway restart
```

Docs: [Update](/cli/update), [Updating](/install/updating).

### What does the onboarding wizard actually do

`openclaw onboard` là đường dẫn thiết lập được khuyến nghị. Ở **chế độ cục bộ** nó sẽ hướng dẫn bạn qua:

- **Thiết lập mô hình/xác thực** (Anthropic **setup-token** được khuyến nghị cho các gói đăng ký Claude, OpenAI Codex OAuth được hỗ trợ, khóa API tùy chọn, các mô hình cục bộ LM Studio được hỗ trợ)
- Vị trí **Workspace** + tệp bootstrap
- **Cài đặt Gateway** (bind/port/auth/tailscale)
- **Nhà cung cấp** (WhatsApp, Telegram, Discord, Mattermost (plugin), Signal, iMessage)
- **Cài đặt Daemon** (LaunchAgent trên macOS; systemd user unit trên Linux/WSL2)
- **Kiểm tra sức khỏe** và lựa chọn **skills**

Nó cũng cảnh báo nếu mô hình được cấu hình của bạn không xác định hoặc thiếu xác thực.

### Tôi có cần đăng ký Claude hoặc OpenAI để chạy cái này không

Không. Bạn có thể chạy OpenClaw với **khóa API** (Anthropic/OpenAI/khác) hoặc với
**các mô hình chỉ cục bộ** để dữ liệu của bạn ở lại trên thiết bị của bạn. Các gói đăng ký (Claude
Pro/Max hoặc OpenAI Codex) là những cách tùy chọn để xác thực các nhà cung cấp đó.

Tài liệu: [Anthropic](/providers/anthropic), [OpenAI](/providers/openai),
[Local models](/gateway/local-models), [Models](/concepts/models).

### Tôi có thể sử dụng gói đăng ký Claude Max mà không cần khóa API không

Có. Bạn có thể xác thực bằng **setup-token**
thay vì khóa API. Đây là đường dẫn gói đăng ký.
Các gói đăng ký Claude Pro/Max **không bao gồm khóa API**, vì vậy đây là cách tiếp cận chính xác cho các tài khoản đăng ký. Quan trọng: bạn phải xác minh với Anthropic rằng cách sử dụng này được phép theo chính sách đăng ký và điều khoản của họ. Nếu bạn muốn con đường được hỗ trợ rõ ràng nhất, hãy sử dụng khóa API của Anthropic.

### Cách xác thực token setup của Anthropic hoạt động như thế nào

`claude setup-token` tạo ra một **chuỗi token** thông qua Claude Code CLI (nó không có sẵn trong bảng điều khiển web). Bạn có thể chạy nó trên **bất kỳ máy nào**. Chọn **Anthropic token (dán setup-token)** trong trình hướng dẫn hoặc dán nó bằng `openclaw models auth paste-token --provider anthropic`. Token được lưu trữ dưới dạng hồ sơ xác thực cho nhà cung cấp **anthropic** và được sử dụng như một khóa API (không tự động làm mới). Chi tiết thêm: [OAuth](/concepts/oauth).

### Tôi tìm setup-token của Anthropic ở đâu

Nó **không** có trong Bảng điều khiển Anthropic. Setup-token được tạo bởi **Claude Code CLI** trên **bất kỳ máy nào**:

```bash
claude setup-token
```

Copy the token it prints, then choose **Anthropic token (paste setup-token)** in the wizard. If you want to run it on the gateway host, use `openclaw models auth setup-token --provider anthropic`. If you ran `claude setup-token` elsewhere, paste it on the gateway host with `openclaw models auth paste-token --provider anthropic`. See [Anthropic](/providers/anthropic).

### Do you support Claude subscription auth (Claude Pro or Max)

Yes - via **setup-token**. OpenClaw no longer reuses Claude Code CLI OAuth tokens; use a setup-token or an Anthropic API key. Generate the token anywhere and paste it on the gateway host. See [Anthropic](/providers/anthropic) and [OAuth](/concepts/oauth).

Note: Claude subscription access is governed by Anthropic's terms. For production or multi-user workloads, API keys are usually the safer choice.

### Why am I seeing HTTP 429 ratelimiterror from Anthropic

That means your **Anthropic quota/rate limit** is exhausted for the current window. If you
use a **Claude subscription** (setup-token or Claude Code OAuth), wait for the window to
reset or upgrade your plan. If you use an **Anthropic API key**, check the Anthropic Console
for usage/billing and raise limits as needed.

Tip: set a **fallback model** so OpenClaw can keep replying while a provider is rate-limited.
See [Models](/cli/models) and [OAuth](/concepts/oauth).

### Is AWS Bedrock supported

Yes - via pi-ai's **Amazon Bedrock (Converse)** provider with **manual config**. You must supply AWS credentials/region on the gateway host and add a Bedrock provider entry in your models config. See [Amazon Bedrock](/providers/bedrock) and [Model providers](/providers/models). If you prefer a managed key flow, an OpenAI-compatible proxy in front of Bedrock is still a valid option.

### How does Codex auth work

OpenClaw supports **OpenAI Code (Codex)** via OAuth (ChatGPT sign-in). The wizard can run the OAuth flow and will set the default model to `openai-codex/gpt-5.3-codex` when appropriate. See [Model providers](/concepts/model-providers) and [Wizard](/start/wizard).

### Do you support OpenAI subscription auth Codex OAuth

Yes. OpenClaw fully supports **OpenAI Code (Codex) subscription OAuth**. The onboarding wizard
can run the OAuth flow for you.

See [OAuth](/concepts/oauth), [Model providers](/concepts/model-providers), and [Wizard](/start/wizard).

### How do I set up Gemini CLI OAuth

Gemini CLI uses a **plugin auth flow**, not a client id or secret in `openclaw.json`.

Steps:

1. Enable the plugin: `openclaw plugins enable google-gemini-cli-auth`
2. Login: `openclaw models auth login --provider google-gemini-cli --set-default`
Điều này lưu trữ các token OAuth trong các hồ sơ xác thực trên máy chủ Gateway. Chi tiết: [Model providers](/concepts/model-providers).

### Có thể sử dụng mô hình cục bộ cho các cuộc trò chuyện bình thường không

Thường thì không. OpenClaw cần ngữ cảnh lớn + bảo mật mạnh; các card nhỏ sẽ cắt ngắn và rò rỉ dữ liệu. Nếu bạn phải làm vậy, hãy chạy bản dựng **lớn nhất** của MiniMax M2.1 mà bạn có thể chạy cục bộ (LM Studio) và xem [/gateway/local-models](/gateway/local-models). Các mô hình nhỏ hơn/được lượng tử hóa làm tăng rủi ro tiêm nhắc - xem [Security](/gateway/security).

### Làm cách nào để giữ lưu lượng mô hình được lưu trữ trong một khu vực cụ thể

Chọn các điểm cuối được ghim theo khu vực. OpenRouter cung cấp các tùy chọn được lưu trữ ở Mỹ cho MiniMax, Kimi và GLM; chọn biến thể được lưu trữ ở Mỹ để giữ dữ liệu trong khu vực. Bạn vẫn có thể liệt kê Anthropic/OpenAI cùng với những tùy chọn này bằng cách sử dụng `models.mode: "merge"` để các fallback vẫn khả dụng trong khi tôn trọng nhà cung cấp được phân vùng mà bạn chọn.

### Tôi có phải mua Mac Mini để cài đặt điều này không

Không. OpenClaw chạy trên macOS hoặc Linux (Windows qua WSL2). Mac mini là tùy chọn - một số người mua nó làm máy chủ luôn bật, nhưng một VPS nhỏ, máy chủ gia đình hoặc hộp loại Raspberry Pi cũng hoạt động.

Bạn chỉ cần Mac **cho các công cụ chỉ dành cho macOS**. Đối với iMessage, hãy sử dụng [BlueBubbles](/channels/bluebubbles) (được khuyến nghị) - máy chủ BlueBubbles chạy trên bất kỳ Mac nào, và Gateway có thể chạy trên Linux hoặc nơi khác. Nếu bạn muốn các công cụ chỉ dành cho macOS khác, hãy chạy Gateway trên Mac hoặc ghép nối một node macOS.

Tài liệu: [BlueBubbles](/channels/bluebubbles), [Nodes](/nodes), [Mac remote mode](/platforms/mac/remote).

### Tôi có cần Mac mini để hỗ trợ iMessage không

Bạn cần **một số thiết bị macOS** được đăng nhập vào Messages. Nó **không** phải là Mac mini - bất kỳ Mac nào cũng hoạt động. **Sử dụng [BlueBubbles](/channels/bluebubbles)**  (được khuyến nghị) cho iMessage - máy chủ BlueBubbles chạy trên macOS, trong khi Gateway có thể chạy trên Linux hoặc nơi khác.

Các thiết lập phổ biến:

- Chạy Gateway trên Linux/VPS, và chạy máy chủ BlueBubbles trên bất kỳ Mac nào được đăng nhập vào Messages.
- Chạy mọi thứ trên Mac nếu bạn muốn thiết lập máy đơn đơn giản nhất.

Tài liệu: [BlueBubbles](/channels/bluebubbles), [Nodes](/nodes),
[Mac remote mode](/platforms/mac/remote).

### Nếu tôi mua Mac mini để chạy OpenClaw, tôi có thể kết nối nó với MacBook Pro của mình không

Có. **Mac mini có thể chạy Gateway**, và MacBook Pro của bạn có thể kết nối như một **node** (thiết bị đi kèm). Các node không chạy Gateway - chúng cung cấp các khả năng bổ sung như màn hình/camera/canvas và `system.run` trên thiết bị đó.

Mẫu phổ biến:

- Gateway trên Mac mini (luôn bật).
- MacBook Pro chạy ứng dụng macOS hoặc máy chủ node và ghép nối với Gateway.
- Sử dụng `openclaw nodes status` / `openclaw nodes list` để xem nó.

Tài liệu: [Nodes](/nodes), [Nodes CLI](/cli/nodes).

### Tôi có thể sử dụng Bun không

Bun **không được khuyến nghị**. Chúng tôi thấy các lỗi runtime, đặc biệt là với WhatsApp và Telegram.
Sử dụng **Node** cho các gateway ổn định.

Nếu bạn vẫn muốn thử nghiệm với Bun, hãy làm điều đó trên một gateway không phải sản xuất mà không có WhatsApp/Telegram.

### Telegram - cái gì vào allowFrom

`channels.telegram.allowFrom` là **ID người dùng Telegram của người gửi** (số). Nó không phải là tên người dùng bot.

Trình hướng dẫn thiết lập ban đầu chấp nhận đầu vào `@username` và giải quyết nó thành ID số, nhưng ủy quyền OpenClaw chỉ sử dụng ID số.
Safer (không có bot của bên thứ ba):

- Gửi tin nhắn riêng cho bot của bạn, sau đó chạy `openclaw logs --follow` và đọc `from.id`.

Official Bot API:

- Gửi tin nhắn riêng cho bot của bạn, sau đó gọi `https://api.telegram.org/bot<bot_token>/getUpdates` và đọc `message.from.id`.

Third-party (ít riêng tư hơn):

- Gửi tin nhắn riêng cho `@userinfobot` hoặc `@getidsbot`.

Xem [/channels/telegram](/channels/telegram#access-control-dms--groups).

### Nhiều người có thể sử dụng một số WhatsApp với các instance OpenClaw khác nhau không

Có, thông qua **multi-agent routing**. Liên kết **tin nhắn riêng** WhatsApp của mỗi người gửi (peer `kind: "direct"`, người gửi E.164 như `+15551234567`) với một `agentId` khác, để mỗi người có không gian làm việc và kho lưu trữ phiên riêng của họ. Các câu trả lời vẫn đến từ **cùng một tài khoản WhatsApp**, và kiểm soát truy cập tin nhắn riêng (`channels.whatsapp.dmPolicy` / `channels.whatsapp.allowFrom`) là toàn cầu cho mỗi tài khoản WhatsApp. Xem [Multi-Agent Routing](/concepts/multi-agent) và [WhatsApp](/channels/whatsapp).

### Tôi có thể chạy một agent chat nhanh và một agent Opus để viết mã không

Có. Sử dụng multi-agent routing: cấp cho mỗi agent mô hình mặc định riêng của nó, sau đó liên kết các tuyến đường đến (tài khoản nhà cung cấp hoặc các peer cụ thể) với mỗi agent. Cấu hình ví dụ nằm trong [Multi-Agent Routing](/concepts/multi-agent). Xem thêm [Models](/concepts/models) và [Configuration](/gateway/configuration).

### Homebrew có hoạt động trên Linux không

Có. Homebrew hỗ trợ Linux (Linuxbrew). Thiết lập nhanh:

```bash
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
echo 'eval "$(/home/linuxbrew/.linuxbrew/bin/brew shellenv)"' >> ~/.profile
eval "$(/home/linuxbrew/.linuxbrew/bin/brew shellenv)"
brew install <formula>
```

If you run OpenClaw via systemd, ensure the service PATH includes `/home/linuxbrew/.linuxbrew/bin` (or your brew prefix) so `brew`-installed tools resolve in non-login shells.
Recent builds also prepend common user bin dirs on Linux systemd services (for example `~/.local/bin`, `~/.npm-global/bin`, `~/.local/share/pnpm`, `~/.bun/bin`) and honor `PNPM_HOME`, `NPM_CONFIG_PREFIX`, `BUN_INSTALL`, `VOLTA_HOME`, `ASDF_DATA_DIR`, `NVM_DIR`, and `FNM_DIR` when set.

### What's the difference between the hackable git install and npm install

- **Hackable (git) install:** full source checkout, editable, best for contributors.
  You run builds locally and can patch code/docs.
- **npm install:** global CLI install, no repo, best for "just run it."
  Updates come from npm dist-tags.

Docs: [Getting started](/start/getting-started), [Updating](/install/updating).

### Can I switch between npm and git installs later

Yes. Install the other flavor, then run Doctor so the gateway service points at the new entrypoint.
This **does not delete your data** - it only changes the OpenClaw code install. Your state
(`~/.openclaw`) and workspace (`~/.openclaw/workspace`) stay untouched.

From npm → git:

```bash
git clone https://github.com/openclaw/openclaw.git
cd openclaw
pnpm install
pnpm build
openclaw doctor
openclaw gateway restart
```
Từ git → npm:

```bash
npm install -g openclaw@latest
openclaw doctor
openclaw gateway restart
```

Doctor detects a gateway service entrypoint mismatch and offers to rewrite the service config to match the current install (use `--repair` trong tự động hóa).

Mẹo sao lưu: xem [Chiến lược sao lưu](/help/faq#whats-the-recommended-backup-strategy).

### Tôi nên chạy Gateway trên laptop hay VPS

Câu trả lời ngắn: **nếu bạn muốn độ tin cậy 24/7, hãy sử dụng VPS**. Nếu bạn muốn giảm thiểu rắc rối và bạn không vấn đề với chế độ ngủ/khởi động lại, hãy chạy nó cục bộ.

**Laptop (Gateway cục bộ)**

- **Ưu điểm:** không có chi phí máy chủ, truy cập trực tiếp vào tệp cục bộ, cửa sổ trình duyệt trực tiếp.
- **Nhược điểm:** chế độ ngủ/mất kết nối mạng = ngắt kết nối, cập nhật OS/khởi động lại gián đoạn, phải luôn hoạt động.

**VPS / đám mây**

- **Ưu điểm:** luôn bật, mạng ổn định, không có vấn đề ngủ laptop, dễ dàng giữ chạy.
- **Nhược điểm:** thường chạy headless (sử dụng ảnh chụp màn hình), chỉ truy cập tệp từ xa, bạn phải SSH để cập nhật.

**Lưu ý dành riêng cho OpenClaw:** WhatsApp/Telegram/Slack/Mattermost (plugin)/Discord đều hoạt động tốt từ VPS. Sự đánh đổi duy nhất là **trình duyệt headless** so với cửa sổ hiển thị. Xem [Trình duyệt](/tools/browser).

**Mặc định được khuyến nghị:** VPS nếu bạn từng gặp sự cố ngắt kết nối gateway trước đây. Cục bộ rất tuyệt vời khi bạn đang sử dụng Mac một cách tích cực và muốn truy cập tệp cục bộ hoặc tự động hóa UI với trình duyệt hiển thị.

### Chạy OpenClaw trên một máy chuyên dụng quan trọng đến mức nào

Không bắt buộc, nhưng **được khuyến nghị để đảm bảo độ tin cậy và cách ly**.

- **Host chuyên dụng (VPS/Mac mini/Pi):** luôn bật, ít gián đoạn ngủ/khởi động lại, quyền hạn sạch hơn, dễ dàng giữ chạy.
- **Laptop/máy tính để bàn dùng chung:** hoàn toàn ổn cho việc kiểm tra và sử dụng tích cực, nhưng hãy dự kiến tạm dừng khi máy ngủ hoặc cập nhật.

Nếu bạn muốn có cả hai thế giới tốt nhất, hãy giữ Gateway trên một host chuyên dụng và ghép nối laptop của bạn như một **node** cho các công cụ màn hình/camera/exec cục bộ. Xem [Nodes](/nodes).
Để biết hướng dẫn bảo mật, hãy đọc [Bảo mật](/gateway/security).

### Yêu cầu VPS tối thiểu và OS được khuyến nghị là gì

OpenClaw rất nhẹ. Đối với Gateway cơ bản + một kênh trò chuyện:

- **Tối thiểu tuyệt đối:** 1 vCPU, 1GB RAM, ~500MB đĩa.
- **Được khuyến nghị:** 1-2 vCPU, 2GB RAM hoặc nhiều hơn để có dư địa (nhật ký, phương tiện, nhiều kênh). Các công cụ node và tự động hóa trình duyệt có thể tiêu tốn nhiều tài nguyên.

OS: sử dụng **Ubuntu LTS** (hoặc bất kỳ Debian/Ubuntu hiện đại nào). Đường dẫn cài đặt Linux được kiểm tra tốt nhất ở đó.

Tài liệu: [Linux](/platforms/linux), [Lưu trữ VPS](/vps).

### Tôi có thể chạy OpenClaw trong VM và yêu cầu là gì

Có. Coi VM giống như VPS: nó cần luôn bật, có thể truy cập được, và có đủ RAM cho Gateway và bất kỳ kênh nào bạn bật.

Hướng dẫn cơ bản:
- **Tối thiểu tuyệt đối:** 1 vCPU, 1GB RAM.
- **Khuyến nghị:** 2GB RAM hoặc nhiều hơn nếu bạn chạy nhiều kênh, tự động hóa trình duyệt hoặc công cụ xử lý phương tiện.
- **Hệ điều hành:** Ubuntu LTS hoặc một Debian/Ubuntu hiện đại khác.

Nếu bạn đang sử dụng Windows, **WSL2 là thiết lập kiểu VM dễ nhất** và có khả năng tương thích công cụ tốt nhất.
Xem [Windows](/platforms/windows), [Lưu trữ VPS](/vps).
Nếu bạn đang chạy macOS trong một VM, xem [macOS VM](/install/macos-vm).
## OpenClaw là gì?

### OpenClaw là gì trong một đoạn văn

OpenClaw là một trợ lý AI cá nhân mà bạn chạy trên các thiết bị của riêng mình. Nó trả lời trên các bề mặt nhắn tin mà bạn đã sử dụng (WhatsApp, Telegram, Slack, Mattermost (plugin), Discord, Google Chat, Signal, iMessage, WebChat) và cũng có thể thực hiện voice + Canvas trực tiếp trên các nền tảng được hỗ trợ. **Gateway** là mặt phẳng điều khiển luôn hoạt động; trợ lý là sản phẩm.

### Đề xuất giá trị là gì

OpenClaw không phải là "chỉ một wrapper Claude." Đó là một **mặt phẳng điều khiển ưu tiên cục bộ** cho phép bạn chạy một trợ lý có khả năng trên **phần cứng của riêng bạn**, có thể truy cập từ các ứng dụng chat mà bạn đã sử dụng, với các phiên có trạng thái, bộ nhớ và công cụ - mà không cần giao quyền kiểm soát quy trình làm việc của bạn cho một SaaS được lưu trữ.

Điểm nổi bật:

- **Thiết bị của bạn, dữ liệu của bạn:** chạy Gateway ở bất kỳ đâu bạn muốn (Mac, Linux, VPS) và giữ không gian làm việc + lịch sử phiên cục bộ.
- **Các kênh thực, không phải sandbox web:** WhatsApp/Telegram/Slack/Discord/Signal/iMessage/v.v., cộng với voice di động và Canvas trên các nền tảng được hỗ trợ.
- **Không phụ thuộc mô hình:** sử dụng Anthropic, OpenAI, MiniMax, OpenRouter, v.v., với định tuyến và failover cho mỗi agent.
- **Tùy chọn chỉ cục bộ:** chạy các mô hình cục bộ để **tất cả dữ liệu có thể ở lại trên thiết bị của bạn** nếu bạn muốn.
- **Định tuyến đa agent:** các agent riêng biệt cho mỗi kênh, tài khoản hoặc tác vụ, mỗi cái có không gian làm việc và mặc định riêng.
- **Mã nguồn mở và có thể hack:** kiểm tra, mở rộng và tự lưu trữ mà không bị khóa nhà cung cấp.

Tài liệu: [Gateway](/gateway), [Channels](/channels), [Multi-agent](/concepts/multi-agent),
[Memory](/concepts/memory).

### Tôi vừa thiết lập xong, tôi nên làm gì trước tiên

Các dự án tốt đầu tiên:

- Xây dựng một trang web (WordPress, Shopify, hoặc một trang tĩnh đơn giản).
- Tạo mẫu một ứng dụng di động (phác thảo, màn hình, kế hoạch API).
- Tổ chức tệp và thư mục (dọn dẹp, đặt tên, gắn thẻ).
- Kết nối Gmail và tự động hóa tóm tắt hoặc theo dõi.

Nó có thể xử lý các tác vụ lớn, nhưng nó hoạt động tốt nhất khi bạn chia chúng thành các giai đoạn và sử dụng các agent phụ để làm việc song song.

### Năm trường hợp sử dụng hàng ngày hàng đầu cho OpenClaw là gì

Những chiến thắng hàng ngày thường trông như:

- **Báo cáo cá nhân:** tóm tắt hộp thư đến, lịch và tin tức mà bạn quan tâm.
- **Nghiên cứu và soạn thảo:** nghiên cứu nhanh, tóm tắt và bản nháp đầu tiên cho email hoặc tài liệu.
- **Nhắc nhở và theo dõi:** các nhắc nhở và danh sách kiểm tra được điều khiển bởi cron hoặc nhịp tim.
- **Tự động hóa trình duyệt:** điền biểu mẫu, thu thập dữ liệu và lặp lại các tác vụ web.
- **Phối hợp đa thiết bị:** gửi một tác vụ từ điện thoại của bạn, để Gateway chạy nó trên máy chủ và nhận kết quả trở lại trong chat.

### OpenClaw có thể giúp với lead gen outreach ads và blogs cho SaaS không

Có cho **nghiên cứu, xác định đủ điều kiện và soạn thảo**. Nó có thể quét các trang web, xây dựng danh sách ngắn, tóm tắt khách hàng tiềm năng và viết bản nháp sao chép tiếp cận hoặc quảng cáo.

Đối với **các lần tiếp cận hoặc chạy quảng cáo**, hãy giữ một con người trong vòng lặp. Tránh spam, tuân theo luật địa phương và chính sách nền tảng, và xem xét bất cứ điều gì trước khi nó được gửi. Mô hình an toàn nhất là để OpenClaw soạn thảo và bạn phê duyệt.
Docs: [Bảo mật](/gateway/security).

### Những ưu điểm so với Claude Code cho phát triển web

OpenClaw là một **trợ lý cá nhân** và lớp điều phối, không phải thay thế IDE. Sử dụng
Claude Code hoặc Codex để có vòng lặp mã hóa trực tiếp nhanh nhất bên trong một kho lưu trữ. Sử dụng OpenClaw khi bạn
muốn bộ nhớ bền vững, truy cập đa thiết bị và điều phối công cụ.

Ưu điểm:

- **Bộ nhớ bền vững + không gian làm việc** trên các phiên
- **Truy cập đa nền tảng** (WhatsApp, Telegram, TUI, WebChat)
- **Điều phối công cụ** (trình duyệt, tệp, lập lịch, hook)
- **Gateway luôn bật** (chạy trên VPS, tương tác từ bất kỳ đâu)
- **Nodes** cho trình duyệt/màn hình/camera/exec cục bộ

Trưng bày: [https://openclaw.ai/showcase](https://openclaw.ai/showcase)
## Skills và tự động hóa

### Làm cách nào để tùy chỉnh Skills mà không làm bẩn repo

Sử dụng managed overrides thay vì chỉnh sửa bản sao repo. Đặt các thay đổi của bạn trong `~/.openclaw/skills/<name>/SKILL.md` (hoặc thêm một thư mục qua `skills.load.extraDirs` trong `~/.openclaw/openclaw.json`). Thứ tự ưu tiên là `<workspace>/skills` > `~/.openclaw/skills` > bundled, vì vậy managed overrides sẽ thắng mà không cần chạm vào git. Chỉ những chỉnh sửa đáng giá upstream mới nên nằm trong repo và được gửi dưới dạng PRs.

### Tôi có thể tải Skills từ một thư mục tùy chỉnh không

Có. Thêm các thư mục bổ sung qua `skills.load.extraDirs` trong `~/.openclaw/openclaw.json` (ưu tiên thấp nhất). Thứ tự ưu tiên mặc định vẫn giữ nguyên: `<workspace>/skills` → `~/.openclaw/skills` → bundled → `skills.load.extraDirs`. `clawhub` cài đặt vào `./skills` theo mặc định, mà OpenClaw coi là `<workspace>/skills`.

### Làm cách nào để sử dụng các mô hình khác nhau cho các tác vụ khác nhau

Ngày nay các mẫu được hỗ trợ là:

- **Cron jobs**: các công việc cô lập có thể đặt một override `model` cho mỗi công việc.
- **Sub-agents**: định tuyến các tác vụ đến các agent riêng biệt với các mô hình mặc định khác nhau.
- **Chuyển đổi theo yêu cầu**: sử dụng `/model` để chuyển đổi mô hình phiên hiện tại bất kỳ lúc nào.

Xem [Cron jobs](/automation/cron-jobs), [Multi-Agent Routing](/concepts/multi-agent), và [Slash commands](/tools/slash-commands).

### Bot bị đông cứng khi thực hiện công việc nặng Làm cách nào để tôi có thể chuyển tải công việc đó

Sử dụng **sub-agents** cho các tác vụ dài hoặc song song. Sub-agents chạy trong phiên riêng của chúng,
trả về một bản tóm tắt, và giữ cho cuộc trò chuyện chính của bạn phản hồi.

Yêu cầu bot của bạn "spawn a sub-agent for this task" hoặc sử dụng `/subagents`.
Sử dụng `/status` trong chat để xem Gateway đang làm gì ngay bây giờ (và liệu nó có bận hay không).

Token tip: các tác vụ dài và sub-agents đều tiêu thụ token. Nếu chi phí là một mối quan tâm, hãy đặt một
mô hình rẻ hơn cho sub-agents qua `agents.defaults.subagents.model`.

Docs: [Sub-agents](/tools/subagents).

### Các phiên subagent liên kết với thread hoạt động như thế nào trên Discord

Sử dụng thread bindings. Bạn có thể liên kết một thread Discord với một subagent hoặc session target để các tin nhắn tiếp theo trong thread đó vẫn ở trên phiên được liên kết đó.

Luồng cơ bản:

- Spawn với `sessions_spawn` sử dụng `thread: true` (và tùy chọn `mode: "session"` cho follow-up liên tục).
- Hoặc liên kết thủ công với `/focus <target>`.
- Sử dụng `/agents` để kiểm tra trạng thái liên kết.
- Sử dụng `/session idle <duration|off>` và `/session max-age <duration|off>` để kiểm soát auto-unfocus.
- Sử dụng `/unfocus` để tách thread.

Cấu hình bắt buộc:

- Mặc định toàn cầu: `session.threadBindings.enabled`, `session.threadBindings.idleHours`, `session.threadBindings.maxAgeHours`.
- Discord overrides: `channels.discord.threadBindings.enabled`, `channels.discord.threadBindings.idleHours`, `channels.discord.threadBindings.maxAgeHours`.
- Auto-bind on spawn: đặt `channels.discord.threadBindings.spawnSubagentSessions: true`.

Docs: [Sub-agents](/tools/subagents), [Discord](/channels/discord), [Configuration Reference](/gateway/configuration-reference), [Slash commands](/tools/slash-commands).

### Cron hoặc reminders không kích hoạt Tôi nên kiểm tra cái gì

Cron chạy bên trong quá trình Gateway. Nếu Gateway không chạy liên tục,
các công việc được lên lịch sẽ không chạy.

Danh sách kiểm tra:
- Xác nhận cron được bật (`cron.enabled`) và `OPENCLAW_SKIP_CRON` không được đặt.
- Kiểm tra Gateway đang chạy 24/7 (không ngủ/khởi động lại).
- Xác minh cài đặt múi giờ cho công việc (`--tz` so với múi giờ máy chủ).

Gỡ lỗi:

```bash
openclaw cron run <jobId> --force
openclaw cron runs --id <jobId> --limit 50
```

Docs: [Cron jobs](/automation/cron-jobs), [Cron vs Heartbeat](/automation/cron-vs-heartbeat).

### How do I install skills on Linux

Use **ClawHub** (CLI) or drop skills into your workspace. The macOS Skills UI isn't available on Linux.
Browse skills at [https://clawhub.com](https://clawhub.com).

Install the ClawHub CLI (pick one package manager):

```bash
npm i -g clawhub
```

```bash
pnpm add -g clawhub
```

### Can OpenClaw run tasks on a schedule or continuously in the background

Yes. Use the Gateway scheduler:

- **Cron jobs** for scheduled or recurring tasks (persist across restarts).
- **Heartbeat** for "main session" periodic checks.
- **Isolated jobs** for autonomous agents that post summaries or deliver to chats.

Docs: [Cron jobs](/automation/cron-jobs), [Cron vs Heartbeat](/automation/cron-vs-heartbeat),
[Heartbeat](/gateway/heartbeat).

### Can I run Apple macOS-only skills from Linux?

Not directly. macOS skills are gated by `metadata.openclaw.os` plus required binaries, and skills only appear in the system prompt when they are eligible on the **Gateway host**. On Linux, `darwin`-only skills (like `apple-notes`, `apple-reminders`, `things-mac`) will not load unless you override the gating.

You have three supported patterns:

**Option A - run the Gateway on a Mac (simplest).**
Run the Gateway where the macOS binaries exist, then connect from Linux in [remote mode](#how-do-i-run-openclaw-in-remote-mode-client-connects-to-a-gateway-elsewhere) or over Tailscale. The skills load normally because the Gateway host is macOS.

**Option B - use a macOS node (no SSH).**
Run the Gateway on Linux, pair a macOS node (menubar app), and set **Node Run Commands** to "Always Ask" or "Always Allow" on the Mac. OpenClaw can treat macOS-only skills as eligible when the required binaries exist on the node. The agent runs those skills via the `nodes` tool. If you choose "Always Ask", approving "Always Allow" in the prompt adds that command to the allowlist.

**Option C - proxy macOS binaries over SSH (advanced).**
Keep the Gateway on Linux, but make the required CLI binaries resolve to SSH wrappers that run on a Mac. Then override the skill to allow Linux so it stays eligible.

1. Create an SSH wrapper for the binary (example: `memo` for Apple Notes):

   ```bash
   #!/usr/bin/env bash
   set -euo pipefail
   exec ssh -T user@mac-host /opt/homebrew/bin/memo "$@"
   ```
2. Đặt wrapper trên `PATH` trên máy chủ Linux (ví dụ `~/bin/memo`).
3. Ghi đè siêu dữ liệu skill (workspace hoặc `~/.openclaw/skills`) để cho phép Linux:

   ```markdown
   ---
   name: apple-notes
   description: Manage Apple Notes via the memo CLI on macOS.
   metadata: { "openclaw": { "os": ["darwin", "linux"], "requires": { "bins": ["memo"] } } }
   ---
   ```

4. Start a new session so the skills snapshot refreshes.

### Do you have a Notion or HeyGen integration

Not built-in today.

Options:

- **Custom skill / plugin:** best for reliable API access (Notion/HeyGen both have APIs).
- **Browser automation:** works without code but is slower and more fragile.

If you want to keep context per client (agency workflows), a simple pattern is:

- One Notion page per client (context + preferences + active work).
- Ask the agent to fetch that page at the start of a session.

If you want a native integration, open a feature request or build a skill
targeting those APIs.

Install skills:

```bash
clawhub install <skill-slug>
clawhub update --all
```

ClawHub installs into `./skills` under your current directory (or falls back to your configured OpenClaw workspace); OpenClaw treats that as `<workspace>/skills` on the next session. For shared skills across agents, place them in `~/.openclaw/skills/<name>/SKILL.md`. Some skills expect binaries installed via Homebrew; on Linux that means Linuxbrew (see the Homebrew Linux FAQ entry above). See [Skills](/tools/skills) and [ClawHub](/tools/clawhub).

### How do I install the Chrome extension for browser takeover

Use the built-in installer, then load the unpacked extension in Chrome:

```bash
openclaw browser extension install
openclaw browser extension path
```

Then Chrome → `chrome://extensions` → bật "Developer mode" → "Load unpacked" → chọn thư mục đó.

Hướng dẫn đầy đủ (bao gồm Gateway từ xa + ghi chú bảo mật): [Chrome extension](/tools/chrome-extension)

Nếu Gateway chạy trên cùng máy với Chrome (thiết lập mặc định), bạn thường **không** cần bất cứ điều gì thêm.
Nếu Gateway chạy ở nơi khác, hãy chạy một node host trên máy trình duyệt để Gateway có thể proxy các hành động trình duyệt.
Bạn vẫn cần nhấp vào nút tiện ích mở rộng trên tab bạn muốn kiểm soát (nó không tự động đính kèm).
## Sandboxing và bộ nhớ

### Có tài liệu sandboxing chuyên dụng không

Có. Xem [Sandboxing](/gateway/sandboxing). Để thiết lập cụ thể cho Docker (gateway đầy đủ trong Docker hoặc hình ảnh sandbox), xem [Docker](/install/docker).

### Docker cảm thấy bị giới hạn. Làm cách nào để bật các tính năng đầy đủ

Hình ảnh mặc định ưu tiên bảo mật và chạy dưới dạng người dùng `node`, vì vậy nó không bao gồm các gói hệ thống, Homebrew hoặc trình duyệt đi kèm. Để thiết lập đầy đủ hơn:

- Duy trì `/home/node` với `OPENCLAW_HOME_VOLUME` để bộ nhớ cache tồn tại.
- Bake các phụ thuộc hệ thống vào hình ảnh với `OPENCLAW_DOCKER_APT_PACKAGES`.
- Cài đặt trình duyệt Playwright qua CLI đi kèm:
  `node /app/node_modules/playwright-core/cli.js install chromium`
- Đặt `PLAYWRIGHT_BROWSERS_PATH` và đảm bảo đường dẫn được duy trì.

Tài liệu: [Docker](/install/docker), [Browser](/tools/browser).

**Tôi có thể giữ tin nhắn riêng ở chế độ cá nhân nhưng công khai các nhóm được sandbox hóa bằng một agent không**

Có - nếu lưu lượng riêng tư của bạn là **tin nhắn riêng** và lưu lượng công khai của bạn là **nhóm**.

Sử dụng `agents.defaults.sandbox.mode: "non-main"` để các phiên nhóm/kênh (khóa không chính) chạy trong Docker, trong khi phiên tin nhắn riêng chính vẫn ở trên máy chủ. Sau đó hạn chế các công cụ có sẵn trong các phiên được sandbox hóa qua `tools.sandbox.tools`.

Hướng dẫn thiết lập + cấu hình ví dụ: [Nhóm: tin nhắn riêng cá nhân + nhóm công khai](/channels/groups#pattern-personal-dms-public-groups-single-agent)

Tham chiếu cấu hình chính: [Cấu hình Gateway](/gateway/configuration#agentsdefaultssandbox)

### Làm cách nào để liên kết thư mục máy chủ vào sandbox

Đặt `agents.defaults.sandbox.docker.binds` thành `["host:path:mode"]` (ví dụ: `"/home/user/src:/src:ro"`). Các liên kết toàn cầu + mỗi agent được hợp nhất; các liên kết mỗi agent bị bỏ qua khi `scope: "shared"`. Sử dụng `:ro` cho bất kỳ thứ gì nhạy cảm và nhớ rằng các liên kết vượt qua các bức tường hệ thống tệp sandbox. Xem [Sandboxing](/gateway/sandboxing#custom-bind-mounts) và [Sandbox vs Tool Policy vs Elevated](/gateway/sandbox-vs-tool-policy-vs-elevated#bind-mounts-security-quick-check) để xem các ví dụ và ghi chú an toàn.

### Bộ nhớ hoạt động như thế nào

Bộ nhớ OpenClaw chỉ là các tệp Markdown trong không gian làm việc agent:

- Ghi chú hàng ngày trong `memory/YYYY-MM-DD.md`
- Ghi chú dài hạn được sắp xếp trong `MEMORY.md` (chỉ các phiên chính/riêng tư)

OpenClaw cũng chạy **xả bộ nhớ nén trước im lặng** để nhắc nhở mô hình viết ghi chú bền vững trước khi nén tự động. Điều này chỉ chạy khi không gian làm việc có thể ghi (các sandbox chỉ đọc bỏ qua nó). Xem [Memory](/concepts/memory).

### Bộ nhớ liên tục quên những thứ. Làm cách nào để làm cho nó tồn tại

Yêu cầu bot **viết sự kiện vào bộ nhớ**. Ghi chú dài hạn thuộc về `MEMORY.md`, ngữ cảnh ngắn hạn đi vào `memory/YYYY-MM-DD.md`.

Đây vẫn là một lĩnh vực mà chúng tôi đang cải thiện. Sẽ hữu ích nếu nhắc nhở mô hình lưu trữ bộ nhớ; nó sẽ biết phải làm gì. Nếu nó liên tục quên, hãy xác minh rằng Gateway đang sử dụng cùng một không gian làm việc trên mỗi lần chạy.

Tài liệu: [Memory](/concepts/memory), [Agent workspace](/concepts/agent-workspace).

### Tìm kiếm bộ nhớ ngữ nghĩa có yêu cầu khóa API OpenAI không

Chỉ khi bạn sử dụng **nhúng OpenAI**. Codex OAuth bao gồm chat/completions và **không** cấp quyền truy cập nhúng, vì vậy **đăng nhập bằng Codex (OAuth hoặc đăng nhập Codex CLI)** không giúp tìm kiếm bộ nhớ ngữ nghĩa. Nhúng OpenAI
vẫn cần một API key thực tế (`OPENAI_API_KEY` hoặc `models.providers.openai.apiKey`).

Nếu bạn không đặt nhà cung cấp một cách rõ ràng, OpenClaw sẽ tự động chọn nhà cung cấp khi
có thể giải quyết một API key (hồ sơ xác thực, `models.providers.*.apiKey`, hoặc biến môi trường).
Nó ưu tiên OpenAI nếu có thể giải quyết khóa OpenAI, nếu không thì Gemini nếu có thể giải quyết khóa Gemini,
sau đó là Voyage, rồi Mistral. Nếu không có khóa từ xa nào khả dụng, tìm kiếm bộ nhớ sẽ vẫn bị vô hiệu hóa cho đến khi bạn cấu hình nó. Nếu bạn có đường dẫn mô hình cục bộ
được cấu hình và có sẵn, OpenClaw
ưu tiên `local`.

Nếu bạn muốn ở cục bộ, hãy đặt `memorySearch.provider = "local"` (và tùy chọn
`memorySearch.fallback = "none"`). Nếu bạn muốn nhúng Gemini, hãy đặt
`memorySearch.provider = "gemini"` và cung cấp `GEMINI_API_KEY` (hoặc
`memorySearch.remote.apiKey`). Chúng tôi hỗ trợ **OpenAI, Gemini, Voyage, Mistral, hoặc cục bộ** các mô hình nhúng - xem [Memory](/concepts/memory) để biết chi tiết thiết lập.

### Bộ nhớ có tồn tại mãi mãi không? Giới hạn là gì?

Các tệp bộ nhớ nằm trên đĩa và tồn tại cho đến khi bạn xóa chúng. Giới hạn là dung lượng lưu trữ của bạn, không phải mô hình. **Ngữ cảnh phiên** vẫn bị giới hạn bởi cửa sổ ngữ cảnh của mô hình, vì vậy các cuộc trò chuyện dài có thể nén hoặc cắt ngắn. Đó là lý do tại sao tìm kiếm bộ nhớ tồn tại - nó chỉ kéo các phần liên quan trở lại ngữ cảnh.

Tài liệu: [Memory](/concepts/memory), [Context](/concepts/context).
## Vị trí lưu trữ dữ liệu

### Tất cả dữ liệu được sử dụng với OpenClaw có được lưu cục bộ không

Không - **trạng thái của OpenClaw là cục bộ**, nhưng **các dịch vụ bên ngoài vẫn nhìn thấy những gì bạn gửi cho họ**.

- **Cục bộ theo mặc định:** phiên, tệp bộ nhớ, cấu hình và không gian làm việc nằm trên máy chủ Gateway
  (`~/.openclaw` + thư mục không gian làm việc của bạn).
- **Từ xa theo yêu cầu:** tin nhắn bạn gửi đến các nhà cung cấp mô hình (Anthropic/OpenAI/v.v.) sẽ đi đến
  các API của họ, và các nền tảng trò chuyện (WhatsApp/Telegram/Slack/v.v.) lưu trữ dữ liệu tin nhắn trên các
  máy chủ của họ.
- **Bạn kiểm soát dấu chân:** sử dụng các mô hình cục bộ giữ các lời nhắc trên máy của bạn, nhưng lưu lượng kênh vẫn
  đi qua các máy chủ của kênh.

Liên quan: [Agent workspace](/concepts/agent-workspace), [Memory](/concepts/memory).

### OpenClaw lưu trữ dữ liệu của nó ở đâu

Mọi thứ nằm dưới `$OPENCLAW_STATE_DIR` (mặc định: `~/.openclaw`):

| Đường dẫn                                                       | Mục đích                                                           |
| --------------------------------------------------------------- | ------------------------------------------------------------------ |
| `$OPENCLAW_STATE_DIR/openclaw.json`                             | Cấu hình chính (JSON5)                                             |
| `$OPENCLAW_STATE_DIR/credentials/oauth.json`                    | Nhập OAuth cũ (được sao chép vào hồ sơ xác thực khi sử dụng lần đầu) |
| `$OPENCLAW_STATE_DIR/agents/<agentId>/agent/auth-profiles.json` | Hồ sơ xác thực (OAuth, khóa API và `keyRef`/`tokenRef` tùy chọn) |
| `$OPENCLAW_STATE_DIR/secrets.json`                              | Tải trọng bí mật được hỗ trợ tệp tùy chọn cho các nhà cung cấp `file` SecretRef |
| `$OPENCLAW_STATE_DIR/agents/<agentId>/agent/auth.json`          | Tệp tương thích cũ (các mục `api_key` tĩnh được xóa)        |
| `$OPENCLAW_STATE_DIR/credentials/`                              | Trạng thái nhà cung cấp (ví dụ: `whatsapp/<accountId>/creds.json`)                  |
| `$OPENCLAW_STATE_DIR/agents/`                                   | Trạng thái cho mỗi agent (agentDir + phiên)                        |
| `$OPENCLAW_STATE_DIR/agents/<agentId>/sessions/`                | Lịch sử và trạng thái cuộc trò chuyện (cho mỗi agent)              |
| `$OPENCLAW_STATE_DIR/agents/<agentId>/sessions/sessions.json`   | Siêu dữ liệu phiên (cho mỗi agent)                                 |

Đường dẫn agent đơn cũ: `~/.openclaw/agent/*` (được di chuyển bởi `openclaw doctor`).

**Không gian làm việc** của bạn (AGENTS.md, tệp bộ nhớ, Skills, v.v.) là riêng biệt và được cấu hình thông qua `agents.defaults.workspace` (mặc định: `~/.openclaw/workspace`).

### AGENTS.md, SOUL.md, USER.md, MEMORY.md nên nằm ở đâu

Các tệp này nằm trong **agent workspace**, không phải `~/.openclaw`.

- **Workspace (cho mỗi agent)**: `AGENTS.md`, `SOUL.md`, `IDENTITY.md`, `USER.md`,
  `MEMORY.md` (hoặc `memory.md`), `memory/YYYY-MM-DD.md`, `HEARTBEAT.md` tùy chọn.
- **Thư mục trạng thái (`~/.openclaw`)**: cấu hình, thông tin xác thực, hồ sơ xác thực, phiên, nhật ký,
  và Skills được chia sẻ (`~/.openclaw/skills`).

Workspace mặc định là `~/.openclaw/workspace`, có thể cấu hình thông qua:

```json5
{
  agents: { defaults: { workspace: "~/.openclaw/workspace" } },
}
```

Nếu bot "quên" sau khi khởi động lại, hãy xác nhận rằng Gateway đang sử dụng cùng một
workspace trên mỗi lần khởi chạy (và nhớ rằng: chế độ từ xa sử dụng **workspace của máy chủ gateway**,
không phải laptop cục bộ của bạn).

Mẹo: nếu bạn muốn một hành vi hoặc tùy chọn bền vững, hãy yêu cầu bot **viết nó vào
AGENTS.md hoặc MEMORY.md** thay vì dựa vào lịch sử trò chuyện.
Xem [Agent workspace](/concepts/agent-workspace) và [Memory](/concepts/memory).

### Chiến lược sao lưu được khuyến nghị là gì

Đặt **agent workspace** của bạn trong một kho git **private** và sao lưu nó ở một nơi private (ví dụ GitHub private). Điều này ghi lại memory + các tệp AGENTS/SOUL/USER, và cho phép bạn khôi phục "tâm trí" của trợ lý sau này.

**Không** commit bất cứ thứ gì dưới `~/.openclaw` (thông tin xác thực, phiên, token, hoặc payload bí mật được mã hóa).
Nếu bạn cần khôi phục đầy đủ, hãy sao lưu cả workspace và thư mục state riêng biệt (xem câu hỏi về migration ở trên).

Tài liệu: [Agent workspace](/concepts/agent-workspace).

### Làm cách nào để gỡ cài đặt hoàn toàn OpenClaw

Xem hướng dẫn chuyên dụng: [Uninstall](/install/uninstall).

### Các agent có thể hoạt động bên ngoài workspace không

Có. Workspace là **cwd mặc định** và neo memory, không phải sandbox cứng.
Các đường dẫn tương đối được phân giải bên trong workspace, nhưng các đường dẫn tuyệt đối có thể truy cập các vị trí máy chủ khác trừ khi sandboxing được bật. Nếu bạn cần cách ly, hãy sử dụng
[`agents.defaults.sandbox`](/gateway/sandboxing) hoặc cài đặt sandbox cho từng agent. Nếu bạn muốn một kho là thư mục làm việc mặc định, hãy trỏ
`workspace` của agent đó đến gốc kho. Kho OpenClaw chỉ là mã nguồn; giữ workspace riêng biệt trừ khi bạn có ý định muốn agent hoạt động bên trong nó.

Ví dụ (kho làm cwd mặc định):

```json5
{
  agents: {
    defaults: {
      workspace: "~/Projects/my-repo",
    },
  },
}
```

### Tôi ở chế độ remote, session store ở đâu

Trạng thái phiên được sở hữu bởi **gateway host**. Nếu bạn ở chế độ remote, session store mà bạn quan tâm nằm trên máy remote, không phải laptop cục bộ của bạn. Xem [Session management](/concepts/session).
## Cơ bản về cấu hình

### Định dạng cấu hình là gì? Nó ở đâu?

OpenClaw đọc một cấu hình **JSON5** tùy chọn từ `$OPENCLAW_CONFIG_PATH` (mặc định: `~/.openclaw/openclaw.json`):

```
$OPENCLAW_CONFIG_PATH
```

If the file is missing, it uses safe-ish defaults (including a default workspace of `~/.openclaw/workspace`).

### I set gatewaybind lan or tailnet and now nothing listens the UI says unauthorized

Non-loopback binds **require auth**. Configure `gateway.auth.mode` + `gateway.auth.token` (or use `OPENCLAW_GATEWAY_TOKEN`).

```json5
{
  gateway: {
    bind: "lan",
    auth: {
      mode: "token",
      token: "replace-me",
    },
  },
}
```

Notes:

- `gateway.remote.token` / `.password` do **not** enable local gateway auth by themselves.
- Local call paths can use `gateway.remote.*` as fallback when `gateway.auth.*` is unset.
- The Control UI authenticates via `connect.params.auth.token` (stored in app/UI settings). Avoid putting tokens in URLs.

### Why do I need a token on localhost now

OpenClaw enforces token auth by default, including loopback. If no token is configured, gateway startup auto-generates one and saves it to `gateway.auth.token`, so **local WS clients must authenticate**. This blocks other local processes from calling the Gateway.

If you **really** want open loopback, set `gateway.auth.mode: "none"` explicitly in your config. Doctor can generate a token for you any time: `openclaw doctor --generate-gateway-token`.

### Do I have to restart after changing config

The Gateway watches the config and supports hot-reload:

- `gateway.reload.mode: "hybrid"` (default): hot-apply safe changes, restart for critical ones
- `hot`, `restart`, `off` are also supported

### How do I enable web search and web fetch

`web_fetch` works without an API key. `web_search` requires a Brave Search API
key. **Recommended:** run `openclaw configure --section web` to store it in
`tools.web.search.apiKey`. Environment alternative: set `BRAVE_API_KEY` for the
Gateway process.

```json5
{
  tools: {
    web: {
      search: {
        enabled: true,
        apiKey: "BRAVE_API_KEY_HERE",
        maxResults: 5,
      },
      fetch: {
        enabled: true,
      },
    },
  },
}
```
Ghi chú:

- Nếu bạn sử dụng danh sách cho phép, hãy thêm `web_search`/`web_fetch` hoặc `group:web`.
- `web_fetch` được bật theo mặc định (trừ khi bị tắt rõ ràng).
- Daemon đọc biến môi trường từ `~/.openclaw/.env` (hoặc môi trường dịch vụ).

Tài liệu: [Web tools](/tools/web).

### Làm cách nào để chạy một Gateway trung tâm với các worker chuyên biệt trên các thiết bị

Mô hình phổ biến là **một Gateway** (ví dụ: Raspberry Pi) cộng với **nodes** và **agents**:

- **Gateway (trung tâm):** sở hữu các kênh (Signal/WhatsApp), định tuyến và phiên.
- **Nodes (thiết bị):** Macs/iOS/Android kết nối như các thiết bị ngoại vi và hiển thị các công cụ cục bộ (`system.run`, `canvas`, `camera`).
- **Agents (workers):** các bộ não/không gian làm việc riêng biệt cho các vai trò đặc biệt (ví dụ: "Hetzner ops", "Dữ liệu cá nhân").
- **Sub-agents:** tạo công việc nền từ một agent chính khi bạn muốn song song hóa.
- **TUI:** kết nối với Gateway và chuyển đổi giữa các agent/phiên.

Tài liệu: [Nodes](/nodes), [Remote access](/gateway/remote), [Multi-Agent Routing](/concepts/multi-agent), [Sub-agents](/tools/subagents), [TUI](/web/tui).

### Trình duyệt OpenClaw có thể chạy ở chế độ headless không

Có. Đó là một tùy chọn cấu hình:

```json5
{
  browser: { headless: true },
  agents: {
    defaults: {
      sandbox: { browser: { headless: true } },
    },
  },
}
```

Default is `false` (headful). Headless is more likely to trigger anti-bot checks on some sites. See [Browser](/tools/browser).

Headless uses the **same Chromium engine** and works for most automation (forms, clicks, scraping, logins). The main differences:

- No visible browser window (use screenshots if you need visuals).
- Some sites are stricter about automation in headless mode (CAPTCHAs, anti-bot).
  For example, X/Twitter often blocks headless sessions.

### How do I use Brave for browser control

Set `browser.executablePath` thành tệp nhị phân Brave của bạn (hoặc bất kỳ trình duyệt dựa trên Chromium nào) và khởi động lại Gateway.
Xem các ví dụ cấu hình đầy đủ trong [Browser](/tools/browser#use-brave-or-another-chromium-based-browser).
## Gateway và node từ xa

### Lệnh được truyền như thế nào giữa Telegram, gateway và node

Các tin nhắn Telegram được xử lý bởi **gateway**. Gateway chạy agent và chỉ sau đó gọi các node qua **Gateway WebSocket** khi cần công cụ node:

Telegram → Gateway → Agent → `node.*` → Node → Gateway → Telegram

Các node không nhìn thấy lưu lượng nhà cung cấp inbound; chúng chỉ nhận các lệnh gọi RPC node.

### Làm cách nào agent của tôi có thể truy cập máy tính của tôi nếu Gateway được lưu trữ từ xa

Câu trả lời ngắn: **ghép máy tính của bạn như một node**. Gateway chạy ở nơi khác, nhưng nó có thể gọi các công cụ `node.*` (màn hình, camera, hệ thống) trên máy cục bộ của bạn qua Gateway WebSocket.

Thiết lập điển hình:

1. Chạy Gateway trên máy chủ luôn bật (VPS/máy chủ nhà).
2. Đặt máy chủ Gateway + máy tính của bạn trên cùng một tailnet.
3. Đảm bảo Gateway WS có thể truy cập được (tailnet bind hoặc SSH tunnel).
4. Mở ứng dụng macOS cục bộ và kết nối ở chế độ **Remote over SSH** (hoặc tailnet trực tiếp) để nó có thể đăng ký như một node.
5. Phê duyệt node trên Gateway:

   ```bash
   openclaw nodes pending
   openclaw nodes approve <requestId>
   ```

No separate TCP bridge is required; nodes connect over the Gateway WebSocket.

Security reminder: pairing a macOS node allows `system.run` on that machine. Only
pair devices you trust, and review [Security](/gateway/security).

Docs: [Nodes](/nodes), [Gateway protocol](/gateway/protocol), [macOS remote mode](/platforms/mac/remote), [Security](/gateway/security).

### Tailscale is connected but I get no replies What now

Check the basics:

- Gateway is running: `openclaw gateway status`
- Gateway health: `openclaw status`
- Channel health: `openclaw channels status`

Then verify auth and routing:

- If you use Tailscale Serve, make sure `gateway.auth.allowTailscale` được đặt chính xác.
- Nếu bạn kết nối qua SSH tunnel, hãy xác nhận tunnel cục bộ đang hoạt động và trỏ đến cổng chính xác.
- Xác nhận danh sách cho phép của bạn (tin nhắn riêng hoặc nhóm) bao gồm tài khoản của bạn.

Tài liệu: [Tailscale](/gateway/tailscale), [Truy cập từ xa](/gateway/remote), [Kênh](/channels).

### Hai instance OpenClaw có thể nói chuyện với nhau qua VPS cục bộ không

Có. Không có cầu nối "bot-to-bot" tích hợp sẵn, nhưng bạn có thể kết nối nó theo một vài cách đáng tin cậy:

**Đơn giản nhất:** sử dụng kênh trò chuyện bình thường mà cả hai bot có thể truy cập (Telegram/Slack/WhatsApp). Để Bot A gửi tin nhắn cho Bot B, sau đó để Bot B trả lời như bình thường.
**CLI bridge (generic):** chạy một script gọi Gateway khác với
`openclaw agent --message ... --deliver`, nhắm tới một kênh nơi bot khác
lắng nghe. Nếu một bot ở trên VPS từ xa, hãy trỏ CLI của bạn tới Gateway từ xa đó
qua SSH/Tailscale (xem [Truy cập từ xa](/gateway/remote)).

Mẫu ví dụ (chạy từ một máy có thể kết nối tới Gateway đích):

```bash
openclaw agent --message "Hello from local bot" --deliver --channel telegram --reply-to <chat-id>
```

Tip: add a guardrail so the two bots do not loop endlessly (mention-only, channel
allowlists, or a "do not reply to bot messages" rule).

Docs: [Remote access](/gateway/remote), [Agent CLI](/cli/agent), [Agent send](/tools/agent-send).

### Do I need separate VPSes for multiple agents

No. One Gateway can host multiple agents, each with its own workspace, model defaults,
and routing. That is the normal setup and it is much cheaper and simpler than running
one VPS per agent.

Use separate VPSes only when you need hard isolation (security boundaries) or very
different configs that you do not want to share. Otherwise, keep one Gateway and
use multiple agents or sub-agents.

### Is there a benefit to using a node on my personal laptop instead of SSH from a VPS

Yes - nodes are the first-class way to reach your laptop from a remote Gateway, and they
unlock more than shell access. The Gateway runs on macOS/Linux (Windows via WSL2) and is
lightweight (a small VPS or Raspberry Pi-class box is fine; 4 GB RAM is plenty), so a common
setup is an always-on host plus your laptop as a node.

- **No inbound SSH required.** Nodes connect out to the Gateway WebSocket and use device pairing.
- **Safer execution controls.** `system.run` is gated by node allowlists/approvals on that laptop.
- **More device tools.** Nodes expose `canvas`, `camera`, and `screen` in addition to `system.run`.
- **Tự động hóa trình duyệt cục bộ.** Giữ Gateway trên VPS, nhưng chạy Chrome cục bộ và chuyển tiếp điều khiển
  với tiện ích mở rộng Chrome + một node host trên laptop.

SSH rất tốt cho truy cập shell ad-hoc, nhưng các node đơn giản hơn cho các quy trình agent đang diễn ra và
tự động hóa thiết bị.

Tài liệu: [Nodes](/nodes), [Nodes CLI](/cli/nodes), [Tiện ích mở rộng Chrome](/tools/chrome-extension).

### Tôi có nên cài đặt trên laptop thứ hai hay chỉ cần thêm một node

Nếu bạn chỉ cần **công cụ cục bộ** (screen/camera/exec) trên laptop thứ hai, hãy thêm nó như một
**node**. Điều đó giữ một Gateway duy nhất và tránh cấu hình trùng lặp. Các công cụ node cục bộ
hiện chỉ hỗ trợ macOS, nhưng chúng tôi có kế hoạch mở rộng chúng sang các hệ điều hành khác.

Chỉ cài đặt Gateway thứ hai khi bạn cần **cách ly cứng** hoặc hai bot hoàn toàn riêng biệt.

Tài liệu: [Nodes](/nodes), [Nodes CLI](/cli/nodes), [Nhiều gateway](/gateway/multiple-gateways).

### Các node có chạy dịch vụ gateway không

Không. Chỉ **một gateway** nên chạy trên mỗi host trừ khi bạn cố ý chạy các hồ sơ cách ly (xem [Nhiều gateway](/gateway/multiple-gateways)). Các node là các thiết bị ngoại vi kết nối
tới gateway (các node iOS/Android, hoặc "node mode" macOS trong ứng dụng menubar). Để kiểm soát CLI node host không có giao diện và điều khiển, xem [Node host CLI](/cli/node).
Cần khởi động lại hoàn toàn để áp dụng các thay đổi `gateway`, `discovery`, và `canvasHost`.

### Có cách nào để áp dụng cấu hình thông qua API RPC không

Có. `config.apply` xác thực + ghi toàn bộ cấu hình và khởi động lại Gateway như một phần của hoạt động.

### configapply đã xóa cấu hình của tôi Làm cách nào để khôi phục và tránh điều này

`config.apply` thay thế **toàn bộ cấu hình**. Nếu bạn gửi một đối tượng một phần, mọi thứ khác sẽ bị xóa.

Khôi phục:

- Khôi phục từ bản sao lưu (git hoặc `~/.openclaw/openclaw.json` được sao chép).
- Nếu bạn không có bản sao lưu, hãy chạy lại `openclaw doctor` và cấu hình lại các kênh/mô hình.
- Nếu điều này không mong muốn, hãy báo cáo lỗi và bao gồm cấu hình cuối cùng đã biết hoặc bất kỳ bản sao lưu nào.
- Một agent mã hóa cục bộ thường có thể tái tạo cấu hình hoạt động từ nhật ký hoặc lịch sử.

Tránh điều này:

- Sử dụng `openclaw config set` cho các thay đổi nhỏ.
- Sử dụng `openclaw configure` cho các chỉnh sửa tương tác.

Tài liệu: [Config](/cli/config), [Configure](/cli/configure), [Doctor](/gateway/doctor).

### Cấu hình tối thiểu hợp lý cho lần cài đặt đầu tiên là gì

```json5
{
  agents: { defaults: { workspace: "~/.openclaw/workspace" } },
  channels: { whatsapp: { allowFrom: ["+15555550123"] } },
}
```

This sets your workspace and restricts who can trigger the bot.

### How do I set up Tailscale on a VPS and connect from my Mac

Minimal steps:

1. **Install + login on the VPS**

   ```bash
   curl -fsSL https://tailscale.com/install.sh | sh
   sudo tailscale up
   ```

2. **Install + login on your Mac**
   - Use the Tailscale app and sign in to the same tailnet.
3. **Enable MagicDNS (recommended)**
   - In the Tailscale admin console, enable MagicDNS so the VPS has a stable name.
4. **Use the tailnet hostname**
   - SSH: `ssh user@your-vps.tailnet-xxxx.ts.net`
   - Gateway WS: `ws://your-vps.tailnet-xxxx.ts.net:18789`

If you want the Control UI without SSH, use Tailscale Serve on the VPS:

```bash
openclaw gateway --tailscale serve
```
Điều này giữ cho gateway bị ràng buộc với local loopback và hiển thị HTTPS qua Tailscale. Xem [Tailscale](/gateway/tailscale).

### Làm cách nào để kết nối một node Mac với Gateway Tailscale Serve từ xa

Serve hiển thị **Gateway Control UI + WS**. Các node kết nối qua cùng một điểm cuối Gateway WS.

Thiết lập được khuyến nghị:

1. **Đảm bảo VPS + Mac nằm trên cùng một tailnet**.
2. **Sử dụng ứng dụng macOS ở chế độ Remote** (mục tiêu SSH có thể là tên máy chủ tailnet).
   Ứng dụng sẽ đường hầm cổng Gateway và kết nối dưới dạng một node.
3. **Phê duyệt node** trên gateway:

   ```bash
   openclaw nodes pending
   openclaw nodes approve <requestId>
   ```

Tài liệu: [Gateway protocol](/gateway/protocol), [Khám phá thiết bị](/gateway/discovery), [macOS remote mode](/platforms/mac/remote).
## Biến môi trường và tải .env

### OpenClaw tải biến môi trường như thế nào

OpenClaw đọc biến môi trường từ quy trình cha (shell, launchd/systemd, CI, v.v.) và thêm vào:

- `.env` từ thư mục làm việc hiện tại
- một fallback toàn cầu `.env` từ `~/.openclaw/.env` (hay `$OPENCLAW_STATE_DIR/.env`)

Không có tệp `.env` nào ghi đè các biến môi trường hiện có.

Bạn cũng có thể định nghĩa biến môi trường nội tuyến trong cấu hình (chỉ được áp dụng nếu thiếu từ biến môi trường của quy trình):

```json5
{
  env: {
    OPENROUTER_API_KEY: "sk-or-...",
    vars: { GROQ_API_KEY: "gsk-..." },
  },
}
```

See [/environment](/help/environment) for full precedence and sources.

### I started the Gateway via the service and my env vars disappeared What now

Two common fixes:

1. Put the missing keys in `~/.openclaw/.env` so they're picked up even when the service doesn't inherit your shell env.
2. Enable shell import (opt-in convenience):

```json5
{
  env: {
    shellEnv: {
      enabled: true,
      timeoutMs: 15000,
    },
  },
}
```

This runs your login shell and imports only missing expected keys (never overrides). Env var equivalents:
`OPENCLAW_LOAD_SHELL_ENV=1`, `OPENCLAW_SHELL_ENV_TIMEOUT_MS=15000`.

### I set COPILOTGITHUBTOKEN but models status shows Shell env off Why

`openclaw models status` reports whether **shell env import** is enabled. "Shell env: off"
does **not** mean your env vars are missing - it just means OpenClaw won't load
your login shell automatically.

If the Gateway runs as a service (launchd/systemd), it won't inherit your shell
environment. Fix by doing one of these:

1. Put the token in `~/.openclaw/.env`:

   ```
   COPILOT_GITHUB_TOKEN=...
   ```
2. Hoặc bật nhập shell (`env.shellEnv.enabled: true`).
3. Hoặc thêm nó vào khối cấu hình của bạn `env` (chỉ áp dụng nếu bị thiếu).

Sau đó khởi động lại Gateway và kiểm tra lại:

```bash
openclaw models status
```

Copilot tokens are read from `COPILOT_GITHUB_TOKEN` (also `GH_TOKEN` / `GITHUB_TOKEN`).
Xem [/concepts/model-providers](/concepts/model-providers) và [/environment](/help/environment).
## Phiên và nhiều cuộc trò chuyện

### Làm cách nào để bắt đầu một cuộc trò chuyện mới

Gửi `/new` hoặc `/reset` dưới dạng một tin nhắn độc lập. Xem [Quản lý phiên](/concepts/session).

### Các phiên có tự động đặt lại nếu tôi không bao giờ gửi tin nhắn mới không

Có. Các phiên hết hạn sau `session.idleMinutes` (mặc định **60**). Tin nhắn **tiếp theo**
bắt đầu một id phiên mới cho khóa trò chuyện đó. Điều này không xóa
các bản ghi - nó chỉ bắt đầu một phiên mới.

```json5
{
  session: {
    idleMinutes: 240,
  },
}
```

### Is there a way to make a team of OpenClaw instances one CEO and many agents

Yes, via **multi-agent routing** and **sub-agents**. You can create one coordinator
agent and several worker agents with their own workspaces and models.

That said, this is best seen as a **fun experiment**. It is token heavy and often
less efficient than using one bot with separate sessions. The typical model we
envision is one bot you talk to, with different sessions for parallel work. That
bot can also spawn sub-agents when needed.

Docs: [Multi-agent routing](/concepts/multi-agent), [Sub-agents](/tools/subagents), [Agents CLI](/cli/agents).

### Why did context get truncated midtask How do I prevent it

Session context is limited by the model window. Long chats, large tool outputs, or many
files can trigger compaction or truncation.

What helps:

- Ask the bot to summarize the current state and write it to a file.
- Use `/compact` before long tasks, and `/new` when switching topics.
- Keep important context in the workspace and ask the bot to read it back.
- Use sub-agents for long or parallel work so the main chat stays smaller.
- Pick a model with a larger context window if this happens often.

### How do I completely reset OpenClaw but keep it installed

Use the reset command:

```bash
openclaw reset
```

Non-interactive full reset:

```bash
openclaw reset --scope full --yes --non-interactive
```

Sau đó chạy lại thiết lập ban đầu:
```bash
openclaw onboard --install-daemon
```

Notes:

- The onboarding wizard also offers **Reset** if it sees an existing config. See [Wizard](/start/wizard).
- If you used profiles (`--profile` / `OPENCLAW_PROFILE`), reset each state dir (defaults are `~/.openclaw-<profile>`).
- Dev reset: `openclaw gateway --dev --reset` (dev-only; wipes dev config + credentials + sessions + workspace).

### Im getting context too large errors how do I reset or compact

Use one of these:

- **Compact** (keeps the conversation but summarizes older turns):

  ```
  /compact
  ```

  or `/compact <instructions>` to guide the summary.

- **Reset** (fresh session ID for the same chat key):

  ```
  /new
  /reset
  ```

If it keeps happening:

- Enable or tune **session pruning** (`agents.defaults.contextPruning`) to trim old tool output.
- Use a model with a larger context window.

Docs: [Compaction](/concepts/compaction), [Session pruning](/concepts/session-pruning), [Session management](/concepts/session).

### Why am I seeing "LLM request rejected: messages.content.tool_use.input field required"?

This is a provider validation error: the model emitted a `tool_use` block without the required
`input`. It usually means the session history is stale or corrupted (often after long threads
or a tool/schema change).

Fix: start a fresh session with `/new` (standalone message).

### Why am I getting heartbeat messages every 30 minutes

Heartbeats run every **30m** by default. Tune or disable them:

```json5
{
  agents: {
    defaults: {
      heartbeat: {
        every: "2h", // or "0m" to disable
      },
    },
  },
}
```
Nếu `HEARTBEAT.md` tồn tại nhưng thực chất trống rỗng (chỉ có dòng trống và tiêu đề markdown như `# Heading`), OpenClaw sẽ bỏ qua lần chạy heartbeat để tiết kiệm lệnh gọi API.
Nếu tệp bị thiếu, heartbeat vẫn chạy và mô hình sẽ quyết định phải làm gì.

Ghi đè cho từng agent sử dụng `agents.list[].heartbeat`. Tài liệu: [Heartbeat](/gateway/heartbeat).

### Tôi có cần thêm tài khoản bot vào nhóm WhatsApp không

Không. OpenClaw chạy trên **tài khoản của bạn**, vì vậy nếu bạn ở trong nhóm, OpenClaw có thể nhìn thấy nó.
Theo mặc định, các trả lời nhóm bị chặn cho đến khi bạn cho phép người gửi (`groupPolicy: "allowlist"`).

Nếu bạn chỉ muốn **bạn** có thể kích hoạt các trả lời nhóm:

```json5
{
  channels: {
    whatsapp: {
      groupPolicy: "allowlist",
      groupAllowFrom: ["+15551234567"],
    },
  },
}
```

### How do I get the JID of a WhatsApp group

Option 1 (fastest): tail logs and send a test message in the group:

```bash
openclaw logs --follow --json
```

Look for `chatId` (or `from`) ending in `@g.us`, like:
`1234567890-1234567890@g.us`.

Option 2 (if already configured/allowlisted): list groups from config:

```bash
openclaw directory groups list --channel whatsapp
```

Docs: [WhatsApp](/channels/whatsapp), [Directory](/cli/directory), [Logs](/cli/logs).

### Why doesn't OpenClaw reply in a group

Two common causes:

- Mention gating is on (default). You must @mention the bot (or match `mentionPatterns`).
- You configured `channels.whatsapp.groups` without `"*"` và nhóm không được phép danh sách.

Xem [Nhóm](/channels/groups) và [Tin nhắn nhóm](/channels/group-messages).

### Các nhóm/luồng có chia sẻ ngữ cảnh với tin nhắn riêng không

Các cuộc trò chuyện trực tiếp sụp đổ thành phiên chính theo mặc định. Các nhóm/kênh có khóa phiên riêng của chúng, và các chủ đề Telegram / luồng Discord là các phiên riêng biệt. Xem [Nhóm](/channels/groups) và [Tin nhắn nhóm](/channels/group-messages).

### Tôi có thể tạo bao nhiêu không gian làm việc và agent

Không có giới hạn cứng. Hàng chục (thậm chí hàng trăm) đều được, nhưng hãy chú ý:
- **Tăng trưởng đĩa:** các phiên và bản ghi lưu trữ nằm dưới `~/.openclaw/agents/<agentId>/sessions/`.
- **Chi phí token:** nhiều agent hơn có nghĩa là sử dụng mô hình đồng thời nhiều hơn.
- **Overhead vận hành:** các hồ sơ xác thực cho mỗi agent, không gian làm việc và định tuyến kênh.

Mẹo:

- Giữ một không gian làm việc **hoạt động** cho mỗi agent (`agents.defaults.workspace`).
- Xóa các phiên cũ (xóa mục JSONL hoặc lưu trữ) nếu đĩa tăng trưởng.
- Sử dụng `openclaw doctor` để phát hiện các không gian làm việc lạc lõng và không khớp hồ sơ.

### Tôi có thể chạy nhiều bot hoặc chat cùng lúc trên Slack và nên thiết lập như thế nào

Có. Sử dụng **Multi-Agent Routing** để chạy nhiều agent cô lập và định tuyến các tin nhắn đến theo kênh/tài khoản/peer. Slack được hỗ trợ như một kênh và có thể được liên kết với các agent cụ thể.

Truy cập trình duyệt rất mạnh nhưng không phải "làm bất cứ điều gì mà con người có thể làm" - chống bot, CAPTCHA và MFA vẫn có thể chặn tự động hóa. Để kiểm soát trình duyệt đáng tin cậy nhất, hãy sử dụng tiện ích mở rộng Chrome relay trên máy chạy trình duyệt (và giữ Gateway ở bất kỳ đâu).

Thiết lập theo thực tiễn tốt nhất:

- Host Gateway luôn hoạt động (VPS/Mac mini).
- Một agent cho mỗi vai trò (liên kết).
- Các kênh Slack được liên kết với các agent đó.
- Trình duyệt cục bộ thông qua tiện ích mở rộng relay (hoặc một node) khi cần.

Tài liệu: [Multi-Agent Routing](/concepts/multi-agent), [Slack](/channels/slack),
[Browser](/tools/browser), [Chrome extension](/tools/chrome-extension), [Nodes](/nodes).
## Mô hình: mặc định, lựa chọn, bí danh, chuyển đổi

### Mô hình mặc định là gì

Mô hình mặc định của OpenClaw là bất kỳ mô hình nào bạn đặt là:

```
agents.defaults.model.primary
```

Models are referenced as `provider/model` (example: `anthropic/claude-opus-4-6`). If you omit the provider, OpenClaw currently assumes `anthropic` as a temporary deprecation fallback - but you should still **explicitly** set `provider/model`.

### What model do you recommend

**Recommended default:** `anthropic/claude-opus-4-6`.
**Good alternative:** `anthropic/claude-sonnet-4-5`.
**Reliable (less character):** `openai/gpt-5.2` - nearly as good as Opus, just less personality.
**Budget:** `zai/glm-4.7`.

MiniMax M2.1 has its own docs: [MiniMax](/providers/minimax) and
[Local models](/gateway/local-models).

Rule of thumb: use the **best model you can afford** for high-stakes work, and a cheaper
model for routine chat or summaries. You can route models per agent and use sub-agents to
parallelize long tasks (each sub-agent consumes tokens). See [Models](/concepts/models) and
[Sub-agents](/tools/subagents).

Strong warning: weaker/over-quantized models are more vulnerable to prompt
injection and unsafe behavior. See [Security](/gateway/security).

More context: [Models](/concepts/models).

### Can I use selfhosted models llamacpp vLLM Ollama

Yes. If your local server exposes an OpenAI-compatible API, you can point a
custom provider at it. Ollama is supported directly and is the easiest path.

Security note: smaller or heavily quantized models are more vulnerable to prompt
injection. We strongly recommend **large models** for any bot that can use tools.
If you still want small models, enable sandboxing and strict tool allowlists.

Docs: [Ollama](/providers/ollama), [Local models](/gateway/local-models),
[Model providers](/concepts/model-providers), [Security](/gateway/security),
[Sandboxing](/gateway/sandboxing).

### How do I switch models without wiping my config

Use **model commands** or edit only the **model** fields. Avoid full config replaces.

Safe options:

- `/model` in chat (quick, per-session)
- `openclaw models set ...` (updates just model config)
- `openclaw configure --section model` (interactive)
- edit `agents.defaults.model` in `~/.openclaw/openclaw.json`

Avoid `config.apply` with a partial object unless you intend to replace the whole config.
If you did overwrite config, restore from backup or re-run `openclaw doctor` để sửa chữa.

Tài liệu: [Mô hình](/concepts/models), [Cấu hình](/cli/configure), [Cấu hình](/cli/config), [Doctor](/gateway/doctor).
### OpenClaw, Flawd và Krill sử dụng mô hình nào

- **OpenClaw + Flawd:** Anthropic Opus (`anthropic/claude-opus-4-6`) - xem [Anthropic](/providers/anthropic).
- **Krill:** MiniMax M2.1 (`minimax/MiniMax-M2.1`) - xem [MiniMax](/providers/minimax).

### Làm cách nào để chuyển đổi mô hình mà không cần khởi động lại

Sử dụng lệnh `/model` dưới dạng một tin nhắn độc lập:

```
/model sonnet
/model haiku
/model opus
/model gpt
/model gpt-mini
/model gemini
/model gemini-flash
```

You can list available models with `/model`, `/model list`, or `/model status`.

`/model` (and `/model list`) shows a compact, numbered picker. Select by number:

```
/model 3
```

You can also force a specific auth profile for the provider (per session):

```
/model opus@anthropic:default
/model opus@anthropic:work
```

Tip: `/model status` shows which agent is active, which `auth-profiles.json` file is being used, and which auth profile will be tried next.
It also shows the configured provider endpoint (`baseUrl`) and API mode (`api`) when available.

**How do I unpin a profile I set with profile**

Re-run `/model` **without** the `@profile` suffix:

```
/model anthropic/claude-opus-4-6
```

If you want to return to the default, pick it from `/model` (or send `/model <default provider/model>`).
Use `/model status` to confirm which auth profile is active.

### Can I use GPT 5.2 for daily tasks and Codex 5.3 for coding

Yes. Set one as default and switch as needed:

- **Quick switch (per session):** `/model gpt-5.2` for daily tasks, `/model gpt-5.3-codex` for coding.
- **Default + switch:** set `agents.defaults.model.primary` to `openai/gpt-5.2`, then switch to `openai-codex/gpt-5.3-codex` khi viết mã (hoặc ngược lại).
- **Sub-agents:** định tuyến các tác vụ viết mã đến các sub-agents với một mô hình mặc định khác.

Xem [Mô hình](/concepts/models) và [Lệnh gạch chéo](/tools/slash-commands).

### Tại sao tôi thấy Model is not allowed và sau đó không có phản hồi
Nếu `agents.defaults.models` được đặt, nó sẽ trở thành **danh sách cho phép** cho `/model` và bất kỳ ghi đè phiên nào. Chọn một mô hình không có trong danh sách đó sẽ trả về:

```
Model "provider/model" is not allowed. Use /model to list available models.
```

That error is returned **instead of** a normal reply. Fix: add the model to
`agents.defaults.models`, remove the allowlist, or pick a model from `/model list`.

### Why do I see Unknown model minimaxMiniMaxM21

This means the **provider isn't configured** (no MiniMax provider config or auth
profile was found), so the model can't be resolved. A fix for this detection is
in **2026.1.12** (unreleased at the time of writing).

Fix checklist:

1. Upgrade to **2026.1.12** (or run from source `main`), then restart the gateway.
2. Make sure MiniMax is configured (wizard or JSON), or that a MiniMax API key
   exists in env/auth profiles so the provider can be injected.
3. Use the exact model id (case-sensitive): `minimax/MiniMax-M2.1` or
   `minimax/MiniMax-M2.1-lightning`.
4. Run:

   ```bash
   openclaw models list
   ```

   and pick from the list (or `/model list` in chat).

See [MiniMax](/providers/minimax) and [Models](/concepts/models).

### Can I use MiniMax as my default and OpenAI for complex tasks

Yes. Use **MiniMax as the default** and switch models **per session** when needed.
Fallbacks are for **errors**, not "hard tasks," so use `/model` or a separate agent.

**Option A: switch per session**

```json5
{
  env: { MINIMAX_API_KEY: "sk-...", OPENAI_API_KEY: "sk-..." },
  agents: {
    defaults: {
      model: { primary: "minimax/MiniMax-M2.1" },
      models: {
        "minimax/MiniMax-M2.1": { alias: "minimax" },
        "openai/gpt-5.2": { alias: "gpt" },
      },
    },
  },
}
```

Then:

```
/model gpt
```
**Tùy chọn B: các agent riêng biệt**

- Agent A mặc định: MiniMax
- Agent B mặc định: OpenAI
- Định tuyến theo agent hoặc sử dụng `/agent` để chuyển đổi

Tài liệu: [Models](/concepts/models), [Multi-Agent Routing](/concepts/multi-agent), [MiniMax](/providers/minimax), [OpenAI](/providers/openai).

### Có phải opus sonnet gpt là các phím tắt tích hợp sẵn

Có. OpenClaw cung cấp một số phím tắt mặc định (chỉ được áp dụng khi mô hình tồn tại trong `agents.defaults.models`):

- `opus` → `anthropic/claude-opus-4-6`
- `sonnet` → `anthropic/claude-sonnet-4-5`
- `gpt` → `openai/gpt-5.2`
- `gpt-mini` → `openai/gpt-5-mini`
- `gemini` → `google/gemini-3-pro-preview`
- `gemini-flash` → `google/gemini-3-flash-preview`

Nếu bạn đặt bí danh của riêng mình với cùng tên, giá trị của bạn sẽ được ưu tiên.

### Làm cách nào để xác định/ghi đè các bí danh phím tắt mô hình

Các bí danh đến từ `agents.defaults.models.<modelId>.alias`. Ví dụ:

```json5
{
  agents: {
    defaults: {
      model: { primary: "anthropic/claude-opus-4-6" },
      models: {
        "anthropic/claude-opus-4-6": { alias: "opus" },
        "anthropic/claude-sonnet-4-5": { alias: "sonnet" },
        "anthropic/claude-haiku-4-5": { alias: "haiku" },
      },
    },
  },
}
```

Then `/model sonnet` (or `/<alias>` when supported) resolves to that model ID.

### How do I add models from other providers like OpenRouter or ZAI

OpenRouter (pay-per-token; many models):

```json5
{
  agents: {
    defaults: {
      model: { primary: "openrouter/anthropic/claude-sonnet-4-5" },
      models: { "openrouter/anthropic/claude-sonnet-4-5": {} },
    },
  },
  env: { OPENROUTER_API_KEY: "sk-or-..." },
}
```

Z.AI (GLM models):
```json5
{
  agents: {
    defaults: {
      model: { primary: "zai/glm-4.7" },
      models: { "zai/glm-4.7": {} },
    },
  },
  env: { ZAI_API_KEY: "..." },
}
```

If you reference a provider/model but the required provider key is missing, you'll get a runtime auth error (e.g. `Không tìm thấy khóa API cho nhà cung cấp "zai"`).

**No API key found for provider after adding a new agent**

This usually means the **new agent** has an empty auth store. Auth is per-agent and
stored in:

```
~/.openclaw/agents/<agentId>/agent/auth-profiles.json
```

Fix options:

- Run `openclaw agents add <id>` and configure auth during the wizard.
- Or copy `auth-profiles.json` from the main agent's `agentDir` into the new agent's `agentDir`.

Do **not** reuse `agentDir` trên các agent; nó gây ra xung đột xác thực/phiên.
## Mô hình dự phòng và "Tất cả các mô hình đã thất bại"

### Dự phòng hoạt động như thế nào

Dự phòng xảy ra trong hai giai đoạn:

1. **Xoay vòng hồ sơ xác thực** trong cùng một nhà cung cấp.
2. **Quay lại mô hình** sang mô hình tiếp theo trong `agents.defaults.model.fallbacks`.

Thời gian chờ áp dụng cho các hồ sơ bị lỗi (backoff theo cấp số nhân), vì vậy OpenClaw có thể tiếp tục phản hồi ngay cả khi nhà cung cấp bị giới hạn tốc độ hoặc tạm thời gặp sự cố.

### Lỗi này có nghĩa là gì

```
No credentials found for profile "anthropic:default"
```

It means the system attempted to use the auth profile ID `anthropic:default`, but could not find credentials for it in the expected auth store.

### Fix checklist for No credentials found for profile anthropicdefault

- **Confirm where auth profiles live** (new vs legacy paths)
  - Current: `~/.openclaw/agents/<agentId>/agent/auth-profiles.json`
  - Legacy: `~/.openclaw/agent/*` (migrated by `openclaw doctor`)
- **Confirm your env var is loaded by the Gateway**
  - If you set `ANTHROPIC_API_KEY` in your shell but run the Gateway via systemd/launchd, it may not inherit it. Put it in `~/.openclaw/.env` or enable `env.shellEnv`.
- **Make sure you're editing the correct agent**
  - Multi-agent setups mean there can be multiple `auth-profiles.json` files.
- **Sanity-check model/auth status**
  - Use `openclaw models status` to see configured models and whether providers are authenticated.

**Fix checklist for No credentials found for profile anthropic**

This means the run is pinned to an Anthropic auth profile, but the Gateway
can't find it in its auth store.

- **Use a setup-token**
  - Run `claude setup-token`, then paste it with `openclaw models auth setup-token --provider anthropic`.
  - If the token was created on another machine, use `openclaw models auth paste-token --provider anthropic`.
- **If you want to use an API key instead**
  - Put `ANTHROPIC_API_KEY` in `~/.openclaw/.env` on the **gateway host**.
  - Clear any pinned order that forces a missing profile:

    ```bash
    openclaw models auth order clear --provider anthropic
    ```

- **Confirm you're running commands on the gateway host**
  - In remote mode, auth profiles live on the gateway machine, not your laptop.

### Why did it also try Google Gemini and fail

If your model config includes Google Gemini as a fallback (or you switched to a Gemini shorthand), OpenClaw will try it during model fallback. If you haven't configured Google credentials, you'll see `No API key found for provider "google"`.

Fix: either provide Google auth, or remove/avoid Google models in `agents.defaults.model.fallbacks` / bí danh nên dự phòng không định tuyến đến đó.

**Thông báo yêu cầu LLM bị từ chối - cần chữ ký tư duy Google Antigravity**

Nguyên nhân: lịch sử phiên chứa **các khối tư duy không có chữ ký** (thường từ một luồng bị hủy/không hoàn chỉnh). Google Antigravity yêu cầu chữ ký cho các khối tư duy.
Sửa: OpenClaw hiện loại bỏ các khối thinking không được ký cho Google Antigravity Claude. Nếu nó vẫn xuất hiện, hãy bắt đầu **phiên mới** hoặc đặt `/thinking off` cho agent đó.

## Hồ sơ xác thực: chúng là gì và cách quản lý

Liên quan: [/concepts/oauth](/concepts/oauth) (Luồng OAuth, lưu trữ token, mẫu đa tài khoản)

### Hồ sơ xác thực là gì

Hồ sơ xác thực là bản ghi thông tin xác thực được đặt tên (OAuth hoặc khóa API) được liên kết với một nhà cung cấp. Hồ sơ nằm trong:

```
~/.openclaw/agents/<agentId>/agent/auth-profiles.json
```

### What are typical profile IDs

OpenClaw uses provider-prefixed IDs like:

- `anthropic:default` (common when no email identity exists)
- `anthropic:<email>` for OAuth identities
- custom IDs you choose (e.g. `anthropic:work`)

### Can I control which auth profile is tried first

Yes. Config supports optional metadata for profiles and an ordering per provider (`auth.order.<provider>`). This does **not** store secrets; it maps IDs to provider/mode and sets rotation order.

OpenClaw may temporarily skip a profile if it's in a short **cooldown** (rate limits/timeouts/auth failures) or a longer **disabled** state (billing/insufficient credits). To inspect this, run `openclaw models status --json` and check `auth.unusableProfiles`. Tuning: `auth.cooldowns.billingBackoffHours*`.

You can also set a **per-agent** order override (stored in that agent's `auth-profiles.json`) via the CLI:

```bash
# Defaults to the configured default agent (omit --agent)
openclaw models auth order get --provider anthropic

# Lock rotation to a single profile (only try this one)
openclaw models auth order set --provider anthropic anthropic:default

# Or set an explicit order (fallback within provider)
openclaw models auth order set --provider anthropic anthropic:work anthropic:default

# Clear override (fall back to config auth.order / round-robin)
openclaw models auth order clear --provider anthropic
```

To target a specific agent:

```bash
openclaw models auth order set --provider anthropic --agent main anthropic:default
```

### OAuth so với khóa API - sự khác biệt là gì

OpenClaw hỗ trợ cả hai:

- **OAuth** thường tận dụng quyền truy cập theo đăng ký (khi có thể).
- **Khóa API** sử dụng thanh toán theo token.

Trình hướng dẫn hỗ trợ rõ ràng thiết lập token Anthropic và OAuth OpenAI Codex và có thể lưu trữ khóa API cho bạn.
## Gateway: cổng, "đã chạy", và chế độ từ xa

### Gateway sử dụng cổng nào

`gateway.port` kiểm soát cổng được ghép kênh duy nhất cho WebSocket + HTTP (Control UI, hooks, v.v.).

Thứ tự ưu tiên:

```
--port > OPENCLAW_GATEWAY_PORT > gateway.port > default 18789
```

### Why does openclaw gateway status say Runtime running but RPC probe failed

Because "running" is the **supervisor's** view (launchd/systemd/schtasks). The RPC probe is the CLI actually connecting to the gateway WebSocket and calling `status`.

Use `openclaw gateway status` and trust these lines:

- `Probe target:` (the URL the probe actually used)
- `Listening:` (what's actually bound on the port)
- `Last gateway error:` (common root cause when the process is alive but the port isn't listening)

### Why does openclaw gateway status show Config cli and Config service different

You're editing one config file while the service is running another (often a `--profile` / `OPENCLAW_STATE_DIR` mismatch).

Fix:

```bash
openclaw gateway install --force
```

Run that from the same `--profile` / environment you want the service to use.

### What does another gateway instance is already listening mean

OpenClaw enforces a runtime lock by binding the WebSocket listener immediately on startup (default `ws://127.0.0.1:18789`). If the bind fails with `EADDRINUSE`, it throws `GatewayLockError` indicating another instance is already listening.

Fix: stop the other instance, free the port, or run with `openclaw gateway --port <port>`.

### How do I run OpenClaw in remote mode client connects to a Gateway elsewhere

Set `gateway.mode: "remote"` and point to a remote WebSocket URL, optionally with a token/password:

```json5
{
  gateway: {
    mode: "remote",
    remote: {
      url: "ws://gateway.tailnet:18789",
      token: "your-token",
      password: "your-password",
    },
  },
}
```

Notes:

- `openclaw gateway` only starts when `gateway.mode` is `local` (hoặc bạn truyền cờ ghi đè).
- Ứng dụng macOS theo dõi tệp cấu hình và chuyển đổi chế độ trực tiếp khi các giá trị này thay đổi.

### Control UI hiển thị unauthorized hoặc liên tục kết nối lại Bây giờ thì sao

Gateway của bạn đang chạy với xác thực được bật (`gateway.auth.*`), nhưng UI không gửi token/mật khẩu phù hợp.

Sự kiện (từ mã):

- Control UI lưu trữ token trong khóa localStorage của trình duyệt `openclaw.control.settings.v1`.

Cách khắc phục:

- Nhanh nhất: `openclaw dashboard` (in + sao chép URL bảng điều khiển, cố gắng mở; hiển thị gợi ý SSH nếu không có giao diện).
- Nếu bạn chưa có token: `openclaw doctor --generate-gateway-token`.
- Nếu ở xa, hãy tạo tunnel trước: `ssh -N -L 18789:127.0.0.1:18789 user@host` rồi mở `http://127.0.0.1:18789/`.
- Đặt `gateway.auth.token` (hoặc `OPENCLAW_GATEWAY_TOKEN`) trên máy chủ gateway.
- Trong cài đặt Control UI, dán cùng một token.
- Vẫn còn gặp vấn đề? Chạy `openclaw status --all` và làm theo [Khắc phục sự cố](/gateway/troubleshooting). Xem [Bảng điều khiển](/web/dashboard) để biết chi tiết xác thực.

### Tôi đặt gatewaybind tailnet nhưng nó không thể bind không có gì lắng nghe

`tailnet` bind chọn một IP Tailscale từ các giao diện mạng của bạn (100.64.0.0/10). Nếu máy không ở trên Tailscale (hoặc giao diện bị ngắt), không có gì để bind.

Cách khắc phục:

- Bắt đầu Tailscale trên máy chủ đó (để nó có địa chỉ 100.x), hoặc
- Chuyển sang `gateway.bind: "loopback"` / `"lan"`.

Lưu ý: `tailnet` là rõ ràng. `auto` ưu tiên loopback; sử dụng `gateway.bind: "tailnet"` khi bạn muốn bind chỉ tailnet.

### Tôi có thể chạy nhiều Gateway trên cùng một máy chủ không

Thường thì không - một Gateway có thể chạy nhiều kênh nhắn tin và agent. Chỉ sử dụng nhiều Gateway khi bạn cần dự phòng (ví dụ: bot cứu hộ) hoặc cách ly cứng.

Có, nhưng bạn phải cách ly:

- `OPENCLAW_CONFIG_PATH` (cấu hình cho mỗi instance)
- `OPENCLAW_STATE_DIR` (trạng thái cho mỗi instance)
- `agents.defaults.workspace` (cách ly không gian làm việc)
- `gateway.port` (cổng duy nhất)

Thiết lập nhanh (được khuyến nghị):

- Sử dụng `openclaw --profile <name> …` cho mỗi instance (tự động tạo `~/.openclaw-<name>`).
- Đặt một `gateway.port` duy nhất trong cấu hình hồ sơ của mỗi hồ sơ (hoặc chuyển `--port` cho các lần chạy thủ công).
- Cài đặt dịch vụ cho mỗi hồ sơ: `openclaw --profile <name> gateway install`.

Hồ sơ cũng thêm hậu tố tên dịch vụ (`ai.openclaw.<profile>`; cũ `com.openclaw.*`, `openclaw-gateway-<profile>.service`, `OpenClaw Gateway (<profile>)`).
Hướng dẫn đầy đủ: [Nhiều gateway](/gateway/multiple-gateways).

### Mã bắt tay không hợp lệ 1008 có nghĩa là gì

Gateway là một **máy chủ WebSocket**, và nó mong đợi thông báo đầu tiên
là một khung `connect`. Nếu nó nhận được bất cứ thứ gì khác, nó sẽ đóng kết nối
với **mã 1008** (vi phạm chính sách).

Nguyên nhân phổ biến:

- Bạn đã mở URL **HTTP** trong trình duyệt (`http://...`) thay vì máy khách WS.
- Bạn đã sử dụng cổng hoặc đường dẫn sai.
- Một proxy hoặc tunnel đã loại bỏ các tiêu đề xác thực hoặc gửi một yêu cầu không phải Gateway.

Sửa nhanh:

1. Sử dụng URL WS: `ws://<host>:18789` (hoặc `wss://...` nếu HTTPS).
2. Không mở cổng WS trong một tab trình duyệt thông thường.
3. Nếu xác thực được bật, hãy bao gồm token/mật khẩu trong khung `connect`.

Nếu bạn đang sử dụng CLI hoặc TUI, URL sẽ trông như:

```
openclaw tui --url ws://<host>:18789 --token <token>
```

Chi tiết giao thức: [Giao thức Gateway](/gateway/protocol).
## Ghi nhật ký và gỡ lỗi

### Nhật ký ở đâu

Nhật ký tệp (có cấu trúc):

```
/tmp/openclaw/openclaw-YYYY-MM-DD.log
```

You can set a stable path via `logging.file`. File log level is controlled by `logging.level`. Console verbosity is controlled by `--verbose` and `logging.consoleLevel`.

Fastest log tail:

```bash
openclaw logs --follow
```

Service/supervisor logs (when the gateway runs via launchd/systemd):

- macOS: `$OPENCLAW_STATE_DIR/logs/gateway.log` and `gateway.err.log` (default: `~/.openclaw/logs/...`; profiles use `~/.openclaw-<profile>/logs/...`)
- Linux: `journalctl --user -u openclaw-gateway[-<profile>].service -n 200 --no-pager`
- Windows: `schtasks /Query /TN "OpenClaw Gateway (<profile>)" /V /FO LIST`

See [Troubleshooting](/gateway/troubleshooting#log-locations) for more.

### How do I start/stop/restart the Gateway service

Use the gateway helpers:

```bash
openclaw gateway status
openclaw gateway restart
```

If you run the gateway manually, `openclaw gateway --force` can reclaim the port. See [Gateway](/gateway).

### I closed my terminal on Windows how do I restart OpenClaw

There are **two Windows install modes**:

**1) WSL2 (recommended):** the Gateway runs inside Linux.

Open PowerShell, enter WSL, then restart:

```powershell
wsl
openclaw gateway status
openclaw gateway restart
```

If you never installed the service, start it in the foreground:

```bash
openclaw gateway run
```

**2) Windows gốc (không được khuyến nghị):** Gateway chạy trực tiếp trong Windows.

Mở PowerShell và chạy:
```powershell
openclaw gateway status
openclaw gateway restart
```

If you run it manually (no service), use:

```powershell
openclaw gateway run
```

Docs: [Windows (WSL2)](/platforms/windows), [Gateway service runbook](/gateway).

### The Gateway is up but replies never arrive What should I check

Start with a quick health sweep:

```bash
openclaw status
openclaw models status
openclaw channels status
openclaw logs --follow
```

Common causes:

- Model auth not loaded on the **gateway host** (check `trạng thái mô hình`).
- Channel pairing/allowlist blocking replies (check channel config + logs).
- WebChat/Dashboard is open without the right token.

If you are remote, confirm the tunnel/Tailscale connection is up and that the
Gateway WebSocket is reachable.

Docs: [Channels](/channels), [Troubleshooting](/gateway/troubleshooting), [Remote access](/gateway/remote).

### Disconnected from gateway no reason what now

This usually means the UI lost the WebSocket connection. Check:

1. Is the Gateway running? `trạng thái gateway openclaw`
2. Is the Gateway healthy? `trạng thái openclaw`
3. Does the UI have the right token? `bảng điều khiển openclaw`
4. If remote, is the tunnel/Tailscale link up?

Then tail logs:

```bash
openclaw logs --follow
```

Docs: [Dashboard](/web/dashboard), [Remote access](/gateway/remote), [Troubleshooting](/gateway/troubleshooting).

### Telegram setMyCommands fails with network errors What should I check

Start with logs and channel status:

```bash
openclaw channels status
openclaw channels logs --channel telegram
```
Nếu bạn đang sử dụng VPS hoặc đằng sau proxy, hãy xác nhận rằng HTTPS đi ra ngoài được phép và DNS hoạt động.
Nếu Gateway ở xa, hãy chắc chắn rằng bạn đang xem nhật ký trên máy chủ Gateway.

Tài liệu: [Telegram](/channels/telegram), [Khắc phục sự cố kênh](/channels/troubleshooting).

### TUI không hiển thị đầu ra Tôi nên kiểm tra cái gì

Trước tiên hãy xác nhận rằng Gateway có thể truy cập được và agent có thể chạy:

```bash
openclaw status
openclaw models status
openclaw logs --follow
```

In the TUI, use `/status` to see the current state. If you expect replies in a chat
channel, make sure delivery is enabled (`/deliver on`).

Docs: [TUI](/web/tui), [Slash commands](/tools/slash-commands).

### How do I completely stop then start the Gateway

If you installed the service:

```bash
openclaw gateway stop
openclaw gateway start
```

This stops/starts the **supervised service** (launchd on macOS, systemd on Linux).
Use this when the Gateway runs in the background as a daemon.

If you're running in the foreground, stop with Ctrl-C, then:

```bash
openclaw gateway run
```

Docs: [Gateway service runbook](/gateway).

### ELI5 openclaw gateway restart vs openclaw gateway

- `openclaw gateway restart`: restarts the **background service** (launchd/systemd).
- `openclaw gateway`: runs the gateway **in the foreground** for this terminal session.

If you installed the service, use the gateway commands. Use `openclaw gateway` when
you want a one-off, foreground run.

### What's the fastest way to get more details when something fails

Start the Gateway with `--verbose` để có thêm chi tiết bảng điều khiển. Sau đó kiểm tra tệp nhật ký để tìm xác thực kênh, định tuyến mô hình và lỗi RPC.
## Phương tiện và tệp đính kèm

### Kỹ năng của tôi đã tạo một hình ảnh/PDF nhưng không có gì được gửi

Các tệp đính kèm gửi đi từ agent phải bao gồm dòng `MEDIA:<path-or-url>` (trên dòng riêng của nó). Xem [Thiết lập trợ lý OpenClaw](/start/openclaw) và [Gửi Agent](/tools/agent-send).

Gửi qua CLI:

```bash
openclaw message send --target +15555550123 --message "Here you go" --media /path/to/file.png
```

Cũng kiểm tra:

- Kênh đích hỗ trợ phương tiện gửi đi và không bị chặn bởi danh sách cho phép.
- Tệp nằm trong giới hạn kích thước của nhà cung cấp (hình ảnh được thay đổi kích thước thành tối đa 2048px).

Xem [Hình ảnh](/nodes/images).
## Bảo mật và kiểm soát truy cập

### Có an toàn khi để OpenClaw tiếp nhận tin nhắn riêng từ bên ngoài không

Coi tin nhắn riêng đến là dữ liệu không đáng tin cậy. Các cài đặt mặc định được thiết kế để giảm rủi ro:

- Hành vi mặc định trên các kênh hỗ trợ tin nhắn riêng là **ghép nối**:
  - Những người gửi không xác định nhận được mã ghép nối; bot không xử lý tin nhắn của họ.
  - Phê duyệt bằng: `openclaw pairing approve --channel <channel> [--account <id>] <code>`
  - Các yêu cầu đang chờ xử lý được giới hạn ở **3 trên mỗi kênh**; kiểm tra `openclaw pairing list --channel <channel> [--account <id>]` nếu mã không đến.
- Mở tin nhắn riêng công khai yêu cầu sự chấp thuận rõ ràng (`dmPolicy: "open"` và danh sách cho phép `"*"`).

Chạy `openclaw doctor` để hiển thị các chính sách tin nhắn riêng rủi ro cao.

### Prompt injection chỉ là mối quan tâm đối với bot công khai thôi sao

Không. Prompt injection là về **nội dung không đáng tin cậy**, không chỉ là ai có thể gửi tin nhắn riêng cho bot.
Nếu trợ lý của bạn đọc nội dung bên ngoài (tìm kiếm web/tìm nạp, trang trình duyệt, email,
tài liệu, tệp đính kèm, nhật ký dán), nội dung đó có thể chứa các hướng dẫn cố gắng
chiếm quyền kiểm soát mô hình. Điều này có thể xảy ra ngay cả khi **bạn là người gửi duy nhất**.

Rủi ro lớn nhất là khi các công cụ được bật: mô hình có thể bị lừa để
rò rỉ ngữ cảnh hoặc gọi công cụ thay mặt bạn. Giảm phạm vi ảnh hưởng bằng cách:

- sử dụng agent "đọc" chỉ đọc hoặc vô hiệu hóa công cụ để tóm tắt nội dung không đáng tin cậy
- giữ `web_search` / `web_fetch` / `browser` tắt cho các agent có công cụ
- sandboxing và danh sách cho phép công cụ nghiêm ngặt

Chi tiết: [Bảo mật](/gateway/security).

### Bot của tôi có nên có tài khoản email, GitHub hoặc số điện thoại riêng không

Có, đối với hầu hết các thiết lập. Cách ly bot bằng các tài khoản và số điện thoại riêng biệt
giảm phạm vi ảnh hưởng nếu có sự cố. Điều này cũng giúp dễ dàng xoay vòng
thông tin xác thực hoặc thu hồi quyền truy cập mà không ảnh hưởng đến tài khoản cá nhân của bạn.

Bắt đầu nhỏ. Chỉ cấp quyền truy cập vào các công cụ và tài khoản bạn thực sự cần, và mở rộng
sau nếu cần thiết.

Tài liệu: [Bảo mật](/gateway/security), [Ghép nối](/channels/pairing).

### Tôi có thể cho nó quyền tự chủ trên tin nhắn văn bản của mình và điều đó có an toàn không

Chúng tôi **không** khuyến nghị quyền tự chủ đầy đủ trên tin nhắn cá nhân của bạn. Mô hình an toàn nhất là:

- Giữ tin nhắn riêng ở **chế độ ghép nối** hoặc danh sách cho phép chặt chẽ.
- Sử dụng **số hoặc tài khoản riêng biệt** nếu bạn muốn nó gửi tin nhắn thay mặt bạn.
- Để nó soạn thảo, sau đó **phê duyệt trước khi gửi**.

Nếu bạn muốn thử nghiệm, hãy làm trên một tài khoản chuyên dụng và giữ nó cách ly. Xem
[Bảo mật](/gateway/security).

### Tôi có thể sử dụng các mô hình rẻ hơn cho các tác vụ trợ lý cá nhân không

Có, **nếu** agent chỉ là trò chuyện và đầu vào được tin cậy. Các tầng nhỏ hơn
dễ bị tấn công lệnh hướng dẫn hơn, vì vậy tránh chúng cho các agent có công cụ
hoặc khi đọc nội dung không đáng tin cậy. Nếu bạn phải sử dụng mô hình nhỏ hơn, hãy khóa
công cụ và chạy bên trong sandbox. Xem [Bảo mật](/gateway/security).

### Tôi đã chạy start trong Telegram nhưng không nhận được mã ghép nối
Mã ghép nối được gửi **chỉ** khi một người gửi tin nhắn không xác định gửi tin nhắn đến bot và
`dmPolicy: "pairing"` được bật. `/start` tự nó không tạo mã.

Kiểm tra các yêu cầu đang chờ xử lý:

```bash
openclaw pairing list telegram
```

If you want immediate access, allowlist your sender id or set `dmPolicy: "open"`
for that account.

### WhatsApp will it message my contacts How does pairing work

No. Default WhatsApp DM policy is **pairing**. Unknown senders only get a pairing code and their message is **not processed**. OpenClaw only replies to chats it receives or to explicit sends you trigger.

Approve pairing with:

```bash
openclaw pairing approve whatsapp <code>
```

List pending requests:

```bash
openclaw pairing list whatsapp
```

Wizard phone number prompt: it's used to set your **allowlist/owner** so your own DMs are permitted. It's not used for auto-sending. If you run on your personal WhatsApp number, use that number and enable `channels.whatsapp.selfChatMode`.
## Lệnh trò chuyện, hủy bỏ tác vụ và "nó sẽ không dừng lại"

### Làm cách nào để ngăn các tin nhắn hệ thống nội bộ hiển thị trong trò chuyện

Hầu hết các tin nhắn nội bộ hoặc công cụ chỉ xuất hiện khi **verbose** hoặc **reasoning** được bật
cho phiên đó.

Sửa trong trò chuyện nơi bạn thấy nó:

```
/verbose off
/reasoning off
```

If it is still noisy, check the session settings in the Control UI and set verbose
to **inherit**. Also confirm you are not using a bot profile with `verboseDefault` set
to `on` in config.

Docs: [Thinking and verbose](/tools/thinking), [Security](/gateway/security#reasoning--verbose-output-in-groups).

### How do I stopcancel a running task

Send any of these **as a standalone message** (no slash):

```
stop
stop action
stop current action
stop run
stop current run
stop agent
stop the agent
stop openclaw
openclaw stop
stop don't do anything
stop do not do anything
stop doing anything
please stop
stop please
abort
esc
wait
exit
interrupt
```

These are abort triggers (not slash commands).

For background processes (from the exec tool), you can ask the agent to run:

```
process action:kill sessionId:XXX
```

Slash commands overview: see [Slash commands](/tools/slash-commands).

Most commands must be sent as a **standalone** message that starts with `/`, but a few shortcuts (like `/status`) cũng hoạt động nội tuyến cho những người gửi được phép.

### Làm cách nào để gửi tin nhắn Discord từ Telegram Nhắn tin liên ngữ cảnh bị từ chối
OpenClaw chặn **nhắn tin giữa các nhà cung cấp** theo mặc định. Nếu một lệnh gọi công cụ được liên kết với Telegram, nó sẽ không gửi đến Discord trừ khi bạn cho phép rõ ràng.

Bật nhắn tin giữa các nhà cung cấp cho agent:

```json5
{
  agents: {
    defaults: {
      tools: {
        message: {
          crossContext: {
            allowAcrossProviders: true,
            marker: { enabled: true, prefix: "[from {channel}] " },
          },
        },
      },
    },
  },
}
```

Restart the gateway after editing config. If you only want this for a single
agent, set it under `agents.list[].tools.message` instead.

### Why does it feel like the bot ignores rapidfire messages

Queue mode controls how new messages interact with an in-flight run. Use `/queue` to change modes:

- `steer` - new messages redirect the current task
- `followup` - run messages one at a time
- `collect` - batch messages and reply once (default)
- `steer-backlog` - steer now, then process backlog
- `interrupt` - abort current run and start fresh

You can add options like `debounce:2s cap:25 drop:summarize` cho các chế độ followup.
## Trả lời chính xác câu hỏi từ ảnh chụp/nhật ký trò chuyện

**Q: "Mô hình mặc định cho Anthropic với khóa API là gì?"**

**A:** Trong OpenClaw, thông tin xác thực và lựa chọn mô hình là riêng biệt. Đặt `ANTHROPIC_API_KEY` (hoặc lưu trữ khóa API Anthropic trong hồ sơ xác thực) cho phép xác thực, nhưng mô hình mặc định thực tế là bất kỳ mô hình nào bạn cấu hình trong `agents.defaults.model.primary` (ví dụ: `anthropic/claude-sonnet-4-5` hoặc `anthropic/claude-opus-4-6`). Nếu bạn thấy `No credentials found for profile "anthropic:default"`, điều đó có nghĩa là Gateway không thể tìm thấy thông tin xác thực Anthropic trong `auth-profiles.json` dự kiến cho agent đang chạy.

---

Vẫn còn gặp vấn đề? Hỏi trong [Discord](https://discord.com/invite/clawd) hoặc mở một [cuộc thảo luận GitHub](https://github.com/openclaw/openclaw/discussions).