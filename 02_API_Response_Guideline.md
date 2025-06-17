# 📘 API Response Guideline – Chuẩn ISC

---

## 🎯 Mục tiêu

Chuẩn hóa định dạng phản hồi của tất cả API trong hệ thống ISC giúp:

- Đảm bảo tính nhất quán trong giao tiếp hệ thống.
- Giúp frontend, mobile app, hệ thống tích hợp xử lý đồng nhất.
- Tăng hiệu quả log, trace, debug và monitoring.
- Hỗ trợ quá trình kiểm thử, tự động hóa test.

---

## ✅ Phản hồi thành công (`success = true`)

```json
{
  "success": true,
  "data": {
    "user_id": 123,
    "username": "john.doe"
  },
  "error": null,
  "meta": {
    "request_id": "req-abc-123",
    "trace_id": "trace-xyz-789",
    "timestamp": "2025-06-16T09:00:00Z"
  }
}
```

## ✅ Phản hồi thành công với phân trang (`success = true`)

```json
{
  "success": true,
  "data": [
    { "user_id": "u123", "name": "Nguyễn Văn A" },
    { "user_id": "u124", "name": "Trần Thị B" }
  ],
  "error": null,
  "meta": {
    "pagination": {
      "page": 1,
      "limit": 10,
      "total_items": 42,
      "total_pages": 5,
      "has_next": true,
      "has_prev": false
    },
    "request_id": "req-xyz-111",
    "trace_id": "trace-abc-222",
    "timestamp": "2025-06-16T09:03:00Z"
  }
}
```

| Trường     | Bắt buộc | Ý nghĩa                                                                 |
|------------|----------|------------------------------------------------------------------------|
| `success`  | ✅        | Luôn là `true` nếu thành công                                           |
| `data`     | ✅        | Payload trả về – object/array/null                                     |
| `error`     | ✅        | Cấu trúc lỗi trả về nếu có phát sinh                                     |
| `code`     | ✅        | Mã kết quả nội bộ – tham khảo `error_codes.md`                         |
| `message`  | ✅        | Mô tả dễ hiểu, có thể hiển thị hoặc log                                 |
| `details`     | ✅        | Chi tiết lỗi – object/array/null                                     |
| `retryable`     | ✅        | Xác định khả thi retry cho client                                     |
| `meta`     | ✅        | Cấu trúc thông tin mô tả thêm                                     |
| `pagination`     | ✅        | Cấu trúc thông tin phân trang                                    |
| `request_id`  | ✅        | ID để định danh request bất kỳ khi phát sinh                           |
| `trace_id`  | ✅        | ID để trace xuyên suốt hệ thống phân tán micro-service                           |
| `timestamp`  | ✅        | Thời điểm cụ thể khi phát sinh response                           |

---

## ❌ Phản hồi lỗi nghiệp vụ (`success = false`, HTTP vẫn là `200 OK`)

```json
{
  "success": false,
  "data": null,
  "error": {
    "code": "INVALID_OTP",
    "message": "Mã OTP không hợp lệ hoặc đã hết hạn",
    "details": {
      "field": "sms_otp"
    },
    "retryable": false
  },
  "meta": {
    "request_id": "req-def-456",
    "trace_id": "trace-uvw-123",
    "timestamp": "2025-06-16T09:01:00Z"
  }
}
```

---

## ❌ Phản hồi lỗi kỹ thuật (4xx - 5xx)
```json
{
  "success": false,
  "data": null,
  "error": {
    "code": "SYS_500",
    "message": "Internal server error. Please try again later.",
    "details": {
      "field": "Can not connect database."
    },
    "retryable": true
  },
  "meta": {
    "request_id": "req-hij-999",
    "trace_id": "trace-klm-000",
    "timestamp": "2025-06-16T09:02:00Z"
  }
}
```

