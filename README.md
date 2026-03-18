# ⚙️ n8n Workflows — Automation Collection

> **EN:** A collection of automated workflows built with n8n, covering ETL data pipelines and certificate generation — all self-hosted via Docker.
>
> **TH:** รวม Workflow อัตโนมัติที่สร้างด้วย n8n ครอบคลุมทั้ง ETL Data Pipeline และระบบออกเกียรติบัตรอัตโนมัติ — ทั้งหมด Self-hosted ด้วย Docker

---

## 📁 Workflows

| Workflow | Description | Tools |
|---|---|---|
| [ETL Pipeline](#-etl-pipeline--automated-sales-report) | Multi-currency sales data pipeline | n8n, Google Sheets, Exchange Rate API, MySQL |
| [Auto Certificate](#-auto-certificate--automated-certificate-generator) | Auto-generate & deliver certificates on quiz pass | n8n, Google Forms, Slides, Gmail, Drive |

---

## 🔄 ETL Pipeline — Automated Sales Report

> **EN:** Fetches sales data from Google Sheets, converts multi-currency prices using Exchange Rate API in real-time, and loads results into MySQL — fully eliminating manual data entry.
>
> **TH:** ดึงข้อมูลยอดขายจาก Google Sheets แปลงสกุลเงินด้วย Exchange Rate API แบบ Real-time แล้วส่งผลลัพธ์เข้า MySQL โดยไม่ต้องทำด้วยมือ

### Workflow Overview

![ETL Pipeline Workflow](etl-pipeline-screenshot.png)

### Pipeline Flow

```
Schedule Trigger
      │
      ▼
Exchange Rate API ──────────────────────────┐
      │                                     │
      ▼                                     │
Extract Sales Data (Google Sheets)          │
      │                                     │
      ▼                                     │
Extract Product Cost (Google Sheets)        │
      │                                     │
      ▼                                     ▼
    Merge (JOIN by Product_Name) ──► Code: Calculate THB
                                          │
                                          ▼
                                       Filter (Remove empty Order_ID)
                                          │
                                          ▼
                                    MySQL Upsert
```

### How It Works / ขั้นตอนการทำงาน

**1. Schedule Trigger** — EN: Runs automatically on schedule / TH: รัน Workflow ตามเวลาที่กำหนด

**2. Exchange Rate API** — EN: Fetches real-time rates (USD, JPY, GBP, THB vs EUR) / TH: ดึงอัตราแลกเปลี่ยน Real-time

**3. Extract Sales & Product Cost** — EN: Pulls order data and cost data from two Google Sheets / TH: ดึงข้อมูลยอดขายและต้นทุนจาก Google Sheets สองชีท

**4. Merge** — EN: JOINs both datasets by `Product_Name` / TH: JOIN ข้อมูลสองชุดด้วย `Product_Name`

**5. JavaScript Calculation** — EN: Converts all prices to THB and calculates profit / TH: แปลงราคาเป็น THB และคำนวณกำไร

```js
Sale_THB   = (Sale_Price / rate_of_currency) × rate_THB
Cost_THB   = (Base_Cost  / rate_of_currency) × rate_THB
Profit_THB = Sale_THB - Cost_THB
```

**6. Filter** — EN: Removes rows without Order_ID / TH: กรองแถวที่ไม่มี Order_ID ออก

**7. MySQL Upsert** — EN: Inserts or updates records using `Order_ID` as unique key (no duplicates) / TH: บันทึกหรืออัปเดตข้อมูลโดยใช้ `Order_ID` เป็น Key หลัก

### Output Fields (MySQL)

| Field | Description | คำอธิบาย |
|---|---|---|
| Order_ID | Order reference | รหัสคำสั่งซื้อ |
| Order_Date | Date of order | วันที่สั่งซื้อ |
| Product_Name | Product name | ชื่อสินค้า |
| Quantity | Units sold | จำนวน |
| Sale_Price | Original price | ราคาขายเดิม |
| Currency | Sale currency | สกุลเงินที่ขาย |
| Sale_THB | Price in THB | ยอดขายเป็น THB |
| Cost_THB | Cost in THB | ต้นทุนเป็น THB |
| Profit_THB | Net profit in THB | กำไรสุทธิเป็น THB |

### Getting Started / วิธีใช้งาน

1. Import `etl-pipeline/N8N_ETL_safe.json` into n8n
2. Set up credentials: Google Sheets OAuth2, MySQL, Exchange Rates API Key
3. Update Google Sheet IDs in `Extract Sales Data` and `Extract Product Cost` nodes
4. Execute manually or wait for Schedule Trigger

---

## 📜 Auto Certificate — Automated Certificate Generator

> **EN:** Automatically generates and delivers a personalized PDF certificate to users via Gmail upon passing a quiz — fully eliminating manual certificate processing.
>
> **TH:** สร้างและส่งใบเกียรติบัตร PDF ส่วนตัวให้ผู้ใช้ผ่าน Gmail โดยอัตโนมัติเมื่อทำแบบทดสอบผ่าน

### Workflow Overview

![Auto Certificate Workflow](auto-cert-screenshot.png)

### How It Works / ขั้นตอนการทำงาน

**1. Google Sheets Trigger** — EN: Monitors linked Google Sheet for new quiz submissions / TH: ตรวจจับแถวใหม่จาก Google Sheet ที่เชื่อมกับ Google Form

**2. Extract Essential Data** — EN: Pulls name, email, score, and status / TH: ดึงชื่อ, อีเมล, คะแนน และสถานะ

**3. Score Checker** — EN: Routes to certificate generation if score ≥ 5 and status ≠ "Sent" / TH: ตรวจสอบคะแนน ≥ 5 และสถานะยังไม่เป็น "Sent"

**4. Copy from Template** — EN: Duplicates the Google Slides template in Drive for this user / TH: คัดลอก Template Google Slides สำหรับผู้ใช้คนนี้

**5. Replace Text** — EN: Fills in `[name]` and other placeholders with user's actual data / TH: แทนที่ `[name]` และ Placeholder อื่นๆ ด้วยข้อมูลจริง

**6. Convert to PDF** — EN: Exports the personalized Slide as PDF / TH: แปลง Slide เป็นไฟล์ PDF

**7. Send to User's Email** — EN: Delivers the PDF certificate via Gmail / TH: ส่ง PDF เกียรติบัตรผ่าน Gmail

**8. Update Status** — EN: Marks the row as "Sent" in Google Sheets to prevent duplicates / TH: อัปเดตสถานะเป็น "Sent" ใน Google Sheets เพื่อป้องกันการส่งซ้ำ

### Getting Started / วิธีใช้งาน

1. Create a Google Form → enable **Quiz Mode** → link to Google Sheet
2. Import `auto-cert/_DSP__Certificate_w__Google_Forms.json` into n8n
3. Set up credentials: Google Sheets, Slides, Drive, Gmail OAuth2
4. Update your Template Slide ID in the **Copy from Template** node
5. Toggle workflow to **Active**

---

## 🛠️ Tech Stack

| Tool | Role |
|---|---|
| n8n | Workflow Automation Engine |
| Docker | Self-hosted n8n deployment |
| Google Sheets | Data source & status tracking |
| Exchange Rates API | Real-time currency rates |
| MySQL | Output storage (ETL) |
| Google Slides | Certificate template |
| Google Drive | File storage |
| Gmail | Certificate delivery |

---

## 📁 Repository Structure / โครงสร้าง Repo

```
n8n-workflows/
├── etl-pipeline/
│   ├── N8N_ETL_safe.json               ← Import into n8n
│   ├── etl_pipeline-screenshot.png                  ← Workflow canvas
│   └── Final_Sales_Report.csv
├── auto-cert/
│   ├── _DSP__Certificate_w__Google_Forms.json  ← Import into n8n
│   └── auto_screenshot.png                  ← Workflow canvas
└── README.md
```

---

*Developed by [Peerawit Aiamoakson (James)](https://github.com/peerawit-dev) and Suchanan Khanphrasaeng — Digital Technology, Phuket Rajabhat University*
*พัฒนาโดย [พีรวิชญ์ เอี่ยมอักษร (James)](https://github.com/peerawit-dev) และ สุชานันท์ ขันพระแสง — สาขาเทคโนโลยีดิจิทัล มหาวิทยาลัยราชภัฏภูเก็ต*
