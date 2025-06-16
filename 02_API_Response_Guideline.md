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
  "code": "USR_200",
  "message": "Thành công",
  "data": {
    "userId": 123,
    "username": "john.doe"
  },
  "traceId": "abc-xyz-789"
}
```

| Trường     | Bắt buộc | Ý nghĩa                                                                 |
|------------|----------|------------------------------------------------------------------------|
| `success`  | ✅        | Luôn là `true` nếu thành công                                           |
| `code`     | ✅        | Mã kết quả nội bộ – tham khảo `error_codes.md`                         |
| `message`  | ✅        | Mô tả dễ hiểu, có thể hiển thị hoặc log                                 |
| `data`     | ✅        | Payload trả về – object/array/null                                     |
| `traceId`  | ✅        | ID để trace xuyên suốt request, log, monitor                           |

---

## ❌ Phản hồi lỗi nghiệp vụ (`success = false`, HTTP vẫn là `200 OK`)

```json
{
  "success": false,
  "code": "INVALID_OTP",
  "message": "Mã OTP không hợp lệ hoặc đã hết hạn",
  "traceId": "xyz-789"
}
```

---

## 🛠️ Lỗi kỹ thuật (do hệ thống / middleware handle)

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

> ✅ Những lỗi trên sẽ được middleware handle, Dev không cần viết tay.

---

## 🧠 Phân loại lỗi

| Loại lỗi  | HTTP Code | Cách xử lý                            |
|-----------|-----------|----------------------------------------|
| Kỹ thuật  | 4xx / 5xx  | Middleware tự động xử lý               |
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

## ⏱️ Lỗi nghiệp vụ (Do Dev xử lý thủ công)

Lỗi nghiệp vụ là lỗi liên quan đến logic hệ thống hoặc nghiệp vụ như:  
OTP sai, tài khoản bị khóa, vi phạm điều kiện...

- HTTP status: `200 OK`
- `success`: `false`
- `code`: mã lỗi nội bộ
- `message`: rõ ràng, dễ hiểu
- `traceId`: để trace log

### Ví dụ:
```json
{
  "success": false,
  "code": "INVALID_OTP",
  "message": "Mã OTP không hợp lệ hoặc đã hết hạn",
  "traceId": "xyz-789"
}
```

---

## ✅ Checklist dành cho Dev khi xử lý response API

| Tiêu chí kiểm tra                                     | Trạng thái | Ghi chú                                     |
|--------------------------------------------------------|------------|---------------------------------------------|
| Trả `success = true` khi xử lý thành công              | ✅         | Bắt buộc                                    |
| Trả `success = false` khi xảy ra lỗi nghiệp vụ         | ✅         | Dùng đúng `code`, `message`, `traceId`      |
| Không dùng `throw` cho lỗi nghiệp vụ                   | ✅         | Tránh crash không cần thiết                 |
| Middleware xử lý lỗi kỹ thuật (4xx/5xx)                | ✅         | Không cần xử lý thủ công                    |
| Luôn có `traceId` trong mọi response                   | ✅         | Dùng để trace log & monitoring              |
| Dùng `code` từ `error_codes.md`                        | ✅         | Không tự đặt mã lỗi                         |
| Không trả HTTP 200 nếu là lỗi kỹ thuật                 | ✅         | Chỉ dùng 200 cho lỗi nghiệp vụ              |

---

## 🧩 Gợi ý bổ sung

- Mã lỗi nên đặt theo cấu trúc: `DOMAIN_CODE`  
  Ví dụ: `AUTH_401`, `USR_200`, `OTP_423`, `SYS_500`
- Mỗi API cần có mô tả chi tiết bằng OpenAPI YAML (.yaml)  
  Đặt tại thư mục: `docs/api/openapi/`
- `traceId` nên được sinh ra từ gateway hoặc service đầu tiên để giữ consistency

---

## 🧪 Khi viết API mới hoặc refactor cần đảm bảo test:

- ☑️ Trường hợp **thành công (happy path)**
- ❌ Trường hợp **sai dữ liệu, trạng thái không hợp lệ, lỗi nghiệp vụ**

---

## ✅ Checklist QA – Review API

| Mục kiểm                              | Bắt buộc | Ghi chú                                                |
|--------------------------------------|----------|--------------------------------------------------------|
| Đúng format JSON response            | ✅       | `success`, `code`, `message`, `traceId`               |
| Dùng đúng mã lỗi chuẩn               | ✅       | Tham khảo `error_codes.md`                            |
| Có YAML mô tả API (OpenAPI 3.0)      | ✅       | File đặt tại `docs/api/openapi/`                      |
| Có `traceId` trong mọi phản hồi      | ✅       | Bắt buộc để trace lỗi chính xác                       |
| Không dùng HTTP 200 cho lỗi hệ thống| ✅       | 200 chỉ dành cho thành công hoặc lỗi nghiệp vụ        |

---

## 📄 YAML OpenAPI bắt buộc

- Mỗi API phải có file `.yaml` tại `docs/api/openapi/`
- Viết theo chuẩn **OpenAPI 3.0**
- Sử dụng [Swagger Editor](https://editor.swagger.io/) để validate file
  
---


## 📚 Tài liệu liên quan

- [`error_codes.md`](./error_codes.md)
- [`API_Naming_Convention.md`](./API_Naming_Convention.md)
- [`Coding_Convention.md`](./Coding_Convention.md)
- `docs/api/openapi/*.yaml` – mô tả API theo chuẩn OpenAPI 3.0

---

📌 **Tài liệu này là tiêu chuẩn bắt buộc áp dụng trong tất cả hệ thống nội bộ ISC.**  
Áp dụng cho tất cả Dev, QA, Reviewer trong quá trình xây dựng và review API.
