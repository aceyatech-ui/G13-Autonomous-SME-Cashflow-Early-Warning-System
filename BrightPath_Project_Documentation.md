# BrightPath Logistics: Autonomous SME Cashflow Early Warning System

Project Documentation, Group 13, TechCrush AI Automation Capstone

In this document N means Nigerian naira and m means million. Day numbers count from Day 0, which is the day the system starts its first forecast.

## 1. Introduction

Many small and medium businesses do not fail because they are unprofitable. They fail because cash arrives later than the bills do. Group 13 chose to look at that problem through one company, BrightPath Logistics, and to build a system that watches its cash every day and warns the owner early, before the account runs low.

The system is built in n8n and uses a Supabase database, Gmail for messages and Google Gemini for the AI explanation. It is made of eight connected workflows, named W01 to W08. One of them is the main controller, and the others handle ingestion, forecasting, risk rules, AI analysis, human approval, notifications and reliability.

This document explains the company and the story we used, the design of the system, the database, each workflow in detail, how we tested it, how to set it up, and what it cannot yet do.

## 2. The company: BrightPath Logistics

BrightPath Logistics Ltd is a delivery company based in Port Harcourt, Rivers State. It was founded in 2019 by Mr. Tamuno Briggs, a former warehouse supervisor who noticed that many businesses in the city could not find a reliable delivery partner. He started with two second hand vans and one driver. Today BrightPath runs 12 vehicles, made up of 8 vans and 4 light trucks, with 12 drivers and a small office team. It delivers goods across the Niger Delta for five main corporate clients.

On paper the business is healthy. It wins contracts, its vehicles are busy and its monthly profit is positive. The weakness is timing. BrightPath pays for fuel and driver wages every week, but its corporate clients pay 45 to 60 days after each invoice. The company therefore funds about seven weeks of operations before it is repaid. Whether it survives a bad month depends on how much cash it holds and on when each payment lands.

Today Mr. Briggs checks the bank balance every morning. That tells him where cash is now, but not where it is heading. Mrs. Ebiere Dokubo, the finance and admin officer, keeps invoices and bills in spreadsheets, and nobody forecasts the next 30 days. BrightPath needs something that does this automatically.

### Company profile

| Item | Detail |
|---|---|
| Company | BrightPath Logistics Ltd, company_id BP-001 |
| Location | Port Harcourt, Rivers State, Nigeria. Founded in 2019 |
| Business | Regional delivery and haulage for corporate clients |
| Fleet | 12 vehicles, 8 vans and 4 light trucks. Three of the trucks are on a finance loan |
| People | 12 drivers paid weekly, 3 office staff and 1 in house mechanic paid monthly. 16 in total |
| Founder and MD | Mr. Tamuno Briggs, the main approver for serious cash decisions |
| Finance and admin officer | Mrs. Ebiere Dokubo, who prepares invoices and bills |
| Operations manager | Mr. Chidi Nwankwo, the backup approver if the founder does not respond |
| Fuel supplier | Harbour Fuels Ltd, paid weekly for diesel |
| Fuel use | About 2,880 litres of diesel per week, which is 240 litres per vehicle |
| Normal diesel price | N1,250 per litre, so a normal weekly fuel bill of N3.6m |
| Safety buffer | N4.0m, about one week of running costs. The owner wants cash to stay above this |

### The five corporate clients

| Client | Business | Payment terms | Open invoice |
|---|---|---|---|
| Coastal Beverages Ltd | Beverage distribution | 45 days | INV-1041, N6.4m, due Day 15 |
| Delta Build Supplies | Building materials | 60 days | INV-1046, N4.8m, due Day 19 |
| FreshMart Stores | Supermarket chain | 45 days | INV-1052, N3.9m, due Day 23 |
| Niger Basin Energy Services | Oilfield services | 60 days | INV-1058, N8.5m, due Day 27 |
| QuickCart Nigeria | E-commerce fulfilment | 45 days | INV-1063, N3.2m, due Day 30 |

The five invoices add up to N26.8m expected from clients over the next 30 days.

## 3. A normal month

In a normal month BrightPath collects about N26.8m from clients and spends about N21.49m, which leaves a profit of about N5.3m. The money goes out like this.

| Cost | When it is paid | Amount | Monthly total |
|---|---|---|---|
| Diesel from Harbour Fuels | Weekly, Days 7, 14, 21 and 28 | N3.6m each | N14.4m |
| Driver wages | Weekly, Days 5, 12, 19 and 26 | N0.48m each | N1.92m |
| Office and mechanic salaries | Monthly, Day 28 | N1.0m | N1.0m |
| Vehicle maintenance and repairs | Days 9 and 23 | N0.6m each | N1.2m |
| Truck loan repayment | Day 10 | N1.5m | N1.5m |
| Vehicle insurance | Day 16 | N0.6m | N0.6m |
| Office rent | Day 2 | N0.45m | N0.45m |
| Tracking software and other costs | Day 20 | N0.42m | N0.42m |
| Total | | | N21.49m |

