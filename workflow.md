# Reject Cycle Count System

## Description:

Reject Management Center Cycle count system where operators pick when to count (no fixed schedule). Min 3 months between customer counts.

### Workflow
- On-demand counting - pick any customer that is due
- Multiple customers at once
- Auto-saves everything (crash recovery built-in)
- Pause anytime, resume later
- Auto reminders depending on time

### The Flow:
0. **Main Page** - Picking progress (3 buttons)
1. **Generate Count** - Creating count for customers (checks 3-month rule)
2. **Cycle Count** - Start scanning pallets
3. **Bundle Scanning** - Scan bundles, pause if needed
4. **Pallet Check** - Verify completion, handle MTF/Scrap
5. **Discrepancy Rescan** - Fix any issues/Continue from checkpoint
6. **Documentation** - Reports & emails

---

## 0. Main Page Navigation

Navigation Menu
Generate Count - Start cycle by selecting customer and generating count id and list

Cycle count - Resume cycle count by scanning cycle id for customer

Discrepancy - Resolve discrepancy goes back to pallets marked with discrepancy, able to view replaced lots and initiate rescan/close pallet entirely with approval

```mermaid
flowchart TD
    Start(["CK3: Main Page"]) --> Main["Display 3 Options:<br>━━━━━━━━━━━━━━━━<br>1. Generate Count<br>2. Cycle Count<br>3. Discrepancy"]
    Main --> Choice{"Operator<br>Selection?"}
    Choice -- Generate Count --> Gen["Proceed to Generate Count<br>(Phase 1)"]
    Choice -- Cycle Count --> CC["Proceed to Cycle Count<br>(Phase 3)"]
    Choice -- Discrepancy --> DR["Proceed to Discrepancy<br>(Phase 6)"]

    style Start fill:#90EE90,stroke:#000,stroke-width:3px,color:#000
    style Main fill:#B2DFDB,stroke:#000,stroke-width:2px,color:#000
    style Choice color:#000
    style Gen fill:#87CEEB,stroke:#000,stroke-width:2px,color:#000
    style CC fill:#87CEEB,stroke:#000,stroke-width:2px,color:#000
    style DR fill:#87CEEB,stroke:#000,stroke-width:2px,color:#000



```

---

## 1. Generate Count (On-Demand)
Generation of customer cycle count (need to be within 3 mos. since last generation)

```mermaid
flowchart TB
    Start(["Start: Generate Count"]) --> n1["Operator: Scan Badge ID"]
    n1 --> n2["CK3: Display Customer Selection Screen<br>Show Available Customers with Last Cycle Date"]
    n2 --> n3["Operator: Select Customer/s<br>(Multiple Selection Allowed)"]
    n3 --> n4{"System: Check Last Cycle Date<br>for Each Selected Customer"}
    n4 -- &lt;3 Months Ago --> n5["CK3: Display Error<br>━━━━━━━━━━━━━━━━<br>Cannot Generate Cycle<br>Last cycle was less than 3 months ago<br>Last Cycle Date: [Date]<br>Next Available: [Date]"]
    n5 --> n2
    n4 -- ≥3 Months Ago --> n6["System: Generate Cycle Count List<br>Per Selected Customer/s"]
    n6 --> B1["Group Pallets by Customer"]
    B1 --> B1a["Sort Pallets by Age<br>Priority: Older Pallet Numbers First"]
    B1b["Select 5% Sample of Aging Pallets"] --> B1c["Select 5% Sample Bundles from Sampled Pallets"]
    B2["Create Cycle Count ID"] --> B3["Update Pallet and Bundle List<br>Record: Pallet - Lot - Bundle Location"]
    B3 --> C1["System: Send Email Cycle Count Generated<br>━━━━━━━━━━━━━━━━<br>To: RMC Team<br>Subject: Cycle Count Generated<br>Details: Count ID, Customer/s, List of Pallets/Bundles"]
    C1 --> C2["System: Activate Reminder Email<br>Weekly reminders, Daily for last week<br>Will send if cycle count remains active"]
    C2 --> n7["Email for remaining customers not checked within the year (Once every 3 months)"]
    B1a --> B1b
    B1c --> B2
    n7 --> Next["Cycle Count Generated Successfully<br>Return to Main Page"]

    style Start fill:#87CEEB,stroke:#000,stroke-width:3px,color:#000
    style n1 color:#000
    style n2 fill:#B2DFDB,stroke:#000,stroke-width:2px,color:#000
    style n3 color:#000
    style n4 color:#000
    style n5 fill:#FFB6C1,stroke:#000,stroke-width:2px,color:#000
    style n6 fill:#FFE4B5,stroke:#000,stroke-width:2px,color:#000
    style B1 color:#000
    style B1a color:#000
    style B1b color:#000
    style B1c color:#000
    style B2 color:#000
    style B3 color:#000
    style C1 fill:#FFE4B5,stroke:#000,stroke-width:2px,color:#000
    style C2 fill:#FFE4B5,stroke:#000,stroke-width:2px,color:#000
    style Next fill:#90EE90,stroke:#000,stroke-width:3px,color:#000




```

