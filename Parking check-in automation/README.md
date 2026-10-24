# Automation Design for Campus Parking Check-In Log

## 1. Process Selection & Methodology

### Selected Workflow
* **Process Name:** Campus Parking & Security Check-In Log.
* **Scope:** Physical pen-and-paper check-in and check-out logbook maintained by security guards at the external parking lot adjacent to the ICFAI campus.

### Investigation & Fieldwork Methodology
To observe and document the workflow without operational disruption or collecting sensitive personal data:
* **Observation Mode:** Direct time-and-motion walkthrough during peak morning arrival hours (08:45 AM – 09:30 AM) and afternoon departure hours.
* **Anonymity & Compliance:** No student names, contact numbers, or full license plate identities were recorded. Data points were recorded purely as aggregated time durations and arrival/departure counts.

---

## 2. Current State Evidence Log

### Step-by-Step Current Process Mapping

1. **Park Vehicle :** Student enters the parking zone, locates an open parking spot, parks the vehicle, and shuts off the engine.
2. **Walk to Security Desk :** Student walks from their parked vehicle across the lot to the security guard desk.
3. **Queueing :** Student waits for other students ahead of them to finish writing in the single physical ledger.
4. **Manual Ledger Entry :** Student writes down name, contact number, vehicle registration number, and check-in time across columns.
5. **Checkout Logging :** Upon leaving, the student walks back to the security desk, locates their original entry row in the ledger, logs their exit time, and proceeds to their parked vehicle.

---

### Dated Evidence Log & Time-and-Motion Measurements

* **Observation Window:** Peak arrival window (08:45 AM – 09:20 AM) across 3 consecutive academic weekdays.
* **Sample Size:** 45 total vehicle arrivals observed at the security desk.

| Step # | Process Action | Average Active Touch Time | Average Queue / Transit Delay |
| :--- | :--- | :--- | :--- |
| **1** | Park & Secure Vehicle | 20 seconds | 0 seconds |
| **2** | Walk to Security Desk | 15 seconds | 0 seconds |
| **3** | Waiting for Ledger | 0 seconds (if clear) | 35–90 seconds (peak rush) |
| **4** | Manual Entry | 45 seconds | 0 seconds |
| **5** | Log exit Time | 30 seconds | 20–45 seconds |
| **Total** | **End-to-End Cycle (Check-In + Check-Out)** | **110 seconds (Touch time)** | **55–135 seconds (Total Delay)** |

---

### Fact vs. Estimate vs. Assumption Breakdown

* **Measured Facts:**
  * Students park their vehicles first before walking to the desk to make a manual entry.
  * Active writing touch time per check-in averages **45 seconds** per student.
  * Only **1 student** can write in the ledger at a time.
* **Estimates:**
  * Non-compliance rate (students parking and walking straight to campus without visiting the desk during morning rush) is estimated at **60% to 70%**.
  * Illegible entries (unreadable vehicle numbers or phone digits due to rushed handwriting) account for **~35% of total entries**.
* **Assumptions:**
  * Total daily parking throughput spans approximately **200 to 400 vehicles**.

---

## 3. Operational Problem & ROI Metrics

### Problem Statement
The requirement to walk to a central security desk and wait in line to manually sign a ledger creates severe friction. Because writing by hand takes ~45 seconds per student and multiple students arrive simultaneously, physical queues form around the desk. To avoid being late for classes after already parking, over 60% of students skip walking to the desk entirely. Security guards stop enforcing check-in to avoid constant confrontation, leaving the security record incomplete when vehicles are blocked or damaged.

---

### ROI & Effort Calculation Formulas

Current Touch Time per Check-In = 45 seconds = 0.75 minutes

Daily Total Check-In Effort (300 Vehicles) = 300 * 0.75 mins = 225 minutes (3.75 hours)

---

### Scaled Impact Analysis (Base: 300 Daily Vehicles)