BrightPath starts Day 0 with N16.0m in its main operating account. With no crisis, the lowest point in the 30 days is N5.29m on Day 14, just after the second fuel payment and one day before Coastal Beverages pays. That is comfortable, so the business is HEALTHY.

## 4. The crisis: a fuel price spike

A supply disruption at the depots serving Port Harcourt, together with higher import costs, pushes diesel prices up sharply in the first week. Harbour Fuels announces three increases in five days.

| Day | Diesel price per litre | Rise from normal | Weekly fuel bill | Extra cost per week |
|---|---|---|---|---|
| Day 0 | N1,250 | None | N3.60m | None |
| Day 1 | N1,500 | 20% | N4.32m | N0.72m |
| Day 3 | N1,750 | 40% | N5.04m | N1.44m |
| Day 5 | N2,000 | 60% | N5.76m | N2.16m |

BrightPath's clients are on fixed rate delivery contracts, so the extra cost cannot be passed on quickly. At N2,000 per litre, the four fuel payments of the month cost N8.64m more than normal. The monthly profit of N5.3m turns into a loss of about N3.3m.

The dangerous part is that nothing looks wrong for a long time. On Day 5 the bank balance is still N15.07m. Without a forecast, nobody would feel the danger until Day 14, when the fuel payment would leave only N0.97m in the account, less than two days of running costs. With the early warning system, the owner hears about the problem on Day 1, which is 13 days before the danger point.

### The 30 day cash calendar

All amounts are in millions of naira.

| Day | Event | Balance, no spike | Balance, diesel at N2,000 |
|---|---|---|---|
| 0 | Opening cash | 16.00 | 16.00 |
| 2 | Rent paid (0.45) | 15.55 | 15.55 |
| 5 | Driver wages (0.48) | 15.07 | 15.07 |
| 7 | Fuel payment (3.60 normal, 5.76 at spike) | 11.47 | 9.31 |
| 9 | Maintenance (0.6) | 10.87 | 8.71 |
| 10 | Truck loan (1.5) | 9.37 | 7.21 |
| 12 | Driver wages (0.48) | 8.89 | 6.73 |
| 14 | Fuel payment, the lowest point | 5.29 | 0.97 |
| 15 | Coastal Beverages pays 6.4 | 11.69 | 7.37 |
| 16 | Insurance (0.6) | 11.09 | 6.77 |
| 19 | Wages (0.48), Delta Build pays 4.8 | 15.41 | 11.09 |
| 20 | Tracking and other costs (0.42) | 14.99 | 10.67 |
| 21 | Fuel payment | 11.39 | 4.91 |
| 23 | Maintenance (0.6), FreshMart pays 3.9 | 14.69 | 8.21 |
| 26 | Driver wages (0.48) | 14.21 | 7.73 |
| 27 | Niger Basin Energy pays 8.5 | 22.71 | 16.23 |
| 28 | Fuel payment, staff salaries (1.0) | 18.11 | 9.47 |
| 30 | QuickCart pays 3.2 | 21.31 | 12.67 |

## 5. Our solution

The idea is simple. Instead of looking at today's balance, the system looks at the balance BrightPath will have on every one of the next 30 days. It does this by taking the current cash, adding each invoice on the day it is due, subtracting each bill and commitment on the day it is due, and pricing diesel at the latest known price. The lowest point on that line is the number that matters, because that is the day the business is closest to trouble.

That number is compared with a set of rules, and the answer decides what happens next. A comfortable forecast only produces a daily summary. A worrying one produces an alert. A dangerous one brings in an AI explanation, a request for a human decision, and an audit record of everything that happened.

We kept three principles throughout the build.

First, the AI never does the arithmetic. All figures come from the forecast and the rules. The AI only receives those verified figures and turns them into a readable explanation and three suggested actions. Its answer is checked before it is used.

Second, a failure must never leave the owner without a warning. If the AI service is down, the system retries and then sends a rule based message built from the same figures, clearly marked as automated.

Third, everything is recorded. Forecasts, alerts, approvals and audit events are stored in the database so that any run can be explained later.

### How a run flows

A run begins with either the daily schedule at 07:00 or an incoming event on the webhook. The controller creates a run ID and then calls the workflows in this order: ingestion (only for webhook events), forecast, risk, AI analysis (for AT_RISK and CRITICAL), human approval (for CRITICAL), notifications and finally the audit record. Results are kept in Supabase for review.

