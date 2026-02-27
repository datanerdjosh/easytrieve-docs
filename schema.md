**only** 
The answer must answer
The user wants a markdown document with a specific structure. Must use only info from source data. No extra commentary. Must follow headings exactly. Provide tables with fields.

We need to extract field definitions.

First Input File: ACHIN.

File description: organization FB (fixed block), record length 94, block length 940.

Common fields: REC (pos 1 len 94 type A) and REC-TYP (pos 1 len 1 type N). Also maybe other common fields but they ask list REC and REC-TYP.

Now 1-Record: File Header fields: DEST-1, ORIG-1, DATE-1, TIME-1, ID-1, BANK-1. Also maybe TR-6-1? Not part of header. So list those.

Need heading, mask if given.

From source: DEST-1 pos 5 len 9 type N heading 'RECV BANK' mask '9999-9999-9'.

ORIG-1 pos 15 len 9 type N heading 'ORIGINATOR' mask '9999-9999-9'.

DATE-1 pos 24 len 6 type N heading 'FILE DATE' mask '99-99-99'.

TIME-1 pos 30 len 4 type N heading 'FILE TIME' mask '99:99'.

ID-1 pos 34 len 1 type A heading 'FILE ID'.

BANK-1 pos 41 len 23 type A heading 'RECEIVING BANK'.

Also there is TR-6-1? Not part of header.

Now 5-Record: Batch Header fields: COMPANY-5, CO-ID-5, SEC-5, DESCRIPT-5, EFF-DATE-5, EFF-YR-5, EFF-MM-5, SET-DATE-5. Also maybe TR-6-5? Not needed.

Heading and mask for each: COMPANY-5 heading 'COMPANY' (no mask). CO-ID-5 heading 'CO. ID'. SEC-5 heading 'SEC'. DESCRIPT-5 heading 'ENTRY DESC.'. EFF-DATE-5 heading 'EFF DATE' mask '99-99-99'. EFF-YR-5 no heading? It says . * YY. We'll leave blank. Same for EFF-MM-5. SET-DATE-5 no heading/mask.

Now 6-Record: Detail Entry fields: list all -6 suffix fields: TC-6, TR-6, ACCT-6, AMT-6, ID-6, NAME-6, DESCR-6, ADDEN-6, TRACE-NO-6. Also maybe TR-6-6 etc but they are internal; but spec says list all -6 suffix fields. The suffix -6 includes many fields: TC-6, TR-6, TR-6H, TR-6-1 etc. However the layout table for 6-Record lists only certain fields: REC-TYP, TC-6, TR-6, TR-6-6, ACCT-6, AMT-6, ID-6, NAME-6, DESCR-6, ADDEN-6, TRACE-NO-6. Also there are fields like TR-6-1 etc but not listed in layout. The instruction: "List all -6 suffix fields". Could interpret as all fields ending with -6 defined in file definition. In file definition earlier we have many: TC-6, TR-6, TR-6H, TR-6-1, TR-6-2, TR-6-3, TR-6-4, TR-6-5, TR-6-6, TR-6-7, TR-6-8, TR-6-9, TR-6-9A, ACCT-6, ACCT-6-IAT, AMT-6, ID-6, NAME-6, DESCR-6, ADDEN-6, TRACE-NO-6. Should we include all? Probably yes.

But the table columns: Position, Length, Type, Heading, Mask, Description. For those without heading/mask we can leave blank.

We need positions and lengths from source definitions.

Let's capture each:

From file definition lines:

TC-6          2  2  N   HEADING ('TRAN', 'CODE')
TR-6          4  9  N   HEADING 'TRANSIT #'
TR-6H         4  8  N                     .  * USED FOR HASH CONTROL
TR-6-1        4  1  N
TR-6-2        5  1  N
TR-6-3        6  1  N
TR-6-4        7  1  N
TR-6-5        8  1  N
TR-6-6        9  1  N
TR-6-7       10  1  N
TR-6-8       11  1  N
TR-6-9       12  1  N
TR-6-9A      12  1  A
ACCT-6       13 17  A   HEADING 'ACCOUNT'
ACCT-6-IAT   40 35  A
AMT-6        30 10  N 2 HEADING 'AMOUNT'
ID-6         40 15  A   HEADING 'CUST ID'
NAME-6       55 22  A   HEADING 'CUST NAME'
DESCR-6      77  2  A   HEADING 'DESCRPT'
ADDEN-6      79  1  N   HEADING 'ADDENDA'
TRACE-NO-6   80 15  A   HEADING 'TRACE #'

Note overlapping positions: ID-6 starts at 40 length 15, overlapping with ACCT-6-IAT (40-74). That's okay.

Now 7-Record: Addenda fields: TYPE-7, DATA-7, ADSEQ-7, DETSEQ-7. Also maybe 710-REC-NAME, 711-ORIG-NAME but they are not suffixed -7; they are separate fields but not required. We'll list those four.

Positions:

TYPE-7          2  2  A   HEADING 'ADDENDA TYPE'
DATA-7          3 80  A   HEADING 'FREE FORM'
ADSEQ-7        84  4  A   HEADING 'ADDENDA SEQ'
DETSEQ-7       88  7  A   HEADING 'DETAIL SEQ'

Now 8-Record: Batch Trailer fields: DEBIT-8, CREDIT-8, BATCH-CT-8, HASH-8. Positions from file definition:

