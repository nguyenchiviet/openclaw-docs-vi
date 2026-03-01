---
summary: Sử dụng các mô hình Amazon Bedrock (Converse API) với OpenClaw
read_when:
  - Bạn muốn sử dụng các mô hình Amazon Bedrock với OpenClaw
  - >-
    Bạn cần thiết lập thông tin xác thực AWS/vùng để thực hiện các lệnh gọi mô
    hình
title: Amazon Bedrock
x-i18n:
  source_path: providers\bedrock.md
  source_hash: 73b0472f571ee6eb9c1b0d851ce94208914f2fa24eacc3b92a571aeb162dbca8
  provider: anthropic
  model: claude-sonnet-4-20250514
  generated_at: '2026-02-27T20:15:22.085Z'
---

# Amazon Bedrock

OpenClaw có thể sử dụng các mô hình **Amazon Bedrock** thông qua nhà cung cấp truyền phát **Bedrock Converse** của pi‑ai. Xác thực Bedrock sử dụng **chuỗi thông tin xác thực mặc định của AWS SDK**, không phải khóa API.
## Pi‑ai hỗ trợ những gì

- Nhà cung cấp: `amazon-bedrock`
- API: `bedrock-converse-stream`
- Xác thực: Thông tin đăng nhập AWS (biến môi trường, cấu hình được chia sẻ hoặc vai trò instance)
- Vùng: `AWS_REGION` hoặc `AWS_DEFAULT_REGION` (mặc định: `us-east-1`)
## Khám phá mô hình tự động

Nếu phát hiện thông tin xác thực AWS, OpenClaw có thể tự động khám phá các mô hình Bedrock hỗ trợ **truyền phát** và **đầu ra văn bản**. Khám phá sử dụng `bedrock:ListFoundationModels` và được lưu vào bộ nhớ đệm (mặc định: 1 giờ).

Các tùy chọn cấu hình nằm dưới `models.bedrockDiscovery`:

```json5
{
  models: {
    bedrockDiscovery: {
      enabled: true,
      region: "us-east-1",
      providerFilter: ["anthropic", "amazon"],
      refreshInterval: 3600,
      defaultContextWindow: 32000,
      defaultMaxTokens: 4096,
    },
  },
}
```

Notes:

- `enabled` defaults to `true` when AWS credentials are present.
- `region` defaults to `AWS_REGION` or `AWS_DEFAULT_REGION`, then `us-east-1`.
- `providerFilter` matches Bedrock provider names (for example `anthropic`).
- `refreshInterval` is seconds; set to `0` to disable caching.
- `defaultContextWindow` (default: `32000`) and `defaultMaxTokens` (default: `4096`)
  được sử dụng cho các mô hình được khám phá (ghi đè nếu bạn biết giới hạn mô hình của mình).
## Thiết lập ban đầu

1. Đảm bảo thông tin xác thực AWS có sẵn trên **máy chủ gateway**:

```bash
export AWS_ACCESS_KEY_ID="AKIA..."
export AWS_SECRET_ACCESS_KEY="..."
export AWS_REGION="us-east-1"
# Optional:
export AWS_SESSION_TOKEN="..."
export AWS_PROFILE="your-profile"
# Optional (Bedrock API key/bearer token):
export AWS_BEARER_TOKEN_BEDROCK="..."
```

2. Add a Bedrock provider and model to your config (no `apiKey` required):

```json5
{
  models: {
    providers: {
      "amazon-bedrock": {
        baseUrl: "https://bedrock-runtime.us-east-1.amazonaws.com",
        api: "bedrock-converse-stream",
        auth: "aws-sdk",
        models: [
          {
            id: "us.anthropic.claude-opus-4-6-v1:0",
            name: "Claude Opus 4.6 (Bedrock)",
            reasoning: true,
            input: ["text", "image"],
            cost: { input: 0, output: 0, cacheRead: 0, cacheWrite: 0 },
            contextWindow: 200000,
            maxTokens: 8192,
          },
        ],
      },
    },
  },
  agents: {
    defaults: {
      model: { primary: "amazon-bedrock/us.anthropic.claude-opus-4-6-v1:0" },
    },
  },
}
```
## Vai trò EC2 Instance

