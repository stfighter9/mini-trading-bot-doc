silver_logic_v1.md:
Đây chính là tài liệu `silver_logic_v1.md` chính thức, đã được phê duyệt.

Tài liệu này là một "gói triển khai" (implementation packet) hoàn chỉnh, bao gồm logic nghiệp vụ, DDL cho `db/init.sql`, và skeleton code cho `src/worker/main.py` để Jules thực thi.

-----

## Silver Logic v1 — Stock Hunter AI

**Story 1.3: Worker Xử lý (Silver Layer "Tinh gọn")**

### 1\. Mục tiêu

Viết worker AppWK đọc hàng đợi `task_q` bằng `FOR UPDATE SKIP LOCKED` trong advisory lock `silver_consume`, áp dụng 5+ "Cheap Sanity Rules", rồi:

  * **Pass:** `UPSERT` vào `ta_silver` hoặc `sa_silver` (Silver layer).
  * **Fail:** Đẩy sang `task_q_dlq` với `reason='sanity_fail'`.
  * **Backlog:** Nếu `task_q` \> 10k → set `control_flags('SCRAPE_SLOW')=true`.

### 2\. Kiến trúc thực thi

  * **File:** `src/worker/main.py`
  * **Entry:** `python -m worker.main silver_consume`
  * **Luồng chính:**
    1.  `pg_try_advisory_lock('silver_consume')` → nếu không lấy được → `exit 0` (AC2).
    2.  `SELECT ... FOR UPDATE SKIP LOCKED` một lô nhỏ (ví dụ: 500) từ `task_q` (AC1).
    3.  Với mỗi task: parse `payload.json`, định tuyến theo `task_type`:
          * `ta.bar` → áp dụng TA sanity → transform → `UPSERT ta_silver` (AC5).
          * `sa.article` → áp dụng SA sanity → transform → `UPSERT sa_silver` (AC5).
    4.  Nếu fail sanity → `INSERT task_q_dlq` (AC4).
    5.  Sau lô: check backlog → set `SCRAPE_SLOW` (AC7).
    6.  `COMMIT`, unlock.

### 3\. “Cheap Sanity Rules” (AC3)

Đây là các quy tắc "rẻ tiền" (không truy vấn DB) để "fail fast".

#### 3.1 TA (Giá/Khối lượng ngày)

Fail bất kỳ rule nào → DLQ.

1.  **Symbol:** Bắt buộc, `regex ^[A-Z]{3,7}$` (Cho phép mã 3 chữ cái và CW/ETF).
2.  **Trade Date:** Không `NULL`, dạng ngày hợp lệ, không ở tương lai.
3.  **Giá:** `open`, `high`, `low`, `close` không `NULL`, **`>= 0`**.
4.  **Quan hệ giá:** `high >= max(open, close)`, `low <= min(open, close)`, `high >= low`.
5.  **Khối lượng:** `volume` không `NULL`, **`>= 0`**.
6.  **Rule nghiệp vụ (Volume):** Nếu `volume == 0` thì `open == high == low == close` (phải là nến "chấm", thường là ngày nghỉ giao dịch).
7.  **Currency:** Bắt buộc (hoặc ngầm định 'VND').

#### 3.2 SA (Bài viết/Tin tức)

Fail bất kỳ rule nào → DLQ.

1.  **URL:** `url_canonical` không rỗng, hợp lệ `http/https`.
2.  **Timestamp:** `publisher_time` không `NULL`, `<= now() + 5m` (không ở tương lai xa).
3.  **Timestamp logic:** `first_seen_time` không được nhỏ hơn `publisher_time - 1 ngày` (chống dữ liệu cũ bất thường).
4.  **Language:** `language` phải thuộc `{vi, en}` (nếu khác, `set lang='unknown'`).
5.  **Nội dung:** `text_normalized` không `NULL`, **`len >= 120`** ký tự (bộ lọc rác hiệu quả, đã loại bỏ HTML).
6.  **Hash:** `content_hash` không rỗng (dùng để dedup).
7.  **Symbols:** Nếu `symbols[]` tồn tại, mỗi mã phải match `^[A-Z]{3,7}$`. (Nếu không có `symbols` → vẫn pass, ví dụ: bài vĩ mô).
8.  **Domain:** `source_domain` không rỗng.

### 4\. Mapping payload → Silver (AC5)

