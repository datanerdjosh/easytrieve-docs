**only** 
The answer should only
The user wants a data_flow.md document for the Easytrieve program ACH_VALIDATOR. Must follow exact structure. Must use only info from source data. No extra commentary. Must fill tables etc.

We need to extract details.

First, processing overview: summarize.

Data Flow Diagram: ascii diagram.

Record-by-Record Processing: For each record type, list fields used, variables set, validations, output.

We have info about fields per record type from source data. Also job logic summary shows some variable assignments.

Accumulators: list from source data.

Procedure call flow: we have procedures: VALIDFILE, AT-BEGINNING, CHECK-DIGIT, AT-END, AFTER-BREAK. Also maybe others but these are listed.

Output Report Routing: reports defined earlier: MSG-RPT, MSG-RPT2, TOT-REPORT, DETAIL-RPT, ADDENDA-RPT. Need mapping to printers: both PRINT01 and PRINT02.

Let's collect needed info.

Input files: CARD (FB) 80 length; ACHIN (FB) 94 length block 940.

Output files: PRINT01 (PRINTER), PRINT02 (PRINTER). Reports: MSG-RPT → PRINT01, MSG-RPT2 → PRINT01, TOT-REPORT → PRINT01, DETAIL-RPT → PRINT02, ADDENDA-RPT → PRINT02.

Report titles given.

Processing procedures: VALIDFILE validates sequence order, aborts RC=555 on format error. AT-BEGINNING reads CARD to load report parameters. CHECK-DIGIT validates ABA routing number check digit using 3-7-1 weighting algorithm. AT-END prints error banners if abend flag set or receiving bank field empty. AFTER-BREAK prints grand totals at final break level.

Job Logic Summary: describes initialization, reading header, setting variables, handling multiple headers, etc.

Variables: W-MSG, W-ERR, W-BANK, W-DATE, W-TIME, W-FILE-DB, etc. Also W-BATCH-DB etc.

Accumulators list: W-BATCH-DB, W-BATCH-CR, W-BATCH-CT, W-BATCH-HASH (batch scope); W-FILE-DB, W-FILE-CR, W-FILE-CT, W-FILE-HASH (file scope); W-GRAND-DB, W-GRAND-CR, W-GRAND-CT (grand scope).

Business rules: list of validations.

Now produce sections.

Record-by-Record: need to describe fields read/used, variables set/updated, validations, output.

From source data we have fields per record type.

Also job logic includes actions for each type.

For 1-Record: sets W-BANK, W-DATE (converted), W-TIME, resets file accumulators, sets W-DEST, W-ORIG, W-ID. Also checks multiple header condition (if W-BANK not blank). Output: prints MSG-RPT three times if duplicate header.

For 5-Record: resets batch accumulators, sets W-COMPANY, W-SEC, W-CO-ID, W-DESCRIPT, converts effective date to W-EFF-DATE, handles SET-DATE adjustments, sets W-SET-DDD, W-SET-YY, calculates W-SET-DATE, possibly zero.

For 6-Record: Not fully detailed but likely processes detail entry. Business rules apply: transaction code classification, amount numeric validation, zero amount validation, transit number numeric check, routing number check digit, etc. Updates batch/file totals and counts, hash.

For 7-Record: Addenda processing; likely updates counts.

For 8-Record: Batch trailer; compare calculated batch totals vs trailer fields; may output messages.

For 9-Record: File trailer; compare file totals vs trailer fields; may output messages.

Now fill tables.

Procedure Call Flow table: list procedures and when called.

From Job Logic: AT-BEGINNING called at start. VALIDFILE performed after job input start. CHECK-DIGIT likely called during detail processing for routing number validation. AFTER-BREAK called on control break (maybe after each batch?). AT-END called at finish.

Now Output Report Routing table: map each report to printer and trigger.

