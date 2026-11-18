# EGM Analysis & Integration Blueprint: *Hands-On AI Trading with Python, QuantConnect, and AWS* × `day-trader-skill`

**วันที่วิเคราะห์:** 2026-11-18  
**เอกสารต้นทาง:** `Hands-on AI Trading with Python, QuantConnect, and AWS.pdf` / `.epub` (9 บท) และโค้ดใน `HandsOnAITradingBook-master`  
**โปรเจกต์เป้าหมาย:** `day-trader-skill`  
**ทีมวิเคราะห์:** `egm-codebase-details`, `egm-design`, `egm-implementation-readiness`, `egm-zoom-out`

---

## 1. สรุปเนื้อหาและคุณค่าของหนังสือทั้ง 9 บท (`egm-codebase-details`)

| บท | หัวข้อหลัก | สาระสำคัญและประโยชน์เชิงปฏิบัติ |
| :---: | :--- | :--- |
| **Ch. 1** | **Foundations of Capital Markets** | กลไกตลาด (SIP, NBBO, Limit Order Book), ความต่างของ `TradeBar` vs `QuoteBar` (Bid/Ask), ต้นทุนแฝง (Spread + Slippage), การจัดการ Corporate Actions (Stock Splits, Dividends) และโหมดปรับข้อมูล (`ADJUSTED`, `RAW`, `SCALED_RAW`) |
| **Ch. 2** | **Foundations of Quantitative Trading** | วงจรพัฒนา (`Research -> Backtest -> Optimization -> Deployment`), การป้องกัน Look-Ahead Bias (เช่น การใช้โมเดล LLM/Foundation ที่เทรนหลังช่วงเวลา Backtest), **Probabilistic Sharpe Ratio (PSR)**, และหลักการลด Overfitting (*Remove, Replace, Reduce*) |
| **Ch. 3** | **Step 1: Problem Definition** | การแปลงสมมติฐานการเทรดเป็นโจทย์ ML ที่วัดผลได้จริงภายใต้ข้อจำกัดด้านสภาพคล่อง ค่าคอมมิชชัน และเวลาถือครอง |
| **Ch. 4** | **Step 2: Dataset Preparation** | การทำความสะอาดข้อมูล, การทำให้อนุกรมเวลานิ่งโดยไม่สูญเสียความจำแนวโน้มด้วย **Fixed-Window Fractional Differentiation (`frac_diff_ffd`)**, การทดสอบ Cointegration (Engle-Granger), **Hurst Exponent** (H < 0.5 Mean Reversion vs H > 0.5 Trending) และ `PCA` |
| **Ch. 5** | **Step 3: Model Choice, Training & Application** | คู่มือโมเดล 5 กลุ่ม: Regression (OLS, LASSO, Ridge, `MarkovRegression`, DecisionTree, Wavelet+SVR), Classification (LightGBM RF, HMM, GaussianNB, Conv1D+LSTM), Ranking (`LGBMRanker`), Clustering (`OPTICS`), และ Foundation/Language Models (`GPT-4`, `Chronos`, `FinBERT`) |
| **Ch. 6** | **Applied Machine Learning (19 Examples)** | กรณีศึกษา 19 ตัวอย่าง (25 โปรเจกต์ย่อยในโฟลเดอร์ `06 Applied Machine Learning`) ครอบคลุมหุ้น, ออปชัน, ฟิวเจอร์ส, ค่าเงิน และคริปโต |
| **Ch. 7** | **Better Hedging with Reinforcement Learning** | การใช้ Policy Gradient RL (`PyTorch`) ทำ Delta Hedging ออปชันแบบคิดค่าคอมมิชชัน โดย Pre-train กับสูตร Black-Scholes ก่อน Fine-tune กับข้อมูลตลาดจริง |
| **Ch. 8** | **AI for Risk Management and Optimization** | **บทที่มีคุณค่าสูงสุดสำหรับการเทรดจริง** (เขียนโดย Ernest Chan): แนวคิด **Corrective AI (Meta-Labeling)** ใช้โมเดลชุดที่สองทำนายโอกาสชนะ (`PoP`) ของสัญญาณหลักเพื่อคัดกรอง/ปรับขนาดไม้ และ **Conditional Parameter Optimization (CPO)** ปรับพารามิเตอร์ตามสภาวะตลาด |
| **Ch. 9** | **LLMs and Generative AI in Trading** | การเลือกใช้ LLM, Prompt Engineering, และสถาปัตยกรรม RAG บน AWS (S3 + Kendra + SageMaker Canvas) พร้อมตารางต้นทุนจริง (~$7,283/เดือน) |

