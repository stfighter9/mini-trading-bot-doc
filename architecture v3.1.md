Tài liệu Kiến trúc Stock Hunter AI (Lean V3.1)

Phiên bản: V3.1 - Data Platform Hardening
Trạng thái: As-Built (Thiết kế Thi công)
Kiến trúc sư: Winston

Tài liệu này đã được phân rã (sharded).

Các phần Kiến trúc

1. Giới thiệu (Introduction)

2. Kiến trúc Cấp cao (High-Level Architecture)

3. Ngăn xếp Công nghệ (Tech Stack)

4. Mô hình Dữ liệu & Schema (SQL DDL)

5. Các Thành phần (Components)

6. API Bên ngoài & Tích hợp (External APIs & Integration)

7. Luồng công việc cốt lõi (Core Workflows)

8. Cấu trúc Thư mục (Source Tree)

9. Hạ tầng & Triển khai (Infrastructure & Deployment)

10. Tiêu chuẩn Triển khai (Implementation Standards)

1. Giới thiệu (Introduction)

Stock Hunter AI là hệ thống hỗ trợ giao dịch định lượng (Quantitative Trading Support System) chuyên biệt dành cho nhà đầu tư cá nhân tại thị trường Việt Nam. Dự án giải quyết bài toán "quá tải thông tin" bằng cách lọc nhiễu và tìm kiếm các tín hiệu dẫn dắt thị trường.

1.1 Triết lý Kiến trúc: "Lean & Hybrid"

Kiến trúc Tinh gọn (Lean Architecture):

Toàn bộ hệ thống được thiết kế để vận hành trơn tru trên VPS $10-15/tháng (NFR7).

Ưu tiên các công cụ hiệu quả cao (PostgreSQL, Docker Compose, Polars) thay vì các hệ thống doanh nghiệp cồng kềnh (Kafka, Spark, K8s).

Trí tuệ Lai (Hybrid AI Intelligence) (V3.0):

Cloud AI (The Reader): Thuê ngoài các tác vụ đọc hiểu nặng nề (NLP) cho Google Gemini API (FR3).

Local Quant (The Thinker): Giữ lại quyền ra quyết định (Scoring, HMM, Backtest) trên máy chủ cục bộ để đảm bảo tốc độ và sự minh bạch.

1.2 Nguyên tắc Cốt lõi (Linh hồn của Hệ thống)

Ba nguyên tắc này (từ V2.1.4) vẫn được giữ nguyên và tuân thủ tuyệt đối:

Temporal Correctness (Sự đúng đắn về thời gian):

Chống tuyệt đối "gian lận thời gian" (look-ahead bias) (NFR4).

Mọi dữ liệu đều được gắn nhãn as_of_time. Hệ thống Training và Backtest chỉ được "nhìn thấy" dữ liệu đã tồn tại tại thời điểm đó.

Elitist Signals (Tín hiệu Tinh hoa):

Không phải mọi nguồn tin đều có giá trị như nhau.

Hệ thống sử dụng Dynamic Weighting (Trọng số động) để đánh giá uy tín của từng nguồn tin (dim_account) (FR4).

Data-Centric AI (AI lấy Dữ liệu làm trung tâm):

Thay vì tinh chỉnh model phức tạp, chúng ta tập trung cải thiện chất lượng dữ liệu.

Sử dụng quy trình Active Learning (FR8) và Weak Supervision (Snorkel) (FR6).

1.3 Quyết định Nền tảng (Foundation Decision)

Loại dự án: Greenfield (Xây dựng mới từ đầu).

Starter Template: KHÔNG sử dụng.

1.4 Phạm vi & Ranh giới (Scope & Boundaries)

In-Scope: Thu thập dữ liệu (TA/SA), Phân loại chế độ thị trường (HMM), Active Learning UI, Auto-Backtest.

Out-of-Scope: Tự động đặt lệnh (Execution), High-Frequency Trading (HFT).

1.5 Change Log (Lịch sử Thay đổi)

Date

Version

Description

Author

2025-11-06

2.1.4

Phiên bản gốc (Base) V2.1.4.

Winston (Arch)

2025-11-17

3.0

