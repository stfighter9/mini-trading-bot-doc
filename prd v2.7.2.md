Tài liệu Yêu cầu Sản phẩm (PRD) - Stock Hunter AI

(Phiên bản V2.7.2 - Data Platform Hardening & Abstraction)

Tài liệu này đã được phân rã (sharded).

Các phần Chính

1. Mục tiêu và Bối cảnh

2. Yêu cầu (FRs/NFRs)

3. Mục tiêu UI/UX

4. Giả định Kỹ thuật

5. Danh sách Epics (Tóm tắt)

Chi tiết Epics

Epic 0: Foundation & Core Infrastructure

Epic 1: Data Platform

Epic 2: NLP & Feature Platform

Epic 3: Labeling & Active Learning

Epic 4: Model Training & A/B

Epic 5: Serving & Control Plane

Phụ lục

9. Ràng buộc & Giả định

10. Rủi ro & Câu hỏi Mở

11. Yêu cầu Vận hành & Triển khai

1. Goals and Background Context (Mục tiêu và Bối cảnh)

1.1 Goals (Mục tiêu)

Các mục tiêu nghiệp vụ (business goals) và mục tiêu người dùng (user goals) cho MVP này là:

Mục tiêu Chính: Tăng cường sự tự tin của nhà giao dịch (trader confidence) bằng cách giảm thiểu sự không chắc chắn (uncertainty) gây ra bởi "tiếng ồn" (noise).

Mục tiêu Phụ: Cải thiện kết quả giao dịch bằng cách giảm các "lỗi không đáng có" (unforced errors) – tức là mua vào khi "Hưng phấn" hoặc bán ra khi "Tích lũy".

Mục tiêu KPI (Giá trị): Đạt "Chỉ số Tin cậy Tín hiệu" (Signal Trust Score - STS) $\ge$ 4.0 (trên thang 1-5).

Mục tiêu KPI (Tương tác): Đạt Tỷ lệ Hoàn thành Active Learning (AL Completion Rate) $\ge$ 70% hàng tuần.

Mục tiêu KPI (Hiệu quả): Chứng minh IC(Elitist) $\ge$ IC(Baseline) (IC Tinh hoa $\ge$ IC Cơ bản) thông qua A/B test (với các "lan can" (guardrails) thống kê).

1.2 Background Context (Bối cảnh)

Vấn đề cốt lõi mà dự án này giải quyết là "Sự Ồn ào" (Noise) trong giao dịch. Các nhà giao dịch cá nhân hiện đang bị quá tải bởi các tín hiệu kỹ thuật giả (false signals) và thông tin "rác" (garbage) từ mạng xã hội, dẫn đến việc họ không thể phân biệt giữa một cú bùng nổ "thật" và một "Bẫy tăng giá" (Bull Trap).

Giải pháp của chúng ta là một Hệ thống Lọc Tín hiệu Tinh hoa (Elitist Signal Filtering System). Đây là một công cụ cộng tác (collaborative tool) mà người dùng (là Admin duy nhất) có thể "dạy" (thông qua Active Learning), được thiết kế để phân loại các "chế độ" (regimes) thị trường.

1.3 Change Log (Lịch sử Thay đổi)

Date

Version

Description

Author

2025-11-06

2.6.1

Sửa lỗi Tuần tự Cron: label_batch chạy sau feature_gold_batch.

John (PM)

2025-11-17

2.7

Chuyển đổi (Pivot) kiến trúc NLP sang sử dụng Gemini API (Batch/Structured).

John (PM)

2025-11-18

2.7.1

Data Platform Strategy: Bổ sung chiến lược API-First và ràng buộc pháp lý (Scraping Ethics).

John (PM)

2025-11-18

2.7.2

Data Platform Hardening: Thêm Data Source Abstraction, News Backfill, và Observability chuyên sâu cho Scraper.

John (PM)

2. Requirements (Yêu cầu)

2.1 Functional (Chức năng)

FR1 (Ingestion - TA): Thu thập OHLCV + Turnover hằng ngày thông qua lớp trừu tượng hóa (Abstraction Layer).

FR2 (Ingestion - SA/News/Macro): Thu thập văn bản (Tiêu đề + Sapo từ 1 nguồn ổn định), lưu đủ thời gian, VÀ thu thập các chỉ số "Core-4" Vĩ mô.

FR3 (Sentiment - API AI): Gán sentiment + trích xuất "Catalyst" (Sự kiện) cho News/SA bằng Gemini API (Batch Mode).

FR4 (Elitist Filter - Dynamic Weighting): Tính điểm có trọng số theo nguồn/tài khoản (với capping và cold-start).

FR5 (Regime Classification - HMM): Dự đoán trạng thái ngày T (4 lớp) (dùng filtering-only, sticky bias, transition mask, VÀ sử dụng các đặc trưng Vĩ mô làm Exogenous Variables).

FR6 (Labeling - Weak Supervision): Tạo silver labels bằng Snorkel (với abstain rate $\le$ 40%) VÀ thêm các LF dựa trên Vĩ mô.