---

## 2. การตรวจสอบโค้ดทั้ง 25 โปรเจกต์ใน `HandsOnAITradingBook-master`

| โฟลเดอร์ตัวอย่าง | สินทรัพย์ | Timeframe / ความถี่ปรับพอร์ต | โมเดลที่ใช้ | มี `main.py`? | ผล Backtest (`footnotes.txt`) |
| :--- | :--- | :--- | :--- | :---: | :--- |
| **`00 Libraries/backtestlib`** | Generic | Daily Close-to-Close | `pandas`, `plotly` | Helper | ไม่คิดค่าคอม/Spread |
| **`06 / 01 ML Trend Scanning`** | `BTCUSD` | Daily | OLS t-statistic (`trend_scanning_labels.py`) | ไม่มี (`ipynb` + helper) | ไม่มี `footnotes.txt` |
| **`06 / 02 Factor Preprocessing`** | `SPY` | Daily (5d fwd) | LightGBM RF + `ffd` + `PCA` | ไม่มี (`ipynb` เท่านั้น) | OOS Acc: Raw `0.6101` > FFD `0.5849` |
| **`06 / 03 Reversion vs Trending`** | `SPY`, `TLT`, `VIX` | Daily | Keras MLP Classifier | ไม่มี (`ipynb` เท่านั้น) | OOS Acc `54.83%` |
| **`06 / 04 / 01–03 HMM`** | `SPY`/`TLT`, Options, `SPX` | Daily | `statsmodels` `MarkovRegression(k_regimes=2)` | มี (`02` มี SyntaxError บรรทัดแรก) | Equities Sharpe `1.041`, DD `13.4%` |
| **`06 / 05 FX SVM Wavelet`** | 4 คู่เงิน FX | Daily (Leverage 20x) | `pywt` (`sym10`) + `SVR` | มี | Sharpe `0.664`, DD `6.7%` |
| **`06 / 06 Dividend Harvesting`** | Top 100 `QQQ` | Monthly | `DecisionTreeRegressor` | มี | Sharpe `0.674`, DD `26.8%` |
| **`06 / 07 Splits`** | หุ้น Tech (`XLK`) | Event-driven (ถือหลายวัน) | `LinearRegression` | มี | Sharpe `0.677`, DD `35.0%` |
| **`06 / 08 / 01–03 Dynamic Stop`** | `KO` + `VIX` | Weekly | `Lasso(alpha=1e-4)` ทำนาย Low | มี | ML Stop Sharpe `0.631` (DD `16%`) ชนะ Fixed Stop (`0.219`, DD `36.3%`) |
| **`06 / 09 Pairs Selection`** | หุ้นใน `IWB` (927 ตัว) | Daily (3 ปี) | `PCA` + `OPTICS` + Cointegration + Hurst | ไม่มี (`ipynb` เท่านั้น) | คัดได้ 30 คู่ Mean-Reverting |
| **`06 / 10 Fundamental Clustering`** | หุ้น Top 100 | Monthly | `PCA` + `LGBMRanker` | มี | Sharpe `0.68`, DD `45.3%` |
| **`06 / 11 Inverse Vol Futures`** | 12 สัญญา Futures | Weekly | `Ridge` ทำนายความผันผวน (w_i ∝ 1/σ_i) | มี | Sharpe `0.819`, DD `18.3%` |
| **`06 / 12 Trading Costs Opt.`** | `BTCUSDC` | **Intraday (Minute)** | `DecisionTreeRegressor` ประเมินต้นทุนแฝง | มี + `ipynb` | ประหยัดต้นทุนได้ `$6,888.5` ใน 366 ไม้ |
| **`06 / 13 PCA StatArb`** | หุ้น Top 100 | Monthly | `PCA(3)` + `OLS` | มี | Sharpe `0.903`, DD `39.1%` |
| **`06 / 14 Temporal CNN`** | Top 3 `QQQ` | Weekly | Keras `Conv1D` | มี | Sharpe `1.044` (หมายเหตุระบุว่าพารามิเตอร์ส่วนใหญ่ขาดทุน) |
| **`06 / 15 Gaussian Classifier`** | Top 10 Tech | Weekly (ถือ 22 วัน) | `GaussianNB` | มี | Sharpe `1.379`, DD `26.5%` |
| **`06 / 16 LLM Tiingo News`** | `TSLA` | **Intraday (Hourly)** | `GPT-4` Sentiment + `ROC(2)` | มี + `ipynb` | Sharpe `2.357` (แต่ทดสอบแค่ 4 เดือน) |
| **`06 / 17 Head & Shoulders CNN`** | `USDCAD` | Daily (ถือ 10 วัน) | `Conv1D` เทรนด้วยกราฟจำลอง | มี + `ipynb` | Sharpe `0.008`, CAGR `1.592%` |
| **`06 / 18 / 01–02 Amazon Chronos`** | Top 5 หุ้นสภาพคล่องสูง | Quarterly (63 วัน) | `amazon/chronos-t5-tiny` + `SLSQP` | มี | Sharpe `0.854` / `0.87` (ติด Look-Ahead Bias ของโมเดลปี 2024) |
| **`06 / 19 / 01–02 FinBERT`** | หุ้นผันผวนสูงสุด 1 ตัว | Monthly | `ProsusAI/finbert` | มี | Base Sharpe `1.531` > Fine-tuned `1.124` |
| **`07 / 01 RL Option Hedging`** | `TSLA` / `AAPL` Options | Daily | `PyTorch` Policy Gradient | มี (ใช้ `aihedging`) | เทรน Refinement บนข้อมูลจริงเพียง ~40 วัน |
| **`08 / 01–02 Risk & Corrective AI`** | ETFs / FX / `GLD`-`GDX` | Daily & **1m Intraday** | `PredictNowClient` (SaaS) | มีเฉพาะ `08/01` | ยก Sharpe จาก `0.88` เป็น `1.29` และลด DD ครึ่งหนึ่งในหนังสือ |

