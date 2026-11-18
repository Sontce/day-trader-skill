# EGM Audit & Development Report: `day-trader-skill`

**วันที่วิเคราะห์:** 2026-11-18  
**โปรเจกต์:** `day-trader-skill`  
**ผู้ตรวจสอบ:** EGM Analysis Team (`egm-codebase-details`, `egm-design`, `egm-zoom-out`, `egm-comprehension`)  
**สถานะชุดทดสอบ (หลังแก้ไขล่าสุด):** `317 / 317` tests passed (`OK`, 23.589s)

---

## 1. บทสรุปผู้บริหาร (Executive Summary & Verdict)

**คำตัดสินความเหมาะสมในการใช้งานจริง:**  
**เหมาะสำหรับการศึกษา, การจำลอง/ทดสอบกลยุทธ์ (Backtest / Monte Carlo) และ Paper Trading แบบ Human-in-the-Loop เท่านั้น — ยังไม่เหมาะสมสำหรับ Live Day Trading ด้วยเงินจริง**

### จุดแข็งของโปรเจกต์
- **Zero Dependencies:** ใช้ Python 3.8+ standard library ล้วน ไม่ต้อง `pip install` (ยกเว้น `pdftotext` ของ Poppler ที่เป็น optional ใน `trader-pelosi`)
- **Offline-First & Reproducible:** มี `--offline` และ `--now` รองรับการทดสอบย้อนหลังบนข้อมูลจริงโดยไม่ยิงเน็ต
- **Safety Posture:** มีระบบจำลองความเสี่ยง (`trader-risk`), การทำ SHA-256 `fingerprint` ผูกกับออเดอร์, และการบังคับให้ผู้ใช้พิมพ์ `"yes"` ยืนยันทีละออเดอร์ใน `trader-broker` รวมถึง Paper Trading เป็นค่าเริ่มต้น

### เหตุผลหลักที่ยังไม่เหมาะกับเงินจริง (Live Capital)
1. **ไม่มี Broker Sync (Local-vs-Broker Split-Brain):** ระบบไม่อ่านสถานะ Position/Order จริงจากโบรกเกอร์ หาก Stop-Loss หรือ Take-Profit จับคู่ฝั่งโบรกเกอร์แล้ว ไฟล์ `trader/portfolio.json` และ `trader/state/broker.json` จะไม่อัปเดตอัตโนมัติ
2. **ช่วงเวลาไร้ Stop-Loss ระหว่าง `flatten` Bracket Order:** ขั้นตอน `flatten` สั่งให้ยกเลิกขากันขาดทุน (`cancel_order_by_id`) ก่อนแล้วค่อยส่งคำสั่งขาย Market ผ่านการอนุมัติของมนุษย์ ทำให้เกิดช่องว่างที่พอร์ตไม่มี Stop-Loss คุ้มครอง
3. **ข้อมูลตลาด Unofficial & Delayed:** พึ่งพา Yahoo Finance `v7/finance/spark` และ `v8/finance/chart` ที่ไม่มี SLA, สแกนทั้งตลาด ~5,900 ตัวใช้เวลาหลายนาที, ยอมรับราคาเก่าได้ถึง 15 นาที (`FRESH_MINUTES = 15`) เพื่อคำนวณ Market Order และไม่ได้ปรับข้อมูลจาก Stock Splits (`events.splits`)
4. **ความเสี่ยงด้านกฎ T+1 / GFV และ File Concurrency:** คืนเงินสดทันทีหลังขาย (เสี่ยง Good Faith Violation ในบัญชี Cash Account) และไม่มี OS File Lock (`fcntl` / `msvcrt`) รองรับการรันพร้อมกัน
5. **ข้อจำกัดทางสถิติของ Time Machine:** สุ่มแท่งเทียน 5 นาทีแบบ IID Bootstrap (ทำลาย Volatility Clustering), ตัดวันที่แท่งไม่ครบ 78 แท่งออก (Survivorship / Tail-Risk Bias) และไม่หัก Spread / Slippage / ค่าธรรมเนียม

---

## 2. ความสามารถของทั้ง 11 สกิล (Capability Matrix)

Pipeline หลักเชื่อมต่อกันผ่านโฟลเดอร์ `trader/*.json` และ `trader/state/*.json`:

```text
scanner -> news -> entry -> exit -> timemachine -> budget -> risk -> broker
  └── alerts (เฝ้า stop / target / flat_by ระหว่างถือ)
  └── analyst / pelosi (ตรวจงบ SEC EDGAR และรายงานซื้อขายสภา)
```

