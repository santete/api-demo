# ⏱️ Quy định cấu hình thời gian Timeout cho API

## 🎯 Mục tiêu
- Đảm bảo hệ thống phản hồi đúng thời gian mong đợi
- Tránh treo hệ thống, kẹt tài nguyên do request quá lâu
- Tăng khả năng quan sát (observability) và kiểm soát lỗi
- Cải thiện trải nghiệm người dùng và tính ổn định hệ thống

---

## ✅ 1. Luôn đặt Timeout cho tất cả các tầng

- Bất kỳ HTTP/gRPC/DB call nào đều phải có timeout rõ ràng
- Không được để gọi API nội bộ hoặc bên thứ 3 **vô thời hạn**

---

## ✅ 2. Timeout theo vai trò/tính chất API

| Loại API                      | Thời gian Timeout khuyến nghị |
|-------------------------------|-------------------------------|
| Truy vấn đơn giản (GET)       | 100ms – 1s                    |
| Ghi dữ liệu (POST/PUT)        | 300ms – 2s                    |
| Tạo job / trigger task        | 500ms – 2s                    |
| Upload file / xử lý ảnh       | 5s – 10s                      |
| Webhook callback              | ≤ 3s                          |
| Nội bộ (microservice call)    | 200ms – 1s                    |

---

## ✅ 3. Timeout phải tương thích với hệ thống Retry

- Tổng thời gian retry không được vượt quá timeout của tầng trên
- Ví dụ: nếu frontend timeout = 5s  
  → Internal service retry: max 1.5s x 2 lần = 3s (ok)

---

## ✅ 4. Cần log mọi request bị timeout

- Log thông tin:
  - endpoint
  - duration thực tế
  - request_id
  - thời gian timeout cấu hình
- Giúp alert và thống kê hiệu suất

---

## ✅ 5. Timeout phải có khả năng cấu hình

- Không hardcode
- Sử dụng `.env`, config server, hoặc centralized config

Ví dụ:
```env
API_TIMEOUT_PAYMENT=2000 # ms

## ✅ 6. Phản hồi rõ ràng khi timeout xảy ra
```json
{
  "success": false,
  "error": {
    "code": "REQUEST_TIMEOUT",
    "message": "The request timed out after 2000ms.",
    "retryable": true
  },
  "meta": {
    "timestamp": "2025-06-21T12:30:45Z"
  }
}
```

## ✅ 7. Giao tiếp timeout giữa các service theo chuẩn
Dùng AbortController (JS), Context.WithTimeout (Go), CancellationToken (.NET)
Truyền theo header:
```yaml
X-Request-Timeout: 2000
```
