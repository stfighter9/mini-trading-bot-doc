# Tài liệu Kiến trúc Frontend (Giao diện Người dùng) - Stock Hunter AI (V1.0)

Tài liệu này đã được phân rã (sharded).

## Các phần Kiến trúc
- [1. Giới thiệu (Introduction)](./1-introduction.md)
- [2. Ngăn xếp Công nghệ (Tech Stack)](./2-tech-stack.md)
- [3. Cấu trúc (Structure)](./3-structure.md)
- [4. Logic Nghiệp vụ (Business Logic)](./4-business-logic.md)
- [5. Bảo mật (Security)](./5-security.md)
- [6. Khả năng Tiếp cận (Accessibility)](./6-accessibility.md)
- [7. Bản địa hóa (Localization)](./7-localization.md)
- [8. Triển khai (Deployment)](./8-deployment.md)

# 1. Introduction (Giới thiệu)
*(Trích từ FE Arch V1.0)*

Tài liệu này phác thảo (outlines) kiến trúc (architecture) frontend (frontend) cho (for) dự án (project) "Stock Hunter AI". Mục tiêu (goal) của (of) nó (it) là (is) định nghĩa (define) các (the) công nghệ (technologies), cấu trúc (structure), và (and) các (the) mẫu (patterns) thiết kế (design) sẽ (will) được (be) sử dụng (used) để (to) xây dựng (build) Giao diện (UI) Bảng điều khiển (Control Plane).

## 1.1 Technology Choice (Lựa chọn Công nghệ)

* **Công nghệ (Technology):** `Streamlit`
* **Lý do (Rationale):** Quyết định (Decision) này được (was) đưa ra (made) trong (during) `front-end-spec.md` (V1.4). `Streamlit` được (is) chọn (chosen) vì (because) nó (it) tuân thủ (respects) **NFR7 (Cost)** (Chi phí) (rẻ (cheap) tài nguyên (resources)), không (no) cần (needs) "build pipeline" (đường ống xây dựng) (build pipeline) phức tạp (complex), và (and) tích hợp (integrates) mượt mà (seamlessly) với (with) `core_lib` (lõi thư viện) (Python) của (of) chúng ta (our).

## 1.2 Architectural Context (Bối cảnh Kiến trúc)

Kiến trúc (Architecture) Frontend (Frontend) này (this) *không* (does not) tồn tại (exist) độc lập (in isolation). Nó (It) là (is) một (a) service (dịch vụ) (tên (named) là (is) `dash` (bảng điều khiển)) bên trong (inside) hệ thống (system) `docker-compose.yml` (V2.1.4) (docker-compose.yml (V2.1.4)) tổng thể (overall).

* Nó (It) bind (liên kết) với (to) `127.0.0.1:8501`.
* Nó (It) được (is) bảo vệ (protected) bởi (by) service (dịch vụ) `proxy` (Nginx/Caddy) (xử lý (handling) `HTTPS` (Bảo mật Lớp Vận tải) và `X-ADMIN-KEY` (Khóa Quản trị X)).

## 1.3 Change Log (Lịch sử Thay đổi)

| Date | Version | Description | Author |
| :--- | :--- | :--- | :--- |
| 2025-11-06 | 1.0 | Khởi tạo (Initial draft) từ (from) `front-end-spec.md` (V1.4) (Đặc tả Giao diện Người dùng). | Winston (Architect) |

# 2. Tech Stack (Ngăn xếp Công nghệ)
*(Trích từ FE Arch V1.0)*

## 2.1 Technology Stack Table (Bảng Ngăn xếp Công nghệ)