| ลำดับ | สกิล | สคริปต์หลัก | คำสั่งหลัก | แหล่งข้อมูล | หน้าที่หลัก |
| :---: | :--- | :--- | :--- | :--- | :--- |
| 1 | `/trader-scanner` | `trader-scanner/scan.py` | `universe`, `daily`, `orb` | Nasdaq Trader, Yahoo `spark` & `v8` | สร้างรายชื่อหุ้นสหรัฐ (~5,900 ตัว), สแกนกราฟ Daily (`breakout_20d`, `bull_flag`, `reclaim_ma20`, `oversold`, `oversold_bounce`) และสแกน Opening-Range Breakout 5 นาที (`orb`) |
| 2 | `/trader-news` | `trader-news/news.py` | CLI (`--tickers` / ไฟล์) | Yahoo RSS, Yahoo `v8`, SEC EDGAR `8-K` | ติดตามข่าวด่วน, ราคาขยับ `>= 3%`, เอกสาร `8-K` (ต้องตั้ง `SEC_USER_AGENT`), ติดแท็กข่าว และจำข่าวเดิม 30 วัน |
| 3 | `/trader-entry` | `trader-entry/entry.py` | `check`, `watch`, `watchlist` | Yahoo `v8` (`1m`/`5m`) | เฝ้าดูจังหวะเข้าฝั่ง Long (`orb` หรือ `level`) ทีละแท่งเทียน พร้อมตรวจกรอบเวลาตลาด NYSE |
| 4 | `/trader-exit` | `trader-exit/exit.py` | `plan`, `show` | Yahoo `v8` (เมื่อใช้ `atr:K`) | วางแผน `stop` (`or-mid`, `or-low`, `price`, `pct`, `atr:K`), `target` (`NR`, `price`) และเวลาปิดสถานะ `flat_by` (`15:55` ET) ด้วย `Decimal` |
| 5 | `/trader-timemachine` | `trader-timemachine/timemachine.py` | `run`, `chart` | Yahoo `v8` (`60d`, `5m`) | จำลอง Monte Carlo 10,000 เส้นทาง + ย้อนรันกฎ ORB บนประวัติจริง + วาดกราฟ ASCII Fan Chart |
| 6 | `/trader-budget` | `trader-budget/size.py` | CLI (Bracket / Weights) | Local JSON (`plan.json`, `config.json`) | คำนวณจำนวนหุ้นเต็มหน่วยจาก `min(max_loss, position_cap, max_order, cash)` และเขียน Bracket Order รูปแบบ Alpaca |
| 7 | `/trader-risk` | `trader-risk/check.py` | CLI | Local JSON (`orders.json`, `portfolio.json`, `state/broker.json`) | ตรวจกฎความเสี่ยงทั้งหมด (Max Loss, Daily Loss, R:R, Max Trades, Trading Window, `pdt_guard`) และออก SHA-256 `fingerprint` (`PASS` / `BLOCK`) |
| 8 | `/trader-broker` | `trader-broker/ticket.py` | `show`, `approve`, `decline`, `result`, `fill`, `flatten`, `log` | Local JSON + Alpaca/IBKR MCP (ผ่าน LLM) | แสดงตั๋วออเดอร์, ตรวจ `"yes"` ยืนยันทีละออเดอร์, ตรวจ `fingerprint` จาก `risk.json`, และทำบัญชีต้นทุน FIFO ใน `portfolio.json` |
| 9 | `/trader-alerts` | `trader-alerts/alerts.py` | `clock`, `quote`, `check`, `add`, `list`, `reset` | Yahoo `v8` / Stooq CSV | เช็กเวลาตลาดและวันหยุด NYSE, ดึงราคาล่าสุด, สร้างและตรวจกฎแจ้งเตือน (`stop`, `target`, `flat_by`, `opening_bell`) |
| 10 | `/trader-analyst` | `trader-analyst/facts.py` | CLI (`tickers` / `--cik`) | SEC EDGAR XBRL & Submissions | วิเคราะห์งบรายไตรมาส/TTM (Revenue, Margin, Net Income, EPS), คำนวณ Q4 จาก 10-K และหาไฟล์ 8-K/10-Q ล่าสุด |
| 11 | `/trader-pelosi` | `trader-pelosi/ptr.py` | `index`, `new`, `fetch`, `parse` | House Clerk ZIP/XML & PTR PDF | ติดตามและแยกข้อมูลรายงานซื้อขายหุ้นของสมาชิกสภาสหรัฐ (ต้องมี `pdftotext` สำหรับอ่าน PDF) |