| HTTP Code | Tên lỗi               | Mô tả thực tế                                                 |
|-----------|------------------------|----------------------------------------------------------------|
| `400`     | Bad Request            | Dữ liệu sai định dạng, thiếu field, lỗi JSON                  |
| `401`     | Unauthorized           | Thiếu token hoặc token không hợp lệ                           |
| `403`     | Forbidden              | Không có quyền truy cập                                       |
| `404`     | Not Found              | URL sai hoặc resource không tồn tại                           |
| `408`     | Request Timeout        | Server mất quá nhiều thời gian để phản hồi                    |
| `426`     | Upgrade Required       | Client cần nâng cấp phiên bản để tiếp tục sử dụng dịch vụ     |
| `429`     | Too Many Requests      | Spam, gửi quá nhiều request                                    |
| `500`     | Internal Server Error  | Lỗi hệ thống, null pointer, crash bất thường...               |
| `502`     | Bad Gateway            | Lỗi proxy hoặc service upstream                               |
| `503`     | Service Unavailable    | Server quá tải, ngắt kết nối hoặc đang bảo trì                |

> ✅ Những lỗi trên nên được `middleware handle` trả code `httpStatus` chính xác tránh gây ảnh hưởng đến biz-flow của client

---

## 🧠 Phân loại lỗi

| Loại lỗi  | HTTP Code | Cách xử lý                            |
|-----------|-----------|----------------------------------------|
| Kỹ thuật  | 4xx / 5xx  | Middleware chủ động xử lý verify lại trước khi trả về client              |
| Nghiệp vụ | 200 OK     | Dev xử lý và trả về `success = false` |

---

## 🏷️ Prefix gợi ý cho mã lỗi nghiệp vụ

| Prefix             | Ý nghĩa                             |
|--------------------|-------------------------------------|
| `INVALID_*`        | Dữ liệu sai định dạng               |
| `MISSING_*`        | Thiếu thông tin bắt buộc            |
| `DUPLICATE_*`      | Dữ liệu bị trùng                    |
| `NOT_FOUND_*`      | Không tìm thấy                      |
| `UNAUTHORIZED_*`   | Không đủ quyền                      |
| `BUSINESS_RULE_*`  | Vi phạm logic nghiệp vụ             |

---

## 📘 Một số tình huống phổ biến

| Tình huống             | HTTP | success | code           |
|------------------------|------|---------|----------------|
| Thành công             | 200  | true    | `USR_200`      |
| User bị khóa           | 200  | false   | `USER_LOCKED`  |
| Thiếu/sai token        | 401  | false   | `AUTH_401`     |
| Validate thất bại      | 400  | false   | `REQ_400`      |
| Lỗi hệ thống           | 500  | false   | `SYS_500`      |

---

## ⏱️ Lỗi nghiệp vụ (Do Dev xử lý thông tin chi tiết)

Lỗi nghiệp vụ là lỗi liên quan đến logic nghiệp vụ như:  
OTP sai, tài khoản bị khóa, vi phạm điều kiện...

- HTTP status: `200 OK`
- `success`: `false`
- `error`: mô tả cấu trúc lỗi rõ ràng, đầy đủ
- `code`: mã lỗi nội bộ
- `message`: rõ ràng, dễ hiểu
- `details`: chi tiết lỗi hoặc danh sách chi tiết
- `trace_id`: để trace log

### Ví dụ:
```json
{
  "success": false,
  "data": null,
  "error": {
    "code": "VALIDATION_FAILED",
    "message": "Missing required field 'product_id'.",
    "details": {
      "field": "product_id"
    },
    "retryable": false
  },
  "meta": {
    "request_id": "abc-uuid",
    "trace_id": "trace-uuid",
    "timestamp": "2025-06-11T14:46:00Z"
  }
}
```

---

## ✅ Checklist dành cho Dev khi xử lý response API