#### 4.1 `ta_silver`

  * **Khóa tự nhiên (Natural Key):** `(symbol, trade_date)`
  * **Upsert Logic:** `ON CONFLICT (symbol, trade_date) DO UPDATE`
  * **Logic nghiệp vụ:** Ghi đè (overwrite) bằng dữ liệu mới nhất (`EXCLUDED`), nhưng **giữ lại (audit)** thời điểm `first_seen_time` cũ nhất.

<!-- end list -->

```sql
-- Logic Upsert cho ta_silver
INSERT INTO ta_silver (symbol, trade_date, open, high, low, close, volume, ...)
VALUES (...)
ON CONFLICT (symbol, trade_date)
DO UPDATE SET
  open = EXCLUDED.open,
  high = EXCLUDED.high,
  low = EXCLUDED.low,
  close = EXCLUDED.close,
  volume = EXCLUDED.volume,
  vwap = EXCLUDED.vwap,
  adj_close = EXCLUDED.adj_close,
  source = EXCLUDED.source,
  -- Logic Audit: Giữ lại thời điểm nhìn thấy đầu tiên
  first_seen_time = LEAST(ta_silver.first_seen_time, EXCLUDED.first_seen_time),
  ingest_time = now(),
  content_hash = EXCLUDED.content_hash;
```

#### 4.2 `sa_silver`

  * **Khóa tự nhiên (Primary):** `url_canonical` (Nếu `UNIQUE`, không `NULL`).
  * **Khóa tự nhiên (Fallback):** `(source_domain, content_hash)` (Nếu `url_canonical` có thể `NULL`).
  * **Upsert Logic:** `ON CONFLICT (url_canonical) DO UPDATE`
  * **Logic nghiệp vụ (Rất quan trọng):** **"Làm giàu dữ liệu" (Enrichment)**. Chúng ta dùng `COALESCE` để đảm bảo dữ liệu mới chỉ cập nhật những trường bị `NULL` trước đó, không ghi đè mù quáng.

<!-- end list -->

```sql
-- Logic Upsert cho sa_silver
INSERT INTO sa_silver (url_canonical, source_domain, publisher_time, ...)
VALUES (...)
ON CONFLICT (url_canonical)
DO UPDATE SET
  -- Logic Audit: Giữ lại publisher/seen time cũ nhất
  publisher_time = LEAST(sa_silver.publisher_time, EXCLUDED.publisher_time),
  first_seen_time = LEAST(sa_silver.first_seen_time, EXCLUDED.first_seen_time),
  
  -- Logic Enrichment: Dùng COALESCE để làm giàu, không ghi đè
  language = COALESCE(EXCLUDED.language, sa_silver.language),
  title = COALESCE(EXCLUDED.title, sa_silver.title),
  text_normalized = COALESCE(EXCLUDED.text_normalized, sa_silver.text_normalized),
  
  -- Logic Enrichment cho Array: Chỉ cập nhật nếu array mới không rỗng
  symbols = CASE WHEN array_length(EXCLUDED.symbols, 1) IS NOT NULL
                 THEN EXCLUDED.symbols ELSE sa_silver.symbols END,
  
  -- Logic Enrichment cho Hype (nếu được tính toán sau)
  hype_raw = COALESCE(EXCLUDED.hype_raw, sa_silver.hype_raw),
  hype_crowd = COALESCE(EXCLUDED.hype_crowd, sa_silver.hype_crowd),
  hype_elitist = COALESCE(EXCLUDED.hype_elitist, sa_silver.hype_elitist),
  
  content_hash = EXCLUDED.content_hash,
  ingest_time = now();
```

### 5\. DLQ (AC4)

  * **Bảng:** `task_q_dlq(id, task_id, reason, rule_id, payload, error_msg, created_at)`
  * **Logic:** Khi fail sanity, `INSERT` vào `task_q_dlq` và `UPDATE` `task_q.status='dlq'`.

### 6\. Control flag `SCRAPE_SLOW` (AC7)

  * **Bảng:** `control_flags(name PRIMARY KEY, value boolean, updated_at)`
  * **Logic:** Sau mỗi batch, `SELECT count(*)` từ `task_q`.
      * Nếu `count > 10000`, `UPSERT control_flags SET value=true`.
      * Ngược lại, `UPSERT control_flags SET value=false`.

### 7\. DDL gợi ý (cho `db/init.sql`)