---

## 2. Cycle Count - Primary Count Execution

Initialising count, finding pallet and verifying

```mermaid
flowchart TD
    Start(["Start: Cycle Count<br>(From Main Page Button 2)"]) --> n1["Operator: Scan Badge and Scan/Input Cycle Count ID on CK3"]
    n1 --> H["CK3: Display Pallet List<br>Show Pallet IDs, Bundle Count & Locations<br>System: Auto-save enabled for session"]
    H --> I["Operator: Navigate to First/Next Pallet Location<br>and Scan Pallet ID"]
    I --> K{"Pallet Matches<br>Count List?"}
    K -- No --> L["CK3: Display Error<br>Invalid Pallet for This Count<br>Pallet Not Found"]
    L --> I
    K -- Yes --> M["CK3: Update Display with Pallet Info<br>Shows Location and Expected Bundle Sample List<br>Ready for Bundle Scanning"]
    M --> Next["Proceed to Bundle Scanning<br>(Phase 3)"]

    style Start fill:#87CEEB,stroke:#000,stroke-width:3px,color:#000
    style n1 color:#000
    style H fill:#B2DFDB,stroke:#000,stroke-width:2px,color:#000
    style I color:#000
    style K color:#000
    style L fill:#FFB6C1,stroke:#000,stroke-width:2px,color:#000
    style M color:#000
    style Next fill:#87CEEB,stroke:#000,stroke-width:2px,color:#000
```

---

## 3. Bundle Scanning & Pause/Resume

Start bundle scanning process flow with cross checking and handling of different scenarios

```mermaid
flowchart TB
    Start(["Start: Bundle Scanning"]) --> n9["System: Initially Blocks Discrepancy Button<br><br>"]
    n9 --> n1["Operator: Scan Bundles Within Pallet"]
    n2{"System: Check if Bundle is<br>Within Sample Bundle List"} -- Yes (Valid Bundle) --> n3["CK3: Sample Bundle is Removed from List<br>Marked as Checked<br>System: Auto-saves Progress"]
    n3 --> n7{"Operator Action?"}
    n7 -- Continue Scanning --> n1
    n7 -- All Bundles Scanned --> n8["Proceed to Completion Verification<br>(Phase 4)"]
    n7 -- Pause & Resume Later --> n10["Operator: Press Discrepancy Button<br>to Mark Bundle as Discrepancy"]
    n10 --> n11["System: Mark Remaining Bundles as Discrepancy<br>Enter reason : To be continued<br>Pallet removed from list<br>Current Progress Saved<br>Return to Main Page"]
    n11 --> n12["To Resume: Use Discrepancy Rescan<br>from Main Page (Phase 6)"]
    n7 -- System Crash/Battery Loss --> n13["System: Auto-saves Progress"]
    n13 --> n14["Operator: Return to Main Page<br>Select Cycle Count Button<br>Select Same Cycle Count ID"]
    n14 --> n15["Resume from Last Scanned Bundle"]
    n15 --> n1
    n2 -- No --> n1
    n1 --> n2

    n9@{ shape: rect}
    style Start fill:#87CEEB,stroke:#000,stroke-width:3px,color:#000
    style n9 fill:#FFE4B5,stroke:#000,stroke-width:2px,color:#000
    style n1 color:#000
    style n2 color:#000
    style n3 fill:#B2DFDB,stroke:#000,stroke-width:2px,color:#000
    style n7 color:#000
    style n8 fill:#87CEEB,stroke:#000,stroke-width:2px,color:#000
    style n10 color:#000
    style n11 fill:#FFE4B5,stroke:#000,stroke-width:2px,color:#000
    style n12 fill:#E1F5FE,stroke:#000,stroke-width:2px,color:#000
    style n13 fill:#90EE90,stroke:#000,stroke-width:2px,color:#000
    style n14 color:#000
    style n15 fill:#B2DFDB,stroke:#000,stroke-width:2px,color:#00
```

