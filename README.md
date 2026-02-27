# ACH_VALIDATOR

## Overview  

`ACH_VALIDATOR` is an Easytrieve‑Plus batch program that reads an Automated Clearing House (ACH) payment file (`ACHIN`) together with a small parameter card (`CARD`). It validates the structural integrity of the ACH file, checks the financial calculations (debits, credits, entry counts, routing‑number hash totals), and verifies key field formats such as ABA routing numbers and transaction codes. All findings are printed to two printer destinations – one for error and summary messages and another for detailed transaction listings.

---

## Business Purpose  

Financial institutions must ensure that every ACH file submitted to the clearing house conforms to NACHA specifications. `ACH_VALIDATOR` provides that safeguard by catching format violations, incorrect routing‑number check digits, mismatched batch totals, and illegal transaction‑code/amount combinations before the file is transmitted. Operations staff and audit teams use the generated reports to confirm that the file will settle without rejection, thereby avoiding costly re‑submissions, settlement delays, and regulatory penalties. Typical error conditions detected include:

* Bad ABA routing‑number check digit  
* Batch hash total mismatch  
* Multiple file‑header records in a single file  
* Prenote entries with a non‑zero amount  
* Invalid transaction codes for the given amount  

---

## Input Files  

### CARD – Report Parameter Card  

| Attribute | Value |
|-----------|-------|
| **Organization** | FB (fixed‑block) |
| **Record Length** | 80 |
| **Block Length** | 80 |
| **Purpose** | Supplies report identification data that are read once at job start (procedure `AT‑BEGINNING`). |

#### Fields  

| Field | Position | Length | Type | Description |
|-------|----------|--------|------|-------------|
| C-REPORT‑NUMBER | 1 | 8 | A | Report number identifier |
| C‑1 | 9 | 1 | A | Unused placeholder |
| C‑REPORT‑BANKNBR | 10 | 6 | A | Bank number displayed in report headers |
| C‑2 | 16 | 1 | A | Unused placeholder |
| C‑REPORT‑NAME | 17 | 40 | A | Full report name (company / processing center) |
| C‑3 | 57 | 1 | A | Unused placeholder |
| C‑REPORT‑MEMBER | 58 | 10 | A | Member identifier shown on the summary report |

---

### ACHIN – ACH Input File  

| Attribute | Value |
|-----------|-------|
| **Organization** | FB (fixed‑block) |
| **Record Length** | 94 |
| **Block Length** | 940 (10 records per block) |
| **Purpose** | The ACH payment file to be validated. It contains all required record types (File Header, Batch Header, Detail, Addenda, Batch Trailer, File Trailer). |
| **Record‑type indicator** | `REC‑TYP` – position 1, length 1, numeric. Determines which logical record layout follows. |

#### Six ACH Record Types  

| Rec‑Typ | Record Name | Key Fields | Role |
|---------|-------------|------------|------|
| **1** | File Header | `DEST‑1`, `ORIG‑1`, `DATE‑1`, `TIME‑1`, `ID‑1`, `BANK‑1` | Begins the file; identifies the receiving and originating banks, file creation date/time, and a one‑character file ID. |
| **5** | Batch Header | `COMPANY‑5`, `CO‑ID‑5`, `SEC‑5`, `DESCRIPT‑5`, `EFF‑DATE‑5`, `SET‑DATE‑5` | Starts a new batch of transactions; carries company name, SEC code, effective date, and settlement date. |
| **6** | Detail Entry | `TC‑6`, `TR‑6`, `ACCT‑6`, `AMT‑6`, `ID‑6`, `NAME‑6`, `DESCR‑6`, `ADDEN‑6`, `TRACE‑NO‑6` | Represents a single ACH transaction (debit or credit). Includes transaction code, routing transit number, account, amount, and optional addenda flag. |
| **7** | Addenda Record | `TYPE‑7`, `DATA‑7`, `ADSEQ‑7`, `DETSEQ‑7` | Provides supplemental information for a preceding Detail entry (e.g., payment‑related remarks). |
| **8** | Batch Trailer | `DEBIT‑8`, `CREDIT‑8`, `BATCH‑CT‑8`, `HASH‑8` | Closes a batch; contains the calculated debit total, credit total, entry/addenda count, and routing‑number hash for the batch. |
| **9** | File Trailer | `DEBIT‑9`, `CREDIT‑9`, `FILE‑CT‑9`, `HASH‑9` | Terminates the file; holds cumulative totals for the entire file (debits, credits, entry count, hash). |

