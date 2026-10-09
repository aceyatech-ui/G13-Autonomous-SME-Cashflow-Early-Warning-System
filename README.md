# BrightPath Cashflow Early Warning System

Group 13, TechCrush AI Automation Capstone

In this project, N means Nigerian naira and m means million.

## About the project

BrightPath Logistics is a delivery company in Port Harcourt, Nigeria. It runs 12 vehicles for five large corporate clients and it makes a profit every month. Its weakness is timing. Fuel and driver wages are paid every week, but the clients pay 45 to 60 days after each invoice, so the company is always funding about seven weeks of work from its own cash. When costs jump, the account can run dry before anyone sees it coming, because the owner only checks today's bank balance and nobody forecasts the next 30 days.

We built a system that does that forecasting automatically and speaks up early. Every day it works out how much cash BrightPath will hold over the next 30 days, decides how serious the situation is, asks an AI model to explain the problem in plain language, and emails the owner. If things are critical it also asks him to approve or reject a set of recommended actions, and it passes the request to the operations manager if he does not answer in time. Every step is written to a database, so a run can be reviewed afterwards.

The story we used to test and demonstrate the system is a fuel price spike. Diesel rises from N1,250 to N2,000 per litre over five days. On Day 5 the bank balance still looks fine at N15.07m, but the forecast shows cash dropping to N0.97m on Day 14, which is less than two days of running costs. The system raises the alarm on Day 1, which gives the owner 13 days of warning.

## How it works

A run starts in one of two ways. A schedule trigger fires every day at 07:00, or a webhook receives an event such as a fuel price update, a payment or a new invoice. The main controller creates a run ID and then calls the other workflows in order.

1. Ingestion checks the incoming event, rejects bad data, ignores duplicates and stores the rest.
2. The forecast engine reads the database and builds the 30 day cash balance, day by day.
3. The risk engine compares the lowest projected balance with the safety buffer of N4.0m and sets the state to HEALTHY, WATCH, AT_RISK or CRITICAL.
4. For AT_RISK and CRITICAL, the AI analysis explains the cause and suggests three actions. If the AI service fails, the system retries three times and then falls back to a rule based summary.
5. For CRITICAL, the human approval workflow emails the founder and waits. If nobody replies, it reminds him and escalates to the operations manager.
6. The notification workflow sends the right email for the state.
7. Everything important is written to the audit log.

## The eight workflows

| ID | Name | What it does |
|---|---|---|
| W01 | Main Controller | Receives the schedule or webhook trigger, creates the run, decides the path and calls the other workflows |
| W02 | Financial Ingestion | Validates fuel price, payment and invoice events, checks for duplicates and stores them |
| W03 | Forecast Engine | Builds the 30 day cash forecast from the database and saves it |
| W04 | Risk Engine | Applies the risk rules, compares with the previous state and saves the alert |
| W05 | AI Analysis | Sends verified figures to Gemini and returns a checked, structured explanation |
| W06 | Human Approval | Asks the founder to approve or reject, reminds, escalates and records the decision |
| W07 | Notifications | Sends the Gmail alerts and the daily summary |
| W08 | Reliability and Audit | Retries the AI call, switches to a fallback when needed and writes events to the audit log |

## Risk rules

The state depends on the lowest projected balance in the next 30 days.

| State | Lowest projected balance | Response |
|---|---|---|
| HEALTHY | N5.0m or more | Daily summary email |
| WATCH | N3.0m up to N5.0m | Alert email to the founder |
| AT_RISK | N1.0m up to N3.0m | Urgent alert with AI explanation and actions |
| CRITICAL | Below N1.0m, or negative | AI analysis, human approval, all alerts |

## Technology used

* n8n for all workflows, including the schedule trigger, webhook, Code nodes and Execute Workflow calls
* Supabase (PostgreSQL) for the database
* Gmail with OAuth2 for alerts and approval emails
* Google Gemini for the AI explanation
* Thunder Client for sending test requests

## Getting started