---

## 4. Pallet Completion Verification

All Sample bundles checked and verified

```mermaid
flowchart TB
    Start(["Start: Pallet Completion Verification"]) --> n1{"Any Sample Bundle<br>Left in List?"}
    n1 -- No (All Bundles Scanned) --> n16["System: Allows to proceed"]
    n16 --> n17["System: Mark Pallet as Completed<br>Auto-save Progress"]
    n17 --> n21["Remove Pallet from Pallet List"]
    n21 --> n18{"All Pallets in<br>Cycle Completed?"}
    n18 -- Yes --> n19(["Proceed to Completion Check<br>(Phase 7)"])
    n18 -- No --> n20(["Return to Pallet List<br>(Phase 2)"])

    style Start fill:#87CEEB,stroke:#000,stroke-width:3px,color:#000
    style n1 color:#000
    style n16 fill:#90EE90,stroke:#000,stroke-width:2px,color:#000
    style n17 fill:#B2DFDB,stroke:#000,stroke-width:2px,color:#000
    style n21 color:#000
    style n18 color:#000
    style n19 fill:#87CEEB,stroke:#000,stroke-width:3px,color:#000
    style n20 fill:#FFF9C4,stroke:#000,stroke-width:2px,color:#000
```
Sample bundles unchecked -> Discrepancy logging
```mermaid
flowchart TB
    Start(["Start: Pallet Completion Verification"]) --> n1{"Any Sample Bundle<br>Left in List?"}
    n1 -- Yes (Bundles Remaining) --> n15["System Block - Discrepancy &amp; Proceed"]
    n15 --> n2["Operator: Press Proceed Button<br>System: Auto-refresh Triggered"]
    n2 --> n3{"Is There Any MTF/Scrap list<br>matching for bundles in List?"}
    n3 -- Yes (MTF/Scrap Found) --> n4["CK3: Display Text Box<br>━━━━━━━━━━━━━━━━<br>MTF/Scrap List for Bundle Detected<br>Bundle Queued for Replacement System"]
    n4 --> n5["System: Mark Sample Bundles as MTF/Scrap<br>Add to Daily Replacement Queue"]
    n5 --> n25["CK3: Display Text Box<br>━━━━━━━━━━━━━━━━<br>Bundle Added to Replacement Queue<br>Marking Pallet as Pending"]
    n25 --> n26["System: Mark Pallet as Pending Replacement"]
    n3 -- No, Discrepancy --> n24["System Unblock - Discrepancy button"]
    n24 --> A(["Rescan &amp; Discrepancy Handling"])
    n26 --> n20(["Allowed to Proceed and Return to Pallet List<br>(Phase 2)"])
    A --> n20

    style Start fill:#87CEEB,stroke:#000,stroke-width:3px,color:#000
    style n1 color:#000
    style n15 fill:#FF5252,stroke:#000,stroke-width:3px,color:#FFF
    style n2 fill:#B2DFDB,stroke:#000,stroke-width:2px,color:#000
    style n3 color:#000
    style n4 fill:#E1BEE7,stroke:#000,stroke-width:2px,color:#000
    style n5 fill:#FFE4B5,stroke:#000,stroke-width:2px,color:#000
    style n25 fill:#FFF59D,stroke:#000,stroke-width:2px,color:#000
    style n26 fill:#FFE4B5,stroke:#000,stroke-width:2px,color:#000
    style n24 fill:#FF5252,stroke:#000,stroke-width:3px,color:#FFF
    style A fill:#FFB74D,stroke:#000,stroke-width:3px,color:#000
    style n20 fill:#FFF9C4,stroke:#000,stroke-width:2px,color:#000

```
Rescan/File discrepancy for bundles not found (need supervisor badge)
```mermaid
---
config:
  layout: fixed
---
flowchart TB
    B(["Rescan/Discrepancy: MTF/Scrap Not Found"]) --> n8["CK3: Display Text Box<br>━━━━━━━━━━━━━━━━<br>MTF/Scrap Not Found<br>Prompt Operator to Rescan"]
    n8 --> n9["Operator: Rescan Pallet"]
    n9 --> n10{"Bundle Found<br>on Rescan?"}
    n10 -- Yes --> n12["Mark as Checked and Remove from List"]
    n12 --> n1(["Allowed to proceed"])
    n10 -- No (Still Missing) --> n11["Operator: Press Discrepancy Button"]
    n11 --> n13["Operator: Select Reason. if missing scan supervisor badge to continue"]
    n13 --> n14["System: Bundle Marked as Not Found<br>Remark Saved and Compiled into Discrepancy Report"]
    n14 --> n1

    style B fill:#FFB74D,stroke:#000,stroke-width:3px,color:#000
    style n8 fill:#E1F5FE,stroke:#000,stroke-width:2px,color:#000
    style n9 color:#000
    style n10 color:#000
    style n12 color:#000
    style n1 fill:#FFB74D,stroke:#000,stroke-width:3px,color:#000
    style n11 color:#000
    style n13 color:#000
    style n14 fill:#FFB6C1,stroke:#000,stroke-width:2px,color:#000
```
Replacement system scheduled to run each day for replacing MTF and bundles confirmed to be not found
```mermaid
flowchart TB
    D1(["Start: Replacement System<br>(Scheduled/ on demand)"]) --> D2["System: Process Replacement Queue"]
    D2 --> D3{"Pending Pallet<br>in Queue?"}
    D3 -- Yes --> D4["System: Retrieve all Pallet Details<br>and MTF/Scrap Bundle List"]
    D4 --> D5{"Sufficient Available<br>Bundles to Replace?"}
    D5 -- Yes --> D6["System: Replace MTF/Confirmed Not found Bundles<br>with Available Bundles from Same Pallet"]
    D6 --> D7["Update Pallet Bundle List"]
    D7 --> D8["Mark Bundle as Ready for Rescan<br>Notify operator with changes"]
    D8 --> D9["Remove Pallet from Replacement Queue"]
    D9 --> D3
    D5 -- No (Insufficient Bundles) --> D10["System: Mark Pallet as Insufficient Bundles"]
    D10 --> D11["Reflect on discrepancy page, prompt operator to scan badge to close pallet"]
    D3 -- No --> D14(["End: Replacement System"])

    style D1 fill:#4CAF50,stroke:#000,stroke-width:3px,color:#FFF
    style D2 fill:#B2DFDB,stroke:#000,stroke-width:2px,color:#000
    style D3 color:#000
    style D4 fill:#E1BEE7,stroke:#000,stroke-width:2px,color:#000
    style D5 color:#000
    style D6 fill:#90EE90,stroke:#000,stroke-width:2px,color:#000
    style D7 fill:#B2DFDB,stroke:#000,stroke-width:2px,color:#000
    style D8 fill:#FFF59D,stroke:#000,stroke-width:2px,color:#000
    style D9 fill:#B2DFDB,stroke:#000,stroke-width:2px,color:#000
    style D10 fill:#FFB6C1,stroke:#000,stroke-width:2px,color:#000
    style D11 fill:#FFB6C1,stroke:#000,stroke-width:2px,color:#000
    style D14 fill:#4CAF50,stroke:#000,stroke-width:3px,color:#FFF

```

