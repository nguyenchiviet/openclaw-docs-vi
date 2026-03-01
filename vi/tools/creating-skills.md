---
title: Tạo Skills
summary: Xây dựng và kiểm tra các kỹ năng workspace tùy chỉnh với SKILL.md
read_when:
  - Bạn đang tạo một kỹ năng tùy chỉnh mới trong không gian làm việc của mình
  - Bạn cần một quy trình khởi động nhanh cho các kỹ năng dựa trên SKILL.md
x-i18n:
  source_path: tools\creating-skills.md
  source_hash: 96de482d2a534b9220f2cea4130ce540cb732a13731764df9eb1748787f3acb8
  provider: anthropic
  model: claude-sonnet-4-20250514
  generated_at: '2026-02-27T20:29:14.418Z'
---

# Tạo Custom Skills 🛠

OpenClaw được thiết kế để dễ dàng mở rộng. "Skills" là cách chính để thêm các khả năng mới cho trợ lý của bạn.

## Skill là gì?

Một skill là một thư mục chứa tệp `SKILL.md` (cung cấp hướng dẫn và định nghĩa công cụ cho LLM) và tùy chọn một số tập lệnh hoặc tài nguyên.

## Từng bước: Skill đầu tiên của bạn

### 1. Tạo thư mục

Skills nằm trong workspace của bạn, thường là `~/.openclaw/workspace/skills/`. Tạo một thư mục mới cho skill của bạn:

```bash
mkdir -p ~/.openclaw/workspace/skills/hello-world
```

### 2. Define the `SKILL.md`

Create a `SKILL.md` file in that directory. This file uses YAML frontmatter for metadata and Markdown for instructions.

```markdown
---
name: hello_world
description: A simple skill that says hello.
---

# Hello World Skill

When the user asks for a greeting, use the `echo` tool to say "Hello from your custom skill!".
```

### 3. Add Tools (Optional)

You can define custom tools in the frontmatter or instruct the agent to use existing system tools (like `bash` or `browser`).

### 4. Refresh OpenClaw

Ask your agent to "refresh skills" or restart the gateway. OpenClaw will discover the new directory and index the `SKILL.md`.

## Best Practices

- **Be Concise**: Instruct the model on _what_ to do, not how to be an AI.
- **Safety First**: If your skill uses `bash`, ensure the prompts don't allow arbitrary command injection from untrusted user input.
- **Test Locally**: Use `openclaw agent --message "use my new skill"` để kiểm tra.

## Shared Skills

Bạn cũng có thể duyệt và đóng góp skills cho [ClawHub](https://clawhub.com).