| Tiêu chí kiểm tra                                     | Trạng thái | Ghi chú                                     |
|--------------------------------------------------------|------------|---------------------------------------------|
| Trả `success = true` khi xử lý thành công              | ✅         | Bắt buộc                                    |
| Trả `success = false` khi xảy ra lỗi nghiệp vụ         | ✅         | Dùng đúng `code`, `message`, `trace_id`      |
| Không dùng `throw` cho lỗi nghiệp vụ                   | ✅         | Tránh crash không cần thiết                 |
| Middleware xử lý lỗi kỹ thuật (4xx/5xx)                | ✅         | Chủ động verify trước khi trả về client                    |
| Luôn có `trace_id` trong mọi response                   | ✅         | Dùng để trace log & monitoring              |
| Dùng `code` từ `error_codes.md`                        | ✅         | Không tự đặt mã lỗi                         |
| Không trả HTTP 200 nếu là lỗi kỹ thuật                 | ✅         | Chỉ có thể 200 cho lỗi nghiệp vụ              |
> ❌ Tránh nhầm lẫn giữa mã lỗi 404 not found và lỗi không tìm thấy dữ liệu trong nghiệp vụ

---

## 🧩 Gợi ý bổ sung

- Mã lỗi nên đặt theo cấu trúc: `DOMAIN_CODE`  
  Ví dụ: `AUTH_401`, `USR_200`, `OTP_423`, `SYS_500`
- Mỗi API cần có mô tả chi tiết bằng OpenAPI YAML (.yaml)  
  Đặt tại thư mục: `docs/api/openapi/`
- `request_id` nên được sinh ra từ gateway API hoặc entry endpoint để giữ consistency
- `trace_id` nên được sinh ra bởi tracing system(ví dụ: Opentelemetry)
- Nên dùng snake_case để viết các input/output thuộc tính API

---

## 🧪 Khi viết API mới hoặc refactor cần đảm bảo test:

- ☑️ Trường hợp **thành công (happy path)**
- ❌ Trường hợp **sai dữ liệu, trạng thái không hợp lệ, lỗi nghiệp vụ**

---

## ✅ Checklist QA – Review API

| Mục kiểm                              | Bắt buộc | Ghi chú                                                |
|--------------------------------------|----------|--------------------------------------------------------|
| Đúng format JSON response            | ✅       | `success`, `code`, `message`, `trace_id`               |
| Dùng đúng mã lỗi chuẩn               | ✅       | Tham khảo `error_codes.md`                            |
| Có YAML mô tả API (OpenAPI 3.0)      | ✅       | File đặt tại `docs/api/openapi/`                      |
| Có `trace_id` trong mọi phản hồi      | ✅       | Bắt buộc để trace lỗi chính xác                       |
| Không dùng HTTP 200 cho lỗi hệ thống| ✅       | 200 chỉ dành cho thành công hoặc lỗi nghiệp vụ khác 4xx       |

---

## 📄 YAML OpenAPI bắt buộc

- Mỗi API phải có file `.yaml` tại `docs/api/openapi/`
- Viết theo chuẩn **OpenAPI 3.0**
- Sử dụng [Swagger Editor](https://editor.swagger.io/) để validate file
  
---

📌 Kết luận
- Một response API nhất quán nên:
- Chuẩn hóa cấu trúc (success, data, error, meta)
- Sử dụng đúng HTTP Status Code
- Có error code rõ ràng
- Có request_id, trace_id để trace
- Phân biệt lỗi retry / không retry
- Không tiết lộ lỗi nội bộ

## 📚 Tài liệu liên quan

- [`error_codes.md`](./error_codes.md)
- [`API_Naming_Convention.md`](./API_Naming_Convention.md)
- [`Coding_Convention.md`](./Coding_Convention.md)
- `docs/api/openapi/*.yaml` – mô tả API theo chuẩn OpenAPI 3.0

---

📌 **Tài liệu này là tiêu chuẩn bắt buộc áp dụng trong tất cả hệ thống nội bộ ISC.**  
Áp dụng cho tất cả Dev, QA, Reviewer trong quá trình xây dựng và review API.