---

## 5. Discrepancy Resolve

Handling discrepancy/ Resuming paused pallets

```mermaid
flowchart TB
    Start(["Start: Discrepancy<br>(From Main Page Button 3)"]) --> n1["Operator: Scan Badge ID"]
    n1 --> n2["CK3: Display All Pallets with Discrepancy Bundles<br>━━━━━━━━━━━━━━━━<br>Shows: Pallet ID, Location, Discrepancy Count"]
    n2 --> n3["Operator: Select Pallet"]
    n3 --> n4["CK3: Display Bundle List for Selected Pallet<br>Shows Only Bundles Marked as Discrepancy"]
    n4 --> n13["Rescan Button"] & n14["Close pallet button"]
    n5["Operator: Scan Discrepancy Bundle"] --> n6{"Bundle Scanned<br>Successfully?"}
    n6 -- Yes (Found) --> n7["System: Remove Discrepancy Mark<br>Mark Bundle as Checked<br>Remove from Discrepancy List<br>Auto-save Progress"]
    n6 -- No (Still Not Found) --> n8["Bundle Remains Marked as Discrepancy<br>Will Appear in Final Report"]
    n7 --> n9{"More Discrepancy<br>Bundles in Pallet?"}
    n8 --> n9
    n9 -- Yes --> n5
    n9 -- No (All Addressed) --> n10["System: Mark Pallet as Completed<br>Remove from Discrepancy List"]
    n10 --> n11{"More Pallets with<br>Discrepancies?"}
    n11 -- Yes --> n2
    n11 -- No (All Completed) --> n12["Return to Main Page<br>or Proceed to Completion Check"]
    n13 --> n5
    n14 --> n15["Scan Supervisor Badge for confirmation"]
    n15 --> n16["Pallet marked as closed and removed from discrepancy page"]

    style Start fill:#87CEEB,stroke:#000,stroke-width:3px,color:#000
    style n1 color:#000
    style n2 fill:#B2DFDB,stroke:#000,stroke-width:2px,color:#000
    style n3 color:#000
    style n4 fill:#E1F5FE,stroke:#000,stroke-width:2px,color:#000
    style n5 color:#000
    style n6 color:#000
    style n7 fill:#90EE90,stroke:#000,stroke-width:2px,color:#000
    style n8 fill:#FFB6C1,stroke:#000,stroke-width:2px,color:#000
    style n9 color:#000
    style n10 fill:#B2DFDB,stroke:#000,stroke-width:2px,color:#000
    style n11 color:#000
    style n12 fill:#87CEEB,stroke:#000,stroke-width:2px,color:#000
```