Hybrid AI Pivot: Thay thế PhoBERT/ONNX (Local) bằng Gemini API (Cloud) để giải quyết NFR7 (Cost/RAM).

Winston (Arch)

2025-11-18

3.1

Data Platform Hardening: Tích hợp yêu cầu từ PRD V2.7.2. Thêm Abstraction Layer (Price/News Source), Legal/ToS Guardrails, và Observability chuyên sâu.

Winston (Arch)

2. Kiến trúc Cấp cao (High-Level Architecture)

2.1 Tóm tắt Kỹ thuật (Technical Summary)

Hệ thống Stock Hunter AI được thiết kế theo mô hình Monolith được đóng gói (Containerized Monolith), tối ưu hóa để vận hành trên một máy chủ duy nhất (Single VPS) với chi phí thấp (NFR7).

Toàn bộ hệ thống được định nghĩa và điều phối bởi Docker Compose, bao gồm 8 services hoạt động phối hợp chặt chẽ.

Điểm đặc biệt của V3.1 là chiến lược Hybrid AI: "Thuê ngoài" các tác vụ đọc hiểu nặng nề (NLP) lên Cloud (Google Gemini) để giải phóng tài nguyên VPS cho các thuật toán định lượng (Quant) chạy cục bộ.

2.2 Sơ đồ Dự án (High Level Project Diagram)

Sơ đồ dưới đây minh họa luồng dữ liệu và sự tương tác giữa các thành phần trong hệ thống V3.1:

graph TD
  subgraph External [Môi trường Bên ngoài]
    News[News Sites/APIs]
    Gemini[Google Gemini API]
    User[Trader/Admin]
  end

  subgraph "Stock Hunter Node (VPS 10$)"
    Proxy[Caddy (Reverse Proxy + Auth)]
    
    subgraph "Data Plane (Tầng Dữ liệu)"
      Scraper[Scraper (Ingestion)]
      Worker[Worker (Batch Processing)]
      DB[(PostgreSQL)]
    end
    
    subgraph "Control Plane (Tầng Điều khiển)"
      API[AppAPI (FastAPI)]
      Dash[Dashboard (Streamlit)]
    end
    
    subgraph "Orchestration (Điều phối)"
      Cron[Cron Sidecar]
    end
  end

  %% Luồng Người dùng
  User -->|HTTPS /predict, /dash| Proxy
  Proxy -->|X-ADMIN-KEY| Dash
  Proxy -->|Rate-limited| API
  
  %% Luồng Thu thập (V3.1)
  Scraper -->|Abstraction Layer| News
  Scraper -->|Raw Data (Bronze)| DB
  
  %% Luồng Xử lý (V3.1 - Hybrid AI)
  Cron -->|Trigger| Worker
  Worker -->|1. Read Raw| DB
  Worker -->|2. Batch Request (JSON)| Gemini
  Gemini -->|3. Sentiment & Catalyst| Worker
  Worker -->|4. Write Silver/Gold| DB
  
  %% Luồng Phục vụ
  API -->|Hot-load .pkl| FS[Filesystem (Artifacts)]
  API -->|Query Features| DB
  Dash -->|Internal API Call| API


2.3 Các Mẫu Kiến trúc & Thiết kế (Architectural & Design Patterns)

1. Hybrid Intelligence (Trí tuệ Lai)

Mô tả: Phân chia rạch ròi trách nhiệm xử lý AI.

Áp dụng:

Cloud AI (The Reader): Sử dụng Gemini API cho các tác vụ NLP (Unstructured Data) (FR3).

Local AI (The Thinker): Sử dụng scikit-learn/HMM cho các tác vụ Quant (Structured Data) (FR5).

2. Data Source Abstraction (Trừu tượng hóa Nguồn Dữ liệu) - [MỚI V3.1]

Mô tả: Định nghĩa các Interface (PriceSource, NewsSource) để tách biệt logic nghiệp vụ khỏi các implementation cụ thể (như vnstock hay API của bên thứ ba).

Áp dụng: scraper service gọi NewsSource.fetch_latest(), worker service gọi PriceSource.get_ohlcv(). Logic bên trong có thể là VnstockPriceSource hoặc VendorPriceSource mà không ảnh hưởng đến phần còn lại của hệ thống (PRD Story 1.4).