---

## 3. การวิเคราะห์จุดแข็งและจุดอ่อนเชิงวิพากษ์ (`egm-design`)

### 3.1 จุดแข็งที่นำมาสร้างความได้เปรียบ (Real Edge) ให้ `day-trader-skill` ได้ทันที
1. **Corrective AI / Meta-Labeling (Ch. 8):** แทนที่จะให้ AI เดาว่าหุ้นจะขึ้นหรือลงตั้งแต่ศูนย์ ให้ใช้สัญญาณเข้าออกที่ชัดเจนของ `day-trader-skill` (เช่น ORB หรือ Level Breakout) เป็นสัญญาณหลัก แล้วใช้โมเดลคำนวณ **โอกาสชนะ (Probability of Profit)** และ **ขนาดไม้ (Bet Sizing)** จากสภาวะตลาดและ Order Flow
2. **MLFinLab Trend Scanning (`t-statistic` ของ OLS Slope, Ch. 6 Ex. 01):** วัดความแข็งแกร่งของเทรนด์ด้วยค่าสถิติ t-value ข้ามหลายหน้าต่างเวลา แทนการใช้กฎตัดเส้นค่าเฉลี่ยแบบตายตัว
3. **Dynamic Volatility & Drawdown-Recovery Stop-Loss (Ch. 6 Ex. 08):** ตั้งจุดตัดขาดทุนตามความผันผวนจริง (`VIX`, `ATR`, `STD`) และความน่าจะเป็นในการฟื้นตัวจาก Drawdown ซึ่งในหนังสือพิสูจน์แล้วว่าลด Max Drawdown จาก `36.3%` เหลือ `16.0%` และเพิ่ม Sharpe เกือบ 3 เท่าเมื่อเทียบกับ Fixed Stop
4. **Execution Cost & Spread Filter (Ch. 6 Ex. 12):** ประเมินต้นทุนการเข้าออก (`spread_pct`, `atr`, `volume`, `top_of_book_size`) เพื่อชะลอคำสั่งในนาทีที่ Bid-Ask Spread ถ่างผิดปกติ
5. **Sentiment Rate-of-Change (`ROC`, Ch. 6 Ex. 16):** การดู **"อัตราการเปลี่ยนแปลงของอารมณ์ข่าว" (ΔS_t ≥ 0)** แม่นยำกว่าการดูแค่ค่า Sentiment เป็นบวก เพราะช่วยจับจังหวะฟื้นตัวและคัดกรองข่าวร้ายที่กำลังแย่ลงต่อเนื่อง
6. **Market Microstructure & Corporate Actions (Ch. 1–2):** การปรับข้อมูล Stock Splits (`SCALED_RAW`) และการคิดราคา Fill บนฝั่ง Bid/Ask จริง