FR7 (AL UI - Queue): Màn AL hiển thị mẫu không chắc chắn (đã Stratified Round-Robin) VÀ ưu tiên các mẫu HunterScore cao / FrothScore thấp.

FR8 (AL Interaction - Micro-review): Gán golden labels $\le$ 3 phút/tuần (quota 5–10 mẫu).

FR9 (Control Plane): UI hiển thị Kill-Switch, model version, drift alert (với audit log).

FR10 (A/B Test - Elitist vs Baseline): Chạy song song, so sánh hiệu quả (với switching rule có Feature Flag).

FR11 (Admin Security): Bảo vệ endpoint quản trị (và toàn bộ Dashboard) bằng X-ADMIN-KEY.

FR12 (Model Serving - Hot-reload): AppAPI giữ model trong RAM; reload an toàn (xác thực artifact_sha256).

FR13 (Queue & DLQ): task_q có retry với backoff, DLQ bắt buộc.

FR13.1 (NLP Task Linking - MỚI): Mọi bản ghi News hợp lệ sau khi Ingestion phải tự động sinh ra một (và chỉ một) task NLP_PROCESS trong task_q (với ràng buộc idempotent) để đảm bảo không có tin tức nào bị "mồ côi".

FR14 (Observability - NÂNG CẤP):

Hệ thống: Ghi metrics và hiển thị các dashboard (STS, AL completion, IC20D).

Data Platform: Cung cấp metrics chuyên sâu cho Scraper (News/day/source, NLP Lag, Queue Depth, Ingestion Error Rate).

Source Health: Dashboard System Health phải hiển thị trạng thái từng nguồn (HEALTHY / DEGRADED / OVERLOADED / PARSER_SUSPECT).

FR15 (As-of Enforcement): Mọi huấn luyện/backtest đọc qua view as-of; vi phạm = 0.

FR16 (News History & Incremental - MỚI):

Hệ thống phải hỗ trợ Backfill tin tức lịch sử (tối thiểu 12-24 tháng) thông qua script chuyên dụng.

Hệ thống phải hỗ trợ Incremental Crawl dựa trên trạng thái (scraper_state) để không lấy trùng lặp.

2.2 Non Functional (Phi chức năng)

NFR1 (API Latency): /predict p95 < 500 ms.

NFR2 (Uptime): $\ge$ 99.0%/tháng.

NFR3 (Batch Window): NLP $\rightarrow$ Label $\rightarrow$ Train hoàn tất trước 06:00.

NFR4 (Temporal Correctness): as-of violations = 0.

NFR5 (Security): Public /predict rate-limit 100 req/giờ/IP; admin endpoints bắt buộc xác thực.

NFR6 (Compliance & Retention): Không lưu full text quá 60 ngày (trừ khi có license).

NFR7 (Cost): Vận hành trong 1 VPS 10–15 USD; tối ưu hóa Docker image size (< 2GB).

NFR8 (Observability): Metrics bắt buộc. Log giữ tối thiểu 30 ngày.

NFR9 (Data Quality): GE tối thiểu cho dữ liệu đầu vào; các bản ghi rác phải được đưa vào _bronze_bad hoặc DLQ.

NFR10 (Scalability): Hỗ trợ $\le$ 500 symbols, $\le$ 5k bài SA/News/ngày.

3. User Interface Design Goals (Mục tiêu Thiết kế Giao diện Người dùng)

3.1 Overall UX Vision (Tầm nhìn UX Tổng thể)

Tầm nhìn UX là "Hiệu quả Tối đa, Tương tác Tối thiểu". Giao diện Streamlit là một "buồng lái" (cockpit) để thực hiện các hành động quan trọng, không phải một sản phẩm để "khám phá".

3.2 Core Screens and Views (Các Màn hình Cốt lõi)

MVP phải bao gồm 5 màn hình cốt lõi sau (cho single-user "Admin"):

01_Overview.py (Dashboard Ngày):

Hiển thị HunterScore, FrothScore (Badge lớn).

Hiển thị trạng thái "chế độ" (regime), plain_explainer, và "safety banner".

Cung cấp nút Kill-Switch (với confirm dialog 2 bước).

Tích hợp khảo sát "Signal Trust Score" (STS) (pop-up 1 lần/ngày).

02_Active_Learning.py (AL Review):

Hiển thị al_queue (ưu tiên HunterScore cao / FrothScore thấp).

Cung cấp giao diện "Confirm/Flip/Snooze" (có phím tắt).

Hiển thị Sandbox Mode.

03_Model_Registry.py (Quản lý Mô hình):

Hiển thị Model Registry (metrics, 95% CI, promotion_suggestion).

Cung cấp các nút (Approve Canary, Promote 100%, Rollback).

04_System_Health.py (Sức khỏe Hệ thống):

Hiển thị các Đồ thị Giám sát (PSI, flip-rate).

Hiển thị trạng thái Scraper (Circuit Breaker state).

Hiển thị Macro mini-hub (MLI, z-scores) và Sector heatmap.

(MỚI) Hiển thị trạng thái từng Nguồn dữ liệu (HEALTHY, DEGRADED, PARSER_SUSPECT).

