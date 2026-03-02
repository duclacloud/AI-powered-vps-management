# Setup OpenClaw với `gpt-oss:20b` Offline (Ollama)

Tài liệu này dành cho luồng chạy model local/offline qua Ollama.

## 1) Yêu cầu tài nguyên

- Khuyến nghị tối thiểu: RAM >= 24GB (hoặc GPU VRAM phù hợp).
- VPS 4GB thường không đủ để chạy `gpt-oss:20b` ổn định.

## 2) Cài Ollama

```bash
curl -fsSL https://ollama.com/install.sh | sh
sudo systemctl enable ollama
sudo systemctl start ollama
ollama --version
```

## 3) Tải và test model local

```bash
ollama pull gpt-oss:20b
ollama run gpt-oss:20b "What does systemctl status nginx do?"
```

Nếu chạy được, phần model local đã OK.

## 4) Cài OpenClaw và chọn model mặc định

```bash
curl -fsSL https://openclaw.ai/install.sh | bash
```

Trong onboarding, nhập model thủ công:

```text
ollama/gpt-oss:20b
```

## 5) Biến môi trường cần thiết (luồng Ollama)

```bash
echo 'export OLLAMA_API_KEY="ollama-local"' >> ~/.bashrc
echo 'export XDG_RUNTIME_DIR=/run/user/$(id -u)' >> ~/.bashrc
source ~/.bashrc
```

## 6) Khởi động gateway

```bash
openclaw gateway install
systemctl --user enable openclaw-gateway.service
systemctl --user start openclaw-gateway.service
openclaw status
openclaw doctor
```

## 7) Lỗi thường gặp

### `model not found`

```bash
ollama pull gpt-oss:20b
```

### Hết RAM/OOM

```bash
free -h
sudo fallocate -l 4G /swapfile
sudo chmod 600 /swapfile
sudo mkswap /swapfile
sudo swapon /swapfile
```

### Telegram bot không phản hồi

```bash
openclaw status
openclaw logs --follow
openclaw pairing approve telegram <PAIRING_CODE>
```