### 3.2 จุดอ่อนและกับดักในหนังสือที่ต้องระวัง (Do Not Blindly Copy)
1. **Timeframe Mismatch (16 จาก 19 ตัวอย่างไม่ใช่ Day Trading):** ตัวอย่างส่วนใหญ่ในบทที่ 6 เป็นการถือระดับสัปดาห์ เดือน หรือไตรมาส (เช่น Chronos ถือ 63 วัน, FinBERT ปรับพอร์ตเดือนละครั้ง, Dividend/Fundamental ถือรายเดือน) ถ้านำโค้ดมาใช้กับกราฟ 1–5 นาทีโดยตรงจะพังเพราะค่าคอมและ Spread
2. **Look-Ahead Bias & Data Leakage ในโค้ดตัวอย่าง:**
   - Ex. 18 (Chronos) และ Ex. 19 (FinBERT) รัน Backtest ย้อนไปปี 2019–2022 ด้วยโมเดลที่เทรนข้อมูลถึงปี 2023–2024
   - Ex. 14 (Temporal CNN) ทำ `StandardScaler` บนหน้าต่างราคาทั้งชุดข้อมูลก่อนตัดแบ่ง และ `footnotes.txt` ยอมรับเองว่า *"พารามิเตอร์ส่วนใหญ่ขาดทุน"*
   - `backtestlib.py` คิดกำไร Close-to-Close โดยไม่หักค่าคอมมิชชันและ Slippage เลย
3. **Training Truncation & Overfitting:** Ex. 18.2 ลด `max_steps` การ Fine-tune เหลือแค่ `3` steps เพื่อไม่ให้เกินโควตาเวลา 10 นาทีของ QuantConnect Cloud; Ex. 16 ทดสอบกับหุ้นตัวเดียว (`TSLA`) แค่ 4 เดือน
4. **Platform Lock-In & SaaS Paywall:** บทที่ 8 เรียกใช้ `PredictNow.ai` (SaaS ปิดโค้ด) และบทที่ 9 ใช้ AWS Kendra + SageMaker ที่มีค่าใช้จ่ายสูงถึง `$7,283/เดือน` — สำหรับโปรเจกต์ของเรา ต้องแปลงมาใช้ไลบรารี Open-Source ในเครื่อง (`statsmodels`, `scikit-learn`, `lightgbm` หรือ Pure Python stdlib) เท่านั้น

---

## 4. การจัดหมวดหมู่เทคนิคจากหนังสือเพื่อนำมาใช้กับ `day-trader-skill`

