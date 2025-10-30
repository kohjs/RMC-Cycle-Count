# Reject Cycle Count System - Flowcharts

## Overview
This system is broken down into 5 main phases for better readability:
1. **System Initialization & Scheduling**
2. **Primary Count Execution**
3. **Bundle Scanning**
4. **Pallet Completion Verification**
5. **Completion & Coverage Check**

---

## 1. System Initialization & Scheduling

```mermaid
flowchart TD
    Start(["Start: Reject Cycle Count System"]) --> n1["Record Full Cycle Count Start Date<br>Record Existing RMC Data"]
    n1 --> A["System: Schedule Cycle Count per Month<br>Generate Customer/s (determined by size and priority) per Month<br>Based on Total Customer Count"]
    A --> B["System: Generate Cycle Count List<br>Per Selected Customer"]
    B --> B1["Group Pallets by Customer"]
    B1b["Select 5% Sample of Aging Pallets"] --> B1c["Select Sample Bundles from Sampled Pallets"]
    B2["Create Cycle Count ID"] --> B3["Update Pallet and Bundle List in Database<br>Status: Pending<br>Record: Pallet - Lot - Bundle Location"]
    B3 --> C["System: Send Email Alert to RMC Team<br>Subject: Cycle Count Due<br>Details: Count ID, Customer, Due Date,<br>List of Pallets, Bundles and Locations to Check"]
    C --> D{"Current Cycle Count<br>Completed?"}
    D -- No --> E["System: Send Weekly Reminder Email<br>Until Count Completed"]
    E --> D
    D -- Yes --> Next["Proceed to Primary Count"]
    B1c --> n3["Internal background refresh system"]
    n2["Filter pallets that have sufficient available bundles"] --> B1b
    B1 --> B1a["Sort Pallets by Age<br>Priority: Older Pallet Numbers First"]
    B1a --> n2
    n3 --> B2

    n3@{ shape: rect}
    n2@{ shape: rect}
    style Start fill:#90EE90,stroke:#000,stroke-width:3px,color:#000
    style n1 color:#000
    style A fill:#FFE4B5,stroke:#000,stroke-width:2px,color:#000
    style B fill:#FFE4B5,stroke:#000,stroke-width:2px,color:#000
    style B1 color:#000
    style B1b color:#000
    style B1c color:#000
    style B2 color:#000
    style B3 color:#000
    style C fill:#FFE4B5,stroke:#000,stroke-width:2px,color:#000
    style D color:#000
    style E fill:#FFE4B5,stroke:#000,stroke-width:2px,color:#000
    style Next fill:#87CEEB,stroke:#000,stroke-width:2px,color:#000
    style B1a color:#000
    style n2 fill:#FFE4B5,stroke:#000,stroke-width:2px,color:#000
    style n3 fill:#FFE4B5,stroke:#000,stroke-width:2px,color:#000
```

---

## 2. Primary Count Execution

```mermaid
flowchart TD
    Start(["Start: Primary Count"]) --> n1["Operator: Scan Badge and Scan/Input Cycle Count ID on CK3"]
    n1 --> H["CK3: Display Bundle List<br>Show Bundle IDs, Pallet IDs & Locations"]
    H --> I["Operator: Navigate to First/Next Pallet Location<br>and Scan Pallet ID"]
    I --> K{"Pallet Matches<br>Count List?"}
    K -- No --> L["CK3: Display Error<br>Invalid Pallet for This Count<br>Pallet Not Found"]
    L --> I
    K -- Yes --> M["CK3: Update Display with Pallet Info<br>Shows Location and Expected Bundle Count List<br>Ready for Bundle Counting"]
    M --> Next["Proceed to Bundle Validation"]

    style Start fill:#87CEEB,stroke:#000,stroke-width:3px,color:#000
    style n1 color:#000
    style H color:#000
    style I color:#000
    style K color:#000
    style L fill:#FFB6C1,stroke:#000,stroke-width:2px,color:#000
    style M color:#000
    style Next fill:#87CEEB,stroke:#000,stroke-width:2px,color:#000
```

---

## 3. Bundle Scanning 