### Risk rules

The state is decided by the lowest projected balance in the next 30 days.

| State | Rule | Response |
|---|---|---|
| HEALTHY | N5.0m or more | No action. Normal daily summary |
| WATCH | N3.0m up to N5.0m | Alert email to the founder, keep monitoring |
| AT_RISK | N1.0m up to N3.0m | Urgent alert with AI explanation and recommended actions |
| CRITICAL | Below N1.0m, or any negative balance | AI analysis, human approval required, all alerts sent |

A breach means the projected balance falls below the N4.0m safety buffer. The forecast also reports days to breach, which is the number of days from the current day until that first happens.

### What the system reports at each price level

| When | Diesel | Lowest projected balance | State | Breach below N4.0m | Warning time |
|---|---|---|---|---|---|
| Day 0 | N1,250 | N5.29m on Day 14 | HEALTHY | None | Not applicable |
| Day 1 | N1,500 | N3.85m on Day 14 | WATCH | Day 14 | 13 days |
| Day 3 | N1,750 | N2.41m on Day 14 | AT_RISK | Day 14 | 11 days |
| Day 5 | N2,000 | N0.97m on Day 14 | CRITICAL | Day 14 | 9 days |

The state therefore moves from HEALTHY to WATCH to AT_RISK to CRITICAL, and when the approval request has to go to the operations manager it reaches HUMAN_ESCALATION, which is recorded in the approvals table. This is the state management the project demonstrates.

### What the AI says at CRITICAL

The AI receives only verified figures. It returns a short summary, the cause, an urgency level and three recommended actions. For the Day 5 situation, the expected content looks like this.

* Summary: cash is projected to fall to N0.97m on Day 14, well below the N4.0m safety buffer.
* Cause: diesel has risen 60% from N1,250 to N2,000 per litre, which adds N2.16m to each weekly fuel payment.
* Action 1: ask Coastal Beverages to pay invoice INV-1041 (N6.4m) by Day 13, offering a 2% early payment discount of N128,000.
* Action 2: ask Harbour Fuels to split the Day 14 payment into two instalments of N2.88m, on Day 14 and Day 17.
* Action 3: apply an 8% temporary fuel surcharge to new invoices and review routes to cut fuel use by 10%.

In the story, Mr. Briggs approves Actions 1 and 2. If Coastal Beverages pays on Day 13 with the 2% discount, the Day 14 balance rises from N0.97m to about N7.2m, safely above the buffer.

## 6. The database

All data lives in Supabase, which is a hosted PostgreSQL database. The workflows read and write these tables.

| Table | What it holds |
|---|---|
| companies | The company record for BrightPath |
| cash_positions | The opening cash balance, with the forecast day it belongs to |
| receivables | Invoices owed to BrightPath, with client, amount, due day and status |
| payables | Bills BrightPath owes, such as diesel and maintenance, with due day and status |
| commitments | Fixed obligations such as wages, salaries, loan, insurance and rent |
| transactions | Payments received, stored by the ingestion workflow |
| fuel_price_events | Every diesel price change that has arrived |
| forecasts | One row for each 30 day forecast that the engine produces |
| risk_alerts | The risk state for each run, with the previous state and the reason |
| approvals | Approval requests and the decision, who made it and when |
| audit_logs | A record of runs, AI attempts, failures, fallbacks and recoveries |

### Sample data

The sample data is loaded exactly as given in the project brief so that every workflow produces the same figures.

The opening cash position is N16,000,000 on Day 0 for company BP-001. The open receivables are INV-1041 for Coastal Beverages (N6,400,000, due Day 15), INV-1046 for Delta Build Supplies (N4,800,000, due Day 19), INV-1052 for FreshMart Stores (N3,900,000, due Day 23), INV-1058 for Niger Basin Energy Services (N8,500,000, due Day 27) and INV-1063 for QuickCart Nigeria (N3,200,000, due Day 30).

The payables and commitments are listed below. Diesel is stored as a quantity of 2,880 litres for each due day, and its amount is calculated from the latest fuel price.

| Item | Type | Amount | Due days |
|---|---|---|---|
| Diesel from Harbour Fuels | Payable | 2,880 litres times the price | 7, 14, 21, 28 |
| Driver wages | Commitment | 480,000 | 5, 12, 19, 26 |
| Staff salaries | Commitment | 1,000,000 | 28 |
| Maintenance | Payable | 600,000 | 9, 23 |
| Truck loan | Commitment | 1,500,000 | 10 |
| Insurance | Commitment | 600,000 | 16 |
| Office rent | Commitment | 450,000 | 2 |
| Tracking and other | Payable | 420,000 | 20 |