---

## 3. ผลการตรวจสอบเชิงลึกตามมุมมอง EGM

### 3.1 `egm-design` (ความเสี่ยงด้านสถาปัตยกรรมและการเทรดจริง)
- **Critical — No Broker Reconciliation:** `ticket.py` และ `check.py` ไม่อ่านข้อมูลจากโบรกเกอร์ ทำให้สถานะในเครื่องคลาดเคลื่อนได้ทันทีเมื่อออเดอร์ฝั่งโบรกเกอร์ถูก Fill, Partial Fill หรือ Cancel
- **Critical — Unprotected Window on `flatten`:** การยกเลิกขากันขาดทุนก่อนส่งคำสั่ง Market Sell เปิดช่องว่างให้พอร์ตไม่มี Stop-Loss ระหว่างรอการอนุมัติ
- **Critical — Unofficial Data & Stale Quote Sizing:** ใช้ Yahoo Finance แบบไม่ปรับ Stock Splits และ `size.py`/`check.py` ยอมรับราคาที่เก่าถึง 15 นาทีมาคำนวณ Market Order
- **Critical — Missing Idempotency & Partial Fill State Machine:** `client_order_id` มีใน `ALPACA_FIELDS` ของ `ticket.py` แต่ไม่ถูกสร้างโดย `size.py` หรือ `ticket.py`
- **High — Stale Risk Verdicts & Missing File Locking:** `risk_verdict()` ใน `ticket.py` เช็กแค่วันที่และ `fingerprint` ทำให้ผล `PASS` ตอนเช้ายังใช้ได้ถึงบ่ายแม้เงินสดหรือราคาจะเปลี่ยนไปแล้ว และไม่มี File Lock ป้องกัน Race Condition
- **High — Cash Settlement (T+1 / GFV) & PDT Batching:** `apply_fill()` คืนเงินสดทันทีและ `pdt_guard` ใน `check.py` นับ `pdt_n` ครั้งเดียวก่อนวนลูป batch
- **Medium — Pre/Post-Market Blindness & 2026–2027 Hardcoded Calendar:** ทิ้งแท่งนอกเวลาทำการทั้งที่ `check.py` อนุญาต `extended_hours` และปฏิทิน NYSE Hardcode ถึงแค่ปี 2027

### 3.2 `egm-zoom-out` (โครงสร้าง, ความซ้ำซ้อน และ Scope)
- **Code Duplication สูง (~8,000+ LOC):**
  - Timezone / DST (`_nth_sunday`, `eastern_offset`, `to_eastern`, `eastern_to_utc`, `parse_now`) ซ้ำใน 8–9 ไฟล์
  - `NYSE_HOLIDAYS` / `NYSE_EARLY_CLOSE` ซ้ำใน 6 ไฟล์
  - Yahoo Finance Fetcher (`query2` -> `query1` fallback + retry) ซ้ำใน 6 ไฟล์ (มี `BACKOFF` ไม่เท่ากันระหว่าง `5.0`s กับ `2.0`s)
  - `ALPACA_FIELDS` และ `fingerprint()` ซ้ำกันระหว่าง `trader-risk/check.py` และ `trader-broker/ticket.py` (เสี่ยง Contract Drift)
- **Scope Creep / YAGNI:** `trader-pelosi` (ดีเลย์ถึง 45 วันตาม STOCK Act), `trader-analyst` (งบรายไตรมาส), และโหมด `weights` ใน `trader-budget` เป็นงาน Swing/Value Investing มากกว่า Intraday Day Trading

### 3.3 `egm-codebase-details` & `egm-comprehension` (ความคลาดเคลื่อนของเอกสารและจุดบกพร่อง)
- `README.md` ระบุ `/trader-*` แต่การติดตั้งผ่าน `/plugin` ของ Claude Code จะติด namespace เป็น `/day-trader-skill:trader-*`
- `README.md` อ้างว่า `"paper": true` ใน `config.json` คุม Paper Trading แต่ฝั่ง Alpaca ตัวคุมจริงคือ API Keys และ `ALPACA_PAPER_TRADE` ใน MCP Config ของ Claude Desktop / Claude Code
- คำสั่งติดตั้งใน `README.md` ขาดคู่มือฝั่ง Windows (`py`/`python`, PowerShell) และขาดวิธีตั้งค่า Alpaca MCP ใน Claude Code (`.mcp.json` / `claude mcp add`)
- `trader-analyst/SKILL.md` อ้างว่าอ่าน `trader/watchlist.json` ได้ แต่ `facts.py` ไม่มี `--watchlist`
- `argument-hint` ใน `SKILL.md` หลายไฟล์ไม่ตรงกับ Subcommand/Flag จริงของสคริปต์