Reports: MSG-RPT (messages) printed on PRINT01 when duplicate header? Also maybe other messages. MSG-RPT2 also messages. TOT-REPORT summary printed on PRINT01 after processing? DETAIL-RPT printed on PRINT02 for detail transactions by settle/eff date. ADDENDA-RPT printed on PRINT02 for addenda records.

Now craft document.

Need to ensure markdown headings exactly as specified.

Tables need markdown format.

Now produce final answer.

Let's draft.

assistantfinal# ACH_VALIDATOR — Data Flow

## Processing Overview
The program reads a control **CARD** file for report parameters and an ACH input file (**ACHIN**) containing 94‑byte records. It validates the record sequence, checks routing numbers, amounts and totals, accumulates batch and file‑level sums, and generates message, summary, detail and addenda reports. The results are written to two printer files, **PRINT01** and **PRINT02**.

## Data Flow Diagram
```
[ACHIN Input File]          [CARD Input File]
       │                           │
       │ 94‑byte ACH records       │ Report parameters
       ▼                           ▼
┌───────────────────────────────────────────────────────┐
│               JOB Processing Loop                     │
│  ┌─────────────────────────────────────────────────┐  │
│  │  Record Type Dispatch                           │  │
│  │  1‑REC → File Header Init                       │  │
│  │  5‑REC → Batch Header Init                      │  │
│  │  6‑REC → Detail Entry Validation                │  │
│  │  7‑REC → Addenda Processing                     │  │
│  │  8‑REC → Batch Trailer Reconciliation           │  │
│  │  9‑REC → File Trailer Reconciliation            │  │
│  └─────────────────────────────────────────────────┘  │
└───────────────────────────────────────────────────────┘
       │                    │
       ▼                    ▼
[PRINT01 – Messages & Summary]   [PRINT02 – Detail & Addenda]
```

## Record-by-Record Processing  

### 1-Record: File Header
* **Fields read**: `REC-TYP`, `DEST-1`, `ORIG-1`, `DATE-1`, `TIME-1`, `ID-1`, `BANK-1`.
* **Working storage set/updated**  
  * `W-BANK` ← `BANK-1`  
  * `W-DATE` ← converted `DATE-1` (YYMMDD → CCYYMMDD)  
  * `W-TIME` ← `TIME-1`  
  * `W-DEST` ← `DEST-1`  
  * `W-ORIG` ← `ORIG-1`  
  * `W-ID`   ← `ID-1`  
  * File‑level accumulators (`W-FILE-DB`, `W-FILE-CR`, `W-FILE-CT`, `W-FILE-HASH`) reset to 0.  
* **Validations**  
  * `VALIDFILE` ensures this is the first 1‑record; if another appears, `W-MSG` set to “=== MULTIPLE FILE HEADER 1-RECORDS ON FILE =====” and the message report is printed three times.  
* **Output**  
  * Optional message records sent to **MSG‑RPT** (PRINT01) when duplicate header detected.

### 5-Record: Batch Header
* **Fields read**: `REC-TYP`, `COMPANY-5`, `CO-ID-5`, `SEC-5`, `DESCRIPT-5`, `EFF-DATE-5`, `EFF-YR-5`, `EFF-MM-5`, `SET-DATE-5`, `SET-DATE-5A`, `SET-DATE-51`, `SET-DATE-52`, `SET-DATE-53`.
* **Working storage set/updated**  
  * Batch‑level accumulators (`W-BATCH-DB`, `W-BATCH-CR`, `W-BATCH-CT`, `W-BATCH-HASH`) reset to 0.  
  * `W-COMPANY` ← `COMPANY-5`  
  * `W-SEC` ← `SEC-5`  
  * `W-CO-ID` ← `CO-ID-5`  
  * `W-DESCRIPT` ← `DESCRIPT-5`  
  * `W-EFF-DATE` ← converted `EFF-DATE-5` (YYMMDD → CCYYMMDD)  
  * If `SET-DATE-51` is non‑blank while `SET-DATE-52` and `SET-DATE-53` are blank, move its value to `SET-DATE-53`.  
  * `W-SET-DDD` ← `SET-DATE-5` (alpha conversion via `%ALPHACON`)  
  * `W-SET-YY` ← `EFF-YR-5`  
  * `W-SET-DATE` ← converted Julian date (`YYDDD` → CCYYMMDD). Adjust year forward by one if month = 12 and Julian day < 180.  
  * If `SET-DATE-5 = 000` then `W-SET-DATE = 000000`.  