The three fuel price events used to trigger the crisis are a rise from 1,250 to 1,500 on Day 1, from 1,500 to 1,750 on Day 3 and from 1,750 to 2,000 on Day 5.

## 7. The workflows in detail

Every workflow except W01 starts with an Execute Workflow Trigger set to accept all data, so that W01 can call it. Workflows pass their results back to W01, which keeps the whole run together in one place.

### W01 Main Controller

**Purpose.** W01 is the entry point. It decides which path a run takes and calls the other seven workflows in the right order.

**Triggers.** There are two. A schedule trigger fires every day at 07:00, and a webhook accepts POST requests at the path brightpath-event for incoming events.

**How it works.**

1. The Create Run Context node builds a run context. It generates a run ID in the form RUN, date, time and four random characters, records the start time and whether the trigger was the schedule or the webhook, and sets the company (BP-001 by default) and the forecast horizon (30 days by default). If the webhook event includes a day, it is kept as the current day.
2. The Needs Ingestion node checks whether the trigger was a webhook. Webhook events go to W02 first. The daily schedule skips this step.
3. After W02, the Continue Run node checks the ingestion status. Only a stored event continues. A duplicate or an invalid event ends the run early, and the run is recorded as skipped or rejected.
4. W03 builds the forecast and W04 turns it into a risk state. After each call, a merge node adds the result to the run context in two forms, as a grouped object and as plain top level fields, so that the next workflow can read the values directly.
5. The Route by Risk State node sends the run down the right path, as shown in the table below.
6. W07 sends the correct message, and the Build Run Summary node creates a final record with the run status, the risk state, the lowest balance, the approval decision, whether the notification was sent and which stages ran.
7. W08 writes that summary to the audit log.

| State | What happens |
|---|---|
| HEALTHY | Daily summary email through W07, then the audit record |
| WATCH | Alert email through W07, then the audit record |
| AT_RISK | AI analysis through W08, then an urgent alert email through W07 |
| CRITICAL | AI analysis through W08, the approval request through W06, then the outcome email through W07 |
| Any other value | The run is recorded as failed with an unknown risk state |

**Error handling.** Every call to another workflow has an error output. If a stage fails, the Build Failure Record node collects the failed stage, the message and the time, and W08 writes that to the audit log. Retries are switched on for W02, W03, W04 and the AI step, with two tries and a short wait, because repeating them is safe. Retries are switched off for W06 and W07, so an approval request or an email is never sent twice.

**Output.** A run summary with the run ID, trigger type, timings, run status, risk state, balance figures, approval decision and the list of stages that ran.

### W02 Financial Ingestion

**Purpose.** W02 is the front door for data. It makes sure that what comes in is complete and correct, that the same event is never stored twice, and that valid events reach the right table.

**Triggers.** W02 can be called by W01, and it also has its own webhook at the path brightpath/financial-ingestion, which we used to test it directly.

**Supported events.** The workflow handles three kinds of event.

| Event type | Required fields | Stored in |
|---|---|---|
| fuel_price_update | fuel, old_price, new_price | fuel_price_events |
| payment_received | invoice_id, amount, payment_day, reference | transactions |
| invoice_created | invoice_id, client_name, amount, due_day, with an optional status | receivables |

Every event must also include event_type, company_id and event_id.

**How it works.**

1. The Validate and Normalize Event node reads the event, whether it arrives in the webhook body or from W01, and checks it. Prices and amounts must be real numbers greater than zero. Day fields must be whole numbers that are not negative. An invoice status, when given, must be Open, Paid, Overdue or Cancelled. Any other event type is rejected as unsupported. The node then returns one clean record with the same fields every time, together with a validity flag and a list of any errors.
2. An invalid event goes straight to a response that says it was rejected and lists the problems.
3. A valid event goes to a switch that sends it to the fuel, payment or invoice branch.
4. In each branch, the workflow looks in the matching table for the same event_id. If it is already there, the event is a duplicate. The workflow answers that no new record was created and stores nothing.
5. If the event is new, it is saved, and the workflow answers with the status stored and the ID of the new record.

**Output.** A response with success true or false, a status of stored, duplicate or invalid, the event ID, a message and, where relevant, the record ID or the validation errors.

**Why duplicates matter.** Events can be sent twice by a retry, a slow network or a person. Without the event_id check, one fuel price rise could be stored twice and distort the forecast.

### W03 Forecast Engine

**Purpose.** W03 builds the 30 day cash forecast. It is the part of the system that does the arithmetic.

**Input.** The company ID, the run ID, the horizon in days and, optionally, the current day.

**How it works.**

