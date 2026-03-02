# Setup OpenClaw với `gpt-5.3` Online (OpenAI API)

Tài liệu này dành cho luồng model online qua OpenAI provider.

## 1) Chuẩn bị API key

```bash
echo 'export OPENAI_API_KEY="sk-..."' >> ~/.bashrc
source ~/.bashrc
```

Kiểm tra key còn hoạt động:

```bash
curl -s https://api.openai.com/v1/models \
  -H "Authorization: Bearer $OPENAI_API_KEY" | head
```

## 2) Kiểm tra model `gpt-5.3` có trong account

```bash
curl -s https://api.openai.com/v1/models \
  -H "Authorization: Bearer $OPENAI_API_KEY" | rg 'gpt-5.3'
```

Nếu không thấy kết quả, account của bạn có thể chưa được cấp model này.

## 3) Cài OpenClaw và chọn model mặc định

```bash
curl -fsSL https://openclaw.ai/install.sh | bash
```

Trong onboarding, nhập model thủ công:

```text
openai/gpt-5.3
```

## 4) Khởi động gateway

```bash
openclaw gateway install
systemctl --user enable openclaw-gateway.service
systemctl --user start openclaw-gateway.service
openclaw status
openclaw doctor
```

## 5) Lưu ý khác với luồng Ollama

- Không cần `ollama login`.
- Không cần `ollama pull` hoặc `ollama run`.
- Không bắt buộc `OLLAMA_API_KEY` nếu bạn chỉ dùng OpenAI provider.

## 6) Lỗi thường gặp

### `401 Unauthorized`

- API key sai/hết hạn/chưa nạp đúng shell.
- Chạy lại:

```bash
echo $OPENAI_API_KEY | head -c 10
```

### `429` hoặc `insufficient_quota`

- Tài khoản hết quota/rate limit.
- Kiểm tra billing và hạn mức API.

### `model_not_found` với `gpt-5.3`

- Model chưa mở cho account hiện tại.
- Tạm đổi model trong OpenClaw sang model OpenAI khả dụng trong danh sách `/v1/models`.