3. Process Separation (Tách biệt Tiến trình)

Mô tả: Tách các loại tải công việc (workload) khác nhau vào các container riêng biệt.

Áp dụng: AppAPI (I/O bound), AppWK (CPU bound), ScrapyW (Network I/O).

4. Database as a Queue (Cơ sở dữ liệu làm Hàng đợi)

Mô tả: Sử dụng PostgreSQL làm hàng đợi tin cậy (NFR7).

Áp dụng: Sử dụng FOR UPDATE SKIP LOCKED cho task_q (FR13) và al_queue (FR7).

5. Sidecar Orchestrator

Mô tả: Sử dụng container cron nhỏ để kích hoạt các tác vụ.

Áp dụng: cron gửi lệnh docker exec sang worker hoặc scraper.

6. Atomic Publishing (Xuất bản Nguyên tử)

Mô tả: Đảm bảo dữ liệu phục vụ (_serving) luôn ở trạng thái hoàn chỉnh.

Áp dụng: Worker tính toán xong ghi vào bảng tạm (_tmp), sau đó dùng ALTER TABLE ... RENAME để tráo đổi.

7. Lean-Armor (Phòng thủ Tinh gọn)

Mô tả: Các cơ chế bảo vệ hệ thống khỏi quá tải và lỗi từ bên ngoài.

Áp dụng: Circuit Breaker, Backpressure (SCRAPE_SLOW), Rate Limiter (FR14).

3. Ngăn xếp Công nghệ (Tech Stack)

3.1 Hạ tầng Cloud (Cloud Infrastructure)

Nhà cung cấp (Provider): VPS (DigitalOcean, Linode, hoặc Hetzner).

Cấu hình tối thiểu (Target Spec): 2 vCPU, 2GB RAM, 50GB SSD (NFR7).

Chi phí mục tiêu: $10 - $15 / tháng.

Vị trí (Region): Singapore.

3.2 Bảng Công nghệ Chi tiết (Technology Stack Table)

Hạng mục

Công nghệ

Phiên bản

Mục đích

Lý do lựa chọn (Rationale)

Language

Python

3.11-slim

Ngôn ngữ chính

Hệ sinh thái Data & Backend.

Runtime

Docker Compose

3.9+

Điều phối Container

Chuẩn cho Single VPS, hỗ trợ profiles.

Database

PostgreSQL

15

Database & Queue

Hiệu năng cao, SKIP LOCKED.

AI - NLP

Google Gemini API

2.5 Flash

Xử lý Ngôn ngữ

V3.0: Thay thế model local. Nhanh, rẻ.

AI - Quant

Scikit-learn, HMMlearn

Latest

Mô hình Định lượng

Nhẹ, chạy nhanh trên CPU.

Backtest

VectorBT

Latest

Kiểm thử chiến lược

Hiệu năng cao (vectorization), WFO.

Backend

FastAPI

0.110+

API Server

Async, hiệu suất cao.

Frontend

Streamlit

1.33+

Dashboard

Pure Python, phát triển nhanh.

Proxy

Caddy

2-alpine

Gateway Bảo mật

Tự động HTTPS, cấu hình đơn giản.

Orchestration

Cron (Sidecar)

Alpine

Lập lịch Batch Job

Nhẹ, tin cậy.

Data Proc

Polars

Latest

Xử lý Dữ liệu

Tiết kiệm RAM hơn Pandas.

Data Wrapper

VnStock

Latest

Dev/PoC Source

V3.1: Wrapper cho PriceSource giai đoạn đầu.

CI/CD

GitHub Actions

Free Tier

Tự động hóa

Tích hợp sẵn.

3.3 Các Dịch vụ Docker (Docker Services)

Hệ thống bao gồm 8 services được định nghĩa trong docker-compose.yml:

proxy: Cổng bảo mật.

db: Cơ sở dữ liệu PostgreSQL.

api: Backend API (FastAPI).

dash: Frontend Dashboard (Streamlit).

scraper: Thu thập dữ liệu.

worker: Xử lý Batch chính (Gemini, HMM, Snorkel).

worker_bt: Worker Backtest chuyên dụng (Profile backtest).

cron: Sidecar điều phối.