* **Current Cumulative Waiting + Entry Time Lost:**
  300 students * (45 student entries + 60 student queue/transit delay) = 31,500 seconds = 8.75 hours of cumulative student time lost daily

* **Proposed Automated Express Check-In Time:**
  * Student scans static QR code posters placed at the desk on their smartphone: 2 seconds
  * System auto-detects profile & vehicle plate, logs entry instantly: 1 second
  * **Total Express Touch Time:** **3 seconds (0.05 minutes)** per student.

* **Cumulative Student Time Spent Daily (Automated):**
  300 students * 3 seconds = 900 seconds = 0.25 hours daily

* **Net Daily Time Saved Across Campus:**
  8.75 hours - 0.25 hours = 8.50 cumulative hours saved daily (~97% time reduction)

* **Compliance Restoration:** Reduces check-in time from 45s to 3s and allows scanning via mobile QR code, restoring check-in compliance from **35% up to ~98%**.

---

## 4. Target State Workflow Design

```text
[Student Parks Vehicle in Lot]
               │
               ▼
[Scans QR Code Poster via Smartphone at Desk]
               │
               ▼
   [Express Validation & Auto-Fill]
   ├── Registered Student? ──► [Instant Auto-Check-In Logged (3s)]
   └── Unregistered Guest? ──► [15-Second Mobile Web Form]
               │
               ▼
[Real-Time Security Dashboard Updated] ──► [One-Tap Mobile Checkout on Exit]
```
---

## 5. Operational Boundaries & Software vs. AI Value Split

* **Accountable Owner:** Campus Chief Security Officer & Administrative Office.
* **Approval Boundary:** Fully automated self-service check-in for pre-registered student vehicles. Guest or unregistered vehicles trigger an alert on the guard's tablet.
* **Audit Record:** Encrypted timestamped entry containing masked vehicle plate number, student ID token, arrival timestamp, and exit timestamp.
* **Fallback Mode:** If mobile network or tablet power fails, guard switches to a secondary offline mobile app or fallback manual entry without halting traffic.

### Software vs. AI Split
* **Standard Software (Forms/DB/QR):** Handles user registration, QR code generation, database timestamps, and security dashboard UI.
* **AI Value Add:** Automated License Plate Recognition (ALPR) using a camera to automatically capture plate numbers as vehicles enter the lot, reducing manual touch time down to 0 seconds.

---

## 6. Synthetic Test Matrix & Risk Analysis

### 5 Synthetic Test Scenarios


| Case ID | Scenario Type | Input / Action | System Behavior & Path | Expected Result & Recovery | Accountable Decider |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **TC-01** | **Normal** | Pre-registered student scans lot QR code on arrival. | App validates session token, logs arrival timestamp, updates gate dashboard. | Instant green checkmark displayed; entry recorded in $< 3$ seconds. | Automated System |
| **TC-02** | **Missing Information** | Student submits digital check-in form without entering vehicle license plate number. | Form validation catches missing required field before database commit. | UI highlights missing field in red with prompt ("Please enter vehicle number"); submission blocked until filled. | Student / UI Validator |
| **TC-03** | **Duplicate** | Student accidentally scans and submits check-in QR code twice within 2 minutes. | System checks active session database and detects an open check-in for this user/vehicle. | App displays message: *"You are already checked in at South Lot"* and prevents duplicate log creation. | Automated System |
| **TC-04** | **Unauthorized Request** | An unrecognized or blocked vehicle ID/token attempts to log entry via the public portal. | Security rules engine flags the ID against the restricted vehicle registry. | Access denied alert triggered on security dashboard; prompts user to report to campus security desk. | Security Guard |
| **TC-05** | **Failed Step** | Mobile network/cellular data drops mid-submission during peak morning arrival hour. | App encounters timeout on HTTP POST request to cloud server. | App automatically switches to local offline cache mode, saves timestamp locally, and syncs automatically once network restores. | IT Security Admin |
