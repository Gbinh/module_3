# TÓM TẮT KIẾN THỨC MODULE 3

## 1. CLIENT–SERVER & HTTP

**Mô hình Client–Server:** Client gửi yêu cầu → Server xử lý → trả lời. Rõ ràng hơn P2P, dễ bảo mật và kiểm soát tập trung.

**HTTP Request/Response:**
- **Request:** Phương thức (GET/POST/PUT/PATCH/DELETE) + URL + headers + body
- **Response:** Status code + headers + body

| Code | Ý nghĩa |
|------|---------|
| 2xx | Thành công |
| 3xx | Redirect |
| 4xx | Lỗi client (400: bad request, 401: chưa auth, 404: không tìm thấy) |
| 5xx | Lỗi server (500: internal error, 503: service unavailable) |

**REST API:** Dùng **danh từ** cho resource (`/students`), **HTTP method** là động từ. Status code phải khớp kết quả thực tế.

---

## 2. XÂY DỰNG BACKEND

**Vòng đời Request:**
1. **Routing:** URL pattern + method → handler
2. **Middleware Pipeline:** Auth, model binding, logging chạy trước handler
3. **Handler Logic:** Xử lý business logic
4. **Response:** Trả về client

**Quản lý State:**
- **HTTP stateless:** Mỗi request độc lập → dễ scale ngang, nhưng cần cơ chế nhận diện user
- **Cookies/Session:** Server lưu state, browser chỉ giữ session ID → dễ thu hồi nhưng cần shared store khi nhiều instance
- **JWT:** Token tự mang claims (đã ký) → server chỉ verify chữ ký → scale ngang tốt; dùng **access token ngắn hạn** + **refresh token** để dễ kiểm soát

**CORS:** Quy tắc bảo mật của **browser** - server phải cho phép bằng header `Access-Control-Allow-Origin`. Chỉ ảnh hưởng JS trong browser.

**Bảo mật cơ bản:**
- **Authentication** (xác định danh tính: bạn là ai)
- **Authorization** (kiểm soát quyền: bạn được làm gì) - kiểm tra mỗi request
- HTTPS bắt buộc, input validation luôn, rate limiting

---

## 3. DATABASE OPTIMIZATION

**3 cấp độ tối ưu:**

### Tier 1: Schema & Chỉ mục
- **SQL:** Schema cố định trước (validation chặt)
- **NoSQL:** Document linh hoạt (schema-on-read)
- **Khóa:** Primary Key định danh dòng; Foreign Key trỏ tới PK
- **N:M:** Cần bảng junction/collection riêng
- **Index:** COLLSCAN/Clustered Scan → IXSCAN/Index Seek (chuyển từ quét cả bảng thành chỉ lấy cần thiết)
- **Quy tắc ESR (Equality → Sort → Range):** Thứ tự field trong composite index quan trọng để tối ưu filter + sort + range query

### Tier 2: Query Optimization
**Nguyên lý:** Làm ít công việc hơn = nhanh hơn. Luôn đo trước khi sửa.

| Vấn đề | Nguyên nhân | Giải pháp |
|--------|-----------|----------|
| **N+1 Problem** | 1 query lấy list + N query từng dòng | GROUP BY / $group (một pass) |
| **NOT IN + NULL** | NULL → kết quả sai âm thầm | Dùng NOT EXISTS thay thế |
| **OFFSET lớn** | Phải walk qua mọi dòng trước | Keyset pagination (`WHERE id > last`) |
| **Subquery lặp** | Chạy đi chạy lại | Window functions (`RANK() OVER`) |

**Write Optimization:**
- **Set-based:** `INSERT...SELECT` / `insertMany` thay vì row-by-row
- **UPSERT:** `MERGE` / `updateOne({upsert:true})` → 1 round-trip, không race condition
- **Batch Transaction:** Gom nhiều write vào 1 transaction nhưng không quá lớn (giữ lock lâu)

### Tier 3: Infrastructure
- **Read Replicas:** Nhân bản data, tách read (write vẫn 1 primary) - không giải quyết data quá lớn
- **Sharding:** Chia **data** sang nhiều instance
  - **Range:** Tốt cho range query, rủi ro **hot shard** (1 shard cháy)
  - **Hashed:** Chia đều, mất range scan xuyên shard
  - Chọn shard key là quyết định khó đổi nhất
- **Partitioning ≠ Sharding:** Partition = 1 instance (quản lý), Sharding = nhiều instance

---

## 4. MOBILE APP (REACT NATIVE + EXPO)

