# 🛠️ Tự Xây Dựng AI-Powered VPS Manager với OpenClaw, Ollama & Telegram

> **Hướng dẫn từng bước để thiết lập một AI agent tự động giám sát, chẩn đoán và khắc phục sự cố server, chỉ từ một tin nhắn Telegram.**

[![Ubuntu](https://img.shields.io/badge/Ubuntu-24.04_LTS-E95420?style=flat&logo=ubuntu&logoColor=white)](#prerequisites)
[![Nginx](https://img.shields.io/badge/Nginx-stable-009639?style=flat&logo=nginx&logoColor=white)](#part-2--install--configure-nginx-static-website)
[![Ollama](https://img.shields.io/badge/Ollama-latest-000000?style=flat&logo=ollama&logoColor=white)](#part-3--install-ollama)
[![Telegram](https://img.shields.io/badge/Telegram-Bot_API-26A5E4?style=flat&logo=telegram&logoColor=white)](#step-15-create-your-telegram-bot)
[![License](https://img.shields.io/badge/License-MIT-blue.svg)](#license)

---

## 📖 Mục lục

- [Tổng quan](#overview)
- [Kiến trúc](#architecture)
- [Điều kiện tiên quyết](#prerequisites)
- [Phần 0 — Triển khai trên AWS EC2 (Pop!_OS 22.04)](#aws-ec2-deployment)
- [Phần 1 — Thiết lập server ban đầu](#part-1--initial-server-setup)
- [Phần 2 — Cài đặt & cấu hình Nginx static website](#part-2--install--configure-nginx-static-website)
- [Phần 3 — Cài đặt Ollama](#part-3--install-ollama)
- [Phần 4 — Cài đặt & cấu hình OpenClaw với Telegram](#part-4--install--configure-openclaw-with-telegram)
- [Phần 5 — Cấp quyền hệ thống cho OpenClaw](#part-5--grant-openclaw-system-permissions)
- [Phần 6 — Cấu hình OpenClaw system prompt](#part-6--configure-openclaw-system-prompt)
- [Phần 7 — Lab test: phá & sửa Nginx qua Telegram](#part-7--lab-test-break--fix-nginx-via-telegram)
- [Tham khảo quản lý service](#service-management-reference)
- [Khắc phục sự cố](#troubleshooting)
- [Lưu ý bảo mật](#security-considerations)
- [Bước tiếp theo](#whats-next)
- [Đóng góp](#contributing)
- [Giấy phép](#license)

---

# **Hướng dẫn từng bước để thiết lập một tác nhân AI tự động giám sát, chẩn đoán và khắc phục sự cố máy chủ của bạn — tất cả chỉ qua một tin nhắn Telegram.**

Hãy tưởng tượng: trang web của bạn bị sập vào lúc 2 giờ sáng. Thay vì vội vàng kết nối SSH vào máy chủ, bạn chỉ cần lấy điện thoại ra, gửi một tin nhắn Telegram —*“Trang web dường như bị sập, bạn có thể sửa nó không?”*— và trong vài giây, một tác nhân AI sẽ chẩn đoán vấn đề, sửa chữa nó và thông báo cho bạn chính xác điều gì đã xảy ra.

Đó chính là điều chúng ta sẽ xây dựng trong hướng dẫn này.

Chúng ta sẽ cài đặt OpenClaw — một tác nhân AI mã nguồn mở — trên một máy chủ ảo (VPS) giá rẻ (4GB RAM, 2 vCPU), kết nối nó với một mô hình ngôn ngữ lớn (LLM) miễn phí trên đám mây thông qua Ollama, và tích hợp tất cả với một bot Telegram. Đến cuối cùng, bạn sẽ có một máy chủ tự phục hồi mà bạn có thể quản lý hoàn toàn từ điện thoại của mình.

# **Điều Chúng Ta Đang Xây Dựng**

Một phòng thí nghiệm máy chủ tự phục hồi nơi:

- Một trang web tĩnh chạy trên Nginx
- OpenClaw (một tác nhân AI mã nguồn mở) giám sát và quản lý máy chủ
- Ollama kết nối OpenClaw với nhiều lựa chọn LLM để test: `minimax-m2.5:cloud` (nhẹ VPS), `gpt-oss:20b` (offline), hoặc mô hình online khác
- Bạn tương tác với tác nhân qua Telegram — từ điện thoại của bạn, ở bất kỳ đâu trên thế giới
- Khi có sự cố, bạn chỉ cần nhắn tin cho bot — nó sẽ chẩn đoán và khắc phục sự cố một cách tự động

<a id="overview"></a>
## 🔍 Tổng quan

### Chúng ta đang xây gì?

Một self-healing server lab nơi:

- Một **static website** chạy trên Nginx
- **OpenClaw** (open-source AI agent) giám sát và quản lý server
- **Ollama** kết nối OpenClaw với **MiniMax M2.5** cloud LLM (không cần GPU hoặc RAM lớn)
- Bạn tương tác với agent qua **Telegram** từ điện thoại
- Khi có sự cố, bạn chỉ cần nhắn bot để agent tự chẩn đoán và sửa lỗi

### OpenClaw là gì?

[OpenClaw](https://github.com/pjasicek/OpenClaw) là một framework open-source, local-first cho autonomous AI agent do Peter Steinberger tạo ra. Nó chạy trên hạ tầng của bạn và kết nối với các nền tảng nhắn tin (Telegram, WhatsApp, Discord, Slack, ...), giúp bạn quản trị server bằng ngôn ngữ tự nhiên. OpenClaw có thể chạy terminal commands, quản lý file, tự động hóa browser và thực hiện các tác vụ system administration.

### Ollama là gì?

[Ollama](https://ollama.com/) là một framework nhẹ để chạy large language models cục bộ hoặc kết nối cloud-hosted models. Nó cung cấp CLI và API đơn giản để tải, quản lý và phục vụ LLM. Với lệnh `ollama launch`, bạn có thể cài và cấu hình công cụ như OpenClaw khá nhanh.

### Vì sao chọn MiniMax M2.5 Cloud?

Với VPS chỉ 4GB RAM, chạy local LLM lớn sẽ tốn hầu hết tài nguyên. Model [`minimax-m2.5:cloud`](https://ollama.com/library/minimax-m2.5) chạy trên server của MiniMax:

| Lợi ích | Chi tiết |
|:--------|:---------|
| **RAM gần như bằng 0 trên VPS** | LLM chạy từ xa, không chạy cục bộ |
| **Function calling** | Cần cho OpenClaw thực thi command |
| **64k+ context** | Context window lớn cho tác vụ phức tạp |
| **Free access** | Quyền truy cập khuyến mãi (có thể thay đổi) |

---

<a id="architecture"></a>
## 🏗️ Kiến trúc

```
┌─────────────────────────────────────────────────────────┐
│                      Your Phone                         │
│                Telegram: "Site is down!"                 │
└────────────────────────┬────────────────────────────────┘
                         │
                         ▼
┌─────────────────────────────────────────────────────────┐
│                 Ubuntu 24.04 VPS                        │
│              4GB RAM | 2 vCPU | 50GB Disk               │
│                                                         │
│   ┌──────────────────────────────────────────────┐      │
│   │              OpenClaw Agent                   │      │
│   │         (receives Telegram messages)          │      │
│   │                    │                          │      │
│   │          ┌─────────┴──────────┐               │      │
│   │          ▼                    ▼               │      │
│   │   Ollama Client        Shell Executor         │      │
│   │       │                    │                  │      │
│   │       ▼                    ▼                  │      │
│   │  MiniMax M2.5       System Commands           │      │
│   │  (cloud LLM)     (systemctl, journalctl,     │      │
│   │                    nginx -t, curl, etc.)      │      │
│   └──────────────────────────────────────────────┘      │
│                                                         │
│   ┌──────────────┐                                      │
│   │    Nginx     │ ◄── Static Website on port 80        │
│   │   (:80)      │                                      │
│   └──────────────┘                                      │
│                                                         │
└─────────────────────────────────────────────────────────┘
```

<details>
<summary><strong>Bấm để mở rộng: Luồng hoạt động</strong></summary>

1. Bạn gửi tin nhắn trong Telegram
2. OpenClaw nhận tin nhắn qua Telegram Bot API
3. OpenClaw chuyển yêu cầu đến MiniMax M2.5 (qua Ollama)
4. LLM quyết định command cần chạy
5. OpenClaw thực thi command trên VPS
6. Kết quả được gửi lại cho LLM để phân tích
7. OpenClaw trả lời lại bạn trên Telegram bằng nội dung dễ hiểu

</details>

---

<a id="prerequisites"></a>
## ✅ Điều kiện tiên quyết

| Hạng mục | Chi tiết |
|:---------|:---------|
| **VPS** | Ubuntu 24.04 LTS, 4GB RAM, 2 vCPU, 50GB+ disk |
| **Access** | Quyền root hoặc sudo SSH vào VPS |
| **Máy local** | Pop!_OS 22.04 (hoặc Linux/macOS/WSL có SSH client) |
| **Telegram** | Tài khoản Telegram trên điện thoại |
| **Internet** | Kết nối ổn định (cần cho cloud LLM và Telegram) |

> **Thời gian setup dự kiến:** 20–30 phút

---

<a id="aws-ec2-deployment"></a>
## ☁️ Phần 0 — Triển khai trên AWS EC2 (Pop!_OS 22.04)

Phần này dành riêng cho bạn nếu chạy trên AWS EC2 và thao tác từ máy local Pop!_OS 22.04.

### 0.1 Tạo EC2 instance

Trong AWS Console, tạo instance với cấu hình đề xuất:

- **AMI**: Ubuntu Server 24.04 LTS
- **Instance type**: `t3.medium` (2 vCPU, 4GB RAM)
- **Storage**: 50GB `gp3` (hoặc lớn hơn)
- **Key pair**: tạo file `.pem` mới (ví dụ `openclaw-ec2.pem`)

Security Group (inbound) tối thiểu:

- TCP `22` từ **My IP** (không mở toàn internet)
- TCP `80` từ `0.0.0.0/0` (và `::/0` nếu dùng IPv6)
- TCP `443` (tùy chọn, nếu dùng HTTPS)

Khuyến nghị thêm:

- Gán **Elastic IP** để không đổi IP public sau mỗi lần restart instance
- Giữ outbound mặc định `Allow all` để instance gọi Ollama cloud/OpenAI/Telegram API

### 0.2 Chuẩn bị máy local Pop!_OS 22.04

```bash
sudo apt update
sudo apt install -y openssh-client curl jq
mkdir -p ~/.ssh
chmod 700 ~/.ssh
```

Di chuyển file key `.pem` về thư mục SSH và set permission:

```bash
mv ~/Downloads/openclaw-ec2.pem ~/.ssh/openclaw-ec2.pem
chmod 400 ~/.ssh/openclaw-ec2.pem
```

### 0.3 SSH vào EC2 (Ubuntu user mặc định)

```bash
ssh -i ~/.ssh/openclaw-ec2.pem ubuntu@YOUR_EC2_PUBLIC_IP
```

> [!IMPORTANT]
> Trên EC2 Ubuntu, user mặc định là `ubuntu`, thường không đăng nhập trực tiếp bằng `root`.

Nếu cần shell root:

```bash
sudo -i
```

### 0.4 Đồng bộ Security Group và UFW

Bạn đang có 2 lớp firewall:

- AWS Security Group
- UFW trong Ubuntu

Hãy đảm bảo cả hai cùng mở đúng cổng (`22`, `80`, và `443` nếu dùng HTTPS), nếu không sẽ gặp lỗi “mở bên ngoài nhưng vẫn không truy cập được”.

---

<a id="part-1--initial-server-setup"></a>
## Phần 1 — Thiết lập server ban đầu

### Bước 1: Kết nối vào VPS

```bash
ssh root@YOUR_VPS_IP
```

Nếu dùng AWS EC2 Ubuntu:

```bash
ssh -i ~/.ssh/openclaw-ec2.pem ubuntu@YOUR_EC2_PUBLIC_IP
```

### Bước 2: Cập nhật hệ thống

```bash
sudo apt update && sudo apt dist-upgrade -y
```

### Bước 3: Cài các package cần thiết

```bash
sudo apt install -y curl git wget build-essential ufw fail2ban
```

### Bước 4: Tạo user chuyên dụng

Chạy service bằng root là rủi ro bảo mật. Hãy tạo user riêng cho OpenClaw:

```bash
sudo adduser openclaw
sudo usermod -aG sudo openclaw
```

Khi được hỏi, hãy đặt mật khẩu mạnh và điền/bỏ qua các trường tùy chọn.

### Bước 5: Cấu hình firewall

```bash
sudo ufw default deny incoming
sudo ufw default allow outgoing
sudo ufw allow 22/tcp       # SSH access
sudo ufw allow 80/tcp       # Nginx HTTP
sudo ufw limit 22/tcp       # Rate-limit SSH to prevent brute force
sudo ufw enable
```

Khi được hỏi xác nhận, nhập `y`.

<details>
<summary><strong>Bấm để kiểm tra: Kết quả firewall mong đợi</strong></summary>

```bash
sudo ufw status
```

```
Status: active

To                         Action      From
--                         ------      ----
22/tcp                     LIMIT       Anywhere
80/tcp                     ALLOW       Anywhere
```

</details>

### Bước 6: Chuyển sang user OpenClaw

```bash
su - openclaw
```

> [!NOTE]
> Tất cả bước còn lại nên chạy bằng user `openclaw`.

---

<a id="part-2--install--configure-nginx-static-website"></a>
## Phần 2 — Cài đặt & cấu hình Nginx static website

### Bước 7: Cài Nginx

```bash
sudo apt install -y nginx
```

### Bước 8: Cho Nginx tự chạy khi boot

```bash
sudo systemctl enable nginx
sudo systemctl start nginx
```

### Bước 9: Tạo static website

Thay trang mặc định của Nginx bằng trang HTML tùy chỉnh:

<details>
<summary><strong>Bấm để mở rộng: source code index.html</strong></summary>

```bash
sudo tee /var/www/html/index.html > /dev/null << 'EOF2'
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>OpenClaw Lab</title>
    <style>
        * { margin: 0; padding: 0; box-sizing: border-box; }
        body {
            font-family: 'Segoe UI', Arial, sans-serif;
            background: linear-gradient(135deg, #0f0c29, #302b63, #24243e);
            color: #ffffff;
            min-height: 100vh;
            display: flex;
            justify-content: center;
            align-items: center;
        }
        .container {
            text-align: center;
            padding: 40px;
            background: rgba(255, 255, 255, 0.05);
            border-radius: 16px;
            border: 1px solid rgba(255, 255, 255, 0.1);
            max-width: 600px;
        }
        h1 { font-size: 2.5em; margin-bottom: 10px; color: #e94560; }
        .status {
            display: inline-block;
            padding: 8px 20px;
            background: #00c853;
            border-radius: 20px;
            font-weight: bold;
            margin: 20px 0;
        }
        p { color: #ccc; line-height: 1.8; }
        .footer { margin-top: 30px; font-size: 0.85em; color: #666; }
    </style>
</head>
<body>
    <div class="container">
        <h1>OpenClaw Lab Server</h1>
        <div class="status">Nginx is Running</div>
        <p>This static website is served by Nginx on Ubuntu 24.04.</p>
        <p>It is monitored and managed by an AI agent (OpenClaw) via Telegram.</p>
        <div class="footer">Powered by OpenClaw + Ollama + MiniMax M2.5</div>
    </div>
</body>
</html>
EOF2
```

</details>

### Bước 10: Kiểm tra website

```bash
curl -s -o /dev/null -w "HTTP Status: %{http_code}\n" http://localhost
```

Kết quả mong đợi:

```
HTTP Status: 200
```

Sau đó mở trình duyệt truy cập `http://YOUR_VPS_IP`. Bạn sẽ thấy trang "OpenClaw Lab Server" với badge xanh "Nginx is Running".

---

<a id="part-3--install-ollama"></a>
## Phần 3 — Cài đặt Ollama

### Bước 11: Cài Ollama

```bash
curl -fsSL https://ollama.com/install.sh | sh
```

### Bước 12: Kiểm tra cài đặt Ollama

```bash
ollama --version
```

### Bước 13: Đảm bảo Ollama service đang chạy

Ollama tự cài dưới dạng systemd service:

```bash
sudo systemctl enable ollama
sudo systemctl status ollama
```

Bạn nên thấy trạng thái `active (running)`.

### Bước 14: Đăng nhập Ollama (bắt buộc cho cloud model)

Cloud model như `minimax-m2.5:cloud` yêu cầu xác thực. Bạn cần tài khoản Ollama.

Nếu chưa có, đăng ký tại [ollama.com](https://ollama.com) (miễn phí).

```bash
ollama login
```

Làm theo hướng dẫn để xác thực. Thông tin đăng nhập sẽ được lưu local để Ollama truy cập cloud model thay bạn.

> [!NOTE]
> Nếu bạn chỉ dùng local model (ví dụ `llama3.2:3b`), có thể bỏ qua bước này.

### Bước 15: Test cloud model

```bash
ollama run minimax-m2.5:cloud "What does the command 'systemctl status nginx' do?"
```

Nếu phản hồi bình thường, nghĩa là:

- [x] Ollama hoạt động
- [x] Ollama login đã xác thực
- [x] MiniMax M2.5 phản hồi ổn

> [!NOTE]
> Model `minimax-m2.5:cloud` chạy trên server MiniMax. VPS của bạn chỉ gửi prompt qua internet và nhận phản hồi. Không có file model lớn được tải xuống VPS.

> [!TIP]
> Nếu gặp `Error: 401 Unauthorized`, chạy lại `ollama login`. Phương án dự phòng:
> ```bash
> ollama pull llama3.2:3b
> ollama run llama3.2:3b "What does the command 'systemctl status nginx' do?"
> ```
> Model này dùng khoảng 2.5GB RAM, không cần login hoặc internet khi inference.

---

<a id="part-4--install--configure-openclaw-with-telegram"></a>
## Phần 4 — Cài đặt & cấu hình OpenClaw với Telegram

<a id="step-15-create-your-telegram-bot"></a>
### Bước 16: Tạo Telegram bot

Trước khi cài OpenClaw, bạn cần tạo Telegram bot để tương tác:

1. Mở **Telegram** trên điện thoại
2. Tìm **@BotFather** và bắt đầu chat
3. Gửi `/newbot`
4. Nhập display name cho bot (ví dụ: `My VPS Manager`)
5. Nhập username cho bot (phải kết thúc bằng `bot`, ví dụ: `myvps_manager_bot`)
6. **BotFather trả về bot token** dạng:
   ```
   7123456789:AAHn8-Kx9BdQ3mZoP_example_token
   ```
7. **Lưu token này** để dùng ở bước sau

> [!CAUTION]
> Giữ kín bot token. Ai có token đều có thể điều khiển bot. Không commit token vào public repository.

### Bước 17: Cài OpenClaw qua installer chính thức

```bash
curl -fsSL https://openclaw.ai/install.sh | bash
```

Luồng onboarding thực tế:

| Prompt | Chọn |
|:-------|:-----|
| Onboarding mode | **QuickStart** |
| Model/Auth Provider | **Skip for now** (không có Ollama trong danh sách) |
| Filter models by provider | **All providers** |
| Default model | **Enter model manually** → `ollama/minimax-m2.5:cloud` |
| Select channel | **Telegram (Bot API)** |
| Bot Token | Dán token từ Bước 16 |
| Skills to install | **Skip for now** |

> [!IMPORTANT]
> Onboarding wizard không hiển thị Ollama trong provider list. Ở bước provider, chọn "Skip for now", sau đó "All providers", rồi nhập model thủ công theo format `provider/model`: `ollama/minimax-m2.5:cloud`.

Sau onboarding, reload shell:

```bash
source ~/.bashrc
```

### Bước 18: Cấu hình environment variables bắt buộc

Hai biến môi trường cần cho OpenClaw trên headless VPS:

```bash
echo 'export OLLAMA_API_KEY="ollama-local"' >> ~/.bashrc
echo 'export XDG_RUNTIME_DIR=/run/user/$(id -u)' >> ~/.bashrc
source ~/.bashrc
```

| Biến | Mục đích |
|:-----|:---------|
| `OLLAMA_API_KEY` | Để OpenClaw nhận Ollama là model provider (bất kỳ giá trị không rỗng đều được) |
| `XDG_RUNTIME_DIR` | Cần cho `systemctl --user` trên headless VPS |

### Bước 19: Khởi động OpenClaw gateway

```bash
openclaw gateway install
systemctl --user start openclaw-gateway.service
systemctl --user enable openclaw-gateway.service
```

Kiểm tra trạng thái:

```bash
openclaw status
```

> [!NOTE]
> Nếu `systemctl --user` báo "Failed to connect to bus: No medium found", hãy kiểm tra `XDG_RUNTIME_DIR` (Bước 18) và bật linger: `sudo loginctl enable-linger openclaw`.

### Bước 20: Kiểm tra health của OpenClaw

```bash
openclaw doctor
```

Lệnh này kiểm tra Node.js version, Ollama connection, model availability và gateway config.

### Bước 21: Duyệt Telegram pairing

OpenClaw chỉ phản hồi sau khi bạn duyệt pairing cho Telegram account.

1. Trên Telegram, gửi `/start` cho bot
2. Bot trả user ID và **pairing code**:
   ```
   Your Telegram user id: 1205954433
   Pairing code: LS37EW67
   Ask the bot owner to approve with: openclaw pairing approve telegram LS37EW67
   ```
3. Trên VPS, chạy:
   ```bash
   openclaw pairing approve telegram LS37EW67
   ```
4. Gửi lại tin nhắn, bot sẽ phản hồi

### Bước 22: Truy cập dashboard (tùy chọn)

Tạo SSH tunnel từ máy local:

```bash
ssh -N -L 18789:127.0.0.1:18789 openclaw@YOUR_VPS_IP
```

Mở URL:

```
http://localhost:18789/#token=YOUR_GATEWAY_TOKEN
```

Lấy token bằng:

```bash
cat ~/.openclaw/openclaw.json | grep '"token"'
```

> [!NOTE]
> Dashboard nên truy cập qua `localhost` (không dùng trực tiếp VPS IP) để đảm bảo secure context trên browser.

---

<a id="part-5--grant-openclaw-system-permissions"></a>
## Phần 5 — Cấp quyền hệ thống cho OpenClaw

Mặc định OpenClaw chạy dưới user `openclaw` và không đủ quyền quản lý system service. Ta sẽ cấp quyền sudo giới hạn cho Nginx.

### Bước 23: Tạo file sudoers cho OpenClaw

**Cho môi trường lab/test**, có thể cấp full passwordless sudo:

```bash
sudo visudo -f /etc/sudoers.d/openclaw
```

Thêm:

```sudoers
openclaw ALL=(ALL) NOPASSWD: ALL
```

> [!CAUTION]
> `NOPASSWD: ALL` tương đương quyền root đầy đủ. Chỉ dùng cho lab private. Production nên dùng bản giới hạn bên dưới.

**Phương án production** (chỉ command liên quan Nginx):

```bash
sudo visudo -f /etc/sudoers.d/openclaw-nginx
```

```sudoers
openclaw ALL=(ALL) NOPASSWD: /usr/bin/systemctl start nginx
openclaw ALL=(ALL) NOPASSWD: /usr/bin/systemctl stop nginx
openclaw ALL=(ALL) NOPASSWD: /usr/bin/systemctl restart nginx
openclaw ALL=(ALL) NOPASSWD: /usr/bin/systemctl reload nginx
openclaw ALL=(ALL) NOPASSWD: /usr/bin/systemctl status nginx
openclaw ALL=(ALL) NOPASSWD: /usr/sbin/nginx -t
openclaw ALL=(ALL) NOPASSWD: /usr/bin/journalctl -u nginx*
```

### Bước 24: Kiểm tra quyền

```bash
sudo -l -U openclaw
```

Bạn nên thấy danh sách command được phép mà không cần password prompt. Test nhanh qua Telegram: gửi **"can you stop nginx?"**.

---

<a id="part-6--configure-openclaw-system-prompt"></a>
## Phần 6 — Cấu hình OpenClaw system prompt

`system prompt` định nghĩa cách OpenClaw hành xử: được làm gì, dùng command nào và trả lời theo cách nào.

### Bước 25: Thiết lập system prompt

```bash
openclaw configure
```

Khi được hỏi về custom instructions, nhập:

<details>
<summary><strong>Bấm để mở rộng: Full system prompt</strong></summary>

```
Bạn là bot quản trị VPS chạy trên Ubuntu 24.04 LTS.
Server này có 4GB RAM, 2 vCPU và chạy Nginx để phục vụ static website trên port 80.

Khi người dùng báo lỗi hoặc hỏi về server:

1. CHẨN ĐOÁN trước:
   - Kiểm tra trạng thái service: sudo systemctl status nginx
   - Kiểm tra cú pháp config: sudo nginx -t
   - Đọc log gần nhất: sudo journalctl -u nginx --no-pager -n 50
   - Kiểm tra HTTP response: curl -s -o /dev/null -w "%{http_code}" http://localhost
   - Kiểm tra tài nguyên: df -h, free -h

2. GIẢI THÍCH phát hiện bằng ngôn ngữ đơn giản.

3. KHẮC PHỤC sự cố:
   - Nếu service bị dừng: sudo systemctl start nginx
   - Nếu config lỗi: xác định lỗi, sửa, rồi reload/restart
   - Nếu hết dung lượng: xác định file lớn chiếm disk

4. XÁC NHẬN sau khi sửa:
   - Kiểm tra lại: curl -s -o /dev/null -w "%{http_code}" http://localhost
   - Xác nhận: sudo systemctl status nginx

5. BÁO CÁO lại những gì đã làm, nguyên nhân gốc (root cause), và cách khắc phục.

Luôn giải thích theo cách người mới có thể hiểu.
Không chạy command phá hủy (rm -rf, dd, mkfs) nếu chưa có xác nhận rõ ràng.
```

</details>

---

<a id="part-7--lab-test-break--fix-nginx-via-telegram"></a>
## Phần 7 — Lab test: phá & sửa Nginx qua Telegram

Giờ bắt đầu phần test thực tế.

---

### Test 1: Dừng Nginx (cơ bản)

**Phá lỗi** (qua SSH):

```bash
sudo systemctl stop nginx
```

**Xác nhận lỗi:**

```bash
curl http://localhost
# Output: curl: (7) Failed to connect to localhost port 80: Connection refused
```

**Sửa qua Telegram — gửi bot:**

```
The website seems to be down. Can you check and fix it?
```

<details>
<summary><strong>Bấm để xem: Hành vi OpenClaw mong đợi</strong></summary>

```
1. Checks: curl http://localhost           → Connection refused
2. Checks: sudo systemctl status nginx     → inactive (dead)
3. Runs:   sudo systemctl start nginx      → Success
4. Checks: curl http://localhost           → 200 OK
5. Replies: "Nginx was stopped. I've restarted it and the
            website is back online (HTTP 200)."
```

</details>

---

### Test 2: Làm hỏng cấu hình Nginx (trung cấp)

**Phá lỗi** (qua SSH):

```bash
sudo bash -c 'echo "THIS_IS_BROKEN;" >> /etc/nginx/nginx.conf'
sudo systemctl restart nginx
# Lệnh này sẽ fail vì config không hợp lệ
```

**Xác nhận lỗi:**

```bash
sudo nginx -t
# Output: nginx: [emerg] unknown directive "THIS_IS_BROKEN" in /etc/nginx/nginx.conf:XX
```

**Sửa qua Telegram — gửi bot:**

```
The website is not loading. Something might be wrong with the server. Please investigate and fix it.
```

<details>
<summary><strong>Bấm để xem: Hành vi OpenClaw mong đợi</strong></summary>

```
1.  Checks: curl http://localhost           → Connection refused
2.  Checks: sudo systemctl status nginx     → failed
3.  Checks: sudo journalctl -u nginx -n 50  → config error
4.  Checks: sudo nginx -t                   → syntax error on line XX
5.  Reads:  /etc/nginx/nginx.conf           → finds "THIS_IS_BROKEN;"
6.  Removes the bad line
7.  Checks: sudo nginx -t                   → syntax is ok
8.  Runs:   sudo systemctl restart nginx    → Success
9.  Checks: curl http://localhost           → 200 OK
10. Replies: "Nginx failed because of an invalid directive
             'THIS_IS_BROKEN' in nginx.conf on line XX. I removed
             the bad line, verified the config, and restarted
             Nginx. Your website is back up."
```

Agent không chỉ restart service, mà còn đọc logs, tìm root cause, sửa config và verify lại.

</details>

---

### Test 3: Yêu cầu báo cáo health tổng quát

**Gửi cho bot qua Telegram:**

```
Give me a full health report of the server.
```

<details>
<summary><strong>Bấm để xem: Ví dụ phản hồi</strong></summary>

```
Server Health Report:
- CPU: 2 vCPU, load average 0.12
- Memory: 1.2GB used / 4GB total (30%)
- Disk: 8GB used / 50GB total (16%)
- Nginx: active (running), uptime 2 hours
- Website: HTTP 200 (responding normally)
- Uptime: 3 days, 7 hours

Everything looks healthy!
```

</details>

---

### Test 4: Diagnose trước, sửa sau khi bạn duyệt

Đây là workflow thực tế khi xảy ra incident.

**Phá lỗi** (chèn rác vào config Nginx):

```bash
sudo sed -i '10i 6556356d' /etc/nginx/nginx.conf
sudo systemctl restart nginx
```

**Gửi cho bot Telegram:**

```
It looks like my website might be down. Please check the current
status of website. If there is an issue, identify the root cause
and show me the relevant logs. Do NOT make any changes yet — just
diagnose and report back to me first.
```

<img width="625" height="644" alt="cli-12" src="https://github.com/user-attachments/assets/14c673cf-95bb-4c95-954a-f576f5b5c4a5" />

<details>
<summary><strong>Bấm để xem: Full conversation flow</strong></summary>

**OpenClaw phản hồi chẩn đoán đầy đủ:**

```
Status: Website is down — nginx failed to start.

Root Cause: Config error. Line 10 has garbage text "6556356d" inserted.

Logs:
  nginx: [emerg] unknown directive "6556356d" in /etc/nginx/nginx.conf:11
  nginx: configuration file /etc/nginx/nginx.conf test failed

Want me to fix it?
```

**Trả lời để duyệt sửa:**

```
yes please fix it
```

**OpenClaw sửa và xác nhận:**

```
Fixed and online — HTTP 200. ✅
```

**Yêu cầu access logs:**

```
Can you please share me the relevant logs who has accessed the website during this time?
```

**OpenClaw trả về bảng access log:**

```
Time (UTC)  | IP              | Request | Status | User-Agent
------------|-----------------|---------|--------|------------------
13:14:34    | 34.78.60.61     | GET /   | 200    | python-requests
13:14:52    | 89.134.25.185   | GET /   | 200    | Chrome (Windows)
15:35:27    | 176.65.148.203  | GET /   | 200    | Mozilla/1.0
```

</details>

> [!NOTE]
> Đây là sức mạnh của setup này: **diagnose, approve, fix, kiểm tra logs** toàn bộ qua Telegram, không cần mở SSH terminal.

---

### Dọn dẹp sau khi test

Sau khi test xong, đảm bảo config Nginx sạch:

```bash
sudo nginx -t
```

Nếu có lỗi, khôi phục config mặc định:

```bash
sudo cp /etc/nginx/nginx.conf.bak /etc/nginx/nginx.conf 2>/dev/null \
  || sudo apt install --reinstall -y nginx
sudo systemctl restart nginx
```

---

<a id="service-management-reference"></a>
## 📋 Tham khảo quản lý service

### Lệnh nhanh

| Tác vụ | Lệnh |
|:-------|:-----|
| Nginx status | `sudo systemctl status nginx` |
| Nginx start | `sudo systemctl start nginx` |
| Nginx restart | `sudo systemctl restart nginx` |
| Nginx config test | `sudo nginx -t` |
| Nginx logs | `sudo journalctl -u nginx --no-pager -n 50` |
| Ollama status | `sudo systemctl status ollama` |
| Ollama restart | `sudo systemctl restart ollama` |
| OpenClaw status | `openclaw status` |
| OpenClaw restart | `systemctl --user restart openclaw-gateway.service` |
| Test website | `curl -s -o /dev/null -w "%{http_code}" http://localhost` |

### Port sử dụng

| Service | Port | Truy cập |
|:--------|:-----|:---------|
| Nginx | `80` | Public (qua UFW) |
| Ollama | `11434` | Chỉ localhost |
| OpenClaw Gateway | `18789` | Chỉ localhost |
| SSH | `22` | Public (đã rate-limit qua UFW) |

---

<a id="troubleshooting"></a>
## 🔧 Khắc phục sự cố

<details>
<summary><strong>OpenClaw Bot không phản hồi trên Telegram</strong></summary>

Kiểm tra đầu tiên: bạn đã hoàn tất **Telegram pairing** (Bước 21) chưa? Bot sẽ không phản hồi trước khi bạn chạy `openclaw pairing approve telegram <CODE>`.

```bash
# Check gateway and channel status
openclaw status

# View live logs
openclaw logs --follow

# Restart gateway nếu cần
systemctl --user restart openclaw-gateway.service
```

</details>

<details>
<summary><strong>Không SSH được EC2 (timeout / Permission denied)</strong></summary>

Checklist nhanh:

1. EC2 đang `running` và có `Public IPv4`.
2. Security Group inbound mở TCP `22` từ đúng IP hiện tại của bạn (`My IP`).
3. Dùng đúng key pair gắn với instance và đúng quyền file key:

```bash
chmod 400 ~/.ssh/openclaw-ec2.pem
ssh -i ~/.ssh/openclaw-ec2.pem ubuntu@YOUR_EC2_PUBLIC_IP
```

4. Dùng đúng username mặc định của Ubuntu AMI: `ubuntu` (không phải `root`).
5. Nếu `timeout`, kiểm tra thêm Route Table có route `0.0.0.0/0` ra Internet Gateway.

</details>

<details>
<summary><strong>"Failed to connect to bus: No medium found" (systemctl --user)</strong></summary>

Đây là lỗi phổ biến trên headless VPS.

```bash
# Set runtime dir cho session hiện tại
export XDG_RUNTIME_DIR=/run/user/$(id -u)

# Lưu vĩnh viễn
echo 'export XDG_RUNTIME_DIR=/run/user/$(id -u)' >> ~/.bashrc

# Enable linger để user session tồn tại
sudo loginctl enable-linger openclaw
```

</details>

<details>
<summary><strong>Lỗi "Unknown model: ollama/minimax-m2.5:cloud"</strong></summary>

OpenClaw cần `OLLAMA_API_KEY` để nhận Ollama là provider:

```bash
echo 'export OLLAMA_API_KEY="ollama-local"' >> ~/.bashrc
source ~/.bashrc
systemctl --user restart openclaw-gateway.service
```

</details>

<details>
<summary><strong>Telegram bot chỉ hiện pairing code, không trả lời</strong></summary>

Đây là hành vi bình thường ở lần dùng đầu tiên. Duyệt pairing trên VPS:

```bash
openclaw pairing approve telegram <YOUR_PAIRING_CODE>
```

Pairing code nằm trong phản hồi đầu tiên của bot khi bạn gửi `/start`.

</details>

<details>
<summary><strong>Lỗi dashboard "Token Mismatch" hoặc "Token Missing"</strong></summary>

Đảm bảo URL có đầy đủ token:

```
http://localhost:18789/#token=YOUR_GATEWAY_TOKEN
```

Lấy token hiện tại:

```bash
cat ~/.openclaw/openclaw.json | grep '"token"'
```

Đồng thời kiểm tra SSH tunnel vẫn đang chạy trước khi mở browser.

</details>

<details>
<summary><strong>Ollama không hoạt động</strong></summary>

```bash
# Check if Ollama is running
sudo systemctl status ollama

# Restart if needed
sudo systemctl restart ollama

# Test the cloud model
ollama run minimax-m2.5:cloud "hello"
```

</details>

<details>
<summary><strong>Lỗi cloud model / timeout</strong></summary>

Nếu MiniMax M2.5 chậm hoặc không khả dụng:

```bash
# Option 1: Switch to a different cloud model
ollama launch openclaw --model kimi-k2.5:cloud

# Option 2: Use a small local model (uses ~2.5GB RAM)
ollama pull llama3.2:3b
ollama launch openclaw --model llama3.2:3b
```

</details>

<details>
<summary><strong>Nginx không khởi động sau khi config bị lỗi</strong></summary>

```bash
# Test the config
sudo nginx -t

# If broken, reinstall default config
sudo apt install --reinstall -y nginx-core
sudo systemctl restart nginx
```

</details>

<details>
<summary><strong>Hết bộ nhớ (Out of Memory)</strong></summary>

Với 4GB RAM, tài nguyên khá hạn chế. Theo dõi usage:

```bash
free -h
```

Nếu thiếu bộ nhớ, thêm swap:

```bash
sudo fallocate -l 2G /swapfile
sudo chmod 600 /swapfile
sudo mkswap /swapfile
sudo swapon /swapfile

# Make it permanent
echo '/swapfile none swap sw 0 0' | sudo tee -a /etc/fstab
```

</details>

---

<a id="security-considerations"></a>
## 🔐 Lưu ý bảo mật

### Điểm an toàn trong setup này

- **Firewall (UFW)** chặn toàn bộ incoming traffic trừ SSH (22) và HTTP (80)
- **OpenClaw gateway** chỉ bind localhost, không exposed trực tiếp ra internet
- **Sudo permissions** giới hạn theo command, tránh root full-time
- **fail2ban** giảm rủi ro SSH brute force
- **Dedicated user** giúp dịch vụ không chạy bằng root

### Các rủi ro cần biết

| Rủi ro | Giảm thiểu |
|:-------|:-----------|
| Cloud LLM thấy prompt của bạn | Dùng local model nếu xử lý dữ liệu nhạy cảm |
| Telegram không có E2E encryption cho bot | Không gửi password/secret qua bot |
| Third-party skills có thể độc hại | Chỉ cài skill từ nguồn tin cậy |
| Prompt injection từ input xấu | Giữ bot private, không thêm vào public group |

### Hardening khuyến nghị (tùy chọn)

<details>
<summary><strong>Bấm để mở rộng: Hardening commands</strong></summary>

```bash
# Disable password-based SSH (use key-based auth instead)
sudo sed -i 's/#PasswordAuthentication yes/PasswordAuthentication no/' /etc/ssh/sshd_config
sudo systemctl restart sshd

# Keep the system updated
sudo apt update && sudo apt upgrade -y

# Set up unattended security updates
sudo apt install -y unattended-upgrades
sudo dpkg-reconfigure -plow unattended-upgrades
```

</details>

---

## 📊 Tổng kết

```
┌──────────────────┬────────────────────────────────────────────┐
│ Thành phần       │ Vai trò                                    │
├──────────────────┼────────────────────────────────────────────┤
│ Ubuntu 24.04 VPS │ Base infrastructure (4GB RAM / 2 vCPU)    │
│ Nginx            │ Serves static website on port 80           │
│ Ollama           │ Connects to MiniMax M2.5 cloud LLM        │
│ OpenClaw         │ AI agent that manages the server           │
│ Telegram Bot     │ Your interface to control everything       │
└──────────────────┴────────────────────────────────────────────┘
```

Toàn bộ stack chạy được trên VPS **4GB RAM / 2 vCPU** vì phần xử lý AI nặng nằm ở cloud (MiniMax M2.5), còn VPS chỉ chạy agent runtime nhẹ và web service.

Bạn có thể làm hỏng Nginx, gửi một tin nhắn Telegram, và để AI agent tự chẩn đoán + sửa lỗi mà không cần thao tác SSH thủ công.

---

<a id="whats-next"></a>
## 🚀 Bước tiếp theo

Khi đã quen với setup này, bạn có thể mở rộng:

- [ ] **Thêm nhiều service hơn**: Docker containers, database, SSL certificates
- [ ] **Thiết lập proactive alerts**: để OpenClaw nhắn trước khi sự cố xảy ra
- [ ] **Chuyển sang local model**: khi nâng cấp VPS, thử `qwen2.5-coder:14b` để tăng riêng tư
- [ ] **Thêm kênh nhắn tin**: WhatsApp hoặc Discord ngoài Telegram
- [ ] **Viết custom skills**: tạo workflow riêng cho hạ tầng của bạn

---

<a id="contributing"></a>
## 🤝 Đóng góp

Đóng góp luôn được hoan nghênh. Nếu bạn có cải tiến:

1. Fork repository này
2. Tạo feature branch (`git checkout -b improve-guide`)
3. Commit thay đổi (`git commit -m 'Add new troubleshooting tip'`)
4. Push branch (`git push origin improve-guide`)
5. Mở Pull Request

---

<a id="license"></a>
## 📄 Giấy phép

Dự án sử dụng MIT License. Xem file [LICENSE](LICENSE) để biết chi tiết.

---

<p align="center">
  <strong>Hướng dẫn đã được kiểm thử trên Ubuntu 24.04 LTS | OpenClaw + Ollama + MiniMax M2.5 Cloud | Tháng 2 năm 2026</strong>
</p>

<p align="center">
  <sub>Nếu thấy hướng dẫn hữu ích, hãy ⭐ star repository này.</sub>
</p>