---

## Output Files / Reports  

All output is directed to two printer destinations.

### PRINT01 – Messages & Summary  

| Report | Title (Line 2) | Content |
|--------|----------------|---------|
| **MSG‑RPT** | *none* | One‑line error messages generated immediately when a validation fails (e.g., “*===NON NUMERIC CHECK DIGIT‑SHOULD BE*”). |
| **MSG‑RPT2** | *none* | Five‑line format‑error report printed only when the record‑sequence validator (`VALIDFILE`) detects a fatal ordering problem. Triggers an abnormal termination with RC = 555. |
| **TOT‑REPORT** | `W‑37SPACES 'MESSAGES'` (for MSG‑RPT) and `W‑37SPACES 'MESSAGES'` (for MSG‑RPT2) – also used for batch/file totals | Printed at each Batch Trailer (record 8) and at the File Trailer (record 9). Shows company, SEC code, effective date, batch/file debit total, credit total, and entry count. Also displays an error banner when any soft error has been recorded (`W‑ABEND='Y'`). |

### PRINT02 – Detail & Addenda Listings  

| Report | Title (Line 2) | Content |
|--------|----------------|---------|
| **DETAIL‑RPT** | `W‑37SPACES  'DETAIL TRANSACTIONS BY SETTLE/EFF DATE'` | Lists every Detail entry (record 6) sorted by settlement date, effective date, transaction code, company, transit number, account, and amount. IAT entries appear only after a matching Type‑11 Addenda is processed. |
| **ADDENDA‑RPT** | `W‑37SPACES 'ADDENDA RECORDS'` | Prints each Addenda record (record 7) with its type, free‑form data, and sequence numbers, linked back to the associated Detail entry. |

---

## Processing Logic  

1. **Job Start – AT‑BEGINNING**  
   * The program opens `CARD` and reads the single parameter record. Values are moved to working‑storage items (`W‑REPORT‑NUMBER`, `W‑BANK`, `W‑NAME`, `W‑MEMBER`). These values are later inserted into report titles and footers.  

2. **Main Loop – Read ACHIN**  
   * For each physical record the program first calls **VALIDFILE** to verify that the record‑type (`REC‑TYP`) follows the allowed sequence (see *File Format / Sequence Validations*).  
   * If `VALIDFILE` sets the format‑error flag (`W‑FORMAT‑ERR='Y'`), a five‑line error block is printed via `MSG‑RPT2` and the job terminates with return‑code 555.  

3. **Dispatch by Record Type**  

   * **Record 1 – File Header**  
     * Initializes file‑level accumulators (`W‑FILE‑DB`, `W‑FILE‑CR`, `W‑FILE‑CT`, `W‑FILE‑HASH`) to zero.  
     * Stores receiving bank name (`BANK‑1`) for later use in the summary report.  

   * **Record 5 – Batch Header**  
     * Resets batch‑level accumulators (`W‑BATCH‑DB`, `W‑BATCH‑CR`, `W‑BATCH‑CT`, `W‑BATCH‑HASH`).  
     * Converts the effective date (`EFF‑DATE‑5`) from `YY‑MM‑DD` to a full eight‑digit date using `%DATECONV`.  
     * Captures settlement date (`SET‑DATE‑5`) for sorting the detail report.  

   * **Record 6 – Detail Entry**  
     * Performs field‑level checks (numeric amount, numeric transit, routing‑number check digit, transaction‑code validity, zero‑amount rules).  
     * Updates batch and file totals: adds the amount to the appropriate debit/credit accumulator, increments entry count, and adds the 8‑digit transit number (without the check digit) to the hash totals.  
     * If the addenda flag (`ADDEN‑6`) is set, the program expects a following Addenda record (type 7).  

   * **Record 7 – Addenda**  
     * Increments the entry/addenda count (both batch and file).  
     * For International ACH Transactions (IAT) the program stores the extended account field (`ACCT‑6‑IAT`) and suppresses the normal detail line until the corresponding Type‑11 addenda is encountered.  

   * **Record 8 – Batch Trailer**  
     * Calls **AFTER‑BREAK** to print the batch subtotal (`TOT‑REPORT`).  
     * Compares the calculated batch totals (debit, credit, entry count, hash) with the trailer fields (`DEBIT‑8`, `CREDIT‑8`, `BATCH‑CT‑8`, `HASH‑8`). Any mismatch generates the appropriate error message and sets the soft‑abend flag (`W‑ABEND='Y'`).  

   * **Record 9 – File Trailer**  
     * Performs the same reconciliation at the file level (debit, credit, entry count, hash) against `DEBIT‑9`, `CREDIT‑9`, `FILE‑CT‑9`, `HASH‑9`.  
     * Prints the final grand‑total report.  

   * **Other Records** – Filler or unexpected types are ignored after the format validator has already flagged them.  