05_Macro_Impact_Config.py (Cấu hình Tác động Vĩ mô):

Hiển thị st.data_editor để quản lý dim_macro_sector_impact.

3.3 Accessibility (Khả năng Tiếp cận)

Yêu cầu: Giao diện 02_Active_Learning.py phải hỗ trợ hoàn toàn bằng bàn phím (Hotkeys C/F/S).

3.4 Target Device and Platforms (Nền tảng Mục tiêu)

Nền tảng: Streamlit (trong dash container).

Thiết bị: Tối ưu hóa cho Desktop.

4. Technical Assumptions (Giả định Kỹ thuật)

Architecture: Hybrid Model.

Local: Chạy các thuật toán nhẹ (scikit-learn, hmmlearn, vectorbt) và Database/App.

Cloud: Offload tác vụ NLP nặng (Sentiment/Extraction) lên Gemini API (Google AI Studio).

Strategy: API-First (MỚI):

Ưu tiên sử dụng API (public hoặc internal JSON) của nguồn dữ liệu để thu thập thông tin thay vì parse HTML.

Chỉ sử dụng biện pháp parse HTML (SSR) như phương án dự phòng cuối cùng.

Sử dụng các thư viện wrapper (như vnstock cho dữ liệu giá) trong giai đoạn đầu để tăng tốc độ phát triển.

External Dependency: Phụ thuộc vào Google Gemini API (Free Tier/Paid) cho pipeline NLP.

Resource Optimization: Worker container KHÔNG cài đặt torch, transformers, onnxruntime. Docker image size phải nhỏ (< 1GB lý tưởng).

Data Privacy: Chấp nhận gửi nội dung tin tức công khai (Public News) lên Cloud để xử lý.

Monorepo: Tách biệt các tiến trình (api, worker, scraper, dash, proxy, cron).

Orchestrator: Cron (Sidecar).

Database: PostgreSQL.

5. Epic List (Danh sách Epics)

Epic 0: Foundation & Core Infrastructure (Docker, CI/CD, Schema).

Epic 1: Data Platform (Scraper, Ingestion, Silver Layer, Abstraction, Backfill).

Epic 2: NLP & Feature Platform (Gemini API NLP, Elitist, Feature Gold).

Epic 3: Labeling & Active Learning (Snorkel, AL Queue).

Epic 4: Model Training & A/B (Auto-Train, Backtest WFO, Auto-Rollback).

Epic 5: Serving & Control Plane (AppAPI, Dashboard, AI Hand-off).

Epic 0: Foundation & Core Infrastructure

Mục tiêu (Goal): Khởi tạo nền tảng kỹ thuật "sống dai" (robust) cho toàn bộ dự án.

Story 0.1: Khởi tạo Monorepo và Docker Compose

Yêu cầu (Acceptance Criteria):

(AC1) Cấu trúc thư mục Monorepo chuẩn (src/api, src/worker, src/core_lib...).

(AC2) docker-compose.yml định nghĩa các services chính (proxy, db, api, worker, scraper, dash, cron).

(AC3) Giới hạn tài nguyên (CPU/RAM) cho từng container.

(AC4, AC5) Thư viện dùng chung core_lib được đóng gói và cài đặt vào các service khác.

(AC8) Quản lý biến môi trường qua .env.

(AC9) Tối ưu Image: Worker image không cài đặt torch, transformers.

Story 0.2: Khởi tạo Schema Database và Logic As-of

Yêu cầu (Acceptance Criteria):

(AC1, AC2) init.sql chứa DDL cho TẤT CẢ các bảng và idempotent.

(AC3) Chuẩn hóa thời gian (Timezone) database là UTC.

(AC5) View v_features_asof được định nghĩa trong db/views.sql (NFR4).

(AC7) core_lib chứa db.py, locks.py (advisory locks).

Story 0.3: Liveness, Readiness, và Security Tối thiểu

Yêu cầu (Acceptance Criteria):

(AC2) Liveness Probe (/healthz) trả về 200 OK.

(AC3) Readiness Probe (/readyz) kiểm tra sâu (DB connection, Model cache, Data freshness > 26h).