DEBIT-8      21 12  N 2 HEADING 'BTCH TRLR DEBITS'
CREDIT-8     33 12  N 2 HEADING 'BTCH TRLR CREDITS'
BATCH-CT-8    5  6  N   HEADING 'BTCH ENTRY/ADDENDA COUNT'
HASH-8       11 10  N   HEADING 'BTCH HASH TOTAL'

Now 9-Record: File Trailer fields: FILL-9, DEBIT-9, CREDIT-9, FILE-CT-9, HASH-9.

Positions:

FILL-9        2 12  A
DEBIT-9      32 12  N 2 HEADING 'FILE TRLR DEBITS'
CREDIT-9     44 12  N 2 HEADING 'FILE TRLR CREDITS'
FILE-CT-9    14  8  N   HEADING 'FILE ENTRY/ADDENDA COUNT'
HASH-9       22 10  N   HEADING 'FILE HASH TOTAL'

Now Input File: CARD.

Description: FB organization, record length 80, block length 80.

Fields:

C-REPORT-NUMBER pos 1 len 8 type A
C-1 pos 9 len 1 type A
C-REPORT-BANKNBR pos 10 len 6 type A
C-2 pos 16 len 1 type A
C-REPORT-NAME pos 17 len 40 type A
C-3 pos 57 len 1 type A
C-REPORT-MEMBER pos 58 len 10 type A

Now Working Storage Variables.

Accumulator Variables: list W-BATCH-DB, W-BATCH-CR, W-BATCH-CT, W-BATCH-HASH, W-GRAND-DB, W-GRAND-CR, W-GRAND-CT, W-FILE-DB, W-FILE-CR, W-FILE-CT, W-FILE-HASH.

Need length, type, decimals (if P(2) indicates 2 decimal places). Provide purpose.

From source:

W-BATCH-DB len 7 type P(2) heading 'DEBIT TOTAL'
W-BATCH-CR len 7 type P(2) heading 'CREDIT TOTAL'
W-BATCH-CT len 7 type P heading 'ENTRY/ADDENDA COUNT'
W-BATCH-HASH len 10 type N heading 'HASH TOTAL'
W-GRAND-DB len 7 type P(2) heading ' '
W-GRAND-CR len 7 type P(2) heading ' '
W-GRAND-CT len 7 type P heading ' '
W-FILE-DB len 12 type N(2) heading 'FILE DEBIT TOTAL'
W-FILE-CR len 12 type N(2) heading 'FILE CREDIT TOTAL'
W-FILE-CT len 12 type N heading 'FILE ENTRY/ADDENDA COUNT'
W-FILE-HASH len 10 type N heading 'FILE HASH TOTAL'

Now Date/Time Work Fields: W-DATE, W-TIME, W-EFF-DATE, W-SET-DATE and group structures.

From source:

W-DATE len 8 type N heading 'FILE DATE'
W-TIME len 4 type N heading 'FILE TIME'
W-EFF-DATE len 8 type N heading 'EFF DT'
W-SET len 7 type N
W-SET-DATE len 8 type N heading 'SETTL.DT'

Group structures: W-SET-CCYY redefines W-SET len 4 type N etc. Provide them.

We'll list each group variable with length, type, mask (if any), purpose.

Masks: none given for these groups.

Now Message/Error Fields: W-MSG, W-MSGB, W-MSGC, W-MSGD, W-MSGE. Also subfields: W-MSG1, W-MSG2, W-MSG-TR, W-MSG-CKDIG, W-MSGC-PT1, etc.

Provide length, type, purpose.

From source:

W-MSG len 55 type A heading 'MESSAGES'
W-MSGB len 55 type A
W-MSGC len 55 type A
W-MSGD len 55 type A
W-MSGE len 55 type A

Subfields:

W-MSG1 redefines W-MSG len 25 type A
W-MSG2 redefines W-MSG+25 len 30 type A
W-MSG-TR redefines W-MSG+39 len 8 type N
W-MSG-CKDIG redefines W-MSG+47 len 1 type N
W-MSGC-PT1 redefines W-MSGC len 11 type A
W-MSGC-PT2 redefines W-MSGC+11 len 1 type N
W-MSGC-PT3 redefines W-MSGC+12 len 20 type A
W-MSGC-PT4 redefines W-MSGC+32 len 1 type N
W-MSGC-PT5 redefines W-MSGC+33 len 22 type A

Now Control/Flag Variables: W-ABEND, W-FORMAT-ERR, W-ERR, W-LOOP, W-PREV-RECTYP.

Lengths and types:

W-ABEND len 1 type A value 'N'
W-FORMAT-ERR len 1 type A value 'N'
W-ERR len 9 type A heading ' '
W-LOOP len 2 type P value '0'
W-PREV-RECTYP len 1 type N value '0'

Now Check Digit Work Fields: W-CKDIG len 2 type N, W-CKSUM len 4 type N, plus subfields: W-CKDIGLO redefines W-CKDIG+1 len 1 type N, W-CKSUMLO redefines W-CKSUM+3 len 1 type N.

Now Data Type Reference table.

Now produce markdown.

Make sure to follow exact headings and table column names.

Check formatting: Use pipes.

Now craft final answer.

Potential issues: Ensure no extra text beyond requested sections.

Let's write.

assistantfinal# ACH_VALIDATOR — Field Definitions & Schema  

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