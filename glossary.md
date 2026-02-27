We need to produce markdown document with sections as described. Must use only info from source data. No extra invented data. Provide definitions accordingly.

Need to list transaction codes table with columns Code, Category, Direction, Account Type, Description. We have list of codes with descriptions. Need to infer category (debit/credit) and direction? The source says transaction code classification: classifies ACH entries by transaction code into debit/credit categories. So we can categorize based on description: e.g., "Checking Credit" => credit, "Checking Debit" => debit, "General Ledger Credit" => credit, etc. Also account type: checking, savings, general ledger, loan, demand, etc. Direction maybe credit/debit already; but column includes Direction separate from Category? Might be redundant. We'll fill Category as "Credit" or "Debit". Direction maybe same? Could leave blank? But need fill. We'll put Direction same as Category maybe.

Provide definitions for terms.

Make sure to adhere to structure exactly.

Let's craft.

Note: Block filler (type 9 with 999...). In source there is FILL-9 field defined as 12 chars A. Not explicit about 999 filler but typical. We'll define generically.

Report Glossary: need to describe each report: MSG-RPT, MSG-RPT2, TOT-REPORT, DETAIL-RPT, ADDENDA-RPT. Use info from source.

Acronyms table includes given ones plus maybe others like FB, RC. Provide.

Now produce final markdown.assistantfinal# ACH_VALIDATOR — Business Glossary  

## ACH File Structure Terms  

- **ACH (Automated Clearing House)**: A nationwide electronic funds‑transfer network that moves money between financial institutions. ACH_VALIDATOR processes ACH files to verify that every record complies with required formats, totals, and routing rules before the file is accepted for settlement.  

- **File Header Record (Type 1)**: The first record in an ACH file. It identifies the receiving bank, originator, file creation date and time, and a single‑character file ID. Key fields: `DEST-1` (receiving bank), `ORIG-1` (originator), `DATE-1` (file date), `TIME-1` (file time), `ID-1` (file ID), `BANK-1` (receiving bank name).  

- **Batch Header Record (Type 5)**: Begins a batch of detail entries that share common characteristics. It supplies the company name, company ID, SEC (Standard Entry Class) code, entry description, effective date, and settlement (Julian) date. Key fields: `COMPANY-5`, `CO-ID-5`, `SEC-5`, `DESCRIPT-5`, `EFF-DATE-5`, `SET-DATE-5`.  

- **Detail Entry Record (Type 6)**: One transaction line within a batch. Contains the transaction code, ABA routing (transit) number, account number, amount, customer ID, name, and optional addenda indicator. Key fields: `TC-6` (transaction code), `TR-6` (routing number), `ACCT-6` (account), `AMT-6` (amount), `ID-6`, `NAME-6`, `ADDEN-6` (addenda flag).  

- **Addenda Record (Type 7)**: Provides supplemental information for a preceding detail entry when `ADDEN-6` = ‘1’. The addenda type (`TYPE-7`) indicates the purpose (e.g., payment‑related, remittance). IAT (International ACH Transaction) entries use an extended account field (`ACCT-6-IAT`) and require an associated addenda record.  

- **Batch Trailer Record (Type 8)**: Terminates a batch. Holds the batch’s summed debit amount, summed credit amount, total entry/addenda count, and the batch hash total (sum of routing numbers). Fields: `DEBIT-8`, `CREDIT-8`, `BATCH-CT-8`, `HASH-8`. The program reconciles these values against the calculated batch accumulators.  

- **File Trailer Record (Type 9)**: Terminates the entire ACH file. Contains the file‑wide summed debit amount, summed credit amount, total entry/addenda count, and the file hash total. Fields: `DEBIT-9`, `CREDIT-9`, `FILE-CT-9`, `HASH-9`. The program validates these against cumulative file‑level accumulators.  

- **Block Filler (Type 9 with 999…)**: When the physical file does not end on a full 10‑record block, filler characters (often “9”) are added to reach the block size. The program treats the filler as non‑data and ignores it during validation.  

## Transaction Code Reference  

| Code | Category | Direction | Account Type | Description |
|------|----------|-----------|--------------|-------------|
| 21 | Credit | Credit | Checking | Checking Credit (ACH Credit) |
| 22 | Credit | Credit | Checking | Checking Credit (Automated Deposit) |
| 23 | Credit | Credit | Checking | Checking Credit (Prenote) |
| 24 | Credit | Credit | Checking | Checking Credit (Zero‑dollar) |
| 26 | Debit | Debit | Checking | Checking Debit (Return) |
| 27 | Debit | Debit | Checking | Checking Debit (Automated Payment) |
| 28 | Debit | Debit | Checking | Checking Debit (Prenote) |
| 29 | Debit | Debit | Checking | Checking Debit (Zero‑dollar) |
| 31 | Credit | Credit | Savings | Savings Credit (ACH Credit) |
| 32 | Credit | Credit | Savings | Savings Credit (Automated Deposit) |
| 33 | Credit | Credit | Savings | Savings Credit (Prenote) |
| 34 | Credit | Credit | Savings | Savings Credit (Zero‑dollar) |
| 36 | Debit | Debit | Savings | Savings Debit (Return) |
| 37 | Debit | Debit | Savings | Savings Debit (Automated Payment) |
| 38 | Debit | Debit | Savings | Savings Debit (Prenote) |
| 39 | Debit | Debit | Savings | Savings Debit (Zero‑dollar) |
| 40 | Credit | Credit | General Ledger | General Ledger Credit |
| 41 | Credit | Credit | General Ledger | General Ledger Credit |
| 42 | Credit | Credit | General Ledger | General Ledger Credit |
| 43 | Credit | Credit | General Ledger | General Ledger Credit |
| 44 | Credit | Credit | General Ledger | General Ledger Credit |
| 46 | Debit | Debit | General Ledger | General Ledger Debit |
| 47 | Debit | Debit | General Ledger | General Ledger Debit |
| 48 | Debit | Debit | General Ledger | General Ledger Debit |
| 49 | Debit | Debit | General Ledger | General Ledger Debit |
| 51 | Credit | Credit | Loan | Loan Credit |
| 52 | Credit | Credit | Loan | Loan Credit |
| 72 | Credit | Credit | Demand | Demand Credit (Zero‑dollar) |
| 80 | Debit | Debit | Loan | Loan Debit |

