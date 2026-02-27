
## Input File: ACHIN (ACH Input File)  
Fixed‑block (FB) organization, record length **94**, blocking factor **940**.  

### Common Fields (All Record Types)  
| Field Name | Position | Length | Type | Description |
|------------|----------|--------|------|-------------|
| REC        | 1        | 94     | A    | Entire raw record |
| REC‑TYP    | 1        | 1      | N    | Record type indicator |

### 1-Record: File Header Fields  
| Field Name | Position | Length | Type | Heading | Mask | Description |
|------------|----------|--------|------|---------|------|-------------|
| DEST‑1     | 5        | 9      | N    | RECV BANK | 9999-9999-9 | Receiving bank routing number |
| ORIG‑1     | 15       | 9      | N    | ORIGINATOR | 9999-9999-9 | Originator routing number |
| DATE‑1     | 24       | 6      | N    | FILE DATE | 99-99-99 | File creation date (YYMMDD) |
| TIME‑1     | 30       | 4      | N    | FILE TIME | 99:99 | File creation time (HHMM) |
| ID‑1       | 34       | 1      | A    | FILE ID |  | File identification character |
| BANK‑1     | 41       | 23     | A    | RECEIVING BANK |  | Receiving bank name / address |

### 5-Record: Batch Header Fields  
| Field Name | Position | Length | Type | Heading | Mask | Description |
|------------|----------|--------|------|---------|------|-------------|
| COMPANY‑5  | 5        | 16     | A    | COMPANY |  | Company name |
| CO‑ID‑5    | 41       | 10     | A    | CO. ID |  | Company identifier |
| SEC‑5      | 51       | 3      | A    | SEC |  | Standard entry class code |
| DESCRIPT‑5 | 54       | 10     | A    | ENTRY DESC. |  | Description of entry |
| EFF‑DATE‑5| 70       | 6      | N    | EFF DATE | 99-99-99 | Effective date (YYMMDD) |
| EFF‑YR‑5   | 70       | 2      | N    |  |  | Year portion of effective date |
| EFF‑MM‑5   | 72       | 2      | N    |  |  | Month portion of effective date |
| SET‑DATE‑5| 76       | 3      | N    |  |  | Settlement date (Julian) |

### 6-Record: Detail Entry Fields  
| Field Name | Position | Length | Type | Heading | Mask | Description |
|------------|----------|--------|------|---------|------|-------------|
| TC‑6       | 2        | 2      | N    | (TRAN, CODE) |  | Transaction code |
| TR‑6       | 4        | 9      | N    | TRANSIT # |  | Transit routing number |
| TR‑6H      | 4        | 8      | N    |  |  | Used for hash control |
| TR‑6‑1     | 4        | 1      | N    |  |  |  |
| TR‑6‑2     | 5        | 1      | N    |  |  |  |
| TR‑6‑3     | 6        | 1      | N    |  |  |  |
| TR‑6‑4     | 7        | 1      | N    |  |  |  |
| TR‑6‑5     | 8        | 1      | N    |  |  |  |
| TR‑6‑6     | 9        | 1      | N    |  |  |  |
| TR‑6‑7     | 10       | 1      | N    |  |  |  |
| TR‑6‑8     | 11       | 1      | N    |  |  |  |
| TR‑6‑9     | 12       | 1      | N    |  |  |  |
| TR‑6‑9A    | 12       | 1      | A    |  |  |  |
| ACCT‑6     | 13       | 17     | A    | ACCOUNT |  | Account number |
| ACCT‑6‑IAT | 40       | 35     | A    |  |  | IAT account extension |
| AMT‑6      | 30       | 10     | N(2) | AMOUNT |  | Transaction amount (2 decimal places) |
| ID‑6       | 40       | 15     | A    | CUST ID |  | Customer identifier |
| NAME‑6     | 55       | 22     | A    | CUST NAME |  | Customer name |
| DESCR‑6    | 77       | 2      | A    | DESCRPT |  | Entry description code |
| ADDEN‑6    | 79       | 1      | N    | ADDENDA |  | Addenda indicator |
| TRACE‑NO‑6 | 80       | 15     | A    | TRACE # |  | Trace number |

