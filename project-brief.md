# Project Brief: Hệ thống AI Săn Mã Chứng khoán (Stock Hunter AI) — **V2.5 (Bản cuối cùng)**
**Facilitator:** Mary (Business Analyst)

> Mục tiêu: xây dựng hệ thống hỗ trợ ra quyết định cho trader cá nhân sành sỏi bằng AI tự học, giảm nhiễu, tăng độ tin cậy khi vào lệnh.

---

## Mục lục
1. [Executive Summary (Tóm tắt Điều hành)](#executive-summary-tóm-tắt-điều-hành)  
2. [Problem Statement (Tuyên bố Vấn đề)](#problem-statement-tuyên-bố-vấn-đề)  
3. [Proposed Solution (Giải pháp Đề xuất)](#proposed-solution-giải-pháp-đề-xuất)  
4. [Target Users (Người dùng Mục tiêu)](#target-users-người-dùng-mục-tiêu)  
5. [Goals & Success Metrics (Mục tiêu & Chỉ số Thành công)](#goals--success-metrics-mục-tiêu--chỉ-số-thành-công)  
6. [MVP Scope (Phạm vi MVP)](#mvp-scope-phạm-vi-mvp)  
7. [Post-MVP Vision (Tầm nhìn sau MVP)](#post-mvp-vision-tầm-nhìn-sau-mvp)  
8. [Technical Considerations (Cân nhắc Kỹ thuật)](#technical-considerations-cân-nhắc-kỹ-thuật)  
9. [Constraints & Assumptions (Ràng buộc & Giả định)](#constraints--assumptions-ràng-buộc--giả-định)  
10. [Risks & Open Questions (Rủi ro & Câu hỏi Mở)](#risks--open-questions-rủi-ro--câu-hỏi-mở)  
11. [Ops Requirements & Rollout (Yêu cầu Vận hành & Triển khai)](#ops-requirements--rollout-yêu-cầu-vận-hành--triển-khai)

---

## Executive Summary (Tóm tắt Điều hành)
Dự án phát triển **Stock Hunter AI**: hệ thống **adaptive AI** phân loại **chế độ thị trường (regime)** (ví dụ: *Tích lũy, Hưng phấn*) bằng cách **lọc tín hiệu tinh hoa** khỏi **tiếng ồn**. Giá trị cốt lõi: cung cấp **tín hiệu rõ ràng, đáng tin cậy**, giảm **Bull Traps**, tăng **tự tin khi vào lệnh**.

---

## Problem Statement (Tuyên bố Vấn đề)
- **Quá tải thông tin** và **tín hiệu nhiễu** là vấn đề lớn nhất.
- Nguồn nhiễu chính:
  - **Nhiễu kỹ thuật:** các chỉ báo (RSI, v.v.) sinh nhiều **false signals**.
  - **Nhiễu xã hội:** mạng xã hội và tin tức chứa **garbage news**, tin đồn, **pump/dump**.
- Hậu quả: khó phân biệt tín hiệu có giá trị (dòng tiền thông minh) và **bẫy** (tăng do FOMO) dẫn đến **thua lỗ đáng kể**.

---

## Proposed Solution (Giải pháp Đề xuất)
**Hệ thống Lọc Tín hiệu Tinh hoa (Elitist Signal Filtering System)** — công cụ **cộng tác**, không phải “hộp đen”. Người dùng có thể **huấn luyện và tinh chỉnh**.

**Năng lực cốt lõi:**
1. **Phân loại chế độ (Regime Classification):** đầu ra đơn giản, rõ ràng (*Tích lũy, Bùng nổ*, …).  
2. **Lọc tín hiệu (Signal Filtering):** tự động phân biệt **nguồn tinh hoa (đã kiểm chứng)** với **đám đông ồn ào**.  
3. **Vòng lặp phản hồi (Feedback Loop):** **Active Learning** để người dùng liên tục “dạy” AI.

**Kế hoạch dự phòng (Fallback Plan) — Patch B:**
- **MVP** gồm cơ chế **A/B test tín hiệu**: chạy song song (1) **Elitist (có trọng số)** và (2) **Baseline (không trọng số)**.  
- **Switching Rule** rẻ tiền: tự động **giảm trọng số** của Elitist nếu **kém Baseline**.

---

## Target Users (Người dùng Mục tiêu)
- **Phân khúc chính:** Trader cá nhân **kỹ thuật cao**.  
- **Chân dung:** Nghiêm túc, coi trọng dữ liệu. Không tìm “chén thánh”, chỉ cần **edge có thể kiểm chứng**.  
- **Yêu cầu:** Chấp nhận **self-host** và đầu tư **3–5 phút/tuần** để cung cấp **feedback Active Learning**.

---

## Goals & Success Metrics (Mục tiêu & Chỉ số Thành công)
**Mục tiêu chính:** Tăng **trader confidence** bằng cách giảm **uncertainty** do nhiễu.

**KPIs:**

| Mã KPI | Mô tả | Ngưỡng thành công |
|---|---|---|
| **KPI 1 (Value)** | **Signal Trust Score (STS)** đo bằng khảo sát pop-up nhẹ 1 lần/ngày khi **dwell time ≥ 10s** | **STS ≥ 4.0** |
| **KPI 2 (Adoption)** | **AL Completion Rate** cho **micro-quota** 5–10 mẫu/tuần | **≥ 70%** |
| **KPI 3 (Efficacy)** | **A/B:** IC(Elitist) so với IC(Baseline) với **CI 95%** | **IC(Elitist) ≥ IC(Baseline)** |

---

## MVP Scope (Phạm vi MVP)
Hệ thống khép kín **“Tín hiệu → Phân loại → Phản hồi.”**

**Năng lực bắt buộc:**  
- **Ingestion:** TA, SA/News (1 nguồn ổn định), Macro (tối thiểu).  
- **Sentiment:** NLP (ví dụ **PhoBERT INT8**).  
- **Elitist Filtering:** **Dynamic Weighting** (FR4).  
- **Regime Classification:** Dự đoán **hàng ngày** (ví dụ **HMM**).  
- **Labeling:** **Auto-label** silver labels (ví dụ **Snorkel**).  
- **Learning (giảm rủi ro):** UI **Active Learning 1-chạm**, micro-quota **5–10 mẫu/tuần**.  
- **Control:** UI **Kill-Switch** và **Model Registry**.  
- **Instrumentation:** Ghi **metrics** (PSI, flip-rate) và **events** (vd `sts_submitted`).  
- **Core Screens:** Tối thiểu 3 màn hình (**Dashboard, AL Review, Health/Control**).  
- **AI Hand-off:** **AI Hand-off Panel** với **Copy MD/JSON** và **Download Pack**.

---

## Post-MVP Vision (Tầm nhìn sau MVP)
- Mở rộng sang **tài sản khác**.  
- Tích hợp **API** với **nền tảng môi giới**.  
- Bổ sung **XAI** giải thích sâu.

---

## Technical Considerations (Cân nhắc Kỹ thuật)
- **Ops Model:** **Self-hosted** (NFR7).  
- **Chi phí hạ tầng:** Chạy trên **VPS 10–15 USD/tháng** (NFR7).  
- **Cài đặt:** **Đơn giản** (ví dụ `docker-compose up`) (NFR7).  

**Spec dữ liệu tối thiểu (Patch 3):**
- **TA:** `symbol, date, OHLCV, turnover`  
- **SA/News:** `id, source, author, url, text, publisher_time, first_seen_time, entities`  
- **Macro:** `policy_rate, USDVND, CPI_yoy, as_of/effective_date`  
- **Elitist meta:** `source_type, account_id, vip_flag`

---

## Constraints & Assumptions (Ràng buộc & Giả định)
**Ràng buộc “Cứng” (nếu sai, dự án thất bại):**
1. **Self-host** được người dùng chấp nhận.  
2. **VPS 10–15 USD** khả thi về kỹ thuật.  
3. Người dùng **chấp nhận micro-review hàng tuần** (Active Learning).

**Giả định “Mềm” (có Fallback):**
- Giả thuyết **“Elitist Signals” > Baseline**. Fallback: **A/B & auto-switch** (mục 3).

---

## Risks & Open Questions (Rủi ro & Câu hỏi Mở)
**User Risk (đã giảm thiểu):**  
- Người dùng lười, không tham gia **Active Learning**.  
- **Mitigation:** Gói trong **MVP** (mục 6.6, 6.8). Thiết kế **micro-quota, 1-chạm**, thưởng gamification **tối thiểu**.

**Model Risk (đã giảm thiểu):**  
- **Elitist Signals** thất bại, còn ồn hơn đám đông.  
- **Mitigation:** **A/B test** với **Baseline** (mục 3.4).

**Resource Risk (đã giảm thiểu):**  
- **VPS 10 USD** bị **OOM** khi batch NLP/Polars.  
- **Mitigation:** (1) Chạy batch **ban đêm**, (2) **Tách tiến trình** (AppWK, AppAPI), (3) **Fail-policy** (NFR3).

**Data Risk:**  
- Nguồn dữ liệu, nhất là **SA/News**, có thể đổi **schema** hoặc **chặn truy cập**.  
- **Mitigation:** **Adapter Pattern**, **Lean-Armor**, **Canary Selector** đã có trong **architecture.md**.

---

## Ops Requirements & Rollout (Yêu cầu Vận hành & Triển khai)
**NFRs Vận hành:**  
- **p95 < 500 ms** (SLO)  
- **Uptime ≥ 99.0%/tháng**  
- **Batch < 06:00** với **Fail-policy**  
- **as-of violations = 0**

**Giao thức A/B (Patch B):**
- **Metric:** **IC T+1** (primary), **RankIC T+5** (secondary)  
- **Significance:** \|z\| ≥ 1.64 (≈ 90%)  
- **Rule:** 4 tuần bất lợi → **-50% weight**; 8 tuần → **tắt Elitist**

**Bảo mật (Patch 4):** **X-ADMIN-KEY**, **rate limit**, **audit logs**.

**Đo lường (Patch 5):** **Events** (*signal_view, sts_submitted, ai_copy_md*), **Dashboards** (*STS, AL, IC20D*).

**Triển khai (Patch 6):** **Shadow (2 tuần)** → **Canary (10–20%)** → **General**.

**Tuân thủ (Patch 9):** **Retention 60 ngày** cho **full text** không rõ **license**.

---

### Phụ lục
- Viết tài liệu, code và dashboard theo **nguyên tắc as-of**.  
- Ưu tiên **độ rõ ràng** hơn **độ phức tạp**; mọi “hộp đen” cần có **giải thích tối thiểu**.