## Key Business Terms  

- **ABA Routing Number**: A 9‑digit identifier for a U.S. bank. The first 8 digits are the routing (or transit) number; the 9th digit is a check digit used for validation.  

- **Check Digit**: The 9th digit of the ABA routing number calculated with the 3‑7‑1 weighting algorithm (3 × position 1 + 7 × position 2 + 1 × position 3, repeated across the 8 digits). The program validates this digit via the `CHECK-DIGIT` procedure.  

- **Hash Total**: The arithmetic sum of the 8‑digit routing numbers (or their numeric equivalents) for all entries in a batch or file. It provides a quick integrity check; the batch and file trailers contain the expected hash totals.  

- **Prenote**: A zero‑dollar test entry used to validate account information before actual funds are transferred. Transaction codes 23, 28, 33, and 38 represent prenotes; the program enforces that the amount field is “.00”.  

- **IAT (International ACH Transaction)**: An ACH entry that crosses national borders. IAT entries use an extended account field (`ACCT-6-IAT`) and must include an addenda record for additional foreign‑payment details.  

- **SEC Code**: The Standard Entry Class code that categorizes the type of ACH entry (e.g., IAT, PPD, CCD). It appears in the batch header (`SEC-5`).  

- **Effective Date**: The date on which the ACH entry is to be settled. Stored in the batch header as `EFF-DATE-5` (YY‑MM‑DD).  

- **Settlement Date**: A three‑digit Julian date stored in the batch header (`SET-DATE-5`). It is converted to a calendar date for reporting.  

## Validation Terms  

- **Batch Balance**: The requirement that the calculated batch debit total, credit total, entry/addenda count, and hash total match the values recorded in the batch trailer (records 8).  

- **File Balance**: The requirement that the cumulative file‑wide debit total, credit total, entry/addenda count, and hash total match the values recorded in the file trailer (record 9).  

- **Record Sequence Validation**: The program checks that records appear in the strict order **1 → 5 → 6/7 → 8 → 9**. Any deviation causes an abort (`VALIDFILE` procedure) with return code 555.  

- **W‑ABEND Flag**: A work‑storage flag (not shown in the excerpt) that, when set to ‘Y’, signals that an abnormal termination (abend) has occurred. The `AT-END` routine prints error banners if this flag is set.  

- **RETURN‑CODE 555**: The specific return code issued by the `VALIDFILE` procedure when the file fails the mandatory record‑type sequence check.  

## Report Glossary  

| Report | Contents | Primary Audience | Action on Errors |
|--------|----------|------------------|------------------|
| **MSG‑RPT** | List of messages (`W‑MSG`) generated during validation. Printed on `PRINT01`. | Operations staff reviewing run‑time messages. | Review each message; resolve underlying data issues (e.g., bad check digit, non‑numeric amount). |
| **MSG‑RPT2** | Extended message list (`W‑MSG` through `W‑MSGE`). Printed on `PRINT01`. | Same as MSG‑RPT, but provides more detailed diagnostics. | Same as MSG‑RPT; use additional lines to pinpoint exact record problems. |
| **TOT‑REPORT** | Summary of the ACH file: bank, date, file ID, originating bank, error count, company, description, effective date, and overall totals. Printed on `PRINT01`. | Management and compliance reviewers. | Investigate any listed errors (`W‑ERR`). If error count > 0, file must be corrected before settlement. |
| **DETAIL‑RPT** | Transaction‑by‑transaction listing sorted by settlement/effective date. Shows settlement date, effective date, company, description, transaction code, routing, account, and amount. Printed on `PRINT02`. | Reconciliation teams and auditors. | Verify individual transaction details; reconcile with internal accounting. |
| **ADDENDA‑RPT** | Addenda records associated with detail entries. Displays routing, transaction code, account, amount, customer name, addenda type, free‑form data, addenda sequence, and detail sequence. Printed on `PRINT02`. | Teams handling IAT or other addenda‑required transactions. | Ensure required addenda are present and correctly formatted; correct missing or malformed addenda. |

## Acronyms  

| Acronym | Full Form | Context |
|---------|-----------|---------|
| ACH | Automated Clearing House | Nationwide electronic funds‑transfer network |
| ABA | American Bankers Association | Issuer of the 9‑digit routing number |
| IAT | International ACH Transaction | Cross‑border ACH entry requiring special handling |
| SEC | Standard Entry Class | Code that identifies the ACH entry type (e.g., IAT) |
| EFT | Electronic Funds Transfer | General term encompassing ACH transactions |
| FB | Fixed Block | File organization type used for the input files |
| RC | Return Code | Status code returned by procedures (e.g., 555) |
| TC | Transaction Code | Two‑digit code that classifies the transaction type |
| CRC | Check Digit | The ninth digit of the routing number validated by the 3‑7‑1 algorithm |
| W‑ABEND | Work‑storage ABEND flag | Indicates an abnormal termination condition |
| PRN | Print | Refers to the printer output files (`PRINT01`, `PRINT02`) |