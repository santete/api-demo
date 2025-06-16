# ✨ CODING_CONVENTIONS.md – Chuẩn viết code ISC (.NET ưu tiên)

Tài liệu quy định chuẩn viết code cho toàn bộ thành viên trong team phát triển dự án ISC.  
Ngôn ngữ chính: **.NET Core**, tuy nhiên nguyên tắc mang tính **ngôn ngữ-độc lập**.

---

## 📌 1. Nguyên tắc chung

| Nguyên tắc                  | Giải thích ngắn gọn                                         |
|-----------------------------|--------------------------------------------------------------|
| ✅ Nhất quán                | Cùng team nên cùng cách đặt tên, indent, comment             |
| ✅ Dễ đọc – dễ hiểu         | Ưu tiên rõ ràng hơn ngắn gọn, dễ review                      |
| ✅ Không lặp lại code       | Dùng hàm/helper tái sử dụng                                 |
| ✅ 1 hàm = 1 chức năng      | Tránh viết hàm đa mục đích                                 |
| ✅ Không hardcode           | Dùng config/const/enum/DB thay vì số liệu cứng               |
| ✅ Luôn có xử lý lỗi        | Không để app crash hoặc báo lỗi chung chung                 |

---

## 📁 2. Quy ước đặt tên

| Thành phần      | Quy ước       | Ví dụ                         |
|------------------|----------------|-------------------------------|
| Biến             | `camelCase`    | `isActive`, `totalAmount`     |
| Hàm              | `PascalCase`   | `CalculateTotal()`, `GetUser()` |
| Class            | `PascalCase`   | `UserService`, `TokenHandler` |
| Const / Enum     | `PascalCase`   | `DefaultTimeout`, `UserType.Admin` |
| Interface        | `I` + Pascal   | `IUserService`, `IMiddleware` |
| Tên file `.cs`   | Trùng tên class| `LoginController.cs`          |

---

## 🧱 3. Quy ước thư mục

### 📂 3.1. Thư mục code `src/`

| Thư mục         | Mục đích                                      |
|------------------|-----------------------------------------------|
| `Controllers/`   | Xử lý route & HTTP                           |
| `Services/`      | Xử lý logic nghiệp vụ                        |
| `Repositories/`  | Giao tiếp DB, Redis, Kafka                   |
| `Models/`        | DTO, schema, entity                          |
| `Configs/`       | Cấu hình hệ thống (auth, logging, timeout...)|
| `Middlewares/`   | Can thiệp request, response                  |
| `Utils/Helpers/` | Hàm tiện ích tái sử dụng                    |
| `Tests/`         | Unit test, integration test                  |

### 📚 3.2. Thư mục tài liệu `docs/`

| Thư mục          | Mục đích cụ thể                                   |
|------------------|----------------------------------------------------|
| `api/`           | Guideline response, error, OpenAPI YAML           |
| `setup/`         | Cách cài môi trường local/dev                    |
| `architecture/`  | Sơ đồ hệ thống, ADR (architecture decision)       |
| `deployment/`    | CI/CD, dockerfile, helm chart, triển khai script |

---

## 🎨 4. Format code

- ✅ 4 space indent
- ✅ Mỗi file chỉ nên chứa 1 class chính
- ❌ Không dùng cả tab và space
- ✅ Dùng `dotnet format` để đảm bảo format nhất quán
- ✅ `{` xuống dòng (C# style)

---

## ✍️ 5. Comment & Documentation

- ✅ Comment các logic phức tạp
- ✅ Dùng XML-doc cho public method

```csharp
/// <summary>
/// Tính tổng đơn hàng theo loại
/// </summary>
public decimal CalculateTotal(List<Order> orders) { ... }
```

- ❌ Tránh comment dư thừa kiểu:
```csharp
// Tăng biến i lên 1
i++;
```

---

## 🛡 6. Bảo mật & Chất lượng

| Nội dung                 | Lưu ý                                               |
|--------------------------|-----------------------------------------------------|
| ❌ Không log dữ liệu nhạy cảm | Ví dụ: email, token, password                  |
| ✅ Validate input đầy đủ | Không tin tưởng FE gửi                            |
| ❌ Không throw exception thô | Gói lại, log rõ nội bộ                         |
| ✅ Dùng middleware handle error | Trả `traceId`, log chi tiết nếu cần         |

---

## 🛌 7. Mẫu commit message chuẩn Git

```bash
feat(auth): thêm xác thực OAuth2 cho Google
fix(user): sửa lỗi không load avatar khi null
refactor(session): gom nhỏ logic khởi tạo session
```

---

## 💪 8. Testing

- ✅ Mỗi service nên có ít nhất 1 unit test
- ✅ Dùng mock repository
- ✅ Test cả success và fail case

---

## 📤 9. Chuẩn hóa phản hồi API

### ✅ 9.1 Success Response

```json
{
  "success": true,
  "code": "USR_200",
  "message": "Thành công",
  "data": { ... },
  "traceId": "abc-xyz-789"
}
```

| Trường     | Bắt buộc | Mô tả                                    |
|------------|----------|-------------------------------------------|
| success    | ✅       | Luôn là `true` khi thành công             |
| code       | ✅       | Mã phản hồi nội bộ                        |
| message    | ✅       | Nội dung mô tả dễ hiểu                    |
| data       | ✅       | Payload trả về (object / array / null)    |
| traceId    | 🔁       | Mã để trace log toàn hệ thống             |

### 🔁 Ví dụ với phân trang:

```json
{
  "success": true,
  "code": "USR_200",
  "message": "Danh sách người dùng",
  "data": [...],
  "metadata": {
    "page": 1,
    "pageSize": 20,
    "total": 100
  },
  "traceId": "xyz-456"
}
```

---

### ❌ 9.2 Error Response

```json
{
  "success": false,
  "code": "AUTH_401",
  "message": "Không có quyền truy cập",
  "traceId": "abc-123"
}
```

| Trường     | Bắt buộc | Mô tả                                       |
|------------|----------|----------------------------------------------|
| success    | ✅       | Luôn là `false` khi lỗi                     |
| code       | ✅       | Mã lỗi nội bộ chuẩn (`AUTH_401`, `REQ_400`) |
| message    | ✅       | Mô tả lỗi rõ ràng                           |
| traceId    | 🔁       | Dùng để trace log                           |

> 📌 Nên trả toàn bộ qua middleware  
> 📌 Không dùng HTTP status code thô để phân biệt lỗi nghiệp vụ

---

## 📌 SUMMARY

- ✅Áp dụng cho toàn bộ các team nội bộ tại FPT, ưu tiên các dự án sử dụng .NET 6+ và triển khai CI/CD, DevSecOps.
- ✅ Mọi project backend đều cần `README.md`, `ERROR_CODES.md`, `api-guideline.md`
- ✅ Tạo repo backend: phải có `docs/` để QA và Dev follow
- ✅ 🌟**Tài liệu này cần được đính kèm trong repo dưới tên `CODING_CONVENTIONS.md` để các thành viên tuân thủ.**

---