### 7-Record: Addenda Fields  
| Field Name | Position | Length | Type | Heading | Description |
|------------|----------|--------|------|---------|-------------|
| TYPE‑7     | 2        | 2      | A    | ADDENDA TYPE | Addenda record type |
| DATA‑7     | 3        | 80     | A    | FREE FORM | Free‑form addenda data |
| ADSEQ‑7    | 84       | 4      | A    | ADDENDA SEQ | Addenda sequence number |
| DETSEQ‑7   | 88       | 7      | A    | DETAIL SEQ | Associated detail record sequence |

### 8-Record: Batch Trailer Fields  
| Field Name | Position | Length | Type | Heading | Description |
|------------|----------|--------|------|---------|-------------|
| DEBIT‑8    | 21       | 12     | N(2) | BTCH TRLR DEBITS | Total debits for the batch |
| CREDIT‑8   | 33       | 12     | N(2) | BTCH TRLR CREDITS | Total credits for the batch |
| BATCH‑CT‑8 | 5        | 6      | N    | BTCH ENTRY/ADDENDA COUNT | Number of entries/addenda in batch |
| HASH‑8     | 11       | 10     | N    | BTCH HASH TOTAL | Hash total for the batch |

### 9-Record: File Trailer Fields  
| Field Name | Position | Length | Type | Heading | Description |
|------------|----------|--------|------|---------|-------------|
| FILL‑9     | 2        | 12     | A    |  | Fill / padding |
| DEBIT‑9    | 32       | 12     | N(2) | FILE TRLR DEBITS | Total debits for the file |
| CREDIT‑9   | 44       | 12     | N(2) | FILE TRLR CREDITS | Total credits for the file |
| FILE‑CT‑9  | 14       | 8      | N    | FILE ENTRY/ADDENDA COUNT | Total entries/addenda in file |
| HASH‑9     | 22       | 10     | N    | FILE HASH TOTAL | Hash total for the file |

## Input File: CARD (Report Parameter Card)  
Fixed‑block (FB) organization, record length **80**, block length **80**.  

| Field Name       | Position | Length | Type | Description |
|------------------|----------|--------|------|-------------|
| C‑REPORT‑NUMBER | 1        | 8      | A    | Report number |
| C‑1              | 9        | 1      | A    |  |
| C‑REPORT‑BANKNBR| 10       | 6      | A    | Reporting bank number |
| C‑2              | 16       | 1      | A    |  |
| C‑REPORT‑NAME   | 17       | 40     | A    | Report name |
| C‑3              | 57       | 1      | A    |  |
| C‑REPORT‑MEMBER | 58       | 10     | A    | Report member identifier |

## Working Storage Variables  

### Accumulator Variables  
| Variable      | Length | Type | Decimals | Purpose |
|---------------|--------|------|----------|---------|
| W‑BATCH‑DB    | 7      | P    | 2        | Batch debit total accumulator |
| W‑BATCH‑CR    | 7      | P    | 2        | Batch credit total accumulator |
| W‑BATCH‑CT    | 7      | P    | 0        | Batch entry/addenda count accumulator |
| W‑BATCH‑HASH  | 10     | N    | 0        | Batch hash total accumulator |
| W‑GRAND‑DB    | 7      | P    | 2        | Grand (file) debit total accumulator |
| W‑GRAND‑CR    | 7      | P    | 2        | Grand (file) credit total accumulator |
| W‑GRAND‑CT    | 7      | P    | 0        | Grand entry/addenda count accumulator |
| W‑FILE‑DB     | 12     | N    | 2        | File debit total accumulator |
| W‑FILE‑CR     | 12     | N    | 2        | File credit total accumulator |
| W‑FILE‑CT     | 12     | N    | 0        | File entry/addenda count accumulator |
| W‑FILE‑HASH   | 10     | N    | 0        | File hash