| Category (Hạng mục) | Technology (Công nghệ) | Version (Phiên bản) | Purpose (Mục đích) | Rationale (Lý do) |
| :--- | :--- | :--- | :--- | :--- |
| **Language (Ngôn ngữ)** | Python | 3.11+ | Ngôn ngữ (Language) chính (primary) | Nhất quán (Consistent) với (with) `api` (giao diện lập trình ứng dụng), `worker` (nhân viên), `scraper` (trình thu thập). Tích hợp (Integrates) `core_lib` (lõi thư viện). |
| **UI Framework (Khung Giao diện)** | Streamlit | 1.33+ | Công cụ (Tool) UI (Giao diện Người dùng) cốt lõi (core) (PRD 3.4). | Nhanh (Fast), "rẻ" (cheap) (NFR7), không (no) cần (needs) build (xây dựng) (PRD 4.1). |
| **Charting (Biểu đồ)** | Plotly | 5.x | Biểu đồ (Charts) tương tác (interactive) (cho (for) `02_AL` (Học Chủ động 02), `04_Health` (Sức khỏe 04)). | Hỗ trợ (Supports) zoom/hover (phóng to/di chuột) (UX Spec 4.2) (tốt (better) hơn (than) `st.line_chart` (biểu đồ đường st) tĩnh (static)). |
| **Data Handling (Xử lý Dữ liệu)** | Pandas | 2.x | Xử lý (Handling) DataFrames (Khung Dữ liệu) (từ (from) API (Giao diện Lập trình ứng dụng)) để (to) vẽ (plot) biểu đồ (chart). | Tích hợp (Integration) gốc (native) tốt (good) với (with) Streamlit. |
| **State Management (Quản lý Trạng thái)**| Streamlit Session State | N/A | Quản lý (Manage) trạng thái (state) (ví dụ: `admin_key_ok`, `sts_done_today`). | Tích hợp (Built-in). |
| **HTTP Client (Máy khách HTTP)** | `httpx` | 0.27+ | Gọi (Calling) các (the) endpoint (điểm cuối) `AppAPI` (Giao diện Lập trình ứng dụng) (127.0.0.1:8000). | Hiện đại (Modern), hỗ trợ (supports) async (bất đồng bộ), nhất quán (consistent) với (with) FastAPI. |
| **UI Testing (Kiểm thử UI)** | `pytest` / `Playwright` | N/A | Unit test (Kiểm thử đơn vị) (logic) / E2E test (Kiểm thử Đầu cuối) (render (kết xuất)) | `pytest` (Kiểm thử py) cho (for) logic (logic), `Playwright` (Nhà viết kịch) cho (for) smoke test (kiểm thử khói) CI (Tích hợp Liên tục). |

# 3. Structure (Cấu trúc)
*(Trích từ FE Arch V1.0)*

## 3.1 Source Tree (Cấu trúc Thư mục)

Cấu trúc (Structure) thư mục (directory) cho (for) `src/dash` (bảng điều khiển src) (service (dịch vụ) Streamlit) sẽ (will) tuân thủ (follow) (follows) mẫu (pattern) "multi-page app" (ứng dụng đa trang) của (of) Streamlit (như (as) đã (already) định nghĩa (defined) trong (in) `architecture.md` (V2.1.4) (Phần 8)):

```plaintext
src/
└── dash/                 # (Service (Dịch vụ) `dash` (bảng điều khiển))
    ├── pyproject.toml    # (Deps (Phụ thuộc): streamlit, plotly, httpx, core-lib)
    ├── app.py            # (Entrypoint (Điểm vào) chính (main) - Chứa `auth_gate` (cổng xác thực))
    └── pages/
        ├── 01_Overview.py
        ├── 02_Active_Learning.py
        ├── 03_Model_Registry.py
        └── 04_System_Health.py


## 3.2 State Management (Quản lý Trạng thái)

Công cụ (Tool): Streamlit Session State (Trạng thái Phiên Streamlit) (st.session_state).

Lý do (Rationale): Cơ chế (Mechanism) gốc (native) của (of) Streamlit. Đủ (Sufficient) cho (for) các (the) nhu cầu (needs) "Single-User" (Một người dùng) của (of) chúng ta (our).

Cách sử dụng (Usage): st.session_state["admin_key_ok"] (trạng thái phiên["admin_key_ok"]) (Auth (Xác thực)), st.session_state[f"sts_done_{date}"] (trạng thái phiên[f"sts_done_{ngày}"]) (STS (Chỉ số Tin cậy Tín hiệu) 1 lần/ngày), st.session_state["al_item_hidden"] (trạng thái phiên["al_item_hidden"]) (Ẩn (Hide) các (the) mẫu (samples) AL (Học Chủ động) đã (already) xử lý (processed)).

## 3.3 Data Fetching (Lấy Dữ liệu)

Mô hình (Pattern): API-driven (Điều khiển bằng API).
Kiến trúc (Architecture): dash (Bảng điều khiển) (Streamlit) KHÔNG (DOES NOT) kết nối (connect) trực tiếp (directly) với (to) db (cơ sở dữ liệu).
Luồng (Flow): dash (Bảng điều khiển) (Client (Máy khách)) $\rightarrow$ proxy (Proxy) (Nginx/Caddy) $\rightarrow$ api (Giao diện Lập trình ứng dụng) (FastAPI) (Server (Máy chủ)).
Caching (Lưu đệm): Các (The) lệnh (commands) gọi (call) API (Giao diện Lập trình ứng dụng) (bên trong (inside) Streamlit) sẽ (will) sử dụng (use) @st.cache_data (bộ đệm dữ liệu st) (với (with) TTL (Thời gian sống) ngắn (short), ví dụ: 60 giây) để (to) giảm (reduce) tải (load) cho (for) AppAPI (Giao diện Lập trình ứng dụng).

```markdown
# 4. Business Logic (Logic Nghiệp vụ)
*(Trích từ FE Arch V1.0)*

