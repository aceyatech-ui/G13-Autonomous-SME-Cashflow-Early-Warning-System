```markdown
# Autonomous SME Cashflow Early Warning System 🚚📉

**TechCrush AI Automation Capstone Project | Cohort 9 — Group 13**  
**Case Study:** BrightPath Logistics Ltd (`BP-001`)

---

## 📌 Problem Statement
BrightPath Logistics Ltd is a regional delivery company based in Port Harcourt, Rivers State, operating a 12-vehicle fleet. While profitable on paper, the business faces a critical operational cashflow timing mismatch: weekly fuel and driver wage obligations must be settled immediately, whereas corporate clients pay on 45-to-60-day invoice terms. 

When unexpected market disruptions occur—such as sudden diesel price surges—cash balances can drop to critical levels without early warning. This system provides automated, 30-day forward-looking cashflow visibility and proactive risk mitigation strategies before cash reserves hit dangerous thresholds.

---

## ⚙️ Key System Features

* **Dual Event Triggers:** Runs on a daily schedule (07:00) for proactive monitoring and supports webhooks for real-time events (fuel price updates, incoming invoice payments).
* **Dynamic State Management:** Evaluates the lowest projected cash balance over 30 days and transitions through 5 distinct system states:
  * 🟢 **HEALTHY:** Projected balance $\ge$ ₦5.0m
  * 🟡 **WATCH:** Projected balance between ₦3.0m and ₦5.0m (Triggers alert email)
  * 🟠 **AT_RISK:** Projected balance between ₦1.0m and ₦3.0m (Urgent alert + AI analysis)
  * 🔴 **CRITICAL:** Projected balance < ₦1.0m or negative (Requires human approval)
  * 🚨 **HUMAN ESCALATION:** Initiated when approval is requested or unresponded
* **AI Risk Analysis & Recommendations:** At critical stages, the AI model evaluates financial drivers and generates structured recovery actions (e.g., invoice early-payment discounts, vendor installment splits).
* **Human-in-the-Loop Approval:** Pauses execution at `CRITICAL` state to email the Managing Director for decision approval, complete with escalation timeouts to backup managers.
* **Reliability, Error Handling & Auditing:** Features automatic retries for failed AI calls, seamless fallback to rule-based logic during outages, and persistent logging of every execution in Supabase.

---

## 🛠 Tech Stack & Integrations

* **Orchestration:** n8n Workflows (Sub-workflows `W01` through `W08`, JavaScript Code Nodes, Loops)
* **Database:** Supabase PostgreSQL (Cash positions, receivables, payables, risk alerts, approvals, audit logs)
* **Communication:** Gmail (Email alerts and human-approval requests)
* **AI Engine:** Structured AI API for risk evaluation and recommendations

---

## 📁 Repository Structure

```text
.
├── Workflow1/          # Master Workflow & Daily Trigger
├── Workflow2/          # Database Fetch (Supabase)
├── Workflow3/          # Cashflow Forecast Engine (n8n Code Node & Loops)
├── Workflow4/          # Risk Rules & State Management
├── Workflow5/          # Structured AI Risk Analysis
├── Workflow6/          # Human-in-the-Loop Approval & Escalation
├── Workflow7/          # Action Execution Engine
├── Workflow8/          # Reliability, Fallback & Audit Logging
├── database/
│   └── schema.sql      # Supabase database schema and baseline data
└── README.md           # Project Documentation

```

---

## 📊 Baseline Scenario & Crisis Simulation

* **Starting Balance (Day 0):** ₦16,000,000
* **Safety Buffer Target:** ₦4,000,000
* **Baseline Diesel Cost:** ₦1,250 / Litre (2,880L/week = ₦3.6m weekly fuel bill)
* **Crisis Scenario:** A 60% fuel price spike pushes diesel to ₦2,000 / Litre (₦5.76m weekly). Without intervention, projected cash drops to a critical low of **₦0.97m on Day 14**.
* **System Intervention:** On Day 1 (13 days before the crash), the system flags `CRITICAL` risk and recommends offering a 2% early-payment discount to Coastal Beverages on invoice `INV-1041`, keeping cash safely above buffer levels.

---

## 👥 Group 13 Team Workflow & Role Assignments

| Member Name | Main Build Responsibility | Supporting Responsibility |
| --- | --- | --- |
| **Chisom Okafor** *(Team Lead)* | **W01 Main Controller:** Controls process flow, creates run IDs, and invokes sub-workflows. | Integrator (merges approved workflows into the n8n environment and leads final review). |
| **Okoye Ngozi Rosemary** *(Assistant Lead)* | **W03 Forecast Engine:** JavaScript logic to build the 30-day cash balance forecast. | Task verification and poll oversight. |
| **Okonji Brendan** | **W02 Financial Ingestion:** Validates, normalizes, and stores incoming transaction/price events. | API/integration research and setting up the simulated fuel price feed. |
| **Ogunsanwo Ibrahim Ademola** | **W04 Risk Engine:** Applies risk rules and compares state changes. | Financial logic lead (defines thresholds, scenario numbers, and state flow). |
| **Okorafor Ogbonnaya Kennedy** | **W05 AI Analysis:** Sends verified figures to the AI for analysis and action suggestions. | Prompt design, structured output formatting, and validation rules. |
| **Oguegbu Ogochukwu Felicity** | **W06 Human Approval:** Manages the approval request email, reminder, and escalation logic. | Slide deck and presentation lead. |
| **Adeleye Okikiola Emmanuel** | **W07 Notifications:** Sends email alerts via Gmail for various risk states. | GitHub custodian (branches/PRs) and cloud migration lead. |
| **Okwor Onyinyechi Favour** | **W08 Reliability & Audit:** Handles retries, fallback logic, and audit logging. | Live failure scenario design (with QA lead). |
| **Olamide Kehinde** | **Database & Data Lead:** Sets up Supabase tables and loads initial sample data. | Supports W02 and W03 data access. |
| **Okeke Ekene Matthew-Daniel** | **QA & Testing Lead:** Builds test cases, performs integration testing, and tests failures. | Demo rehearsal and requirement checklist verification. |
| **Ogunleye Oluwatimilehin Favour** | **Documentation Lead:** Project diary and tracking. | GitHub documentation, screenshot archiving, and final submission contribution. |

---

## 🚀 Setup & Execution Instructions

1. **Database Setup:**
Execute the contents of `database/schema.sql` inside your Supabase SQL Editor to set up tables (`cash_positions`, `receivables`, `payables`, `risk_alerts`, `approvals`, `audit_logs`) and seed initial data for `BP-001`.
2. **Environment & Credential Setup:**
Configure n8n credentials for:
* **Supabase** (Service Role API key & Host URL)
* **Gmail OAuth2 / SMTP** (For alert & approval emails)
* **AI API** (For structured analysis)


3. **Workflow Import:**
Navigate into each directory (`Workflow1/` through `Workflow8/`) and import the `.json` files into your n8n workspace. Ensure sub-workflow nodes in `Workflow1` properly reference `Workflow2` through `Workflow8`.
4. **Testing the Live Crisis Scenario:**
* Run the master workflow (`Workflow1`) on `Day 0` data to confirm a `HEALTHY` state.
* Trigger the fuel price update webhook with a diesel price of ₦2,000/L to test immediate risk recalculation, state transition to `CRITICAL`, email approval dispatch, and audit log generation.



```

```