4. Mô hình Dữ liệu & Schema (SQL DDL)

Trạng thái: As-Built (V3.1) | Database: PostgreSQL 15

(Phần này chứa toàn bộ DDL từ db/init.sql và các migrations, bao gồm các bảng cốt lõi và các bảng mới cho V3.1)

-- ==== HÀNG ĐỢI & KIỂM SOÁT ====
CREATE EXTENSION IF NOT EXISTS pgcrypto;

CREATE TABLE IF NOT EXISTS task_q (
  id BIGSERIAL PRIMARY KEY,
  kind TEXT NOT NULL,           -- 'NLP_PROCESS', 'TA_PROCESS'
  payload JSONB NOT NULL,
  next_run_at TIMESTAMPTZ NOT NULL DEFAULT now(),
  retry_count INT NOT NULL DEFAULT 0,
  max_retries INT NOT NULL DEFAULT 3,
  created_at TIMESTAMPTZ DEFAULT now()
);
CREATE INDEX IF NOT EXISTS idx_task_q_next_run_at ON task_q (next_run_at);

CREATE TABLE IF NOT EXISTS task_q_dlq (LIKE task_q INCLUDING ALL, moved_at TIMESTAMPTZ DEFAULT now());

CREATE TABLE IF NOT EXISTS control_flags (
  flag TEXT PRIMARY KEY,
  enabled BOOL NOT NULL DEFAULT true,
  reason TEXT,
  updated_at TIMESTAMPTZ
);

-- ==== DỮ LIỆU THÔ (BRONZE) ====
CREATE TABLE IF NOT EXISTS raw_bronze (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  source_name TEXT NOT NULL,
  content_hash TEXT NOT NULL,  -- SHA256 để chống trùng lặp
  payload_json JSONB,          -- Title, Sapo, Link
  publisher_time TIMESTAMPTZ,
  first_seen_time TIMESTAMPTZ NOT NULL DEFAULT now(),
  as_of_time TIMESTAMPTZ NOT NULL, -- max(publisher, first_seen)
  created_at TIMESTAMPTZ DEFAULT now()
);
CREATE UNIQUE INDEX IF NOT EXISTS idx_raw_bronze_content_hash ON raw_bronze (content_hash);

-- Bảng dữ liệu rác (MỚI V3.1)
CREATE TABLE IF NOT EXISTS raw_bronze_bad (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  source_name TEXT,
  payload_json JSONB,
  reason TEXT,                 -- 'INVALID_TIMESTAMP', 'TITLE_TOO_SHORT'
  created_at TIMESTAMPTZ DEFAULT now()
);

-- Bảng trạng thái Scraper (MỚI V3.1)
CREATE TABLE IF NOT EXISTS scraper_state (
    source_id TEXT PRIMARY KEY,
    last_cursor TEXT,           -- last_id hoặc last_timestamp
    last_run_at TIMESTAMPTZ
);

-- ==== DỮ LIỆU SẠCH (SILVER) ====
CREATE TABLE IF NOT EXISTS ta_silver (
  symbol TEXT NOT NULL,
  trade_date DATE NOT NULL,
  open DOUBLE PRECISION NOT NULL,
  high DOUBLE PRECISION NOT NULL,
  low DOUBLE PRECISION NOT NULL,
  close DOUBLE PRECISION NOT NULL,
  volume BIGINT NOT NULL,
  as_of_time TIMESTAMPTZ NOT NULL,
  PRIMARY KEY (symbol, trade_date)
);

CREATE TABLE IF NOT EXISTS sa_silver (
  url_canonical TEXT PRIMARY KEY,
  source_name TEXT NOT NULL,
  publisher_time_utc TIMESTAMPTZ,
  as_of_time TIMESTAMPTZ NOT NULL,
  text_norm_hash TEXT NOT NULL,
  text_len INT NOT NULL,
  language TEXT,
  bronze_ref_id UUID REFERENCES raw_bronze(id),
  sentiment_score REAL,         -- Từ Gemini API
  validated_at TIMESTAMPTZ DEFAULT now()
);
CREATE UNIQUE INDEX IF NOT EXISTS idx_sa_silver_text_norm_hash ON sa_silver (text_norm_hash);