## 4.1 Business Logic Location (Vị trí Logic Nghiệp vụ)

* **Quyết định (Decision):** (BẮT BUỘC (MANDATORY)) **KHÔNG (NO)** logic (logic) nghiệp vụ (business) *phức tạp* (complex) nào (at all) được (be) phép (allowed) tồn tại (to exist) bên trong (inside) `dash` (bảng điều khiển) (Streamlit) container (thùng chứa).
* **Lý do (Rationale):**
    1.  **Tuân thủ (Compliance) NFR7 (Cost (Chi phí)):** Service (Dịch vụ) `dash` (bảng điều khiển) được (is) cấp (allocated) rất (very) ít (low) RAM (RAM) ($\le$ 256MB).
    2.  **Đơn Trách nhiệm (Single Responsibility):** `dash` (Bảng điều khiển) là (is) "lớp hiển thị" (Presentation Layer). `AppAPI` (Giao diện Lập trình ứng dụng) / `AppWK` (Nhân viên App) là (are) "lớp nghiệp vụ" (Business Logic Layer).
* **Luồng (Flow):** `dash` (Bảng điều khiển) (Streamlit) gọi (calls) `POST /al/label` (Đăng /al/label). `AppAPI` (Giao diện Lập trình ứng dụng) (FastAPI) nhận (receives) và (and) thực thi (executes) logic (logic) (ghi (writes) vào (into) `labels_golden` (nhãn vàng)).

## 4.2 Error Handling (Xử lý Lỗi)

* **Kiến trúc (Architecture):** `dash` (Bảng điều khiển) (Streamlit) được (is) coi (considered) là (is) một (a) "client (máy khách) câm" (dumb client).
* **Luồng (Flow) (Spec 4.2) (Đặc tả 4.2):**
    1.  `dash` (Bảng điều khiển) gọi (calls) `GET /predict` (dự đoán).
    2.  `AppAPI` (Giao diện Lập trình ứng dụng) kiểm tra (checks) `KILL_SWITCH` (Nút dừng) và (and) `data_freshness` (độ mới của dữ liệu).
    3.  Nếu (If) có (a) vấn đề (problem), `AppAPI` (Giao diện Lập trình ứng dụng) trả về (returns) `503` (Lỗi 503) (với (with) `reason` (lý do) và (and) `retry_after_hint` (gợi ý thử lại sau)) (PRD 5.1/AC6).
    4.  `dash` (Bảng điều khiển) (Streamlit) BẮT BUỘC (MUST) phải (must) bắt (catch) lỗi (error) này (this) (ví dụ: `httpx.HTTPStatusError` (Lỗi Trạng thái HTTP httpx)) và (and) hiển thị (render) `st.error` (lỗi st) (hoặc (or) `safety_banner` (Biểu ngữ An toàn) (5.2)).


# 5. Security (Bảo mật)
*(Trích từ FE Arch V1.0)*

## 5.1 Authentication (Xác thực)

* **Mô hình (Model):** "Single-User" (Một người dùng) (Admin).
* **Quyết định Kiến trúc (Architectural Decision):** (BẮT BUỘC (MANDATORY)) `Streamlit` (Streamlit) (service (dịch vụ) `dash` (bảng điều khiển)) **KHÔNG (DOES NOT)** xử lý (handle) xác thực (authentication).
* **Luồng (Flow) (Arch 9.0) (Kiến trúc 9.0):**
    1.  `dash` (Bảng điều khiển) (Streamlit) bind (liên kết) với (to) `127.0.0.1:8501`.
    2.  Service (Dịch vụ) `proxy` (Nginx/Caddy) (chạy (running) ở (on) cổng (port) 443) là (is) "cổng" (gate) duy nhất (only).
    3.  `Proxy` (Proxy) BẮT BUỘC (MUST) phải (must) thực thi (enforce) `Basic Auth` (Xác thực Cơ bản) (hoặc (or) `X-ADMIN-KEY` (Khóa Quản trị X)) cho (for) *tất cả* (all) các (the) truy cập (access) vào (to) `dash:8501` (bảng điều khiển:8501).