| หมวดหมู่ | เทคนิค / บทในหนังสือ | แนวทางประยุกต์ใช้กับ `day-trader-skill` |
| :--- | :--- | :--- |
| **A. นำมาใช้ได้ทันที (คุณค่าสูงสุดสำหรับ Day Trading)** | • **Ch. 1–2:** Split Adjustment & Spread/Slippage Modeling<br>• **Ch. 6 Ex. 01:** Trend Scanning (t-statistic)<br>• **Ch. 6 Ex. 12:** Execution Cost & Spread Guard<br>• **Ch. 8:** Corrective AI (Meta-Labeling Filter & Sizing) | ใช้แก้จุดอ่อน Critical ของ `day-trader-skill` โดยตรง: ปรับราคา Split ใน `scan.py`/`timemachine.py`, ให้คะแนนความแข็งแกร่งเทรนด์ด้วย t-stat, หัก Slippage ใน Time Machine และกรองไม้ที่มีโอกาสแพ้สูงออกก่อนส่งออเดอร์ |
| **B. นำมาดัดแปลงให้เข้ากับ Intraday ได้** | • **Ch. 6 Ex. 03 & 04:** Regime Detection (HMM / Volatility Classifier)<br>• **Ch. 6 Ex. 08:** Volatility & Drawdown-Recovery Stop-Loss<br>• **Ch. 6 Ex. 11:** Inverse-Volatility Position Sizing<br>• **Ch. 6 Ex. 13:** PCA Portfolio Concentration Check<br>• **Ch. 6 Ex. 16 & 19:** LLM/FinBERT News Sentiment + `ROC` | ปรับสเกลเวลาจากรายสัปดาห์/รายเดือน มาเป็น Pre-Market & Intraday (5m/15m): คัดแยกว่าวันนี้ตลาดเป็น `TRENDING` หรือ `MEAN_REVERTING`, ตั้ง Stop ตามความผันผวนจริง, ลดขนาดไม้ในหุ้นที่เหวี่ยงแรง และบล็อกการเปิดหุ้นที่วิ่งทิศเดียวกันเกินไป |
| **C. ไม่ควรนำมาใช้ (ไม่เหมาะกับ Retail Day Trading)** | • **Ch. 6 Ex. 02, 05, 06, 07, 10, 14, 15, 17, 18**<br>• **Ch. 7 (RL Option Hedging)**<br>• **Ch. 9 ($7.2k/mo AWS Kendra RAG)** | เป็นกลยุทธ์ถือยาวรายเดือน/ไตรมาส, มีปัญหา Overfitting สูง, หรือมีต้นทุน Cloud สูงเกินความจำเป็น |

---

## 5. แผนผังสถาปัตยกรรมและแผนพัฒนา 4 เฟส (`egm-implementation-readiness`)

### สถาปัตยกรรม "Dual-Ring" Hybrid (รักษาความเสถียร 100% + เพิ่มพลัง Quant/AI)
เพื่อให้ `day-trader-skill` ยังคงรันได้ทันทีด้วย **Python Standard Library** (และผ่านชุดทดสอบ 317 ข้อเดิม 100%) พร้อมกับเปิดรับโมเดล Quant/AI จากหนังสือ จึงออกแบบเป็น 2 วงแหวนที่คุยกันผ่านไฟล์ `trader/*.json`:

```text
┌────────────────────────────────────────────────────────────────────────────┐
│ RING 1: DETERMINISTIC CORE & GUARDRAILS (Pure Python 3.8+ Stdlib)          │
│  - trader_lib/: clock, nyse_calendar, market_data (split-adjusted),        │
│    contracts (fingerprint + client_order_id), fs_atomic (FileLock)         │
│  - stats_stdlib.py: Pure-Python OLS Trend t-stat, Wilder ATR,              │
│    Stationary Block Bootstrap, Inverse-Volatility Sizing                   │
│  - Hardened Skills: scanner, news, entry, exit, timemachine, budget,       │
│    risk, broker, alerts                                                    │
└───────────────────────────────▲────────────────────────────────────────────┘
                                │ เชื่อมต่อผ่าน JSON State Bus
                                │ (regime.json, sentiment.json, plan.json)
┌───────────────────────────────▼────────────────────────────────────────────┐
│ RING 2: OPTIONAL QUANT / AI SIDECAR (`trader_quant/`)                      │
│  - regime_hmm.py (Ch.6 Ex.03/04: HMM + Regime Classifier)                  │
│  - dynamic_stop.py (Ch.6 Ex.08: Volatility & MAE Drawdown Stop)            │
│  - cost_model.py (Ch.6 Ex.12: Slippage & Spread Cost Estimator)            │
│  - sentiment_nlp.py (Ch.6 Ex.16/19: FinBERT / LLM Sentiment + 2h ROC)      │
└────────────────────────────────────────────────────────────────────────────┘
```

### แผนการพัฒนาแบบเป็นขั้นตอน (Phased Roadmap)