---

## 4. การพัฒนาและแก้ไขบั๊กที่ดำเนินการแล้ว (Implemented Fixes)

ในรอบการตรวจสอบและพัฒนานี้ ได้แก้ไขบั๊กที่ยืนยันแล้วโดยไม่กระทบ Compatibility ของ Test Contract เดิม:

1. **`trader-scanner/scan.py` (`orb_window`)**
   - เพิ่มการเลื่อน `no_entries_after` ขึ้น 3 ชั่วโมง (เช่น `15:30` -> `12:30`) ในวันตลาดปิดเร็ว (1:00 PM ET) ให้สอดคล้องกับ `trader-entry/entry.py` และ `trader-risk/check.py`
2. **`trader-scanner/scan.py` (`cmd_daily`, `cmd_orb`)**
   - แก้ `a.offline.rstrip("/")` เป็น `a.offline.rstrip("/\\")` เพื่อรองรับ Path ที่มี Trailing Backslash บน Windows
3. **`trader-news/news.py` (`eastern_to_utc`, `parse_now`, `main`)**
   - เพิ่ม `parse_now()` เพื่อตีความ `--now` ที่ไม่มี Timezone เป็นเวลา New York (ET) แทน UTC ให้ตรงกับสคริปต์อื่นทั้งระบบ
   - เพิ่ม Regression Test `test_parse_now_naive_is_eastern` ใน `tests/test_news.py`
4. **`trader-broker/ticket.py` (`parse_yes`)**
   - แก้ปัญหาคำภาษาอังกฤษพิมพ์เล็กทั่วไป (เช่น `"Yes, buy it"`) ถูกตีความผิดว่าเป็นชื่อหุ้นเมื่อ Batch มี Ticker สั้น (เช่น `IT`, `GO`) โดยแยกตรวจ Exact Uppercase Token ออกจากคำ 3 ตัวอักษรขึ้นไป
   - เพิ่ม Regression Test `test_short_tickers_do_not_block_english_words` ใน `tests/test_broker.py`
5. **`trader-exit/exit.py` (`cmd_show`)**
   - กรองค่า `None` ออกจาก `plan` ก่อนรวมกับค่า Default เพื่อไม่ให้พิมพ์ `"None"` เมื่อ `stop_label` หรือ `target_label` เป็น `null`
6. **`trader-timemachine/timemachine.py` (`_write`)**
   - ปรับ `_write()` ให้เขียนไฟล์แบบ Atomic ผ่านไฟล์ `.tmp.<pid>` แล้ว `os.replace()` ป้องกันไฟล์ `timemachine.json` เสียหายหากถูกขัดจังหวะ

### ผลการทดสอบหลังแก้ไข
```text
python -m unittest discover -s tests -v
Ran 317 tests in 23.589s
OK
```

---

## 5. แนวทางพัฒนาต่อ (Production-Readiness Roadmap)

1. **สกัด Shared Core (`trader_lib/`):** รวม `clock.py`, `http_yahoo.py`, `contracts.py` (`fingerprint`, `DEFAULTS`, Atomic JSON + File Lock) เพื่อลดความซ้ำซ้อนและป้องกัน Contract Drift
2. **Market Data Adapter:** เพิ่ม Provider สำหรับ Real-time Quotes/Bars (เช่น Alpaca / Polygon / IBKR) สำหรับ `entry.py` และ `alerts.py` แทนการ Polling Yahoo Finance
3. **Broker Reconciliation & Atomic Flatten:** เพิ่มคำสั่ง `ticket.py sync` ดึง Position/Order จริงจากโบรกเกอร์, ใส่ `client_order_id` สำหรับ Idempotency, และปรับ `risk_verdict()` ให้มี TTL สั้น (เช่น 60–120 วินาที) แทนการใช้ได้ทั้งวัน
4. **Risk & Compliance Hardening:** เพิ่มการติดตาม T+1 Unsettled Cash (ป้องกัน GFV), ปรับ `pdt_guard` ให้นับข้าม Batch, และอัปเดตปฏิทิน NYSE ให้ครอบคลุมปี 2028+
5. **Onboarding & Windows Docs:** อัปเดต `README.md` และ `trader-broker/SKILL.md` ให้รองรับ Windows PowerShell, Claude Code MCP registration, และชี้แจงความต่างระหว่าง `"paper": true` กับ API Keys อย่างชัดเจน