* **Dự phòng (Fallback):** Thành phần (Component) `auth_gate` (Cổng Xác thực) (Spec 5.1) (Đặc tả 5.1) (bên trong (inside) Streamlit) chỉ (only) là (is) "lan can" (guardrail) dự phòng (backup) cuối cùng (last-line).

## 5.2 Authorization (Phân quyền)

* **Mô hình (Model):** "Single-User" (Một người dùng) (Admin).
* **Quyết định (Decision):** (Đã (Already) "Thu hẹp Phạm vi" (De-scoped)). Không (No) có (is) logic (logic) "phân quyền" (authorization). Người dùng (User) duy nhất (single) (đã (already) được (been) xác thực (authenticated) bởi (by) `Proxy` (Proxy)) có (has) quyền (access) truy cập (access) vào (to) tất cả (all) 4 màn hình (screens).

## 5.3 Input Validation (Xác thực Đầu vào)

* **Quyết định (Decision):** (BẮT BUỘC (MANDATORY)) `dash` (Bảng điều khiển) (Streamlit) **KHÔNG (DOES NOT)** thực hiện (perform) xác thực (validation) logic (logic) nghiệp vụ (business).
* **Luồng (Flow):** `dash` (Bảng điều khiển) $\rightarrow$ `POST /al/label` (Đăng /al/label) $\rightarrow$ `AppAPI` (Giao diện Lập trình ứng dụng) (Pydantic) (Pydantic) $\rightarrow$ `422` (422) (Nếu (If) lỗi (Error)).

## 5.4 CORS (Chia sẻ Tài nguyên Giữa các Nguồn gốc)

* **Quyết định (Decision):** (BẮT BUỘC (MANDATORY)) CORS (Chia sẻ Tài nguyên Giữa các Nguồn gốc) phải (must) được (be) **TẮT (OFF)** ở (at) cấp độ (level) `Proxy` (Proxy) (Nginx/Caddy).


# 6. Accessibility (Khả năng Tiếp cận)
*(Trích từ FE Arch V1.0)*

## 6.1 Standards (Tiêu chuẩn)

* **Kiến trúc (Architecture):** Hệ thống (System) sẽ (will) tuân thủ (adhere to) các (the) nguyên tắc (principles) **WCAG AA** ở (at) mức (level) "nỗ lực (effort) tốt nhất" (best-effort) trong (within) các (the) ràng buộc (constraints) của (of) `Streamlit`.
* **Rủi ro (Risk):** `Streamlit` (Streamlit) (framework (khung) "cấp cao" (high-level)) có thể (may) không (not) cho phép (allow) kiểm soát (control) chi tiết (granular) (ví dụ: `aria-labels` (nhãn aria)) trên (on) mọi (every) thành phần (component).

## 6.2 Implementation (Triển khai)

* **Font Size (Cỡ chữ) (PRD Persona V1.3):** Cỡ (Size) chữ (Font) cơ bản (base) BẮT BUỘC (MUST) phải (must) được (be) cấu hình (configured) $\ge$ `14px` (14 điểm ảnh) (ví dụ: qua (via) `config.toml` (cấu hình.toml) của (of) Streamlit).
* **Color Contrast (Độ tương phản Màu sắc) (Spec 6.2):** Các (The) màu (colors) chủ đề (theme) (Xanh (Green)/Vàng (Yellow)/Đỏ (Red) cho (for) `regime_chart` (Biểu đồ Chế độ)) BẮT BUỘC (MUST) phải (must) được (be) kiểm tra (checked) (checked) để (to) đạt (meet) `WCAG AA` (WCAG AA).
* **Keyboard Navigation (Điều hướng bằng Bàn phím) (Spec 6.3):**
    * Logic (Logic) trong (in) `02_Active_Learning.py` BẮT BUỘC (MUST) phải (must) triển khai (implement) phím tắt (hotkeys) (`C/F/S`).
    * Các (The) nút (buttons) (`Confirm`/`Flip`/`Snooze`) (Xác nhận/Lật/Báo lại) phải (must) có (have) `tooltips` (chú giải công cụ) hiển thị (displaying) các (the) phím tắt (hotkeys) (`(C)`, `(F)`, `(S)`) một cách (visually) rõ ràng.


# 7. Localization (Bản địa hóa)
*(Trích từ FE Arch V1.0)*

## 7.1 Default Language (Ngôn ngữ Mặc định)

* **Ngôn ngữ (Language):** **Tiếng Việt (vi)**.
* **Quyết định (Decision):** Toàn bộ (The entire) Giao diện (UI) (các (the) nút (buttons), tiêu đề (titles), `plain_explainer` (giải thích đơn giản), v.v.) BẮT BUỘC (MUST) phải (must) được (be) viết (written) bằng (in) tiếng Việt (Vietnamese).