```mermaid
flowchart TD
    Start(["Start: Bundle Scanning"]) --> n9["System: Blocks back to Pallet List button"]
    n1["Operator: Scan Bundles Within Pallet"] --> n2{"System: Check if Bundle is<br>Within Sample Bundle List"}
    n2 -- No --> n1
    n3["CK3: Sample Bundle is Removed from List<br>Marked as Checked"] --> n7{"Operator: Finish scanning bundles in current pallet?"}
    n7 -- Yes --> n8["Proceed to Completion Verification"]
    n7 -- No, Continue Scanning --> n1
    n2 -- Yes --> n3
    n9 --> n1

    n9@{ shape: rect}
    style Start fill:#87CEEB,stroke:#000,stroke-width:3px,color:#000
    style n1 color:#000
    style n2 color:#000
    style n3 color:#000
    style n7 color:#000
    style n8 fill:#87CEEB,stroke:#000,stroke-width:2px,color:#000
    style n9 fill:#FF5252,stroke:#000,stroke-width:3px,color:#FFF
```

---

## 4. Pallet Completion Verification

```mermaid
flowchart TD
    Start(["Start: Pallet Completion Verification"]) --> n1{"Any Sample Bundle<br>Left in List?"}
    n1 -- No --> n16["System Unblocks, Allowed to go back to pallet list"]
    n1 -- Yes --> n15["System Block - Unable to go back to pallet list"]
    n15 --> n2["CK3: Refresh Function"]
    n2 --> n3{"Is There Any MTF/Scrap list matching for bundlesin List?"}
    n3 -- Yes --> n4["CK3: Display Text Box<br>MTF Request/Scrap list for Bundle Detected<br>Refreshing Bundles"]
    n4 --> n5["System: Mark Sample Bundles as MTF/Scrap list<br>Replace Bundles with Different Bundle<br>Within Same Box"]
    n5 --> n6["CK3: Update List with Different Bundles<br>from the Same Pallet"]
    n6 --> n9["Operator: Rescan Pallet"]
    n3 -- No --> n8["CK3: Display Text Box<br>MTF Request Not Found for Bundles in List<br>Prompt Operator to Rescan"]
    n8 --> n9
    n9 --> n10{"Found?"}
    n10 -- Yes --> n12["Mark as Checked and Remove from List"]
    n12 --> n1
    n10 -- No, Discrepancy --> n11["Operator: Press Discrepancy Button"]
    n11 --> n13["Operator: Scan Badge ID and Select Reason"]
    n13 --> n14["System: Bundle Marked as Discrepancy<br>Remark Saved and Compiled into Discrepancy Report"]
    n14 --> n1
    n16 --> n17["System: Mark Pallet as Completed"]
    n17 --> n21["Remove Pallet from Pallet list"]
    n18{"All Pallets in<br>Cycle Completed?"} -- Yes --> n19(["Proceed to Cycle Completion Check"])
    n18 -- No --> n20(["Go Back to Primary Count"])
    n21 --> n18

    style Start fill:#87CEEB,stroke:#000,stroke-width:3px,color:#000
    style n1 color:#000
    style n16 fill:#90EE90,stroke:#000,stroke-width:2px,color:#000
    style n15 fill:#FF5252,stroke:#000,stroke-width:3px,color:#FFF
    style n2 fill:#B2DFDB,stroke:#000,stroke-width:2px,color:#000
    style n3 color:#000
    style n4 fill:#E1BEE7,stroke:#000,stroke-width:2px,color:#000
    style n5 fill:#FFE4B5,stroke:#000,stroke-width:2px,color:#000
    style n6 fill:#B2DFDB,stroke:#000,stroke-width:2px,color:#000
    style n9 color:#000
    style n8 fill:#E1F5FE,stroke:#000,stroke-width:2px,color:#000
    style n10 color:#000
    style n12 color:#000
    style n11 color:#000
    style n13 color:#000
    style n14 fill:#FFB6C1,stroke:#000,stroke-width:2px,color:#000
    style n17 color:#000
    style n18 color:#000
    style n19 fill:#87CEEB,stroke:#000,stroke-width:3px,color:#000
    style n20 fill:#FFF9C4,stroke:#000,stroke-width:2px,color:#000
```

---

## 5. Completion & Coverage Check