CREATE TABLE IF NOT EXISTS catalyst_flags (
  sa_silver_ref_id TEXT REFERENCES sa_silver(url_canonical),
  flag_name TEXT NOT NULL,      -- 'M&A', 'EARNINGS_SURPRISE' (Từ Gemini)
  as_of_time TIMESTAMPTZ NOT NULL,
  PRIMARY KEY (sa_silver_ref_id, flag_name)
);

-- ==== DỮ LIỆU VÀNG (GOLD / FEATURE STORE) ====
CREATE TABLE IF NOT EXISTS features_gold (
  symbol TEXT NOT NULL,
  effective_date DATE NOT NULL,
  feature_set_version TEXT NOT NULL,
  feature_name TEXT NOT NULL,
  value DOUBLE PRECISION,
  PRIMARY KEY (symbol, effective_date, feature_set_version, feature_name)
) PARTITION BY RANGE (effective_date);
CREATE TABLE IF NOT EXISTS features_gold_default PARTITION OF features_gold DEFAULT;

-- Bảng Serving (dạng rộng, được hoán đổi nguyên tử)
CREATE TABLE IF NOT EXISTS features_gold_serving (
    symbol TEXT,
    effective_date DATE,
    hmm_state INT,
    HunterScore REAL,
    FrothScore REAL,
    -- ... các features khác ...
    PRIMARY KEY (symbol, effective_date)
);

-- ==== ML & NHÃN (LABELING) ====
CREATE TABLE IF NOT EXISTS dim_account (
  account_id TEXT PRIMARY KEY,
  source_id TEXT,
  cred_score REAL DEFAULT 0.5 -- Điểm uy tín
);

CREATE TABLE IF NOT EXISTS labels_silver (
  symbol TEXT NOT NULL,
  effective_date DATE NOT NULL,
  state_snorkel INT,
  probability DOUBLE PRECISION,
  entropy DOUBLE PRECISION,     -- Độ không chắc chắn
  is_golden BOOLEAN DEFAULT false,
  PRIMARY KEY (symbol, effective_date)
);

CREATE TABLE IF NOT EXISTS labels_golden (
  id BIGSERIAL PRIMARY KEY,
  symbol TEXT NOT NULL,
  effective_date DATE NOT NULL,
  old_label INT,
  new_label INT NOT NULL,
  actor TEXT DEFAULT 'admin',
  created_at TIMESTAMPTZ DEFAULT now()
);

CREATE TABLE IF NOT EXISTS al_queue (
  id BIGSERIAL PRIMARY KEY,
  symbol TEXT NOT NULL,
  effective_date DATE NOT NULL,
  reason TEXT,                  -- 'entropy', 'hunter_score'
  status TEXT DEFAULT 'pending',
  dedup_key TEXT UNIQUE
);

-- ==== QUẢN LÝ MODEL & VẬN HÀNH (OPS) ====
CREATE TABLE IF NOT EXISTS model_registry (
  model_version TEXT PRIMARY KEY,
  model_type TEXT NOT NULL,
  file_path TEXT NOT NULL,
  artifact_sha256 TEXT,         -- Hash để xác thực
  metrics JSONB,                -- Sharpe, IC, Drawdown
  is_active BOOLEAN DEFAULT false,
  promotion_suggestion TEXT
);

CREATE TABLE IF NOT EXISTS model_promotion_history (
  id BIGSERIAL PRIMARY KEY,
  model_version TEXT REFERENCES model_registry(model_version),
  action TEXT NOT NULL,         -- 'APPROVE_CANARY', 'PROMOTE', 'ROLLBACK'
  actor TEXT DEFAULT 'admin',
  created_at TIMESTAMPTZ DEFAULT now()
);

CREATE TABLE IF NOT EXISTS monitoring_logs (
  log_time TIMESTAMPTZ PRIMARY KEY DEFAULT now(),
  metric_name TEXT NOT NULL,    -- 'psi_top_10', 'nlp_lag_minutes' (MỚI V3.1)
  value DOUBLE PRECISION,
  metadata JSONB
);