1. Five Supabase nodes read the data for the company: the cash position, the receivables, the payables, the commitments and the fuel price events.
2. The Build Forecast code node takes the Day 0 row of the cash position as the opening balance. If there is no cash position, the workflow stops with a clear error instead of guessing.
3. It takes the newest row in fuel_price_events as the latest diesel price. If no fuel event exists, it uses the normal price of N1,250.
4. It keeps the receivables whose status is Open, and the payables and commitments that are not settled.
5. A diesel payable is recognised by its name or by having a quantity but no amount. Its amount is the quantity in litres multiplied by the latest price, and that same price is used for all four weekly fuel payments, as the brief requires.
6. The code then loops through every day from Day 0 to the end of the horizon. On each day it adds the invoices due, subtracts the bills and commitments due, updates the balance, and keeps track of the lowest balance, the day it happens and the first day the balance falls below the N4.0m buffer.
7. Days to breach is that first breach day minus the current day. The current day comes from the webhook day, or from the day of the latest fuel event. If neither is known it counts from Day 0.
8. The forecast is saved as one row in the forecasts table, and the full result is returned.

**Output.** The forecast run ID, current cash, expected inflows, expected outflows, the minimum projected balance, the day of the minimum, days to breach, the safety buffer, the fuel price used, the list of open invoices and the day by day forecast.

**Check against the brief.** We ran the forecast against the sample data at each price and compared it with the project brief.

| Diesel price | Lowest balance | Day | Days to breach |
|---|---|---|---|
| N1,250 | N5.29m | 14 | None |
| N1,500 | N3.85m | 14 | 13 |
| N1,750 | N2.41m | 14 | 11 |
| N2,000 | N0.97m | 14 | 9 |

At the normal price, expected inflows are N26.8m and expected outflows are N21.49m, which also matches the brief.

### W04 Risk Engine

**Purpose.** W04 turns the forecast into a risk state and remembers what the state was before.

**Input.** The company ID, the run ID and the forecast figures, mainly the minimum projected balance, the day of the minimum and the days to breach.

**How it works.**

1. The Get Previous State node reads the earlier rows in risk_alerts for the company. The newest one gives the previous state. If there is none, the previous state is HEALTHY.
2. The Apply Risk Rules code node compares the minimum projected balance with the thresholds. Below N1.0m is CRITICAL, below N3.0m is AT_RISK, below N5.0m is WATCH, and anything else is HEALTHY.
3. It writes a short reason, for example that the lowest projected balance is N0.97m on Day 14.
4. It works out whether the state changed by comparing the new state with the previous one.
5. The Save Alert node stores the alert, and the Return Result node returns it.

**Output.** The risk state, the previous state, whether the state changed and the reason, along with the figures it received.

**Why the previous state is kept.** It lets the system show how the situation develops, from HEALTHY to WATCH to AT_RISK to CRITICAL, and it makes a change of state visible in the data.

### W05 AI Analysis

**Purpose.** W05 asks the AI model to explain the situation in plain language and to suggest actions, while making sure the AI cannot invent figures.

**Input.** The risk state, the lowest projected balance and its day, days to breach, current cash, the diesel price and the open invoices.

**How it works.**

1. The Build Prompt node writes the prompt. It lists the verified figures, including the diesel price, the percentage rise, the extra cost on each weekly payment of 2,880 litres and the open invoices. It tells the model to use only these figures and not to invent any number, client or date. It asks for a JSON answer with exactly four keys, which are summary, cause, urgency and actions. It also asks for three different kinds of action: asking a named client to pay early with a small discount, asking Harbour Fuels to split a payment into two instalments, and a temporary fuel surcharge or a cut in fuel use.
2. The Gemini node sends the prompt to a Gemini Flash model.
3. The Validate Output node reads the answer, removes any code fences, and checks it. The summary and cause must be text. The urgency must be LOW, MEDIUM, HIGH or CRITICAL. There must be exactly three actions and none can be empty. If any check fails, the node throws an error.
4. A good answer is returned with the company ID, run ID, risk state and a note that the source was Gemini.

**Why the error matters.** W05 is meant to fail loudly. The error is what tells W08 that the AI step did not work, so it can retry and, if needed, fall back.

### W06 Human Approval

**Purpose.** At the CRITICAL stage, W06 puts a human in charge. Nothing is approved by the system on its own.

**Input.** The risk figures and the AI summary, cause and actions.

**How it works.**