```sql
-- TA Silver
CREATE TABLE IF NOT EXISTS ta_silver (
  symbol TEXT NOT NULL,
  trade_date DATE NOT NULL,
  open DOUBLE PRECISION NOT NULL,
  high DOUBLE PRECISION NOT NULL,
  low  DOUBLE PRECISION NOT NULL,
  close DOUBLE PRECISION NOT NULL,
  volume BIGINT NOT NULL,
  vwap DOUBLE PRECISION,
  adj_close DOUBLE PRECISION,
  currency TEXT NOT NULL DEFAULT 'VND',
  price_multiplier DOUBLE PRECISION NOT NULL DEFAULT 1.0,
  source TEXT,
  first_seen_time TIMESTAMPTZ,
  ingest_time TIMESTAMPTZ NOT NULL DEFAULT now(),
  content_hash TEXT,
  PRIMARY KEY(symbol, trade_date)
);

-- SA Silver
CREATE TABLE IF NOT EXISTS sa_silver (
  url_canonical TEXT UNIQUE NOT NULL, -- Ưu tiên UNIQUE NOT NULL cho khóa chính
  source_domain TEXT NOT NULL,
  publisher_time TIMESTAMPTZ NOT NULL,
  first_seen_time TIMESTAMPTZ NOT NULL,
  language TEXT,
  title TEXT,
  text_normalized TEXT,
  content_hash TEXT NOT NULL,
  symbols TEXT[],
  author TEXT,
  topic_tags TEXT[],
  hype_raw DOUBLE PRECISION,
  hype_crowd DOUBLE PRECISION,
  hype_elitist DOUBLE PRECISION,
  account_weights_applied BOOLEAN,
  ingest_time TIMESTAMPTZ NOT NULL DEFAULT now(),
  PRIMARY KEY (url_canonical) -- Đơn giản hóa khóa
);
-- Chỉ mục (Index)
CREATE INDEX IF NOT EXISTS idx_sa_silver_pub ON sa_silver(publisher_time);
CREATE INDEX IF NOT EXISTS idx_sa_silver_symbols ON sa_silver USING GIN (symbols);
CREATE INDEX IF NOT EXISTS idx_sa_silver_hash_fallback ON sa_silver(source_domain, content_hash);

-- Task Queues
CREATE TABLE IF NOT EXISTS task_q (
  id BIGSERIAL PRIMARY KEY,
  task_type TEXT NOT NULL,   -- 'ta.bar' | 'sa.article'
  payload JSONB NOT NULL,
  status TEXT NOT NULL DEFAULT 'ready',
  priority INT NOT NULL DEFAULT 100,
  first_seen TIMESTAMPTZ NOT NULL DEFAULT now(),
  last_attempt TIMESTAMPTZ
);
CREATE INDEX IF NOT EXISTS idx_task_q_status ON task_q(status, priority, first_seen) WHERE status = 'ready';

CREATE TABLE IF NOT EXISTS task_q_dlq (
  id BIGSERIAL PRIMARY KEY,
  task_id BIGINT,
  reason TEXT NOT NULL,
  rule_id TEXT,
  payload JSONB,
  error_msg TEXT,
  created_at TIMESTAMPTZ NOT NULL DEFAULT now()
);

-- Control Flags
CREATE TABLE IF NOT EXISTS control_flags (
  name TEXT PRIMARY KEY,
  value BOOLEAN NOT NULL,
  updated_at TIMESTAMPTZ NOT NULL DEFAULT now()
);
```

### 8\. Skeleton `src/worker/main.py` (Pseudo-code)