#### **Phase 1: สร้างฐานรากความแม่นยำและความปลอดภัยสำหรับเทรดจริง (`trader_lib/` + Hardening)**
1. **สกัด `trader_lib/`:** รวมโค้ดเวลา Eastern Time, ปฏิทินวันหยุด NYSE อัตโนมัติ (2020–2040), `fingerprint()`, `make_client_order_id()`, และ `FileLock` (`msvcrt`/`fcntl`) ไว้ที่เดียว
2. **รองรับ Stock Splits (`SCALED_RAW` จาก Ch. 1 & Ch. 6 Ex. 07):** ปรับราคาและ Volume ย้อนหลังอัตโนมัติเมื่อตรวจพบ `events.splits` จาก Yahoo `v8`
3. **อัปเกรด `trader-risk` และ `trader-broker`:**
   - นับ `pdt_guard` ต่อเนื่องใน Batch เดียวกัน
   - ติดตามเงินสดรอชำระราคา `T+1` (`unsettled_cash`) ป้องกัน Good Faith Violation (GFV) ในบัญชีเงินสด
   - กำหนดอายุ `risk.json` (TTL 120 วินาที), บีบอายุราคา `FRESH_MINUTES` เหลือ 2 นาทีสำหรับ Market Order, และเพิ่มคำสั่ง `ticket.py sync` เพื่อซิงค์สถานะออเดอร์/พอร์ตจริงจากโบรกเกอร์

#### **Phase 2: อัปเกรด Time Machine ด้วย Block Bootstrap & ต้นทุนแฝง (Ch. 1 & Ch. 6 Ex. 12)**
1. **Stationary Block Bootstrap:** สุ่มแท่งเทียน 5 นาทีเป็นบล็อกต่อเนื่อง (3–6 แท่ง) โดยคัดจากวันที่เปิดตลาดด้วยความผันผวนระดับเดียวกัน เพื่อรักษาอาการเกาะกลุ่มของความผันผวน (Volatility Clustering)
2. **Spread & Slippage Modeling (`SpreadSlippageModel`):** หักค่า Spread/Slippage ทุกครั้งที่เข้าและออก และคิดการขาดทุนเกิน `-1.0R` เมื่อราคาเปิดกระโดดข้ามจุด Stop (`Gap-through-stop`)
3. **ตัด Survivorship Bias:** เก็บวันที่ตลาดมีความผิดปกติหรือแท่งเทียนไม่ครบ 78 แท่งไว้ในการทดสอบย้อนหลัง

#### **Phase 3: ยกระดับสัญญาณเข้า-ออกและขนาดไม้ด้วย Quant Models (Ch. 6 Ex. 01, 03, 04, 08, 11 & Ch. 8)**
1. **Trend Scanning (t-statistic, Ex. 01) & Regime Filter (Ex. 03/04):**
   - คำนวณ `trend_t_value` ใน `scan.py` และ `entry.py` เพื่อคัดเฉพาะหุ้นที่มีแนวโน้มแข็งแกร่งทางสถิติ (|t| ≥ 2.0)
   - แยกสภาวะตลาดรายวัน (`TRENDING` vs `MEAN_REVERTING`) เพื่อเลือกกลยุทธ์ให้ตรงกับสภาพตลาด
2. **Dynamic Volatility & Drawdown-Recovery Stop (`--stop vol-dd`, Ex. 08):**
   - เพิ่มโหมดคำนวณระยะ Stop จากความผันผวนและสถิติ Maximum Adverse Excursion (MAE) 60 วันย้อนหลังใน `exit.py`
3. **Inverse-Volatility Position Sizing (`--inv-vol`, Ex. 11) & Corrective AI Sizing (Ch. 8):**
   - ปรับลดงบความเสี่ยงต่อไม้อัตโนมัติเมื่อหุ้นมีความผันผวนสูงกว่าค่ามาตรฐาน (w_i ∝ 1/σ_i)

#### **Phase 4: ระบบกรองข่าว AI Sentiment ROC และความเสี่ยงพอร์ต PCA (Ch. 6 Ex. 13, 16, 19)**
1. **Sentiment Score & 2-Period ROC (`trader-news`):**
   - ให้คะแนนข่าวย้อนหลังและคำนวณอัตราการเปลี่ยนแปลงของอารมณ์ข่าว (ΔS_t) พร้อมสั่ง `hard_veto` ทันทีหากพบเอกสาร `8-K` ที่มีความเสี่ยงร้ายแรงหรืออารมณ์ข่าวติดลบและกำลังดิ่งลง
2. **PCA / Correlation Guard (`trader-risk`):**
   - ป้องกันการเปิดสถานะหุ้นตัวใหม่ที่มีค่าสหสัมพันธ์ (Correlation) กับหุ้นที่ถืออยู่แล้วเกิน `0.80` (ลดความเสี่ยงการถือหุ้นกลุ่มเดียวกันซ้ำซ้อน)