## 7.2 Internationalization (i18n) (Quốc tế hóa)

* **Phạm vi (Scope):** **Ngoài Phạm vi (Out of Scope)** cho (for) MVP (Sản phẩm Khả thi Tối thiểu).
* **Hành động (Action):** Chúng ta (We) sẽ (will) **mã hóa cứng (hardcode)** các (the) chuỗi (strings) tiếng Việt (Vietnamese) trực tiếp (directly) vào (into) code (mã) Streamlit. Chúng ta (We) sẽ (will) *không* (not) triển khai (implement) một (a) hệ thống (system) i18n (ví dụ: `gettext` (lấy văn bản) hoặc (or) các (the) tệp (files) JSON (JSON) ngôn ngữ (language)) để (to) tuân thủ (respect) NFR7 (Cost (Chi phí) / Tinh gọn (Lean)).


# 8. Deployment (Triển khai)
*(Trích từ FE Arch V1.0)*

## 8.1 Deployment Strategy (Chiến lược Triển khai)

* **Môi trường (Environment):** VPS (Máy chủ riêng ảo) 10-15 đô (NFR7).
* **Công cụ (Tool):** `Docker Compose` (Docker Compose).
* **Kiến trúc (Architecture):** Service (Dịch vụ) `dash` (bảng điều khiển) (Streamlit) là (is) một (a) trong (of) 6 services (dịch vụ) (cộng (plus) `profiles` (hồ sơ)) trong (in) file (tệp) `docker-compose.yml` (V2.1.4) (docker-compose.yml (V2.1.4)).
* **Luồng (Flow):** `User` (Người dùng) $\rightarrow$ `Proxy (Caddy)` (Proxy (Caddy)) (Cổng (Port) 443, xử lý (handles) `HTTPS` (Bảo mật Lớp Vận tải) + `Auth` (Xác thực)) $\rightarrow$ `dash:8501` (bảng điều khiển:8501) (127.0.0.1).

## 8.2 CI/CD Pipeline (Đường ống CI/CD)

* **Công cụ (Tool):** `GitHub Actions` (Hành động GitHub) (PRD Story 0.4).
* **Các bước (Steps) (Liên quan (Related) đến (to) `dash` (bảng điều khiển)):**
    1.  **On Commit (Khi Cam kết):**
    2.  `Lint` (Kiểm tra lỗi) (Ruff/Black) (AC3, 0.4).
    3.  `Build` (Xây dựng) (docker compose build dash) (AC2, 0.4) (Kiểm tra (check) `pyproject.toml` (pyproject.toml) hợp lệ (valid)).
    4.  `Test` (Kiểm thử) (pytest src/dash/tests/) (AC4, 0.4).
    5.  `Smoke Test` (Kiểm thử Khói) (Spin up (Khởi động) compose (soạn thảo), `curl /readyz` (cuộn /sẵn sàng z)) (AC7, 0.4).
* **Triển khai (Deployment):** (Hiện tại (Currently)) là (is) **Thủ công (Manual)**. (Admin/User (Quản trị viên/Người dùng) `ssh` (Giao thức Vỏ bảo mật) vào (into) VPS (Máy chủ riêng ảo), `git pull` (kéo git), `docker compose up -d --build`) (docker compose up -d --build).

## 8.3 Environment Configuration (Cấu hình Môi trường)

* **Secrets (Bí mật):** Service (Dịch vụ) `dash` (bảng điều khiển) BẮT BUỘC (MUST) phải (must) đọc (read) toàn bộ (all) cấu hình (configuration) từ (from) file (tệp) `.env` (môi trường) (đã (already) được (been) nạp (loaded) bởi (by) `docker-compose` (soạn thảo docker)).
* **Các biến (Variables) chính (Key) (cho (for) `dash` (bảng điều khiển)):** `ADMIN_KEY` (Khóa Quản trị), `API_BASE_URL="http://api:8000"`.
* **Log Rotation (Xoay vòng Log):** (BẮT BUỘC (MANDATORY)) `docker-compose.yml` (V2.1.4) (docker-compose.yml (V2.1.4)) BẮT BUỘC (MUST) phải (must) cấu hình (configure) `logging driver` (trình điều khiển ghi log) (ví dụ: `max-size: "100m", max-file: "5"`) (kích thước tối đa: "100m", tệp tối đa: "5") cho (for) service (dịch vụ) `dash` (bảng điều khiển) (NFR8).