**Setup:**
- Expo quản lý native config; Expo Go chạy tức thì
- Metro bundler chạy dev server (laptop & điện thoại cùng Wi-Fi)
- Expo Router: file-based routing (`app/index.tsx` → `/`, `app/item/[id].tsx` → `/item/123`)

**UI System:**
- **View:** Container; **Text:** mọi text phải bên trong Text
- **StyleSheet.create:** Tĩnh, tối ưu (camelCase, không kế thừa CSS)
- **Design tokens:** Gom `colors/spacing/radius/fontSize` vào 1 file `theme.ts` → đổi 1 chỗ, cả app cập nhật
- **FlatList:** Thay `.map()` - chỉ render item trên màn hình (virtualization)

**Backend + Auth:**
- Không dùng `localhost` - dùng **LAN IP** của máy dev hoặc **ngrok**
- JWT auth: login → lưu token `SecureStore` → mỗi request gửi `Authorization: Bearer <token>`; 401 → xóa token, quay lại login
- **SecureStore** cho token (mã hóa OS), **AsyncStorage** cho cache không nhạy cảm

**CRUD + Release:**
- List (GET) → Detail (GET/:id) → Edit (PUT) → Refresh list
- Form 2 mode: tái dùng form, phân biệt bằng sự hiện diện của `id`
- Delete có confirmation alert (style destructive) + optimistic update (sửa UI ngay)
- **Offline cache:** Hiện cache ngay, fetch mới nền - offline vẫn xem được list
- **APK:** `npx expo prebuild` + `./gradlew assembleRelease` (local) hoặc EAS Build (cloud)

---

## 5. REQUEST → RESPONSE END-TO-END

**Journey:**
DNS → TCP+TLS → CDN/Edge → Load Balancer → Reverse Proxy → App Server

**Edge Protection (dừng request xấu sớm):**
- Rate limiting (429)
- WAF (SQLi/XSS)
- DDoS + geo-blocking
- TLS enforcement

**Load Balancing:**
- **L4:** IP+port (nhanh, không đọc HTTP)
- **L7:** Đọc path/header/cookie (route `/api/*` chi tiết)
- **Reverse proxy (Nginx):** Load balancer + SSL termination + cache + compression
- **Health-check & rolling update:** Zero-downtime deployment

**Auth Nâng cao:**
- **OAuth2/OIDC:** Delegate login tới Identity Provider (Google, Azure AD) - PKCE flow
- **Refresh token rotation:** Phát token mới mỗi lần dùng, token cũ vô hiệu
- **API keys:** Cho service-to-service (không bao giờ client-side)
- **RBAC:** Role cố định (user, admin); **ABAC:** Attribute-based (context-aware)

**Data Optimization (ngoài index):**
- **Caching + TTL:** Chống cache stampede (khi cache hết, nhiều request cùng vào DB)
- **Eager loading:** Lấy đủ field ngay (tránh N+1)
- **Projection:** Chỉ lấy field cần (DTO)
- **Pagination:** Offset đắt ở sâu, keyset/cursor phẳng

**Messaging & Async:**
- **Outbox:** Ghi message cùng transaction với business logic (không mất message)
- **Saga:** Chuỗi bước + compensating action (rollback)
- **Idempotent consumer:** Delivery "at least once" → consumer phải chịu duplicate
- **Dead-letter queue:** Message lỗi, thử lại sau
- **Backpressure:** Giới hạn prefetch để không bị ngập

**Resilience:**
- **Retry + backoff:** Thử lại với khoảng cách tăng dần
- **Circuit breaker:** Sau N lỗi → open circuit (fail fast) 30s → half-open (thử lại)
- **Correlation ID:** Xuyên service để tracking

**Response:**
- **ETag + 304:** Chỉ gửi nếu thay đổi
- **Envelope:** `{ success, data, error }` nhất quán
- **Security headers:** CSP, X-Frame-Options, HSTS
- **Async response:** 202 Accepted + polling, hoặc webhook callback
- **Compression:** Gzip/Brotli (không nén ảnh/video/zip)

---

## 6. LIÊN HỆ FOOD ROULETTE PROJECT

| Kiến thức | Áp dụng |
|-----------|--------|
| REST API, JWT Bearer | Backend Express `/api/v1`, auth login/refresh |
| Database (Prisma + MySQL) | Index, tránh N+1, eager loading |
| React Native + Expo | Mobile app, Expo Router, design tokens |
| Auth flow | Login → access + refresh token → SecureStore |
| Offline cache | AsyncStorage cho list, fetch nền |
| Storage | Supabase Storage (bucket `lockets`) cho ảnh |

---

**Bài học lớn nhất:** Luôn đo (profile) trước khi tối ưu. Thứ hạng chi phí không tự chuyển giữa các engine.