CREATE TABLE IF NOT EXISTS event_log (
  id BIGSERIAL PRIMARY KEY,
  event_time TIMESTAMPTZ NOT NULL DEFAULT now(),
  event_name TEXT NOT NULL,     -- 'ai_pack_dl', 'signal_view'
  actor TEXT DEFAULT 'admin',
  meta JSONB
);

-- ==== DỮ LIỆU VĨ MÔ (MACRO) ====
CREATE TABLE IF NOT EXISTS macro_clean (
    metric_name TEXT PRIMARY KEY,
    value DOUBLE PRECISION,
    last_updated TIMESTAMPTZ
);

CREATE TABLE IF NOT EXISTS sector_stats (
    sector TEXT PRIMARY KEY,
    momentum DOUBLE PRECISION,
    breadth DOUBLE PRECISION,
    last_updated TIMESTAMPTZ
);

CREATE TABLE IF NOT EXISTS macro_ui_override (
    sector TEXT PRIMARY KEY,
    weight double precision,
    ttl_until date
);


5. Các Thành phần (Components)

Hệ thống bao gồm 8 services container hóa, mỗi service đảm nhận một trách nhiệm riêng biệt:

proxy (Caddy): Cổng bảo mật duy nhất, xử lý HTTPS và Xác thực (Auth).

db (PostgreSQL): Trái tim lưu trữ, Hàng đợi (task_q), và Feature Store.