```mermaid
flowchart TD
    Start(["Start: Completion Check"]) --> DC1{"System: Check for<br>Discrepancies?"}
    DC1 -- Discrepancies Found --> DC2["System: Generate Discrepancy Report"]
    DC2 --> DC3["Display Discrepancy Report:<br>━━━━━━━━━━━━━━━━━━━━<br>• Pallet - Lot - Bundle Details<br>• MTF Bundles and Replacements<br>• Reasons for Discrepancies<br>• Timestamps"]
    DC3 --> DC4["System: Send Email Notification<br>To: RMC Team &amp; Supervisors<br>Subject: Cycle Count Complete - Discrepancies Found<br>Include: Summary of Issues"] & n1["View Discrepancy List"]
    DC4 --> DC5["CK3: Display Final Summary<br>━━━━━━━━━━━━━━━━━━━━<br>Completion Rate (Current Cycle): [%]<br>Completion Rate (Total Cycles): [%]<br>✓ Total Pallets Counted: [count]<br>✓ Total Bundles Scanned: [count]<br>⚠ Discrepancies Found: [count]<br>⚠ MTF Bundles: [count]<br>⚠ Scrap list Bundles: [count]<br>Report Generated &amp; Emailed"]
    DC1 -- No Discrepancies --> ND1["System: Generate Summary Report<br>All Items Accounted For"]
    NEXT2["CK3: Display Next Cycle Information<br>━━━━━━━━━━━━━━━━━━━━<br><br>Date of Next Cycle for current customer: [1 year/6 months from now]<br><br>"] --> Restart["Return to System Initialization<br>(Phase 1)"]
    ND1 --> DC5
    DC5 --> NEXT2
    n1 --> n2["CK3: Scan bundle with discrepancy to remove discepancy and mark as completed"]
    Restart --> n3["End"]
    n2 -- Update --> DC5

    style Start fill:#87CEEB,stroke:#000,stroke-width:3px,color:#000
    style DC1 color:#000
    style DC2 fill:#FFE4B5,stroke:#000,stroke-width:2px,color:#000
    style DC3 fill:#FFA500,stroke:#000,stroke-width:2px,color:#000
    style DC4 fill:#FFE4B5,stroke:#000,stroke-width:2px,color:#000
    style n1 fill:#E1F5FE,stroke:#000,stroke-width:2px,color:#000
    style DC5 fill:#BBDEFB,stroke:#000,stroke-width:2px,color:#000
    style ND1 fill:#FFE4B5,stroke:#000,stroke-width:2px,color:#000
    style NEXT2 fill:#E1F5FE,stroke:#000,stroke-width:2px,color:#000
    style Restart fill:#87CEEB,stroke:#000,stroke-width:2px,color:#000
    style n2 color:#000
    style n3 fill:#90EE90,stroke:#000,stroke-width:3px,color:#000



```

---

## Internal background refresh

```mermaid
flowchart TD
    n1(["Internal background refresh system for pallet picking"]) --> n4["Scheduled (every week/everytime scrap list generated) trigger refresh of list"]
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

## Process Flow Summary

```
Phase 1: Initialization & Scheduling
    ↓
Phase 2: Primary Count Execution
    ↓
Phase 3: Bundle Scanning
    ↓
Phase 4: Pallet Completion Verification
    ↓ (loops back to Phase 2 if more pallets)
    ↓ (when all pallets counted)
Phase 5: Completion & Coverage Check
    ↓ (if incomplete coverage)
    ↓ loops back to Phase 1
    ↓ (if complete)
    END
```

---

## Color Legend

- 🟢 **Green (#90EE90)**: Success/Completion states
- 🔵 **Blue (#87CEEB)**: Transition/Connection points between phases, Start/End nodes
- 🟡 **Yellow (#FFE4B5)**: System automated actions (scheduling, notifications, save functions, data generation)
- 🟠 **Orange (#FFA500)**: Warnings (discrepancies, MTF alerts, missing items)
- 🔴 **Pink (#FFB6C1)**: Errors/Flags requiring attention (discrepancy logging, error messages)
- 🔴 **Red (#FF5252)**: Critical System Block (requires supervisor override)
- 🟣 **Purple (#E1BEE7)**: MTF-related actions (MTF bundle display and handling)
- 🔷 **Light Blue (#E1F5FE)**: Information display (rescan operations, summary displays)
- 🟦 **Teal/Cyan (#B2DFDB)**: Refresh/Update actions (save progress, update lists, replacements)