```python
import json, os, sys, re
from contextlib import contextmanager
import psycopg

BATCH_SIZE = int(os.getenv("BATCH_SIZE", "500"))
BACKLOG_THRESHOLD = int(os.getenv("BACKLOG_THRESHOLD", "10000"))

@contextmanager
def pg_conn():
    # Kết nối tới DB
    with psycopg.connect(os.getenv("PG_DSN")) as conn:
        conn.autocommit = False
        yield conn

def try_lock(conn, lock_name):
    # (AC2) Thử lấy advisory lock
    with conn.cursor() as cur:
        cur.execute("SELECT pg_try_advisory_lock(hashtext(%s))", (lock_name,))
        ok = cur.fetchone()[0]
        if not ok:
            conn.rollback() # Không cần commit nếu chỉ SELECT
        return ok

def fetch_batch(conn):
    # (AC1) Lấy batch mới bằng FOR UPDATE SKIP LOCKED
    with conn.cursor() as cur:
        cur.execute("""
            WITH cte AS (
                SELECT id FROM task_q
                WHERE status='ready'
                ORDER BY priority, first_seen
                LIMIT %s
                FOR UPDATE SKIP LOCKED
            )
            SELECT t.id, t.task_type, t.payload
            FROM task_q t JOIN cte ON t.id = cte.id
        """, (BATCH_SIZE,))
        return cur.fetchall()

def apply_sanity_rules_ta(p):
    # (AC3) Triển khai các rule TA trong Mục 3.1
    # ...
    # if (fail): return False, 'ta_rule_name', 'failure message'
    return True, None, None

def apply_sanity_rules_sa(p):
    # (AC3) Triển khai các rule SA trong Mục 3.2
    # ...
    # if (fail): return False, 'sa_rule_name', 'failure message'
    return True, None, None

def upsert_ta(conn, p):
    # (AC5) Triển khai logic INSERT ... ON CONFLICT (Mục 4.1)
    pass

def upsert_sa(conn, p):
    # (AC5) Triển khai logic INSERT ... ON CONFLICT (Mục 4.2)
    pass

def move_to_dlq(conn, task_id, payload, reason, rule_id=None, error_msg=None):
    # (AC4) Ghi vào DLQ và cập nhật status task_q
    with conn.cursor() as cur:
        cur.execute("""
            INSERT INTO task_q_dlq(task_id, reason, rule_id, payload, error_msg, created_at)
            VALUES (%s, %s, %s, %s::jsonb, %s, NOW())
        """, (task_id, reason, rule_id, json.dumps(payload), error_msg))
        cur.execute("UPDATE task_q SET status='dlq', last_attempt=now() WHERE id=%s", (task_id,))

def mark_done(conn, task_id):
    # Xóa task đã xử lý thành công khỏi hàng đợi
    with conn.cursor() as cur:
        cur.execute("DELETE FROM task_q WHERE id=%s", (task_id,))

def backlog_control(conn):
    # (AC7) Kiểm soát cờ SCRAPE_SLOW
    with conn.cursor() as cur:
        cur.execute("SELECT count(*) FROM task_q WHERE status='ready'")
        n = cur.fetchone()[0]
        val = (n > BACKLOG_THRESHOLD)
        cur.execute("""
            INSERT INTO control_flags(name, value, updated_at) VALUES ('SCRAPE_SLOW', %s, NOW())
            ON CONFLICT (name) DO UPDATE SET value=EXCLUDED.value, updated_at=NOW()
        """, (val,))

def main():
    lock_name = 'silver_consume'
    with pg_conn() as conn:
        if not try_lock(conn, lock_name):
            print(f"Lock '{lock_name}' đang bận; worker thoát 0.")
            return 0
        
        try:
            tasks = fetch_batch(conn)
            if not tasks:
                print("Không có task nào, worker kết thúc.")
                conn.commit() # Vẫn commit để backlog_control chạy
            
            for task_id, task_type, payload in tasks:
                p = payload if isinstance(payload, dict) else json.loads(payload)
                try:
                    if task_type == 'ta.bar':
                        ok, rid, msg = apply_sanity_rules_ta(p)
                        if not ok:
                            move_to_dlq(conn, task_id, p, 'sanity_fail', rid, msg)
                            continue
                        upsert_ta(conn, p)
                        mark_done(conn, task_id)
                    
                    elif task_type == 'sa.article':
                        ok, rid, msg = apply_sanity_rules_sa(p)
                        if not ok:
                            move_to_dlq(conn, task_id, p, 'sanity_fail', rid, msg)
                            continue
                        upsert_sa(conn, p)
                        mark_done(conn, task_id)
                    
                    else:
                        move_to_dlq(conn, task_id, p, 'unknown_task_type')
                
                except Exception as e:
                    # Bắt lỗi trong vòng lặp để không làm hỏng toàn bộ batch
                    print(f"Lỗi xử lý task {task_id}: {e}")
                    move_to_dlq(conn, task_id, p, 'exception', error_msg=str(e))
            
            # (AC7) Luôn chạy backlog control sau mỗi batch
            backlog_control(conn)
            
            # Commit toàn bộ batch
            conn.commit()
            
        except Exception as e:
            print(f"Lỗi nghiêm trọng, rollback batch: {e}")
            conn.rollback()
        finally:
            # Luôn giải phóng lock
            with conn.cursor() as cur:
                cur.execute("SELECT pg_advisory_unlock(hashtext(%s))", (lock_name,))
            conn.commit() # Commit việc giải phóng lock
    
    return 0

if __name__ == '__main__':
    sys.exit(main())
```