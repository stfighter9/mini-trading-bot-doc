# Đặc tả UI/UX (Front‑end Spec) — **Stock Hunter AI**  
**Phiên bản V1.4 — Đã “Siết Ốc” vận hành**

> Tôn chỉ UX: **Maximum Efficacy, Minimum Interaction** — buồng lái tối giản cho Admin/Trader đơn lẻ, ưu tiên trả lời nhanh, thao tác ít, chống FOMO.

---

## Mục lục
1. [Interface Overview (Tổng quan về Giao diện)](#interface-overview-tổng-quan-về-giao-diện)  
2. [User Personas (Chân dung Người dùng) — V1.4](#user-personas-chân-dung-người-dùng--v14)  
3. [User Flows (Luồng Người dùng) — V1.2](#user-flows-luồng-người-dùng--v12)  
4. [Wireframes (Khung sườn) — V1.4](#wireframes-khung-sườn--v14)  
5. [Component Specification (Đặc tả Thành phần) — V1.1](#component-specification-đặc-tả-thành-phần--v11)  
6. [Accessibility (Khả năng Tiếp cận)](#accessibility-khả-năng-tiếp-cận)  
7. [Localization (Bản địa hóa)](#localization-bản-địa-hóa)  
8. [Security & Privacy (Bảo mật & Quyền riêng tư)](#security--privacy-bảo-mật--quyền-riêng-tư)  
9. [Deployment (Triển khai)](#deployment-triển-khai)

---

## Interface Overview (Tổng quan về Giao diện)

### 1.1 Project Summary (Tóm tắt Dự án)
Tài liệu này định nghĩa các yêu cầu UI/UX cho **Stock Hunter AI**. Mục tiêu là tạo **Control Plane** bằng **Streamlit** cho **Single‑User** (Admin/Trader). Trọng tâm: **Hiệu quả tối đa, tương tác tối thiểu**. Giao diện như **buồng lái** để:  
1) **Monitor** tín hiệu, 2) **Teach** AI qua Active Learning, 3) **Hand‑off** ngữ cảnh cho công cụ AI ngoài (ví dụ: Gemini).

### 1.2 Target Platform (Nền tảng Mục tiêu)
- **Công nghệ:** Streamlit  
- **Thiết bị:** Tối ưu Desktop  
- **Kiến trúc:** UI chạy trong container dashboard riêng, bind `127.0.0.1`  
- **Bảo mật:** Truy cập bảo vệ ở lớp **Proxy** (Nginx/Caddy) thông qua **X‑ADMIN‑KEY** hoặc **Basic Auth**

### 1.3 Change Log (Lịch sử Thay đổi)
| Date       | Version | Description                                                                                          | Author   |
|------------|---------|------------------------------------------------------------------------------------------------------|----------|
| 2025-11-06 | 1.4     | Hoàn thiện Spec với Persona V1.4, đã vá lỗi “macro‑lite”.                                           | Sally (UX) |

---

## User Personas (Chân dung Người dùng) — V1.4 — “Đã Siết Ốc”

### Persona v1.4 — “David” (Solo Admin/Trader, non tay, học nhanh)

**1) Snapshot**  
- **Vai trò:** 1 người, kiêm Admin (vận hành VPS) + Trader (dùng signal)  
- **Kỹ thuật:** VPS/Docker/.env OK  
- **Trading:** Novice‑to‑Medium; skill 1–2/3 (biết MA/RSI, đang học risk/reward), dễ nhiễu do **FOMO**  
- **Vốn & Thị trường (giả định):** < 50k USD; ưu tiên VN30/Mid‑cap  
- **Risk Appetite:** Medium‑Low; **max_drawdown chấp nhận < 15%**  
- **Thiết bị:** Desktop/Laptop (chính)  
- **Di động:** View‑only. Chỉ hiển thị **Regime** + **Safety Banner**. **Không** có Active Learning/Kill‑Switch  
- **Thời gian:** 3–5 phút/tuần cho AL; 1–2 lần/ngày lướt signal (chủ yếu pre‑market)  
- **Ngôn ngữ:** Tiếng Việt (mặc định)

**2) Jobs To Be Done (JTBD)**  
- **Regime rõ ràng** khi thị trường ồn ào để tránh panic mis‑click  
- Trước khi hành động, cần **short explainer** + **AI Hand‑off pack** để hỏi thêm Gemini/ChatGPT **(verify)**  
- Hàng tuần muốn **teach nhanh 5–10 samples** để giảm mơ hồ mô hình  
- Khi signal **lạ**, cần **Kill‑Switch** + **Health** để biết khi nào **không** nên tin

**3) Nhu cầu UX chi tiết**  
- **Focus on “The Answer”:** Regime + Confidence + 3 bullet **Do/Don’t**  
- **Guardrails cho Newbie:** Beginner Mode, 3‑step checklist, AL sandbox, **2‑step confirm** cho Kill‑Switch  
- **Learn with AI:** Button **Copy MD/JSON**, **Download AI Pack** (có privacy_note), **What‑if presets** “rẻ”  
- **Reduce Noise:** Hiển thị **Macro‑lite compressed** chỉ 1 badge (vd: *Macro: Neutral* / *Macro: Rate Alert*)  
- **Anti‑Patterns:** Không scoreboard, không popup triền miên, không plain explainer > 2 câu  
- **FOMO Guard:** Nếu Regime = **Distribution**, hiển thị banner **Off‑plan** (vd: “Cảnh báo: Không phù hợp mở vị thế mới”)  
- **Accessibility:** Font ≥ 14px, contrast ≥ WCAG AA, hotkeys **C/F/S** hiển thị rõ ràng  
- **Rescue UX:** Khi **DEGRADED**/**KILL**, UI đề xuất **hành vi mặc định**: “Stay out/Reduce position 50%”

**4) Acceptance Signals (KPIs)**  
| KPI | Mục tiêu |
|-----|----------|
| **STS** trung bình ≥ **4/5** sau 2 tuần |  
| **AL completion** ≥ **70%/tuần** với thời gian < 5 phút |  
| **Honeypot pass rate** không giảm |  
| **STS_reason coverage** ≥ **80%** |  
| **Off‑plan action rate** < **10%** |

---

## User Flows (Luồng Người dùng) — V1.2 (đã vá)

### 3.1 Luồng 1: Sử dụng hằng ngày (Overview + AI Hand‑off) — 1–2 phút/ngày

```mermaid
graph TD
    A[Bắt đầu] --> B(Mở Dashboard);
    B --> C{Auth qua Proxy OK?};
    C -- Sai --> D[Hiển thị lỗi 401];
    C -- Đúng --> E(Tải 01_Overview);
    E --> F{Gọi /readyz OK? (Stale/Kill)};
    F -- Lỗi (503) --> G[Rescue UX: "Hệ thống đang bảo trì"];
    F -- Sẵn sàng --> H{BEGINNER_MODE?};
    H -- True --> I[Show Guided Checklist (3 bước)];
    H -- False --> J[Show Signal + Macro-lite Badge];
    I --> J;
    J --> K[AI Hand-off Panel];
    K --> L{User Copy/Download?};
    L -- Có --> M[GET /export/context|pack];
    M --> N{503 (Stale/Kill)?};
    N -- Có --> O[Error: "Pack tạm khoá"];
    N -- Không --> P[Copy/Download (log ai_copy_/ai_pack_dl)];
    L -- Không --> Q[Survey STS (1 lần/ngày)];
    O --> Q;
    P --> Q;
    Q --> Z[Kết thúc];
```

### 3.2 Luồng 2: Active Learning (Hàng tuần) — 3–5 phút/tuần

```mermaid
graph TD
    A[Bắt đầu] --> B(Mở 02_Active_Learning);
    B --> C(Tải Queue: GET /al/queue);
    C --> D[Show Progress & "Honeypot Quality: On Track"];
    D --> E{Sandbox Mode?};
    E -- Có --> F[set is_sandbox=true];
    E -- Không --> G[set is_sandbox=false];
    F & G --> H[Show Sample #1 (as-of guard)];
    H --> I[Show suggested_label];
    I --> J[Reason Pill & Micro-Tip];
    J --> K{User nhấn C/F/S};
    K -- Confirm/Flip --> L[POST /al/label {action:"confirm/flip"}];
    K -- Snooze --> M[POST /al/label {action:"snooze"}];
    L --> N[st.success("Đã ghi nhận!")];
    M --> N;
    N --> O[Ẩn Sample, log al_decide, auto advance];
    O --> P{Còn samples?};
    P -- Có --> H;
    P -- Không --> Z[Show "Complete!"];
```

### 3.3 Luồng 3: Quản lý Mô hình (Có Auto‑Rollback)

```mermaid
flowchart TD
    subgraph Manual Flow
        A[Bắt đầu] --> B(Mở 03_Model_Registry);
        B --> C(Load model_registry);
        C --> D[Show "Challenger" (promotion_suggestion='ready_for_canary')];
        D --> E[User: Approve Canary];
        E --> F[POST /admin/model/activate (mode=canary)];
        F --> G[is_active=canary];
        G --> H[Log model_promotion_history];
        H --> Z[Kết thúc];
    end

    subgraph Automatic Flow (Epic 4)
        I[Cron: monitor_canary_batch (hourly)] --> J{Guardrail Breach? (Drawdown/IC_gap)};
        J -- Không --> K[Không làm gì];
        J -- Có --> L[Auto-Rollback BẮT BUỘC];
        L --> M[SET is_active=false (canary)];
        M --> N[SET is_active=true (model cũ)];
        N --> O[Log promotion_history (actor="system")];
        O --> P[Fire Alert];
    end
```

---

## Wireframes (Khung sườn) — V1.4

### 4.1 `01_Overview.py` (Màn hình Chính) — Layout: `wide`

**Sidebar — AI Hand‑off Panel**

```python
st.caption(f"As-of: {utc_time} (UTC)")
st.caption(f"Model: {model_ver} | Status: {safety_banner}")

with st.expander("🔬 Nghiên cứu Sâu (Bằng chứng)"):
    st.caption("Charts tuân thủ as-of, pre-calculated.")
    st.plotly_chart(sma_chart)     # TA: Price vs SMA20/50
    st.plotly_chart(sa_chart)      # SA: hype_kol vs vol_norm
    st.plotly_chart(macro_chart)   # Macro: CPI_yoy vs Policy Rate

st.subheader("Học cùng Gemini")
st.download_button("Copy Prompt cho Gemini (MD)", data=prompt_md, file_name="prompt_gemini.md")
st.download_button("Tải Gói Dữ liệu (Full Data Pack) (.zip)", data=zip_bytes, file_name="ai_pack.zip")

with st.expander("Prompt Cheat-Sheet"):
    st.markdown(cheatsheet_md)
```

**Main Area**

```python
# Safety
st.error("KILL-SWITCH ON")   # hoặc st.warning("DEGRADED")
if st.toggle("KILL-SWITCH"):
    confirm_two_steps()

# Top metrics
c1, c2, c3 = st.columns(3)
c1.metric("Chế độ (Regime)", "TÍCH LŨY", delta_color="normal")
c2.metric("Độ tin cậy (Confidence)", "MEDIUM")
c3.badge("Macro-lite: Neutral")

# Regime background chart (shaded) + legend
st.plotly_chart(regime_chart)
st.info("Plain Explainer: ...")

# What this means (3 bullets Do/Don't)
st.markdown(what_this_means_card_md)

# STS survey (1 lần/ngày)
st.radio("STS hôm nay?", ["1","2","3","4","5"], horizontal=True)
```

### 4.2 `02_Active_Learning.py` — Layout: `wide`

```python
st.title("Active Learning Review")
c1, c2, c3 = st.columns(3)
sandbox = c1.toggle("Sandbox Mode")
c2.progress(completed_count / 10)
c3.caption("Review Quality: On Track")

st.info("Honeypots có thể xuất hiện để đảm bảo chất lượng.")

with st.container(border=True):
    st.subheader(f"{symbol} @ {effective_date}")
    st.badge("Reason: Entropy High")
    st.caption("Micro-tip: ...")

    with st.expander("Hiển thị Bằng chứng (SMA, Hype, v.v.)"):
        st.plotly_chart(sma_chart)  # clipped as-of
        st.plotly_chart(sa_chart)   # clipped as-of

    st.info("Gợi ý hệ thống: TÍCH LŨY")
    b1, b2, b3 = st.columns(3)
    b1.button("Confirm (C)")
    b2.button("Flip (F)")
    b3.button("Snooze (S)")
```

### 4.3 `03_Model_Registry.py` & `04_System_Health.py` (mô tả cấp cao)

- **03_Model_Registry:** Hiển thị `st.dataframe(model_registry)`; `st.selectbox` chọn `model_version`; các nút **Approve Canary**/**Promote** gọi `POST /admin/model/activate`.
- **04_System_Health:** Hiển thị các **st.plotly_chart** (Macro & Performance pre‑calculated) và trạng thái **Scraper**.

---

## Component Specification (Đặc tả Thành phần) — V1.1

### 5.1 `auth_gate` (Cổng Xác thực)
- **Mô tả:** Hàm `check_auth()` bắt buộc ở đầu mỗi page  
- **Hành vi:** Ưu tiên header **X‑ADMIN‑KEY**, fallback `st.text_input` với **brute‑force lock 3 lần**; lưu `st.session_state` TTL **8 giờ**  
- **Telemetry:** `log_event("auth_attempt", …)`

### 5.2 `safety_banner` (Biểu ngữ An toàn)
- **Mô tả:** Hiển thị 1 trong 4 trạng thái
- **Logic & Mapping:**  
  - **KILL →** `st.error`  
  - **DEGRADED** hoặc **Stale > 24h →** `st.warning`  
  - **CANARY →** `st.info`  
- **Telemetry:** `log_event("safety_banner_view", …)`

### 5.3 `regime_chart` (Biểu đồ Chế độ)
- **Mô tả:** Chart chính tại `01_Overview.py`
- **Hành vi:** `st.plotly_chart` với Zoom/Hover; **as‑of guard** vẽ **đường đỏ as_of_cutoff**, disable hover sau đó; **downsample ≤ 800 điểm**; **Legend bắt buộc** (Xanh = **Accumulation**)  
- **Telemetry:** `log_event("regime_chart_rendered", …)`

### 5.4 `ai_handoff_panel` (Bảng điều khiển Giao tay AI)
- **Mô tả:** Toàn bộ `st.sidebar`
- **Hành vi:** Gọi **GET `/export/context`** và **GET `/export/pack`**; Button **Copy** dùng `st.download_button`; **Expander** “Deep Research” lazy‑load charts (TA/SA/Macro)
- **Telemetry:** `log_event("ai_handoff_copy", …)`, `log_event("ai_handoff_download", …)`

### 5.5 `al_sample_card` (Thẻ Mẫu AL)
- **Mô tả:** Thành phần core cho vòng lặp AL
- **Hành vi:** Header hiển thị **symbol**, **effective_date**, **Reason Pill**; Evidence Expander lazy‑load **plotly** (Price+SMA, 1 feature). **As‑of guard**: chart **MUST** clip tại `effective_date`. Actions: **Confirm/Flip/Snooze** với hotkeys **C/F/S** hiển thị. **API**: mọi action **MUST** gọi `POST /al/label` với `idempotency_key`. Feedback: thông báo **trung lập** “Đã ghi nhận!”  
- **Telemetry:** `log_event("al_decide", …)`

---

## Accessibility (Khả năng Tiếp cận)
- **Tiêu chuẩn:** Best‑effort **WCAG AA** trong ràng buộc Streamlit  
- **Font:** **BẮT BUỘC** base font ≥ **14px**  
- **Color Contrast:** **BẮT BUỘC** xanh/vàng/đỏ đạt **WCAG AA**  
- **Keyboard:** **BẮT BUỘC** `02_Active_Learning` hỗ trợ hotkeys **C/F/S**

---

## Localization (Bản địa hóa)
- **Ngôn ngữ:** Tiếng Việt (`vi`)  
- **Quyết định:** **Hardcode** chuỗi tiếng Việt; **i18n Out‑of‑Scope** (tuân thủ **NFR7: Cost**)

---

## Security & Privacy (Bảo mật & Quyền riêng tư)
- **Authentication (BẮT BUỘC):** xử lý 100% tại **Proxy** (Nginx/Caddy) với **Basic Auth / X‑ADMIN‑KEY** (PRD 5.2/AC3)  
- **Authorization:** Single‑User (Admin). **Không có** Role Gate bên trong Streamlit  
- **Privacy (Hand‑off):** **AI Pack MUST sanitize data** (**NO full text**) theo **NFR6**

---

## Deployment (Triển khai)
- **Công cụ:** **Docker Compose** (profiles: `backtest`; log rotation `100MB x 5`)  
- **Triển khai:** Thủ công: `git pull` → `docker compose up -d --build`  
- **Cấu hình:**  
  ```env
  API_BASE_URL="http://api:8000"  # Dash gọi API qua mạng nội bộ Docker
  ```

---

### Phụ lục
- **Hotkeys:** `C` = Confirm, `F` = Flip, `S` = Snooze  
- **Badges/Status:** Macro-lite badge tối đa 1 trạng thái/khung thời gian  
- **FOMO Guard:** Khi Regime = Distribution, bật banner **Off‑plan** chặn mở vị thế mới
