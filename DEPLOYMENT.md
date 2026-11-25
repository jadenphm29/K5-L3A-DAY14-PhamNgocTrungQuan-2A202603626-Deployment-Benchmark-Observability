# Deployment Information — Day 14 Production AI Agent

> Thông tin triển khai dịch vụ Production AI Agent lên Cloud Platform (Render / Railway).  
> Tự kiểm tra 4 lệnh tiêu chuẩn theo hướng dẫn tại [CHECKPOINTS.md](CHECKPOINTS.md#cp6--cloud-deployment-250--315).

## Public URL
`https://k5-day14-production-agent.onrender.com`

*(Ghi chú: Đối với nền tảng Railway: `https://production-ai-agent-k5.up.railway.app`)*

## Platform
- **Nền tảng chính:** Render (Web Service - Docker runtime, Singapore Region)
- **Cấu hình Blueprint:** `06-lab-complete/render.yaml` & `06-lab-complete/railway.toml`
- **Data Store:** Managed Redis (Upstash / Redis Cloud / Render Redis)

---

## Test Commands & Verification Results

### 1. Health Check (Liveness Probe)
Kiểm tra container còn sống:
```bash
curl -sS -o /dev/null -w '%{http_code}\n' https://k5-day14-production-agent.onrender.com/health
# Trả về: 200
```
Chi tiết response body:
```json
{"status": "ok", "environment": "production"}
```

### 2. Readiness Check (Readiness Probe)
Kiểm tra kết nối Redis sẵn sàng nhận tải:
```bash
curl -sS -o /dev/null -w '%{http_code}\n' https://k5-day14-production-agent.onrender.com/ready
# Trả về: 200
```
Chi tiết response body:
```json
{"ready": true}
```
*(Nếu Redis mất kết nối, endpoint tự động trả về HTTP 503 Service Unavailable trong khi `/health` vẫn trả về 200)*.

### 3. API Authentication — Thiếu hoặc sai API Key (Expect 401)
```bash
curl -sS -o /dev/null -w '%{http_code}\n' -X POST https://k5-day14-production-agent.onrender.com/ask \
  -H "Content-Type: application/json" \
  -d '{"question": "Hello"}'
# Trả về: 401
```
Chi tiết response body:
```json
{"detail": "Invalid or missing API key"}
```

### 4. API Authentication — Có API Key hợp lệ (Expect 200)
```bash
curl -sS -o /dev/null -w '%{http_code}\n' -X POST https://k5-day14-production-agent.onrender.com/ask \
  -H "Content-Type: application/json" \
  -H "X-API-Key: prod_sec_day14_k5_valid_key" \
  -d '{"question": "Hello"}'
# Trả về: 200
```
Chi tiết response body:
```json
{
  "question": "Hello",
  "answer": "Đây là câu trả lời từ AI agent (mock)...",
  "cost_usd": 0.000035
}
```

---

## Environment Variables Set

Các biến môi trường được cấu hình an toàn trên Dashboard của Cloud Platform, tuyệt đối không commit secret vào source code hay file git:

| Variable Name | Giá trị mẫu / Mô tả | Mục đích |
|---|---|---|
| `PORT` | `8000` | Port lắng nghe do nền tảng cấp tự động |
| `ENVIRONMENT` | `production` | Đánh dấu môi trường vận hành |
| `AGENT_API_KEY` | *(Set an toàn qua UI / generateValue)* | Khóa bí mật xác thực client qua header `X-API-Key` |
| `REDIS_URL` | `rediss://default:***@singapore-redis.render.com:6379` | URL kết nối Redis dùng cho Rate Limiter và Cost Guard |
| `RATE_LIMIT_PER_MINUTE` | `10` | Giới hạn 10 request/phút (Sliding window 60s) |
| `DAILY_BUDGET_USD` | `1.0` | Giới hạn ngân sách tối đa $1.0 / ngày |
| `MONTHLY_BUDGET_USD` | `10.0` | Giới hạn ngân sách tối đa $10.0 / tháng |

---

## Screenshots

- **Deployment Dashboard:**  
  ![Deployment dashboard](screenshots/dashboard.png)

- **Service Running Logs:**  
  ![Service running](screenshots/running.png)

- **Self-Test Results (4 CP6 Commands):**  
  ![Test results](screenshots/test.png)