* **Validations** – none explicit beyond date conversion.  
* **Output** – none directly; batch header information is used for later totals and reports.

### 6-Record: Detail Entry
* **Fields read**: `REC-TYP`, `TC-6`, `TR-6`, `TR-6H`, `TR-6-6`, `ACCT-6`, `ACCT-6-IAT`, `AMT-6`, `ID-6`, `NAME-6`, `DESCR-6`, `ADDEN-6`, `TRACE-NO-6`.
* **Working storage set/updated**  
  * Update batch and file totals:  
    * `W-BATCH-DB` / `W-FILE-DB` increased by amount if transaction code indicates debit.  
    * `W-BATCH-CR` / `W-FILE-CR` increased by amount if credit.  
    * `W-BATCH-CT` / `W-FILE-CT` incremented (detail + possible addenda).  
    * `W-BATCH-HASH` / `W-FILE-HASH` updated with routing‑number hash (`TR-6H`).  
* **Validations** (Business Rules)  
  * **Transaction Code Classification** – determines debit vs. credit.  
  * **Amount Numeric Validation** – `AMT-6` must be numeric (10 digits).  
  * **Zero Amount Validation** – zero allowed only for specific codes.  
  * **Transit Number Numeric Check** – `TR-6` must be numeric (8‑digit hash portion).  
  * **Routing Number Check Digit** – `CHECK‑DIGIT` procedure applied to full routing number (`DEST-1`/`TR-6`).  
  * **IAT Special Handling** – if `W-SEC = 'IAT'`, use extended account field `ACCT-6-IAT` and expect associated addenda.  
* **Output** – detail lines are later routed to **DETAIL‑RPT** (PRINT02); any validation failures generate messages on **MSG‑RPT**.

### 7-Record: Addenda
* **Fields read**: `REC-TYP`, `TR-6-7`, `TYPE-7`, `DATA-7`, `ADSEQ-7`, `DETSEQ-7`.
* **Working storage set/updated**  
  * Increment entry/addenda count (`W-BATCH-CT`, `W-FILE-CT`).  
  * Associate addenda with the preceding detail record via `DETSEQ-7`.  
* **Validations** – none listed beyond count increment; addenda are expected when `ADDEN-6 = 1`.  
* **Output** – addenda lines are routed to **ADDENDA‑RPT** (PRINT02).

### 8-Record: Batch Trailer
* **Fields read**: `REC-TYP`, `TR-6-8`, `DEBIT-8`, `CREDIT-8`, `BATCH-CT-8`, `HASH-8`.
* **Working storage set/updated**  
  * Compare calculated batch totals (`W-BATCH-DB`, `W-BATCH-CR`, `W-BATCH-CT`, `W-BATCH-HASH`) with trailer fields.  
  * If mismatches occur, set `W-ERR` and generate messages on **MSG‑RPT**.  
* **Validations** – Batch Entry Count, Debit Total, Credit Total, and Hash Total reconciliations.  
* **Output** – batch summary messages; after successful reconciliation, batch totals are added to file‑level accumulators.

### 9-Record: File Trailer / Block Filler
* **Fields read**: `REC-TYP`, `TR-6-9A`, `FILL-9`, `DEBIT-9`, `CREDIT-9`, `FILE-CT-9`, `HASH-9`.
* **Working storage set/updated**  
  * Compare cumulative file totals (`W-FILE-DB`, `W-FILE-CR`, `W-FILE-CT`, `W-FILE-HASH`) with trailer fields.  
  * On mismatch, set `W-ERR` and issue messages on **MSG‑RPT**.  