---

## 6. Completion & Coverage Check

System generates summary and sends emails.

```mermaid
flowchart TD
    Start(["Start: Completion Check"]) --> DC1{"System: Check for<br>Discrepancies?"}
    DC1 -- Discrepancies Found --> DC2["System: Generate Discrepancy Report"]
    DC2 --> DC3["Display Discrepancy Report:<br>━━━━━━━━━━━━━━━━━━━━<br>• Pallet - Lot - Bundle Details<br>• MTF Bundles and Replacements<br>• Reasons for Discrepancies<br>• Timestamps<br>• Operator Badge IDs"]
    DC3 --> DC4["System: Send Email Notification<br>━━━━━━━━━━━━━━━━<br>To: RMC Team &amp; Supervisors<br>Subject: Cycle Count Complete - Discrepancies Found<br>Include: Summary of Issues &amp; Attached Report"]
    DC1 -- No Discrepancies --> ND1["System: Generate Summary Report<br>All Items Accounted For"]
    ND1 --> DC5["CK3: Display Customers that have yet to have cycle count wihin year<br>"]
    DC5 --> NEXT2["System: Update Customer Last Cycle Date<br>Mark Cycle for Customer as Completed for the year<br>Stop Weekly Reminder Emails"]
    NEXT2 --> Restart["Return to Main Page<br>(Phase 0)"]
    DC4 --> DC5

    style Start fill:#87CEEB,stroke:#000,stroke-width:3px,color:#000
    style DC1 color:#000
    style DC2 fill:#FFE4B5,stroke:#000,stroke-width:2px,color:#000
    style DC3 fill:#FFA500,stroke:#000,stroke-width:2px,color:#000
    style DC4 fill:#FFE4B5,stroke:#000,stroke-width:2px,color:#000
    style ND1 fill:#FFE4B5,stroke:#000,stroke-width:2px,color:#000
    style DC5 fill:#BBDEFB,stroke:#000,stroke-width:2px,color:#000
    style NEXT2 fill:#90EE90,stroke:#000,stroke-width:2px,color:#000
    style Restart fill:#87CEEB,stroke:#000,stroke-width:2px,color:#000

```