api (FastAPI): Backend server phục vụ (/predict, /export/pack) và Quản trị (/admin/*).

dash (Streamlit): Giao diện điều khiển (Control Plane) cho Admin.

scraper (Python/httpx): Thu thập dữ liệu. Thực thi Abstraction Layer (NewsSource, PriceSource) và các quy tắc "Lean-Armor".

worker (Python/Polars): Xử lý Batch chính. Chịu trách nhiệm gọi Gemini API, chạy HMM, Snorkel, và tính toán Feature.

worker_bt (Python/VectorBT): Worker chuyên dụng (profile backtest) để chạy Backtest WFO nặng.

cron (Sidecar): Điều phối, gửi lệnh docker exec để kích hoạt worker và scraper theo lịch.

6. API Bên ngoài & Tích hợp (External APIs & Integration)

Đây là khu vực được "Hardening" (gia cố) trong V3.1.

6.1 Data Source Abstraction Layer (MỚI V3.1)

Hệ thống KHÔNG được phụ thuộc cứng vào bất kỳ thư viện hay API "ngầm" nào. Mọi hoạt động thu thập dữ liệu phải thông qua các Interface (Protocol) được định nghĩa trong core_lib/data_source.py:

PriceSource(Protocol):

get_ohlcv(symbols, start, end) -> DataFrame

Implementation ban đầu: VnstockPriceSource (dùng thư viện vnstock cho PoC/Dev).

Implementation tương lai: VendorPriceSource (dùng API trả phí chính thức).

NewsSource(Protocol):

fetch_latest(since: datetime) -> List[NewsItem]

Implementation ban đầu: CafeFSpider (parse HTML) hoặc FireAntApiSource (gọi API JSON).

6.2 Legal & Compliance Guardrails (MỚI V3.1)

Việc triển khai NewsSource phải tuân thủ nghiêm ngặt các quy tắc sau:

Robots.txt: Spider phải kiểm tra robots.txt của trang đích.

ToS (Terms of Service): Chỉ thu thập dữ liệu ở mức "Fair Use" (công khai, phi thương mại).

Data Minimization: Chỉ lấy Title + Sapo. Không lưu Full-text (NFR6).

No Bypass: Cấm tuyệt đối các hành vi bypass CAPTCHA, Cloudflare Challenge, hay các cơ chế bảo vệ nâng cao (PRD 9).

6.3 Tích hợp Cloud AI (Gemini API Integration)

Model: gemini-2.5-flash (Stable).

Phương thức: Batch API (gửi lô lớn) với Structured Output (ép buộc trả về JSON) để tối ưu chi phí và độ tin cậy.

Fallback: Nếu API lỗi (Quota, Content Policy), Worker sẽ ghi log, đẩy vào DLQ, và tiếp tục xử lý các tin khác (Fail-soft).

6.4 Kỷ luật "Lean-Armor" (Scraper)

Rate Limiting: Giới hạn 60 reqs/phút/domain.

Circuit Breaker: Ngắt mạch 10 phút nếu có 5 lỗi liên tiếp.

Backpressure: Tự động giảm tốc độ nếu cờ SCRAPE_SLOW bật.

7. Luồng công việc cốt lõi (Core Workflows)

7.1 Luồng Thu thập & Pipeline NLP (Ingestion & NLP) - [CẬP NHẬT V3.1]

Luồng này kết nối Scraper và Worker, đảm bảo dữ liệu được xử lý ngay lập tức.

sequenceDiagram
    participant Scraper as Scraper
    participant DB as PostgreSQL
    participant Worker as Worker
    participant Gemini as Gemini API

    Scraper->>External: 1. Fetch (qua Abstraction Layer)
    Scraper->>DB: 2. Validate (Dirty Check) & Insert `raw_bronze` (Idempotent)
    
    alt News Hợp lệ
        Scraper->>DB: 3. Enqueue `task_q (NLP_PROCESS)` (FR13.1)
    else News Rác
        Scraper->>DB: 3b. Ghi vào `raw_bronze_bad`
    end
    
    Note over Worker: Cron Trigger (nlp_batch)
    Worker->>DB: 4. Lấy task từ `task_q` (SKIP LOCKED)
    Worker->>Gemini: 5. Gửi Batch Request (Title+Sapo)
    Gemini-->>Worker: 6. Nhận JSON (Sentiment, Catalysts)
    Worker->>DB: 7. Ghi vào `sa_silver` & `catalyst_flags`


7.2 Luồng Batch Đêm (Nightly Batch Pipeline)

Đây là chuỗi Cron Job nối tiếp, đảm bảo thứ tự thực thi:

01:00 (nlp_batch): Chạy Gemini API $\to$ Ghi sa_silver.

02:00 (elitist_batch): Tính trọng số $\to$ Ghi account_weights.

02:30 (feature_gold_batch): Chạy HMM, tính HunterScore $\to$ Atomic Publish features_gold_serving.

02:40 (label_batch): Chạy Snorkel $\to$ Ghi labels_silver $\to$ Ghi al_queue.

02:50 (train_check_batch): Kiểm tra Triggers. Nếu "CÓ" $\to$ Huấn luyện, Ghi model_registry (với is_active=False).

7.3 Luồng Backtest & Auto-Rollback

Chủ Nhật, 03:00 (backtest_batch):

Chạy trên profile worker_bt (nhiều RAM).

So sánh 1v1 (Champion vs Challenger) bằng WFO.

Cập nhật promotion_suggestion ('ready_for_canary' hoặc 'rejected').

Hàng giờ (monitor_canary_batch):

Kiểm tra hiệu suất Canary live.

Nếu vi phạm Guardrail $\to$ Auto-Rollback: Tắt Canary, Bật lại Champion cũ, Gửi Alert.

8. Cấu trúc Thư mục (Source Tree)

Cấu trúc Monorepo được tối ưu cho V3.1:

stock-hunter-ai/
├── .github/
│   └── workflows/
│       └── ci.yml
├── db/
│   ├── migrations/
│   │   └── 02_add_admin_tables.sql
│   ├── init.sql                # Schema chính (đã thêm scraper_state V3.1)
│   └── views.sql               # Logic As-of
├── scripts/
│   ├── cron/                   # 9 cron jobs
│   └── backfill_news.py        # (MỚI V3.1) Script Backfill tin tức
├── src/
│   ├── api/                    # Service: API (FastAPI)
│   ├── core_lib/               # Library: Dùng chung
│   │   ├── core_lib/
│   │   │   ├── db.py
│   │   │   ├── locks.py
│   │   │   ├── queue.py
│   │   │   └── data_source.py  # (MỚI V3.1) Abstraction Layer
│   │   ├── tests/
│   │   └── pyproject.toml
│   ├── dash/                   # Service: Dashboard (Streamlit)
│   │   ├── pages/
│   │   │   ├── 01_Overview.py
│   │   │   └── 05_Macro_Impact_Config.py
│   │   ├── app.py
│   │   └── Dockerfile
│   ├── scraper/                # Service: Thu thập dữ liệu
│   │   ├── scraper/
│   │   │   ├── adapters.py
│   │   │   ├── ingestion.py
│   │   │   ├── main.py         # Entrypoint & Backpressure
│   │   │   └── utils.py        # Rate Limiter & Circuit Breaker
│   │   ├── Dockerfile
│   │   └── pyproject.toml
│   └── worker/                 # Service: Xử lý Batch & AI
│       ├── worker/
│       │   ├── main.py         # Xử lý Silver Layer
│       │   ├── tasks_nlp.py    # (MỚI V3.0) Tích hợp Gemini API
│       │   ├── tasks_feature_gold.py # HMM, HunterScore
│       │   ├── tasks_label.py      # Snorkel, AL Sampling
│       │   ├── tasks_train.py      # Huấn luyện (Local)
│       │   ├── tasks_backtest.py   # Backtest WFO (Local)
│       │   └── tasks_monitor.py    # Auto-Rollback (Local)
│       ├── Dockerfile          # (MỚI V3.0) Tối ưu: Đã loại bỏ torch
│       └── pyproject.toml
├── .env.example
├── Caddyfile
└── docker-compose.yml          # Đã cập nhật 8 services


9. Hạ tầng & Triển khai (Infrastructure & Deployment)

(File docker-compose.yml (V3.0) và Caddyfile đã phản ánh chính xác 8 services và cấu hình bảo mật. Giữ nguyên file architecture v3.0.md).

10. Tiêu chuẩn Triển khai (Implementation Standards)

10.1 Chiến lược Xử lý Lỗi (Error Handling)

API (AppAPI): Trả về JSON có cấu trúc lỗi rõ ràng. Trả 503 nếu KILL_SWITCH bật.

Worker & Scraper: Mọi task lỗi 3 lần $\to$ Đẩy vào task_q_dlq với lý do cụ thể.

10.2 Tiêu chuẩn Mã hóa (Coding Standards)

Không Import Chéo: api KHÔNG ĐƯỢC import từ worker hay scraper.

AppAPI Siêu Nhẹ: src/api TUYỆT ĐỐI KHÔNG cài đặt pandas, polars, scikit-learn.

Hybrid AI: NLP Bắt buộc dùng Gemini API. Quant (HMM, LGBM) chạy Local.

As-of View: Mọi truy vấn huấn luyện, backtest BẮT BUỘC đọc qua v_features_asof.

Advisory Locks: Mọi cron job phải gọi pg_try_advisory_lock trước khi chạy.

Abstraction: Mọi thao tác lấy dữ liệu bên ngoài BẮT BUỘC đi qua PriceSource / NewsSource.

10.3 Chiến lược Kiểm thử (Testing Strategy)

Unit Test: Tập trung vào core_lib (đặc biệt là asof.py, queue.py, data_source.py).

Integration Test: Test luồng task_q $\to$ worker $\to$ DLQ.

Time-Travel Test: Unit test (test_as_of_attack_failure) BẮT BUỘC phải tồn tại để xác minh logic as-of.

Smoke Test: curl /readyz trong CI.

Backtest (WFO): Chạy WFO có Compute Budget (Wallclock Limit, Symbol Cap).

10.4 Observability & Data Quality (MỚI V3.1)

Data Quality Rules (Ingestion):

Drop/Quarantine: Các bản ghi có published_at vô lý (tương lai > 1 ngày) hoặc title quá ngắn (< 10 chars) phải bị loại bỏ và ghi vào raw_bronze_bad (NFR9).

Scraper Metrics (Observability):

Hệ thống phải log vào monitoring_logs các chỉ số (FR14):

news_count_per_source_24h

nlp_lag_minutes (Thời gian từ ingestion đến khi NLP xong)

task_q_depth

ingestion_error_rate

10.5 Bảo mật & Vận hành (Security & Ops)

Secrets: Mọi API Key (GOOGLE_API_KEY), mật khẩu DB, Admin Key phải đọc từ .env.

Access Control: Proxy (Caddy) thực thi Basic Auth. API kiểm tra X-ADMIN-KEY (Defense in Depth).

Docker Optimization: Log Rotation (max-size: 100m) (NFR8).

Migrations: Mọi thay đổi schema phải thông qua file migration .sql (ví dụ: 02_add_admin_tables.sql).