4. **End‑of‑Job – AT‑END**  
   * If `W‑ABEND='Y'` (any soft error occurred) a banner “=== ERRORS THIS RUN ===” and the accumulated error count are printed on `PRINT01`.  
   * If the receiving‑bank field (`W‑BANK`) is empty, a special warning (“= POSSIBLE ERROR - RECEIVING BANK FLD EMPTY ==”) is issued.  
   * The job then closes all files and returns control to the operating system.  

---

## Validations Performed  

### File Format / Sequence Validations (`VALIDFILE`)  

| Current Record | Allowed Predecessor(s) | Rule |
|----------------|------------------------|------|
| **1** (File Header) | Start of file or after a previous File Trailer (9) | Guarantees a single opening header per file. |
| **5** (Batch Header) | After a File Header (1) or after a previous Batch Trailer (8) | Starts a new batch. |
| **6** (Detail Entry) | After a Batch Header (5) or after another Detail (6) or Addenda (7) | Belongs to the current batch. |
| **7** (Addenda) | Must follow a Detail Entry (6) that has its addenda flag set | Provides supplemental data. |
| **8** (Batch Trailer) | After the last Detail/Addenda of the batch (6/7) | Closes the batch. |
| **9** (File Trailer) | After the final Batch Trailer (8) | Ends the file. |
| **Any other order** | – | Sets `W‑FORMAT‑ERR='Y'`; prints MSG‑RPT2 and aborts with RC = 555. |
| **Multiple File Headers** | More than one record‑type 1 in the same file | Generates “=== MULTIPLE FILE HEADER 1‑RECORDS ON FILE ===”. |

### Batch‑Level Balance Checks (performed at Record 8)  

| Check | Calculated Value | Trailer Field | Action on Mismatch |
|-------|------------------|---------------|--------------------|
| Entry/Addenda Count | `W‑BATCH‑CT` | `BATCH‑CT‑8` | Print “=========== BATCH COUNT ERROR ======================” |
| Debit Total | `W‑BATCH‑DB` | `DEBIT‑8` | Print “=========== BATCH DEBIT ERROR ======================” |
| Credit Total | `W‑BATCH‑CR` | `CREDIT‑8` | Print “=========== BATCH CREDIT ERROR ======================” |
| Routing‑Number Hash | `W‑BATCH‑HASH` | `HASH‑8` | Print “=========== BATCH HASH ERROR   ======================” |

### File‑Level Balance Checks (performed at Record 9)  

| Check | Calculated Value | Trailer Field | Action on Mismatch |
|-------|------------------|---------------|--------------------|
| Entry/Addenda Count | `W‑FILE‑CT` | `FILE‑CT‑9` | Print “=========== FILE COUNT ERROR ======================” |
| Debit Total | `W‑FILE‑DB` | `DEBIT‑9` | Print “=========== FILE DEBIT ERROR ======================” |
| Credit Total | `W‑FILE‑CR` | `CREDIT‑9` | Print “=========== FILE CREDIT ERROR ======================” |
| Routing‑Number Hash | `W‑FILE‑HASH` | `HASH‑9` | Print “=========== FILE HASH ERROR   ======================” |

### Field‑Level Validations (performed at Record 6)  

