Feature Logic v1 — Stock Hunter AI
Story 2.3: Batch Job feature_gold_batch (Atomic Publish)

Mục tiêu: Lúc 02:30 hằng ngày, tính HMM (filtering-only), các điểm số HunterScore, FrothScore, Macro_Impact_Score và các feature Macro/Breadth; đọc dữ liệu qua v_features_asof; xuất bản bảng rộng _serving theo cơ chế swap atomically.

Nguồn dữ liệu & Ngưỡng thời gian
Nguồn: Mọi phép tính phải đọc qua view: v_features_asof (NFR4).

Cửa sổ mặc định:

Z-score: 36 tháng cho macro; 180 phiên cho SA crowds; 252 phiên cho TA width/vol.

RS: 21 và 63 phiên.

Pocket-pivot: xét 10 phiên gần nhất trong nền 3–8 tuần.

Nhóm 1 — Định nghĩa “Tiếng ồn”: FrothScore (0–100)
Mục tiêu: Đo mức “sôi”/hưng phấn thiếu chất. Tính theo 4 trụ, chuẩn hóa 0–1 rồi quy về 0–100.

1. Cảm xúc đám đông (Crowd sentiment) — CÓ dùng
Logic: S_crowd = clip( z_pos(hype_crowd_z) , 0, +∞ )

Lý do: Chỉ có sự hưng phấn (z-score dương) của đám đông mới bị tính là "nhiễu". Sự sợ hãi (z-score âm) bị bỏ qua trong mô hình này.

2. Khối lượng tin tức không có xúc tác — CÓ tăng
Logic:

burst = z_pos(news_count_crowd)

S_burst = burst * ( catalyst_active ? 0.4 : 1.0 )

Lý do: Nhiều tin tức (burst) mà không có lý do (catalyst) là "tin rác" (nhân 1.0). Nếu có lý do, nó là "tin có chất" và được giảm trọng số (nhân 0.4).

3. Kỹ thuật “ồn”/bẫy — CÓ tính
Logic:

Parabolic slope: slope = rel(EMA5/EMA20 - 1)

Extension: ext = rel(close/MA50 - 1)

Gap/limit-up streak

Turnover acceleration: vol_3d/vol_10d

Breadth trái chiều: (price_up) & (%ma20_sector < 0.4)

Hợp nhất:

S_tech = w1*slope + w2*ext + w3*turnover + w4*gap_streak + w5*breadth_contra

(w1..w5 ≈ 0.25,0.25,0.2,0.2,0.1; chuẩn hóa 0–1)

4. Tổng hợp & Thang điểm
Logic:

Froth_raw = 0.25*S_crowd + 0.20*S_burst + 0.40*S_tech + 0.15*breadth_contra

FrothScore = round( 100 * clip(Froth_raw, 0, 1) )

Ngưỡng: ≤20 (Thấp), 21–60 (Vừa), >60 (Cao), >80 (Rất cao).

Nhóm 2 — Định nghĩa “Tín hiệu”: HunterScore (0–100)
Mục tiêu: Đo “tín hiệu tinh hoa” trước bứt phá. 3 trụ, chuẩn hóa 0–1, quy về 0–100.

1. Phân kỳ Elitist vs Crowd — LÕI (CÓ)
Logic:

div = z(hype_elitist_z) - z(hype_crowd_z)

H_div = clip( sigmoid(div), 0, 1 )

Lý do: Tín hiệu mạnh nhất là khi "tiền thông minh" (elitist) hành động trước khi "đám đông" (crowd) kịp nhận ra (đo lường Bất đối xứng thông tin).

2. TA “thông minh” đồng pha — CÓ thưởng
Logic:

Volatility contraction (VCP): ATR%20↓ & BBWidth↓

Pivot zone: nền 3–8 tuần, biên ≤25%

Pocket pivot/accumulation

RS21/RS63 top decile (Sức mạnh tương đối)

Hợp nhất: H_ta = wv*VCP + wp*Pocket + wrs*RS + wpivot*Pivot (Trọng số ~0.3 cho VCP, Pocket, RS).

3. Catalyst flags — CÓ thưởng
Logic: H_cat = min(1.0, Σ flag_i * weight_i )

Lý do: Cung cấp "lý do" cơ bản cho việc bứt phá (ví dụ: M&A, KQKD vượt kỳ vọng).

4. Tổng hợp & Thang điểm (trước khi phạt)
Logic:

Hunter_raw = 0.50*H_div + 0.30*H_ta + 0.20*H_cat

Hunter_base = clip(Hunter_raw, 0, 1)

HunterScore_pre = round( 100 * Hunter_base )

Nhóm 3 — Mối quan hệ Hunter vs Froth
1. Chiến lược chuẩn (Story 3.2)
Tín hiệu tốt nhất khi: HunterScore ≥ 80 VÀ FrothScore ≤ 20.

2. Phạt Hunter khi Froth cao — CÓ
Mục tiêu: Tự động "hạ nhiệt" tín hiệu khi thị trường quá hưng phấn (rủi ro cao).

Công thức phạt:

penalty = (1 - γ * FrothScore/100) (với γ gợi ý là 0.6)

Hunter_adj = clip( Hunter_base * penalty, 0, 1)

HunterScore = round( 100 * Hunter_adj )

Nhóm 4 — Macro_Impact_Score (thang -10..+10)
Mục tiêu: Gói gọn bối cảnh vĩ mô (gió thuận/gió ngược) thành một chỉ số duy nhất cho ngành.

1. Inputs
Các biến Z-score 36 tháng (ví dụ: z_cpi, z_ib7d, z_fx).

Bảng cấu hình dim_macro_sector_impact (chứa trọng số w[factor, sector]).

2. Chuẩn hóa dấu & Cường độ
Sử dụng tanh để đưa z-score về thang [-1, +1], giữ hướng tự nhiên của biến.

g_ib7d = -tanh(z_ib7d/2) (lãi suất tăng thường là gió ngược)

g_cpi = -tanh(z_cpi/2)

g_fx = tanh(z_fx/2)

3. Công thức
Gán trọng số α cho từng yếu tố vĩ mô (ví dụ: α_ib7d=0.35, α_cpi=0.25...).

S = α_ib7d * w[ib7d,sector]*g_ib7d + α_cpi * w[cpi ,sector]*g_cpi + ...

Macro_Impact_Score = round( 10 * clip(S, -1, 1) )

Macro_Impact_Bucket: Tailwind (≥+3) | Neutral (-2..+2) | Headwind (≤-3)

HMM (AC2) — Filtering-only, sticky bias, transition mask
Số trạng thái: 4 (Tích lũy, Bùng nổ, Hưng phấn, Phân phối).

Filtering-only: Ước lượng trạng thái tại T chỉ dùng dữ liệu đến T.

Sticky bias (κ): Thêm trọng số κ (ví dụ: 0.6) cho self-transition để giảm "nhấp nháy" trạng thái.

Transition mask: Chặn các bước chuyển phi thực tế (ví dụ: Tích lũy -> Phân phối).

Tính toán & Publish (AC1/AC3/AC6/AC9)
Lịch & Khóa (AC1):

Cron 02:30: feature_gold_batch()

BẮT BUỘC: SELECT pg_advisory_lock( hashtext('feature_gold_batch') );

Pipeline logic (AC3, AC4):

Đọc duy nhất từ v_features_asof ngày T.

Tính HMM (AC2).

Tính FrothScore, HunterScore (đã phạt), Macro_Impact_Score.

Ghi ra bảng rộng: features_gold_vX_tmp.

Validate: Kiểm tra NULL, số dòng, phân phối điểm số.

Chỉ mục (AC9):

CREATE INDEX CONCURRENTLY IF NOT EXISTS idx_fg_tmp_symbol ON features_gold_vX_tmp(symbol);

CREATE INDEX CONCURRENTLY IF NOT EXISTS idx_fg_tmp_effdate ON features_gold_vX_tmp(effective_date);

Swap - Atomic Publish (AC6):

Thực thi trong một transaction:

SQL

BEGIN;
ALTER TABLE features_gold_serving RENAME TO features_gold_vX_bak;
ALTER TABLE features_gold_vX_tmp RENAME TO features_gold_serving;
-- (Optional: DROP TABLE features_gold_vX_bak;)
COMMIT;
Mở khóa: SELECT pg_advisory_unlock( hashtext('feature_gold_batch') );

Tiện ích chuẩn hóa (gợi ý)
def z_pos(z): return max(0.0, z) def rel(x, lo=0.0, hi=0.3): return clip((x-lo)/(hi-lo), 0, 1) def clip(x, a=0.0, b=1.0): return min(max(x, a), b) def sigmoid(x): return 1/(1+exp(-x))