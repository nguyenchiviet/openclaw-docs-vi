---
title: Quy trình CI
description: How the OpenClaw CI pipeline works
summary: 'Biểu đồ công việc CI, cổng phạm vi, và các lệnh tương đương cục bộ'
read_when:
  - Bạn cần hiểu tại sao một công việc CI đã chạy hoặc không chạy
  - Bạn đang gỡ lỗi các kiểm tra GitHub Actions bị thất bại
x-i18n:
  source_path: ci.md
  source_hash: 303f65e807d7b700c903b166aa70c15721ff3d54886eafd554a1d4282767867c
  provider: anthropic
  model: claude-sonnet-4-20250514
  generated_at: '2026-02-27T19:30:58.862Z'
---

# CI Pipeline

CI chạy trên mỗi lần push tới `main` và mỗi pull request. Nó sử dụng phạm vi thông minh để bỏ qua các công việc tốn kém khi chỉ có tài liệu hoặc mã native thay đổi.

## Tổng quan công việc

| Công việc         | Mục đích                                        | Khi nào chạy              |
| ----------------- | ----------------------------------------------- | ------------------------- |
| `docs-scope`      | Phát hiện thay đổi chỉ tài liệu                 | Luôn luôn                 |
| `changed-scope`   | Phát hiện khu vực nào thay đổi (node/macos/android) | PR không phải tài liệu    |
| `check`           | Kiểu TypeScript, lint, định dạng                | Thay đổi không phải tài liệu |
| `check-docs`      | Lint Markdown + kiểm tra liên kết bị hỏng       | Tài liệu thay đổi         |
| `code-analysis`   | Kiểm tra ngưỡng LOC (1000 dòng)                 | Chỉ PR                    |
| `secrets`         | Phát hiện bí mật bị rò rỉ                        | Luôn luôn                 |
| `build-artifacts` | Build dist một lần, chia sẻ với các công việc khác | Không phải tài liệu, thay đổi node |
| `release-check`   | Xác thực nội dung npm pack                      | Sau khi build             |
| `checks`          | Test Node/Bun + kiểm tra giao thức              | Không phải tài liệu, thay đổi node |
| `checks-windows`  | Test đặc thù Windows                            | Không phải tài liệu, thay đổi node |
| `macos`           | Swift lint/build/test + test TS                 | PR có thay đổi macos      |
| `android`         | Gradle build + test                             | Không phải tài liệu, thay đổi android |

## Thứ tự Fail-Fast

Các công việc được sắp xếp để kiểm tra rẻ thất bại trước khi những cái đắt chạy:

1. `docs-scope` + `code-analysis` + `check` (song song, ~1-2 phút)
2. `build-artifacts` (bị chặn bởi các công việc trên)
3. `checks`, `checks-windows`, `macos`, `android` (bị chặn bởi build)

## Runner

| Runner                           | Công việc                                  |
| -------------------------------- | ------------------------------------------ |
| `blacksmith-16vcpu-ubuntu-2404`  | Hầu hết công việc Linux, bao gồm phát hiện phạm vi |
| `blacksmith-16vcpu-windows-2025` | `checks-windows`                           |
| `macos-latest`                   | `macos`, `ios`                             |

## Tương đương cục bộ

```bash
pnpm check          # types + lint + format
pnpm test           # vitest tests
pnpm check:docs     # docs format + lint + broken links
pnpm release:check  # validate npm pack
```