---

## Pallet picking logic

System auto-refreshes sample lists when scrap lists get generated.

```mermaid
flowchart TD
    n1(["Internal background refresh system for pallet picking"]) --> n4["Scheduled (everytime scrap list generated) trigger refresh of list"]
    n2["Determine amount of available bundles within remaining aging pallets"] --> n3["Able to make sample size?"]
    n4 --> n7["Is sample bundle in scrap list?"]
    n3 -- No --> n5["Replace sample pallet with another pallet of the same customer with sufficient available bundles and generate a new list of sample bundles"]
    n3 -- Yes --> n6["Keep pallet and list"]
    n7 -- Yes --> n8["Replace with sample bundle with available bundle"]
    n7 -- No --> n9["No replacement needed"]
    n8 --> n2
    n9 --> n2

    n3@{ shape: diam}
    n7@{ shape: diam}

    style n1 fill:#87CEEB,stroke:#000,stroke-width:3px,color:#000
    style n2 color:#000
    style n3 color:#000
    style n4 fill:#FFE4B5,stroke:#000,stroke-width:2px,color:#000
    style n5 fill:#FFE4B5,stroke:#000,stroke-width:2px,color:#000
    style n6 fill:#90EE90,stroke:#000,stroke-width:2px,color:#000
    style n7 color:#000
    style n8 fill:#B2DFDB,stroke:#000,stroke-width:2px,color:#000
    style n9 fill:#90EE90,stroke:#000,stroke-width:2px,color:#000
```

---

## Email Notifications

The system sends these emails automatically:

### Type 1: Cycle Count Generated
**When:** New cycle count created (Phase 1)  
**To:** RMC Team  
**Contains:**
- Cycle Count ID
- Customer(s) selected
- Pallet lists
- Generation date & operator badge

### Type 2: Weekly Reminders after activation
**When:** Every week while count is active (daily in final week)  
**To:** RMC Team & Supervisors  
**Contains:**
- Cycle Count ID
- Customer(s)
- Days until due date
- Date when count started
- Current progress (%)
- **Stops:** When cycle completed (Phase 6)

### Type 3: Customers Not Checked (Every 3 Months)
**When:** Every 3 Months
**To:** RMC Team & Management  
**Contains:**
- Customers not checked within the year
- Last cycle dates

### Type 4: Completion summary
**When:** Cycle done (Phase 6)  
**To:** RMC Team & Supervisors  
**Contains:**
- Full discrepancy report if any
- Issue summary
- Total discrepancy count if any
- MTF/Scrap bundle counts

---

## Process Flow Summary

```
Phase 0: Main Page
    ├─ Button 1 → Phase 1: Generate Count
    │              └─ Back to Main Page
    │
    ├─ Button 2 → Phase 2: Start Counting
    │              ↓
    │              Phase 3: Scan Bundles
    │              ↓ (pause → Phase 5 or crash recovery)
    │              ↓
    │              Phase 4: Check Pallet
    │              ↓ (loops to Phase 2 if more pallets)
    │              ↓
    │              Phase 6: Finish
    │              └─ Back to Main Page
    │
    └─ Button 3 → Phase 5: Fix Discrepancies
                   └─ Back to Phase 6 or Main Page


```

---

## Color Legend

What the colors mean in the flowcharts:

- Green (#90EE90) - Success/completion
- Blue (#87CEEB) - Phase transitions, start/end
- Yellow (#FFE4B5) - System operations (auto-save, emails, scheduling)
- Orange (#FFA500) - Notifications (discrepancies, MTF alerts, missing items)
- Pink (#FFB6C1) - Errors/needs attention (discrepancy logs, error messages)
- Red (#FF5252) - System blocks prevention of progressing
- Purple (#E1BEE7) - MTF-related
- Light Blue (#E1F5FE) - Info displays (rescan ops, summaries)
- Teal (#B2DFDB) - Updates happening (saving progress, updating lists, replacements)