| Field | Validation | Failure Message |
|-------|------------|-----------------|
| **Amount (`AMT‑6`)** | Must be 10 numeric characters. | “*========= 10‑DIG AMT NOT NUMERIC *****************” |
| **Transit Number (`TR‑6`)** | First 8 digits must be numeric. | “*========= TRANSIT # NOT NUMERIC. CHGD TO 0 *****” |
| **Routing Check Digit** | Apply 3‑7‑1 weighting algorithm to the 9‑digit routing number; compare with supplied digit. | “*===TR HAS BAD CHECK DIGIT. SHOULD BE ***********” |
| **Transaction Code (`TC‑6`)** | Must be a valid ACH code (e.g., 22, 27, 32, 37, etc.). | “========= BAD TRANCODE ON FILE *****************” |
| **Zero‑Amount Rule** | For non‑prenote codes, amount cannot be zero. | “*========= .00 AMT NOT VALID FOR TC ===========*” |
| **Prenote Amount** | If transaction code indicates a prenote, amount must be zero. | “*========= PRENOTE AMOUNT NOT .00 ***************” |

### IAT (International ACH Transaction) Special Handling  

* IAT entries use the extended account field `ACCT‑6‑IAT` (35 characters) instead of the standard 17‑character `ACCT‑6`.  
* The detail line for an IAT is suppressed in `DETAIL‑RPT` until a matching Type‑11 Addenda record is read, at which point the combined information is printed.  
* IAT entries still participate in all batch and file totals (debits/credits, hash, counts).  

---

## Error Handling  

| Flag / Variable | Meaning | Effect |
|-----------------|---------|--------|
| `W‑FORMAT‑ERR` | Set to ‘Y’ when a record‑sequence violation is detected. | Immediate printing of `MSG‑RPT2` (five‑line format error) and job termination with **RETURN‑CODE = 555**. |
| `W‑ABEND` | Set to ‘Y’ when any soft validation fails (field‑level, batch‑level, or file‑level). | Job continues to completion; at `AT‑END` a banner “=== ERRORS THIS RUN ===” and the total error count are printed on `PRINT01`. |
| `W‑ERR` | Holds the literal “===ERR===” when at least one error message has been generated. | Inserted into the `TOT‑REPORT` header to highlight that the batch/file contained errors. |
| `W‑ERR‑COUNT` | Accumulator of all soft errors encountered. | Reported in the final error banner. |

### Complete List of Error Messages  

* `*========= .00 AMT NOT VALID FOR TC ===============*`  
* `*========= TRANSIT # NOT NUMERIC. CHGD TO 0 =======*`  
* `*===NON NUMERIC CHECK DIGIT‑SHOULD BE          ====*`  
* `============ FILE COUNT ERROR =====================`  
* `=== MULTIPLE FILE HEADER 1‑RECORDS ON FILE =====`  
* `============ BATCH HASH ERROR   ==================`  
* `============ BATCH COUNT ERROR ==================`  
* `============ FILE DEBIT ERROR =====================`  
* `========== BAD TRANCODE ON FILE ==================`  
* `*========= PRENOTE AMOUNT NOT .00 =================*`  
* `*===TR HAS BAD CHECK DIGIT. SHOULD BE          ====*`  
* `*========= 10‑DIG AMT NOT NUMERIC *****************`  
* `============ FILE HASH ERROR   ==================`  
* `============ BATCH DEBIT ERROR ==================`  
* `============ ERRORS THIS RUN ==================`  
* `============ BATCH CREDIT ERROR ==================`  
* `!!!!!! FILE FORMAT ERROR - CONTACT PROGRAMMER !!!!!`  
* `= POSSIBLE ERROR - RECEIVING BANK FLD EMPTY ==`  
* `============ FILE CREDIT ERROR ==================`  

When any of these messages are emitted, the corresponding flag (`W‑ABEND` or `W‑FORMAT‑ERR`) is set, and the appropriate banner/report is generated.

---

## Operations  

| Attribute | Value |
|-----------|-------|
| **Language** | Easytrieve Plus |
| **Primary Input** | `ACHIN` – Fixed‑block 94‑byte ACH file |
| **Parameter Input** | `CARD` – Fixed‑block 80‑byte report card |
| **Output Destinations** | `PRINT01` (messages & summary) <br> `PRINT02` (detail & addenda) |
| **Reports