1. The Prepare Request node builds the email. It states the lowest projected balance, the number of days to breach, the AI summary and cause, and the three recommended actions. The addresses of Mr. Briggs and Mr. Nwankwo are set at the top of this node. During testing they point to a team inbox, and they must be changed to the real addresses for live use.
2. The Create Approval Record node saves a row in the approvals table with the decision Pending and the risk state CRITICAL.
3. The Ask Founder node emails Mr. Briggs using Gmail Send and Wait, with Approve and Disapprove buttons, and the workflow pauses. The wait is 2 hours in real life and 2 minutes in the demo.
4. When he answers, the Evaluate Founder Reply node turns the answer into Approved or Rejected, and records who decided and when.
5. If he does not answer in time, the decision becomes Expired. A reminder is sent to him, and the request is then sent to Mr. Nwankwo in the same way with the subject marked as escalated. The Evaluate Ops Reply node records his decision, and the risk state is set to HUMAN_ESCALATION.
6. The Record Decision node updates the approvals row with the decision, the person, the time and the risk state.
7. The Return Result node returns the decision, who made it and when.

**Allowed decisions.** The approvals table accepts the values Pending, Approved, Rejected and Expired, so those are the words the workflow uses.

**Note.** W06 records the decision. The actions themselves, such as calling a client or Harbour Fuels, are carried out by people.

### W07 Notifications

**Purpose.** W07 sends the emails that tell the owner what is going on.

**Input.** A message type and the content to show.

**How it works.**

1. The Build Email node chooses a subject from the message type. The four types are WATCH, AT_RISK, CRITICAL and DAILY_SUMMARY. It builds an HTML email with the status, the current cash, the lowest projected balance and its day, the safety buffer, the days to breach, and, when they exist, the AI summary, cause and numbered actions.
2. If the AI was not available and the fallback was used, the email opens with a clear note saying it is an automated summary built from the forecast figures.
3. The Send Gmail node sends the message.
4. The Return Result node returns whether it was sent and the Gmail message ID.

**Recipient.** The default recipient is set in the Build Email node. A different address can be passed in the input.

### W08 Reliability and Audit

**Purpose.** W08 makes the system dependable. It protects the AI step against failure and it writes the audit trail.

W08 has two modes. The first node checks whether the input contains an audit event. If it does, W08 is acting as the audit logger. If it does not, W08 is acting as the AI wrapper.

**Audit mode.** The Log Audit Event node writes one row to audit_logs with the run ID, the component, the event type, a status and the details. W01 uses this for the final run summary and for stage failures. The event types used by W01 are RUN_COMPLETED, RUN_ENDED_EARLY, RUN_FAILED and STAGE_FAILED.

**AI wrapper mode.**

1. W08 calls W05 up to three times. Each call is set to continue through an error output instead of stopping the run.
2. When an attempt fails, a log node writes an ai_attempt_failed row that includes the attempt number and the error. The original input is then restored for the next attempt, because the log step replaces the data.
3. If an attempt succeeds, an ai_success row is written and the AI result is returned.
4. If all three attempts fail, the Build Fallback node creates a rule based answer from the forecast figures. It sets the urgency from the state, which is CRITICAL for CRITICAL, HIGH for AT_RISK and MEDIUM otherwise. It writes a summary, a cause and three actions, which are an early payment request with a 2% discount, a split of the next fuel payment into two instalments, and a temporary fuel surcharge with a route review to cut fuel use by 10%. The result is marked with the source fallback, a fallback_used row is written, and the answer is returned.

**Why it matters.** The owner always receives a warning, even when the AI service is down, and the audit log shows exactly what failed, what the system did instead and when it recovered.

## 8. How the requirements are met

| No. | Requirement | How BrightPath satisfies it |
|---|---|---|
| 1 | Two triggers | A daily 07:00 schedule for the risk check and a webhook for incoming events |
| 2 | Three or more integrations | Supabase, Gmail, the Gemini AI API and a simulated fuel price feed sent through the webhook |
| 3 | Persistent database | Supabase tables hold transactions, forecasts, risk alerts, approvals and audit logs |
| 4 | AI component | Gemini explains the risk and recommends actions in a structured format |
| 5 | n8n Code node | JavaScript in W03 builds the forecast and in W04 applies the risk rules, with more Code nodes elsewhere |
| 6 | Sub workflow | W01 calls the other workflows with Execute Workflow nodes |
| 7 | Loop or batch | W03 loops through every day and every payment and invoice |
| 8 | Human approval | W06 pauses at CRITICAL until the founder approves or rejects |
| 9 | Retry and error handling | W08 retries the AI call, uses a fallback and escalates, and W01 has error outputs on every call |
| 10 | Audit log | Every run, failure, decision and recovery is written to audit_logs |
| 11 | Live failure | The AI service is broken on purpose during the demo |
| 12 | State management | The state moves through HEALTHY, WATCH, AT_RISK, CRITICAL and HUMAN_ESCALATION |

