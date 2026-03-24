# IPM Pro Edition - Interior Project Manager

Hệ thống quản lý dự án nội thất fullstack với quản lý dự án, tài nguyên, đặt lịch, ví điện tử, theo dõi tiến độ AI và bảng xếp hạng nhà thầu.

![Status](https://img.shields.io/badge/status-completed-brightgreen)
![.NET](https://img.shields.io/badge/.NET-8.0-512BD4)
![ASP.NET Core](https://img.shields.io/badge/ASP.NET%20Core-8.0-512BD4)
![Vue.js](https://img.shields.io/badge/Vue.js-3.x-4FC08D)
![TypeScript](https://img.shields.io/badge/TypeScript-5.x-3178C6)
![Vite](https://img.shields.io/badge/Vite-5.x-646CFF)
![Tailwind CSS](https://img.shields.io/badge/Tailwind%20CSS-3.x-06B6D4)
![Docker](https://img.shields.io/badge/Docker-ready-2496ED)
![SQL Server](https://img.shields.io/badge/SQL%20Server-2022-CC2927)
![EF Core](https://img.shields.io/badge/EF%20Core-8.0-6DB33F)
![Redis](https://img.shields.io/badge/Redis-7.x-DC382D)
![SignalR](https://img.shields.io/badge/SignalR-WebSocket-7A3E9D)
![Hangfire](https://img.shields.io/badge/Hangfire-Background%20Jobs-1E1E1E)
![Gemini AI](https://img.shields.io/badge/Gemini-AI%20Integration-4285F4)

## Tổng quan

- FE: Vue 3 + TypeScript + Pinia + Tailwind + SignalR client
- BE: ASP.NET Core 8 Web API theo Clean Architecture
- DB/Infra: SQL Server + EF Core + Redis + Hangfire
- AI: Gemini cho phân tích tiến độ và OCR hóa đơn

## Tính năng chính

### 1. Quản lý dự án & công việc
- Quản lý dự án theo trạng thái Planning, Ongoing, Handover, Completed
- Kanban task theo phase, cập nhật tiến độ theo thời gian thực
- Theo dõi chi phí ước tính và chi phí thực tế theo project

### 2. Smart Booking tài nguyên
- Đặt lịch tài nguyên/nhân công theo khung giờ
- Hỗ trợ optimistic locking (RowVersion) tránh xung đột
- Gắn booking với project để đồng bộ ngân sách dự án

### 3. Ví điện tử & giao dịch
- Ví thành viên và ví dự án
- Nạp/rút tiền, duyệt yêu cầu nạp, theo dõi lịch sử giao dịch
- Chữ ký SHA256 chống sửa đổi giao dịch

### 4. AI Progress Tracking
- Upload ảnh công trình để AI phân tích tiến độ
- OCR hóa đơn hỗ trợ đối soát nhanh dữ liệu tài chính

### 5. Leaderboard ELO
- Xếp hạng nhà thầu dựa trên hiệu suất hoàn thành dự án
- Redis Sorted Sets cho truy vấn leaderboard nhanh

## Kiến trúc hệ thống

- Domain: entities, enums, business interfaces
- Application: services, validators, DTOs, mappings
- Infrastructure: EF Core, Redis, Hangfire, repositories, external integrations
- API: controllers, middleware, DI, auth pipeline

## Cấu trúc thư mục

```text
FullStackProject/
|-- be/
|   |-- src/
|   |   |-- IPM.Domain/
|   |   |-- IPM.Application/
|   |   |-- IPM.Infrastructure/
|   |   |-- IPM.API/
|   |-- Dockerfile
|   `-- IPM.slnx
|-- fe/
|   |-- src/
|   |   |-- components/
|   |   |-- views/
|   |   |-- stores/
|   |   |-- services/
|   |   |-- composables/
|   |   `-- types/
|   |-- Dockerfile
|   `-- nginx.conf
|-- docker-compose.yml
`-- README.md
```

## Hướng dẫn chạy nhanh

### Yêu cầu

- Docker Desktop
- Node.js 20+ (nếu chạy FE local)
- .NET 8 SDK (nếu chạy BE local)

### Cách 1: Chạy toàn bộ bằng Docker (khuyến nghị)

```bash
git clone <repo-url>
cd FullStackProject
docker compose up -d --build
```

Truy cập:

- Frontend: http://localhost
- API: http://localhost:8080
- Hangfire Dashboard: http://localhost:8080/hangfire

Lệnh quản trị:

```bash
docker compose ps
docker compose logs -f api
docker compose down
```

Reset sạch dữ liệu (xóa volume DB + Redis):

```bash
docker compose down -v
docker compose up -d --build
```

### Cách 2: Hybrid (BE Docker + FE local)

```bash
docker compose up -d db redis api
cd fe
npm install
npm run dev
```

FE local cần trỏ API đúng địa chỉ backend, ví dụ:

```env
VITE_API_URL=http://localhost:8080/api
```

### Cách 3: Chạy local cả BE và FE

```bash
docker compose up -d db redis
cd be
dotnet restore
dotnet run --project src/IPM.API
```

Terminal khác:

```bash
cd fe
npm install
npm run dev
```

## Cấu hình ứng dụng

### appsettings.json (mẫu)

```json
{
  "ConnectionStrings": {
    "DefaultConnection": "Server=localhost;Database=IPMProDB;...",
    "RedisConnection": "localhost:6379",
    "HangfireConnection": "Server=localhost;Database=IPMProDB;..."
  },
  "Jwt": {
    "Key": "your-32-plus-char-secret",
    "Issuer": "https://localhost:7136",
    "Audience": "https://localhost:7136",
    "ExpireHours": 24
  },
  "Database": {
    "TablePrefix": "123"
  },
  "Email": {
    "SmtpServer": "smtp.gmail.com",
    "SmtpPort": 587,
    "FromEmail": "your-email@gmail.com",
    "FromPassword": "your-gmail-app-password",
    "EnableSsl": true
  },
  "Gemini": {
    "ApiKey": "your-gemini-api-key",
    "Model": "gemini-3.0-pro"
  },
  "App": {
    "FrontendUrl": "http://localhost:5173"
  }
}
```

Ghi chú SMTP Gmail:

- Bật 2-Step Verification cho tài khoản Gmail
- Dùng App Password cho FromPassword
- Không dùng mật khẩu Gmail thường

### docker-compose api environment (mẫu chuẩn env)

```yaml
services:
  api:
    environment:
      - ASPNETCORE_ENVIRONMENT=Production
      - ASPNETCORE_URLS=http://+:8080
      - ConnectionStrings__DefaultConnection=Server=db;Database=IPMProDB;User Id=sa;Password=${DB_PASSWORD};TrustServerCertificate=True;MultipleActiveResultSets=true
      - ConnectionStrings__RedisConnection=redis:6379,password=${REDIS_PASSWORD}
      - ConnectionStrings__HangfireConnection=Server=db;Database=IPMProDB;User Id=sa;Password=${DB_PASSWORD};TrustServerCertificate=True
      - Jwt__Key=${JWT_SECRET_KEY}
      - Jwt__Issuer=https://localhost:7136
      - Jwt__Audience=https://localhost:7136
      - Jwt__ExpireHours=24
      - Database__TablePrefix=123
      - Gemini__ApiKey=${GEMINI_API_KEY}
      - Gemini__Model=gemini-3.0-pro
      - Email__SmtpServer=smtp.gmail.com
      - Email__SmtpPort=587
      - Email__FromEmail=${SMTP_FROM_EMAIL}
      - Email__FromPassword=${SMTP_APP_PASSWORD}
      - Email__EnableSsl=true
      - App__FrontendUrl=http://web:80
```

### Danh sách biến môi trường khuyến nghị

| Variable | Ý nghĩa |
|---|---|
| DB_PASSWORD | Mật khẩu SQL Server `sa` |
| REDIS_PASSWORD | Mật khẩu Redis |
| JWT_SECRET_KEY | JWT signing key (>= 32 ký tự) |
| GEMINI_API_KEY | API key Gemini |
| SMTP_FROM_EMAIL | Email gửi SMTP |
| SMTP_APP_PASSWORD | App Password Gmail |

## API chính

### Authentication
- POST /api/auth/register
- POST /api/auth/login
- POST /api/auth/refresh
- POST /api/auth/logout

### Wallet
- GET /api/wallets/balance
- GET /api/wallets/transactions
- POST /api/wallets/deposit
- POST /api/wallets/withdraw

### Booking
- GET /api/bookings
- POST /api/bookings
- PUT /api/bookings/{id}
- DELETE /api/bookings/{id}

### AI
- POST /api/ai/analyze-progress
- POST /api/ai/extract-invoice

### Leaderboard
- GET /api/leaderboard

## Database schema (3 nhóm)

### 1. Bảng nghiệp vụ (prefix `123_`)

- 123_Members
- 123_Projects
- 123_Tasks
- 123_Bookings
- 123_WalletTransactions
- 123_ProjectBudgets
- 123_Resources
- 123_News
- 123_RefreshTokens
- 123_MediaFiles
- 123_AiLogs

### 2. Bảng xác thực phân quyền (ASP.NET Identity)

- AspNetUsers
- AspNetRoles
- AspNetUserRoles
- AspNetUserClaims
- AspNetRoleClaims
- AspNetUserLogins
- AspNetUserTokens

### 3. Bảng job nền (Hangfire)

- HangFire.Job
- HangFire.State
- HangFire.JobQueue
- HangFire.Server
- HangFire.Set / Hash / List / Counter / AggregatedCounter
- HangFire.Schema

Ngoài ra có `__EFMigrationsHistory` để theo dõi migration đã áp dụng.

## Tài khoản mẫu seed

| Vai trò | Email | Mật khẩu |
|---|---|---|
| Admin | admin@ipm.vn | Admin@123 |
| Accountant | accountant@ipm.vn | Accountant@123 |
| Project Manager | pm@ipm.vn | Pm@12345 |
| Subcontractor | contractor@ipm.vn | Contractor@123 |
| Client | client@ipm.vn | Client@123 |

## Kiểm tra seed nhanh sau khi chạy

```bash
docker compose exec -T db /opt/mssql-tools18/bin/sqlcmd -S localhost -U sa -P "<SA_PASSWORD>" -C -d IPMProDB -Q "SET NOCOUNT ON; SELECT 'users', COUNT(*) FROM AspNetUsers UNION ALL SELECT 'projects', COUNT(*) FROM [123_Projects] UNION ALL SELECT 'tasks', COUNT(*) FROM [123_Tasks] UNION ALL SELECT 'bookings', COUNT(*) FROM [123_Bookings] UNION ALL SELECT 'news', COUNT(*) FROM [123_News] UNION ALL SELECT 'wallet_transactions', COUNT(*) FROM [123_WalletTransactions];"
```

## Background jobs & realtime

### Hangfire jobs
- BookingCleanupJob
- DailySummaryJob
- EloUpdateJob

### SignalR hub
- /hubs/notification
  - OnTaskUpdated
  - OnBookingChanged
  - OnWalletTransaction

## Troubleshooting

- Port 80/8080/1433/6379 bị chiếm: dừng service đang dùng hoặc đổi port trong docker-compose
- FE gọi sai API: kiểm tra VITE_API_URL và cấu hình CORS
- Lỗi JWT: kiểm tra khóa ký và thời gian hết hạn
- Lỗi SMTP Gmail: kiểm tra App Password và bật 2FA
- Lỗi migration/seed trên DB cũ: chạy `docker compose down -v` rồi up lại

## Lưu ý bảo mật

- Không commit secret thật vào git (DB password, JWT key, Gmail app password, Gemini key)
- Dùng biến môi trường cho môi trường staging/production
- Tài khoản seed chỉ dành cho demo/dev

## License

MIT License - Dùng cho mục đích học tập và nghiên cứu.