(AC5, AC6) Admin Security: Bảo vệ các endpoint (/admin/*) bằng X-ADMIN-KEY.

Story 0.4: Thiết lập CI/CD và Unit Test

Yêu cầu (Acceptance Criteria):

(AC1) .github/workflows/ci.yml được tạo.

(AC2) Pipeline tự động build tất cả services.

(AC3) Pipeline chạy linters (Ruff, Black).

(AC4) Pipeline chạy pytest.

(AC5) Unit test (core_lib/test_asof.py) kiểm tra logic "gian lận thời gian".

(AC7) Pipeline chạy smoke test (curl /readyz).

Epic 1: Data Platform

Mục tiêu (Goal): Xây dựng nền tảng dữ liệu bền vững, có khả năng mở rộng, tuân thủ pháp lý, và dễ dàng thay thế nguồn dữ liệu, tích hợp cơ chế backfill và giám sát chi tiết.

Story 1.1: Xây dựng Worker Thu thập (Scraper "Lean-Armor")

Yêu cầu (Acceptance Criteria):

(AC1) Adapter Pattern: Sử dụng Pydantic để chuẩn hóa dữ liệu.

(AC2) Cơ chế Phòng thủ: Rate Limiter & Circuit Breaker.

(AC3) Backpressure (Điều áp): Tự động giảm tốc độ nếu cờ SCRAPE_SLOW bật.

(AC6) Cron Job Vĩ mô: Lên lịch thu thập dữ liệu Vĩ mô.

(AC7 - MỚI) Legal Guard: Mọi Spider phải có cơ chế kiểm tra robots.txt (nếu có) và tuân thủ rate limit khai báo.

Story 1.2: Triển khai Thu thập Dữ liệu (As-of & Idempotent)

Yêu cầu (Acceptance Criteria):

(AC1) Tính toán as_of_time = max(publisher_time, first_seen_time).

(AC2) Chuẩn hóa UTC: Mọi timestamp lưu vào DB đều phải đưa về múi giờ UTC.

(AC3) Content Hash: Tạo mã băm (SHA256) từ nội dung.

(AC4, AC5) Idempotent Write: Ghi vào raw_bronze (ON CONFLICT (content_hash) DO NOTHING).

(AC6) Idempotent Enqueue: Chỉ đẩy task vào task_q sau khi đã lưu raw_bronze thành công.

Story 1.3: Triển khai Worker Xử lý (Silver Layer "Tinh gọn")

Yêu cầu (Acceptance Criteria):

(AC1, AC2) Xử lý hàng đợi an toàn: Dùng Advisory Lock và FOR UPDATE SKIP LOCKED.

(AC3) Quy tắc Sanity (Lành mạnh): Kiểm tra nhanh dữ liệu (Giá > 0, Text không quá ngắn).

(AC4) Xử lý lỗi (DLQ): Nếu vi phạm, chuyển task vào task_q_dlq.

(AC5) Ghi vào Silver: Dữ liệu sạch được UPSERT vào ta_silver / sa_silver.

(AC7) Kiểm soát Backpressure: Bật cờ SCRAPE_SLOW nếu task_q > 10,000.

Story 1.4: Data Source Abstraction (MỚI)

As a (Với tư cách là) Architect,
I want (Tôi muốn) Data Platform phải định nghĩa các Interface (lớp trừu tượng) cho nguồn dữ liệu,
so that (để) hệ thống không bị phụ thuộc cứng (Hard-coding) vào vnstock hay bất kỳ API vendor nào, giúp dễ dàng thay thế trong tương lai.

Yêu cầu (Acceptance Criteria):

(AC1.4.1) Định nghĩa Interface PriceSource(Protocol) và NewsSource(Protocol) (bao gồm Pydantic Models) trong core_lib.

(AC1.4.2) Triển khai ít nhất 2 implementations: VnstockPriceSource (cho Dev/PoC) và VendorPriceSource (Placeholder cho API trả phí).

(AC1.4.3) Dependency Injection: Không module nào trong worker hay api được gọi trực tiếp vnstock hay HTTP client của một trang web cụ thể. Mọi cuộc gọi phải đi qua Interface.

Story 1.5: News Backfill & Incremental Engine (MỚI)

As a (Với tư cách là) Data Engineer,
I want (Tôi muốn) hệ thống có khả năng lấy dữ liệu lịch sử (Backfill) và chỉ lấy dữ liệu mới (Incremental) khi chạy cron,
so that (để) tôi có đủ dữ liệu để huấn luyện model và tiết kiệm tài nguyên vận hành.

Yêu cầu (Acceptance Criteria):

(AC1.5.1) Bảng scraper_state: Lưu trữ con trỏ (cursor/last_id/last_time) cho từng nguồn dữ liệu trong DB.

(AC1.5.2) Logic Incremental: Spider khi chạy (fetch_latest) phải đọc scraper_state để chỉ request các bản ghi mới hơn điểm đánh dấu.

(AC1.5.3) Script backfill_news.py: Cho phép chạy lùi thời gian (ví dụ: 12-24 tháng) với tốc độ an toàn (low rate limit) để điền dữ liệu lịch sử.

Epic 2: NLP & Feature Platform

Mục tiêu (Goal): Chuyển đổi dữ liệu sạch (Silver) thành các đặc trưng (Features) thông minh, bao gồm cả tín hiệu định lượng (Quant) và tín hiệu ngôn ngữ (NLP).

Story 2.1: Triển khai Batch Job NLP (Gemini API Integration)

(Cập nhật V2.7: Thay thế mô hình Local bằng Gemini API)

As a (Với tư cách là) AppWK (App-Worker),
I want (Tôi muốn) thực thi một Cron job (01:00) để gửi batch tin tức (Title + Sapo) lên Google Gemini API,
so that (để) tôi nhận về kết quả Cảm xúc (Sentiment) và Sự kiện (Catalyst) một cách chính xác mà không tốn tài nguyên VPS (NFR7).

Yêu cầu (Acceptance Criteria):

(AC1) Lịch chạy: Cron job nlp_batch chạy lúc 01:00 sáng, sử dụng advisory_lock('nlp_batch').

(AC2) Batch Request: Gửi request theo lô (Batch) để tối ưu chi phí và tránh rate limit.

(AC3) Structured Output: Yêu cầu Gemini API trả về JSON theo schema Pydantic (gồm sentiment_score, catalysts list).

(AC4) Fallback: Nếu API lỗi, ghi log và bỏ qua (fail-soft), không làm sập toàn bộ job.

(AC5) Security: API Key đọc từ .env.

Story 2.2: Triển khai Batch Job "Elitist Signal" (Dynamic Weighting)

As a (Với tư cách là) AppWK (App-Worker),
I want (Tôi muốn) thực thi một Cron job (02:00) để tính toán trọng số uy tín (credibility weights) (FR4) (tuân thủ as-of),
so that (để) bảng account_weights sẵn sàng cho bước Tổng hợp (Aggregation).

Yêu cầu (Acceptance Criteria):

(AC1) Lịch chạy: Cron job elitist_batch chạy lúc 02:00 (sau nlp_batch).

(AC3) Logic As-of: Tính toán trọng số BẮT BUỘC tuân thủ as_of_time.

(AC4) Capping: Áp dụng giới hạn (ví dụ: Top 1% $\le$ 25% tổng trọng số) để tránh thao túng.

(AC6) Cold-start: Nếu ngày có < 5 nguồn, hệ thống tự động pha trộn (blend) với tín hiệu trung bình (Baseline).

Story 2.3: Triển khai Batch Job "Feature Gold" (Atomic Publish)

As a (Với tư cách là) AppWK (App-Worker),
I want (Tôi muốn) thực thi một Cron job (02:30) để kết hợp tất cả dữ liệu (TA, SA đã gán trọng số, HMM, Vĩ mô) và "xuất bản nguyên tử" (atomically publish) bảng _serving,
so that (để) AppAPI có thể đọc dữ liệu mới một cách an toàn (without downtime).

Yêu cầu (Acceptance Criteria):

(AC1) Lịch chạy: Cron job feature_gold_batch chạy lúc 02:30.

(AC2) HMM (Regime): Triển khai HMM (filtering-only, sticky bias, transition mask) để phân loại 4 chế độ thị trường.

(AC3) Scoring: Tính toán HunterScore (Tín hiệu) và FrothScore (Rủi ro) dựa trên logic nghiệp vụ (feature_logic_v1).

(AC6, AC9) Atomic Publish: Tạo bảng _tmp $\to$ Validate $\to$ ALTER TABLE ... RENAME trong một transaction để hoán đổi bảng, đảm bảo API không bị gián đoạn.

Epic 3: Labeling & Active Learning

Mục tiêu (Goal): Tạo ra bộ dữ liệu huấn luyện chất lượng cao thông qua quy trình "bán tự động": kết hợp Weak Supervision (Snorkel) và Active Learning (Admin Review).

Story 3.1: Triển khai Batch Job "Weak Supervision" (Snorkel)

As a (Với tư cách là) AppWK (App-Worker),
I want (Tôi muốn) thực thi một Cron job (02:40) để áp dụng các Hàm Dán nhãn (LFs) và huấn luyện Snorkel Label Model,
so that (để) tôi có thể tạo ra các "nhãn bạc" (silver labels) (FR6) có thể tái lập và kiểm soát chất lượng.

Yêu cầu (Acceptance Criteria):

(AC1) Lịch chạy: Cron job label_batch chạy lúc 02:40 (sau feature_gold_batch).

(AC2) LFs: Triển khai tối thiểu 3 LFs (ví dụ: LF_rule_RSI, LF_hmm_top1, LF_macro_tailwind).

(AC3) Reproducibility: Lưu label_model_version và lf_fingerprint (hash của code LFs) vào DB.

(AC4) Output: Ghi kết quả vào labels_silver (gồm state_snorkel, probability, entropy).

(AC5) Quality Guardrail: Job fail nếu abstain_rate > 40% hoặc coverage < 60%.

(AC8) Auto-Accept: Tự động chấp nhận nhãn nếu độ tin cậy $\ge 0.9$.

Story 3.2: Triển khai Logic "Active Learning" (Stratified Sampling)

As a (Với tư cách là) AppWK (App-Worker),
I want (Tôi muốn) lọc và chọn ra các mẫu cần người dùng review (Active Learning) ngay sau khi tạo nhãn bạc,
so that (để) tôi tối ưu hóa thời gian của Admin vào những mẫu "đáng giá" nhất.

Yêu cầu (Acceptance Criteria):

(AC2) Candidate Filter: Chỉ chọn các mẫu "Không chắc chắn" (entropy cao) HOẶC các mẫu "Tín hiệu mạnh" (HunterScore $\ge$ 80 VÀ FrothScore $\le$ 20).

(AC3, AC5) Diversification: Chọn mẫu rải đều theo Ngành (sector) và Vốn hóa (cap_tercile). Giới hạn 2 mẫu/mã/tuần.

(AC6) Hàng đợi & Chống trùng: Ghi vào al_queue với dedup_key (symbol:date) để không review lại.

(AC7) Quota: Giới hạn WEEKLY_AL_QUOTA = 10 mẫu/tuần (FR8).

Story 3.3: Triển khai Giao diện "AL Review" (Active Learning UI)

As a (Với tư cách là) Admin,
I want (Tôi muốn) một giao diện 02_Active_Learning.py trên Dashboard để duyệt nhanh các mẫu trong hàng đợi,
so that (để) tôi có thể gán "nhãn vàng" (Golden Label) chính xác và nhanh chóng.

Yêu cầu (Acceptance Criteria):

(AC1) UI Component: Trang Streamlit hiển thị từng mẫu một (Single-item view).

(AC3) As-of Guard (Quan trọng): Biểu đồ giá và các chỉ số BẮT BUỘC phải bị cắt (clip) tại effective_date. Không hiển thị dữ liệu giá T+1, T+2...

(AC4) Tương tác nhanh: Hỗ trợ 3 hành động với phím tắt: Confirm (C), Flip (F), Snooze (S).

(AC9) Sandbox Mode: Có nút gạt (Toggle) để Admin tập gán nhãn mà không ảnh hưởng model chính.

(AC6, AC7) Output: Khi Admin xác nhận, hệ thống gọi API POST /al/label để ghi vào bảng labels_golden.

(AC8) Trigger: Việc tạo ra nhãn vàng mới sẽ kích hoạt cờ trigger_retrain_needed.

Epic 4: Model Training & A/B

Mục tiêu (Epic Goal): Tự động hóa vòng đời của mô hình AI, từ việc huấn luyện lại (Retraining) khi cần, kiểm thử (Backtesting) khắt khe, và giám sát an toàn (Auto-Rollback).

Story 4.1: Triển khai Batch Job "Model Training" (Có Điều kiện & Tái lập)

As a (Với tư cách là) AppWK (App-Worker),
I want (Tôi muốn) thực thi một Cron job (02:50) để kiểm tra các điều kiện kích hoạt (triggers), và chỉ huấn luyện lại mô hình khi thực sự cần thiết,
so that (để) tôi tiết kiệm tài nguyên VPS (NFR7) và đảm bảo mô hình luôn được cập nhật.

Yêu cầu (Acceptance Criteria):

(AC1) Lịch chạy: Job train_check_batch chạy lúc 02:50 (sau Labeling).

(AC2) Triggers: Chỉ chạy huấn luyện nếu:

(1) Độ trôi dữ liệu (PSI Drift) > 0.2.

(2) Hiệu suất mô hình giảm (IC Drop) > 0.03.

(3) Có > 20 nhãn vàng (Golden Labels) mới.

(4) Mô hình đã quá cũ (> 60 ngày).

(AC3) Cooldown: Bắt buộc nghỉ tối thiểu 7 ngày giữa 2 lần huấn luyện.

(AC5) Refit HMM: Khi huấn luyện lại, phải Refit cả HMM, sử dụng Biến Ngoại sinh Vĩ mô (Macro Exogenous Variables).

(AC7) Model Registry: Mô hình mới (.pkl) được lưu, hash (SHA256) và đăng ký vào model_registry với trạng thái is_active=False.

Story 4.2: Triển khai "Backtesting Engine" (An toàn & Thống kê)

As a (Với tư cách là) AppWK (App-Worker),
I want (Tôi muốn) chạy một quy trình Backtest (WFO) vào cuối tuần để so sánh mô hình mới (Challenger) với mô hình cũ (Champion),
so that (để) tôi có bằng chứng thống kê chắc chắn trước khi đưa mô hình mới ra sử dụng.

Yêu cầu (Acceptance Criteria):

(AC1) Lịch chạy: Job backtest_batch chạy hàng tuần (Chủ Nhật).

(AC2) Docker Profile: Chạy dưới profile backtest (được cấp nhiều RAM/CPU hơn).

(AC3) WFO (Walk-Forward Optimization): Sử dụng vectorbt để chạy backtest cửa sổ trượt.

(AC4) Compute Budget: Giới hạn số lượng mã (100) và thời gian chạy (30 phút) để không làm treo VPS.

(AC6, AC9) Metrics: Tính toán và lưu trữ Sharpe, Max Drawdown, và IC (Information Coefficient).

Story 4.3: Triển khai Logic "Auto-Suggest" & "Auto-Rollback"

As a (Với tư cách là) Hệ thống,
I want (Tôi muốn) tự động đề xuất thăng hạng (Auto-Suggest) mô hình tốt, VÀ tự động quay lui (Auto-Rollback) mô hình lỗi,
so that (để) giảm thiểu rủi ro giao dịch cho người dùng.

Yêu cầu (Acceptance Criteria):

(AC1, AC2) Auto-Suggest Guardrails: Sau khi backtest, tự động đánh giá:

Hysteresis: Mô hình mới phải tốt hơn đáng kể (Sharpe tăng > 0.1).

Risk Guard: Max Drawdown không được tệ hơn quá 20%.

(AC3) Output: Gán nhãn ready_for_canary hoặc rejected vào model_registry.

(AC6, AC7) Auto-Rollback (Canary Monitor):

Cron job monitor_canary_batch chạy hàng giờ.

Nếu phát hiện mô hình Canary đang chạy thật bị sụt giảm nghiêm trọng (Drawdown Breach), hệ thống lập tức TẮT Canary (is_active=False), kích hoạt lại Champion cũ, và gửi cảnh báo.

Epic 5: Serving & Control Plane

Mục tiêu (Goal): Khởi chạy hệ thống phục vụ (Serving) và giao diện quản trị (Control Plane) để đưa "trí tuệ" của AI đến tay người dùng.

Story 5.1: Triển khai AppAPI (Serving Endpoint "Thông minh")

As a (Với tư cách là) AppAPI (API-Server),
I want (Tôi muốn) hot-load (tải nóng) mô hình đã được xác thực, phục vụ dự đoán /predict, và cung cấp các công cụ xuất dữ liệu (export),
so that (để) Người dùng (Admin) có thể nhận tín hiệu an toàn và trích xuất dữ liệu để phân tích thêm.

Yêu cầu (Acceptance Criteria):

(AC1) Health Checks: Vượt qua /healthz (liveness) và /readyz (readiness).

(AC2) Hot-Loading An toàn: Khi khởi động, API tải model is_active=true và xác thực artifact_sha256.

(AC4) Rich Response: Phản hồi /predict bao gồm regime, confidence, plain_explainer, và safety_banner.

(AC6) Fail-safe (Kill-Switch): Nếu cờ KILL_SWITCH bật, API trả về 503.

(AC7, AC9) AI Hand-off Endpoints: GET /export/context (Markdown) và GET /export/pack (ZIP).

Story 5.2: Triển khai Dashboard (Foundation & Proxy Auth)

As a (Với tư cách là) Admin/User,
I want (Tôi muốn) một Dashboard (Streamlit) chạy trên localhost (127.0.0.1) và được bảo vệ bởi Proxy,
so that (để) tôi có một giao diện an toàn, tập trung để quản lý hệ thống.

Yêu cầu (Acceptance Criteria):

(AC1, AC2) Local Binding: Container dash (8501) và api (8000) chỉ lắng nghe trên mạng nội bộ Docker.

(AC3) Proxy Authentication: Nginx/Caddy thực thi Basic Auth (hoặc X-ADMIN-KEY) cho mọi truy cập.

Story 5.3: Triển khai Control Plane (Overview, AL & Health)

As a (Với tư cách là) Admin/User,
I want (Tôi muốn) các màn hình chức năng: Tổng quan, Active Learning, Quản lý Mô hình và Sức khỏe Hệ thống,
so that (để) tôi nắm toàn quyền kiểm soát vận hành của bot.

Yêu cầu (Acceptance Criteria):

(AC1) Overview (01_Overview.py): Hiển thị HunterScore, FrothScore, Regime, và AI Hand-off Panel.

(AC2) Active Learning UI (02_Active_Learning.py): Giao diện review (Confirm/Flip/Snooze) có "As-of Guard".

(AC3, AC4) Model Registry (03_Model_Registry.py): Hiển thị danh sách model, metrics, và các nút Approve Canary / Promote / Rollback.

(AC6) System Health (04_System_Health.py): Hiển thị Macro Mini-Hub (chỉ số vĩ mô) và Sector Heatmap.

Story 5.4: Kế hoạch Vận hành (Ops Plan)

As a (Với tư cách là) Admin,
I want (Tôi muốn) hệ thống có cơ chế ghi log, telemetry và backup,
so that (để) tôi có thể vận hành hệ thống ổn định ("production-ready").

Yêu cầu (Acceptance Criteria):

(AC1) Telemetry: Ghi lại các sự kiện quan trọng (ví dụ: ai_pack_dl, signal_view) vào event_log.

(AC3) Log Rotation: Cấu hình Docker xoay vòng log (ví dụ: max 100MB/file, giữ 5 file).

(AC4) Backup: Script tự động backup database (pg_dump) định kỳ.

Story 5.7: UI Quản lý Ma trận Tác động Vĩ mô (MỚI)

As a (Với tư cách là) Admin/Trader,
I want (Tôi muốn) một màn hình (05_Macro_Impact_Config.py) để xem và chỉnh sửa dim_macro_sector_impact,
so that (để) tôi có thể điều chỉnh "quan điểm" của hệ thống về tác động vĩ mô mà không cần sửa code.

Yêu cầu (Acceptance Criteria):

(AC1, AC2) UI Editor: Sử dụng st.data_editor để hiển thị và chỉnh sửa bảng cấu hình.

(AC4) Persistence: Nút "Lưu" gọi API POST /admin/macro/impact để cập nhật thay đổi vào Database.

9. Constraints & Assumptions (Ràng buộc & Giả định)

Ràng buộc "Cứng" (Hard Constraints):

Self-host: Hệ thống phải chạy được trên hạ tầng do người dùng tự quản lý.

VPS 10–15 USD: Giới hạn tài nguyên nghiêm ngặt (RAM < 2GB, CPU < 2 cores) (NFR7).

Active Learning: Người dùng phải chấp nhận thực hiện micro-review hàng tuần (FR8).

Ràng buộc "Pháp lý & Tuân thủ" (Legal & Compliance) - [MỚI 2.7.1]:
4.  Scraping Ethics:
* Bắt buộc kiểm tra robots.txt và ToS.
* Không bypass CAPTCHA/Cloudflare Challenge phức tạp.
* Chỉ thu thập dữ liệu công khai ở mức "Fair Use".
5.  Data Minimization:
* Chỉ lấy Title, Sapo, Link. Không lưu Full-text nếu không có bản quyền (NFR6).

Giả định "Mềm" (Soft Assumptions):

Giả thuyết “Elitist Signals” > Baseline. Fallback: A/B & auto-switch (FR10).

10. Risks & Open Questions (Rủi ro & Câu hỏi Mở)

1. User Risk (Rủi ro Người dùng) - Đã giảm thiểu:

Rủi ro: Người dùng (Admin) "lười", không tham gia Active Learning.

Giảm thiểu: Thiết kế Micro-review 1-chạm, Quota siêu nhỏ (5-10 mẫu/tuần) (FR8).

2. Model Risk (Rủi ro Mô hình) - Đã giảm thiểu:

Rủi ro: Tín hiệu "Elitist" sai lệch, hoạt động kém hơn Baseline.

Giảm thiểu: Luôn chạy A/B Test (FR10) và có Auto-Rollback (Story 4.3).

3. Resource Risk (Rủi ro Tài nguyên) - Đã giải quyết (V2.7):

Rủi ro: VPS $10 bị tràn RAM (OOM) khi chạy NLP.

Giảm thiểu: Đã chuyển sang kiến trúc Hybrid: Offload NLP nặng lên Gemini API (FR3).

4. Dependency Risk (Rủi ro Phụ thuộc) - Mới:

Rủi ro: Google Gemini API bị gián đoạn (downtime) hoặc thay đổi chính sách giá/quota.

Giảm thiểu: Thiết kế tasks_nlp.py theo Adapter Pattern để dễ dàng chuyển đổi sang OpenAI/Claude nếu cần.

11. Ops Requirements & Rollout (Yêu cầu Vận hành & Triển khai)

1. NFRs Vận hành (Operational NFRs):

Latency: API /predict phải phản hồi dưới 500ms (p95) (NFR1).

Uptime: Cam kết 99.0% uptime mỗi tháng (NFR2).

Batch Window: Các tác vụ batch (NLP $\to$ Train) phải hoàn tất trước 06:00 sáng (NFR3).

As-of Integrity: Tuyệt đối không có vi phạm as-of (NFR4).

2. Bảo mật (Security):

Bắt buộc dùng X-ADMIN-KEY cho mọi thao tác quản trị (FR11).

Áp dụng Rate Limit (100 req/giờ) cho public endpoint (NFR5).

Ghi Audit Log cho mọi hành động thay đổi trạng thái.

3. Đo lường & Giám sát (Observability) - [NÂNG CẤP 2.7.2]:

Hệ thống: Theo dõi các metric chính: STS, AL Completion Rate, Model IC.

Data Platform: Dashboard 04_System_Health.py phải hiển thị trạng thái từng nguồn:

HEALTHY: Có dữ liệu mới trong 24h.

DEGRADED: Không có dữ liệu mới > 24h hoặc tỷ lệ lỗi > 5%.

OVERLOADED: Hàng đợi xử lý (task_q) bị tắc nghẽn.

PARSER_SUSPECT: Tỷ lệ bản ghi rỗng/sai format tăng đột biến.

4. Kế hoạch Triển khai (Rollout Plan):

Giai đoạn 1: Shadow Mode (2 tuần): Hệ thống chạy, thu thập dữ liệu, tính toán tín hiệu nhưng Admin chỉ quan sát Dashboard để kiểm tra độ ổn định.

Giai đoạn 2: Canary Release (10-20% vốn): Bắt đầu giao dịch thật với số vốn nhỏ, sử dụng chế độ BEGINNER_MODE.

Giai đoạn 3: General Availability: Mở rộng quy mô, tắt BEGINNER_MODE.

5. Tuân thủ (Compliance):

Dữ liệu tin tức không rõ bản quyền chỉ được lưu trữ full-text tối đa 60 ngày (NFR6).

Phụ lục (Appendices)

Nguyên tắc As-of: Mọi dòng code, tài liệu và dashboard đều phải tuân thủ nguyên tắc "tại thời điểm đó" (point-in-time), không được sử dụng dữ liệu tương lai.

Ưu tiên sự Rõ ràng: Thà dùng một mô hình đơn giản mà hiểu rõ (như HMM + Rules) còn hơn dùng một "hộp đen" phức tạp.