## 9. Testing

We tested each workflow on its own first, using the BrightPath sample data, and then ran them together through W01. The main cases are below.

### W02 cases

| Case | Expected result |
|---|---|
| A new fuel price event | Stored, with a record ID |
| The same fuel price event again | Duplicate, nothing new stored |
| A new payment, then the same payment again | Stored, then duplicate |
| A new invoice, then the same invoice again | Stored, then duplicate |
| A payment with a negative amount and no reference | Invalid, with both problems listed |
| An unsupported event type such as bank_transfer | Invalid, with an unsupported event type message |
| An invoice with a status of Pending | Invalid, with an invalid status message |

### W03 and W04 cases

W03 was checked at each diesel price against the table in section 5. W04 was tested by sending the four lowest balances from that table in order.

| Lowest balance | Expected state | Previous state | State changed |
|---|---|---|---|
| N5.29m | HEALTHY | HEALTHY | No |
| N3.85m | WATCH | HEALTHY | Yes |
| N2.41m | AT_RISK | WATCH | Yes |
| N0.97m | CRITICAL | AT_RISK | Yes |

### W05, W06 and W07 cases

| Workflow | Case | Expected result |
|---|---|---|
| W05 | The CRITICAL figures with a diesel price of N2,000 | A summary mentioning N0.97m on Day 14, a cause mentioning the 60% rise, urgency of HIGH or CRITICAL and exactly three actions |
| W06 | The founder clicks Approve | Approved, decided by Mr. Briggs |
| W06 | The founder clicks Disapprove | Rejected, decided by Mr. Briggs |
| W06 | Nobody replies within the wait | A reminder, then escalation, and a decision by Mr. Nwankwo recorded with the state HUMAN_ESCALATION |
| W07 | A CRITICAL message with the AI source set to fallback | An email with the automated summary note, and sent shown as true |
| W07 | A WATCH message with no AI content | A shorter email that still reads properly |

### W08 cases

| Case | Expected result |
|---|---|
| Normal path | The AI result is returned and one ai_success row is written |
| Live failure with a wrong API key | Three ai_attempt_failed rows, one fallback_used row and a result marked fallback |
| Recovery with the key restored | A new ai_success row, so the log shows failure, fallback and recovery |

### Full runs through W01

| Case | Expected result |
|---|---|
| Daily schedule with diesel at N2,000 | N0.97m on Day 14, CRITICAL, AI analysis, approval email, outcome email and a RUN_COMPLETED audit row |
| A fuel price webhook with day 5 | The event is stored, the forecast uses N2,000 and days to breach is 9 |
| The same webhook sent again | The run is skipped as a duplicate and a RUN_ENDED_EARLY audit row is written |
| Live failure during a full run | Fallback email, approval request still delivered, failures visible in the audit log |

## 10. The demonstration

The demo runs for 15 minutes and follows the story.

| Time | Focus | Presenter |
|---|---|---|
| 0:00 to 2:00 | Introduce BrightPath, the cash timing problem and the architecture | Chisom Okafor |
| 2:00 to 5:00 | Post the first fuel price events and show ingestion and stored data | Okonji Brendan and Olamide Kehinde |
| 5:00 to 8:00 | Show the 30 day forecast, the loop and the projected dip to N0.97m | Okoye Ngozi Rosemary |
| 8:00 to 10:00 | Show the status moving to CRITICAL and the AI analysis | Ogunsanwo Ibrahim Ademola and Okorafor Ogbonnaya Kennedy |
| 10:00 to 12:00 | Show the approval email, the founder's decision and the notifications | Oguegbu Ogochukwu Felicity and Adeleye Okikiola Emmanuel |
| 12:00 to 14:00 | Break the AI service on purpose and show retry, fallback and recovery | Okwor Onyinyechi Favour and Okeke Ekene Matthew Daniel |
| 14:00 to 15:00 | Show the audit log, the final state and the requirement checklist | Chisom Okafor |

Before the demo, the test rows should be removed from the tables, the latest fuel event should match the point the demo starts from, and the risk_alerts table should start from a healthy state.

## 11. Setup and deployment

**What you need.** An n8n instance, a Supabase project, a Gmail account and a Google Gemini API key.

**Credentials.** Create three credentials inside n8n: a Supabase API credential, a Gmail OAuth2 credential and a Google Gemini API credential. They are never saved in the workflow files and they are never committed to GitHub. The exported files refer to them only by name.

**Importing.** Import the workflows in this order, saving each one: W02, W03, W04, W05, W06, W07, W08, then W01. W01 comes last because it needs the ID of every other workflow.