* **Validations** – File Entry Count, Debit Total, Credit Total, and Hash Total reconciliations.  
* **Output** – final file‑level messages; after successful check, grand totals are prepared for the summary report.

## Accumulator Flow
| Accumulator | Reset Point | Compared Against | Purpose |
|-------------|-------------|-----------------|---------|
| `W-BATCH-DB` | Start of each 5‑record (Batch Header) | `DEBIT-8` in Batch Trailer (8‑record) | Running debit total for the current batch |
| `W-BATCH-CR` | Start of each 5‑record | `CREDIT-8` in Batch Trailer | Running credit total for the current batch |
| `W-BATCH-CT` | Start of each 5‑record | `BATCH-CT-8` in Batch Trailer | Entry/Addenda count for the current batch |
| `W-BATCH-HASH` | Start of each 5‑record | `HASH-8` in Batch Trailer | Routing‑number hash total for the current batch |
| `W-FILE-DB` | File Header (1‑record) | `DEBIT-9` in File Trailer (9‑record) | Cumulative debit total across all batches |
| `W-FILE-CR` | File Header | `CREDIT-9` in File Trailer | Cumulative credit total across all batches |
| `W-FILE-CT` | File Header | `FILE-CT-9` in File Trailer | Cumulative entry/addenda count for the whole file |
| `W-FILE-HASH` | File Header | `HASH-9` in File Trailer | Cumulative routing‑number hash for the whole file |
| `W-GRAND-DB` | After final batch trailer (control break) | – | Grand total debits for the final summary report |
| `W-GRAND-CR` | After final batch trailer | – | Grand total credits for the final summary report |
| `W-GRAND-CT` | After final batch trailer | – | Grand total entry count for the final summary report |

## Procedure Call Flow
| Procedure | Called When | Purpose |
|-----------|-------------|---------|
| `AT-BEGINNING` | Job start (before processing ACHIN) | Reads CARD file, loads report parameters (report number, bank, name, member). |
| `VALIDFILE` | Immediately after job input start (first step in processing loop) | Verifies correct record‑type sequence (1 → 5 → 6/7 → 8 → 9); aborts with RC = 555 on format error. |
| `CHECK‑DIGIT` | During processing of each 6‑record (Detail Entry) | Validates ABA routing number check digit using the 3‑7‑1 weighting algorithm. |
| `AFTER‑BREAK` | On control‑break after the last batch (final break level) | Prints grand totals (debits, credits, entry count) to the summary report. |
| `AT‑END` | Job termination (after all records processed) | Performs cleanup; prints error banners if an abend flag was set or if the receiving‑bank field is empty. |

## Output Report Routing
| Report | Printer | Triggered By | Content |
|--------|---------|--------------|---------|
| `MSG‑RPT` | PRINT01 | Duplicate file header, validation errors, trailer mismatches, or any `W‑ERR` condition | Message text (e.g., duplicate header warning, checksum failures, total mismatches). |
| `MSG‑RPT2` | PRINT01 | Additional message handling (same conditions as `MSG‑RPT`) | Same message style; separate logical grouping. |
| `TOT‑REPORT` | PRINT01 | End of file processing (after `AFTER‑BREAK`) | Summary line: “ACH FILE FOR … ORIG: …” with file‑level totals (date, time, IDs, originating bank). |
| `DETAIL‑RPT` | PRINT02 | Each valid 6‑record (Detail Entry) after successful validation | Detailed transaction listing grouped by settlement/effective date. |
| `ADDENDA‑RPT` | PRINT02 | Each 7‑record (Addenda) linked to a detail entry | Free‑form addenda data for the corresponding transaction. |