Khi chạy OpenClaw trên một EC2 instance với vai trò IAM được gắn kèm, AWS SDK
sẽ tự động sử dụng dịch vụ siêu dữ liệu instance (IMDS) để xác thực.
Tuy nhiên, phát hiện thông tin xác thực của OpenClaw hiện chỉ kiểm tra các biến môi trường,
không phải thông tin xác thực IMDS.

**Giải pháp:** Đặt `AWS_PROFILE=default` để báo hiệu rằng thông tin xác thực AWS
có sẵn. Xác thực thực tế vẫn sử dụng vai trò instance thông qua IMDS.

```bash
# Add to ~/.bashrc or your shell profile
export AWS_PROFILE=default
export AWS_REGION=us-east-1
```

**Required IAM permissions** for the EC2 instance role:

- `bedrock:InvokeModel`
- `bedrock:InvokeModelWithResponseStream`
- `bedrock:ListFoundationModels` (for automatic discovery)

Or attach the managed policy `AmazonBedrockFullAccess`.
## Bắt đầu nhanh (đường dẫn AWS)

```bash
# 1. Create IAM role and instance profile
aws iam create-role --role-name EC2-Bedrock-Access \
  --assume-role-policy-document '{
    "Version": "2012-10-17",
    "Statement": [{
      "Effect": "Allow",
      "Principal": {"Service": "ec2.amazonaws.com"},
      "Action": "sts:AssumeRole"
    }]
  }'

aws iam attach-role-policy --role-name EC2-Bedrock-Access \
  --policy-arn arn:aws:iam::aws:policy/AmazonBedrockFullAccess

aws iam create-instance-profile --instance-profile-name EC2-Bedrock-Access
aws iam add-role-to-instance-profile \
  --instance-profile-name EC2-Bedrock-Access \
  --role-name EC2-Bedrock-Access

# 2. Attach to your EC2 instance
aws ec2 associate-iam-instance-profile \
  --instance-id i-xxxxx \
  --iam-instance-profile Name=EC2-Bedrock-Access

# 3. On the EC2 instance, enable discovery
openclaw config set models.bedrockDiscovery.enabled true
openclaw config set models.bedrockDiscovery.region us-east-1

# 4. Set the workaround env vars
echo 'export AWS_PROFILE=default' >> ~/.bashrc
echo 'export AWS_REGION=us-east-1' >> ~/.bashrc
source ~/.bashrc

# 5. Verify models are discovered
openclaw models list
```
## Ghi chú

- Bedrock yêu cầu **quyền truy cập mô hình** được bật trong tài khoản/vùng AWS của bạn.
- Khám phá thiết bị tự động cần quyền `bedrock:ListFoundationModels`.
- Nếu bạn sử dụng hồ sơ, hãy đặt `AWS_PROFILE` trên máy chủ Gateway.
- OpenClaw hiển thị nguồn thông tin xác thực theo thứ tự này: `AWS_BEARER_TOKEN_BEDROCK`,
  sau đó `AWS_ACCESS_KEY_ID` + `AWS_SECRET_ACCESS_KEY`, sau đó `AWS_PROFILE`, rồi đến
  chuỗi AWS SDK mặc định.
- Hỗ trợ suy luận phụ thuộc vào mô hình; kiểm tra thẻ mô hình Bedrock để biết
  khả năng hiện tại.
- Nếu bạn thích luồng khóa được quản lý, bạn cũng có thể đặt proxy tương thích OpenAI
  phía trước Bedrock và cấu hình nó dưới dạng nhà cung cấp OpenAI thay thế.