You need an n8n instance, a Supabase project, a Gmail account and a Google Gemini API key.

1. Create the database tables in Supabase and load the BrightPath sample data. The tables are companies, transactions, cash_positions, receivables, payables, commitments, fuel_price_events, forecasts, risk_alerts, approvals and audit_logs. The documentation lists the sample values.
2. In n8n, create three credentials: a Supabase API credential, a Gmail OAuth2 credential and a Google Gemini API credential.
3. Import the workflows in this order, and save each one: W02, W03, W04, W05, W06, W07, W08, then W01.
4. Open the Execute Workflow nodes and paste the workflow ID of the workflow each one should call. The three AI attempt nodes in W08 call W05. The call nodes in W01 call W02, W03, W04, W06, W07 and W08. The AI analysis node and both audit nodes in W01 all call W08.
5. Select your credentials on every Supabase, Gmail and Gemini node.
6. Put the founder's and the operations manager's email addresses in the Prepare Request node in W06 and the Build Email node in W07.
7. Activate W01 so the daily schedule and the webhook run. For testing you can use the test URL instead.

Credentials stay inside n8n. The exported workflow files only refer to them by name, and no keys or passwords are stored in this project.

## Sending an event

Post an event to the W01 webhook at the path brightpath-event. This example is the third diesel price rise from the story.

```json
{
  "event_type": "fuel_price_update",
  "company_id": "BP-001",
  "event_id": "DEMO-FP-003",
  "fuel": "diesel",
  "old_price": 1750,
  "new_price": 2000,
  "day": 5
}
```

The day field is optional. When it is present, the forecast counts days to breach from that day, which gives the 9 days of warning shown in the story. Payment and invoice events use the same webhook. A payment needs invoice_id, amount, payment_day and reference. An invoice needs invoice_id, client_name, amount and due_day. Every event needs event_type, company_id and event_id, and the event_id must be new, because a repeated one is treated as a duplicate.

## What the demo shows

| Diesel price | Lowest projected balance | State | Warning time |
|---|---|---|---|
| N1,250 | N5.29m on Day 14 | HEALTHY | Not applicable |
| N1,500 | N3.85m on Day 14 | WATCH | 13 days |
| N1,750 | N2.41m on Day 14 | AT_RISK | 11 days |
| N2,000 | N0.97m on Day 14 | CRITICAL | 9 days |

The demo also breaks the AI service on purpose. The system retries three times, records each failure, sends a clearly marked automated summary and still delivers the approval request. After the key is restored, the normal AI path works again and the audit log shows the whole story.

## Team

Group 13 has 11 members. Ten of them each own one workflow or a main build area, and one is the documentation lead.

| Member | Main responsibility | Supporting responsibility |
|---|---|---|
| Chisom Okafor (Team Lead) | W01 Main Controller | Integrator and final review |
| Okoye Ngozi Rosemary (Assistant Team Lead) | W03 Forecast Engine | Task verification and poll oversight |
| Okonji Brendan | W02 Financial Ingestion | API and integration research, simulated fuel price feed |
| Ogunsanwo Ibrahim Ademola | W04 Risk Engine | Financial logic lead |
| Okorafor Ogbonnaya Kennedy | W05 AI Analysis | Prompt design and validation rules |
| Oguegbu Ogochukwu Felicity | W06 Human Approval | Slide deck and presentation lead |
| Adeleye Okikiola Emmanuel | W07 Notifications | GitHub custodian and cloud migration lead |
| Okwor Onyinyechi Favour | W08 Reliability and Audit | Design of the live failure scenario |
| Olamide Kehinde | Database and data lead | Support for W02 and W03 data access |
| Okeke Ekene Matthew Daniel | QA and testing lead | Demo rehearsal and requirement checklist |
| Ogunleye Oluwatimilehin Favour (Documentation Lead and Co Lead) | Project diary and tracking | GitHub documentation, screenshot archive and the final contribution PDF |

## More detail

The full project documentation explains the company, the scenario, the database, every workflow node by node, the tests and the limits of the system.