**Linking the workflows.** Each Execute Workflow node needs the ID of the workflow it calls. You can find the ID in the address bar when that workflow is open, as the code after workflow in the web address. Choose the By ID option when you paste it. The three AI attempt nodes in W08 call W05. In W01, the nodes for W02, W03, W04, W06 and W07 call those workflows, and the AI analysis node and the two audit nodes all call W08. A called workflow must be saved, because W01 runs the saved version.

**Configuring.** Select your credentials on every Supabase, Gmail and Gemini node. Set the real email addresses for Mr. Briggs and Mr. Nwankwo in W06 and the recipient in W07. If the approval links need to be opened from another device, n8n must be reachable from outside, for example on a cloud n8n instance.

**Starting.** Activate W01 so the daily schedule and the webhook run. The W01 webhook path is brightpath-event.

## 12. Limitations and future work

We want to be clear about what the system does not do yet.

A payment received event is stored as a transaction, but it does not automatically mark the matching invoice as paid, so the receivables table has to be updated for the forecast to stop counting that invoice.

The system works with one company, BP-001. The tables carry a company ID, so more companies could be added, but the workflows have only been tested with BrightPath.

The forecast assumes the latest diesel price applies to all four weekly fuel payments. It does not predict where the price will go next.

The approval workflow records a decision, but it does not carry out the approved actions. People still have to contact the client and the fuel supplier.

The email addresses are set inside the workflow nodes. A later version should read them from the database so they can be changed without editing a workflow.

The 2 minute wait in W06 is meant for the demo. For real use it should be 2 hours, as the brief describes.

Future improvements could include a channel such as WhatsApp or SMS for approvals, automatic invoice matching, a live fuel price feed instead of a simulated one, and a simple dashboard that reads the forecasts and risk_alerts tables.

## 13. Team and responsibilities

The project has 11 members. Ogunleye Oluwatimilehin Favour is the Documentation Lead and does not own a workflow. The other ten members each take one main build responsibility and one supporting responsibility.

| Member | Main responsibility | Supporting responsibility |
|---|---|---|
| Chisom Okafor (Team Lead) | W01 Main Controller | Integrator, merges approved workflows and leads the final review |
| Okoye Ngozi Rosemary (Assistant Team Lead) | W03 Forecast Engine | Task verification and poll oversight |
| Okonji Brendan | W02 Financial Ingestion | API and integration research, and the simulated fuel price feed |
| Ogunsanwo Ibrahim Ademola | W04 Risk Engine | Financial logic lead for thresholds, scenario numbers and the state flow |
| Okorafor Ogbonnaya Kennedy | W05 AI Analysis | Prompt design, structured output format and validation rules |
| Oguegbu Ogochukwu Felicity | W06 Human Approval | Slide deck and presentation lead |
| Adeleye Okikiola Emmanuel | W07 Notifications | GitHub custodian and cloud migration lead |
| Okwor Onyinyechi Favour | W08 Reliability and Audit | Design of the live failure scenario with the QA lead |
| Olamide Kehinde | Database and data lead, Supabase tables and the BrightPath sample data | Support for W02 and W03 data access |
| Okeke Ekene Matthew Daniel | QA and testing lead, test cases, integration tests and the failure test | Demo rehearsal and requirement checklist |
| Ogunleye Oluwatimilehin Favour (Documentation Lead and Co Lead) | Project diary and tracking | GitHub documentation, screenshot archive and the final contribution PDF with the Team Lead |

### Review partners

Each workflow is reviewed by a second member before it is merged.

| Workflow | Owner | Reviewer |
|---|---|---|
| W01 | Chisom Okafor | Okoye Ngozi Rosemary |
| W02 | Okonji Brendan | Olamide Kehinde |
| W03 | Okoye Ngozi Rosemary | Chisom Okafor |
| W04 | Ogunsanwo Ibrahim Ademola | Okonji Brendan |
| W05 | Okorafor Ogbonnaya Kennedy | Ogunsanwo Ibrahim Ademola |
| W06 | Oguegbu Ogochukwu Felicity | Adeleye Okikiola Emmanuel |
| W07 | Adeleye Okikiola Emmanuel | Oguegbu Ogochukwu Felicity |
| W08 | Okwor Onyinyechi Favour | Okeke Ekene Matthew Daniel |

## 14. Conclusion

BrightPath Logistics is a profitable company that can still be hurt by a single bad week, because its money comes in slowly and goes out fast. Our system gives its owner what a bank balance cannot, which is a view of the next 30 days and a clear warning while there is still time to act. It forecasts the cash, applies simple and visible rules, uses AI only to explain verified figures, keeps a human in charge of serious decisions, survives the failure of its own AI service, and records everything it does.
