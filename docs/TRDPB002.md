# TRDPB002 COBOL Program Documentation

## 1. High-Level Overview
The `TRDPB002` COBOL program is designed to handle security bookings. It performs the following main functions:
- Debits the seller's security account.
- Credits the buyer's security account.

### Key Inputs
- Security order details such as order quantity, currency, and account numbers.
- SQL database interactions to fetch and update security account balances.

### Key Outputs
- Updated security account balances.
- Exception reports in case of errors.

## 2. Data Structures

### Input Data Formats

| Name                  | Source        | Fields                                                                 |
|-----------------------|---------------|------------------------------------------------------------------------|
| Security Order Details| Input File/DB | TRD-ORDER-QTY (PIC 9(5)), TRD-CURRENCY (PIC X(3)), TRD-FIGI (PIC X(12)), TRD-BUYER-SEC-ACC-NUM (PIC X(10)), TRD-SELLER-SEC-ACC-NUM (PIC X(10)) |

### Output Data Formats

| Name                  | Destination   | Fields                                                                 |
|-----------------------|---------------|------------------------------------------------------------------------|
| Security Account Balance | Output File/DB | POS-BALANCE (PIC 9(10)V99), POS-CURRENCY (PIC X(3)), POS-SAC-NUMBER (PIC X(10)), POS-FIGI (PIC X(12)) |
| Exception Report      | Output File/DB | EXCEPTION-TYPE (PIC X(10)), EXCEPTION-RECORD-LEN (PIC 9(4)), EXCEPTION-DESC (PIC X(100)) |

## 3. Functional Decomposition

### 0000-MAINLINE
#### Description
Main entry point of the program. It initializes the process by setting initial flags and calling the booking procedure.

#### Inputs
- None

#### Outputs
- TRD-STATUS
- CREDIT-BUYER
- DEBIT-SELLER
- WS-CR-DB-SWTICH
- TRD-SAC-BOOKING-DONE
- TRD-SAC-BOOKING-FAILED

#### Logic
1. Set `TRD-SAC-BOOKING-FAILED` to TRUE.
2. Set `DEBIT-SELLER` to TRUE.
3. Perform `1000-SAC-BOOKING`.
4. If `TRD-SAC-BOOKING-DONE` is TRUE:
   - Set `TRD-SAC-BOOKING-FAILED` to TRUE.
   - Set `CREDIT-BUYER` to TRUE.
   - Perform `1000-SAC-BOOKING`.
5. Return control to the caller.

### 1000-SAC-BOOKING
#### Description
Handles the actual booking process by debiting the seller's account and crediting the buyer's account.

#### Inputs
- TRD-ORDER-QTY
- SQLCODE
- DEBIT-SELLER
- WS-CR-DB-SWTICH
- TRD-FIGI
- TRD-BUYER-SEC-ACC-NUM
- TRD-SELLER-SEC-ACC-NUM
- SQL-INIT-DONE
- POS-BALANCE
- TRD-CURRENCY
- SQL-INIT-FLAG

#### Outputs
- RETURN-CODE
- SQLCODE
- TRD-STATUS
- WS-SQLCODE
- DATA-EXCEPTION
- TRD-CURRENCY
- WS-EXCP-TYPE
- TRD-FIGI
- TRD-BUYER-SEC-ACC-NUM
- TRD-SELLER-SEC-ACC-NUM
- POS-BALANCE
- POS-CURRENCY
- POS-SAC-NUMBER
- TRD-SAC-BOOKING-DONE
- POS-FIGI
- TRD-SAC-BOOKING-FAILED
- WS-EXCEPTION

#### Logic
1. If `DEBIT-SELLER` is TRUE:
   - Move `TRD-CURRENCY` to `POS-CURRENCY`.
   - Move `TRD-SELLER-SEC-ACC-NUM` to `POS-SAC-NUMBER`.
   - Move `TRD-FIGI` to `POS-FIGI`.
   - Display relevant details.
2. Else:
   - Move `TRD-CURRENCY` to `POS-CURRENCY`.
   - Move `TRD-BUYER-SEC-ACC-NUM` to `POS-SAC-NUMBER`.
   - Move `TRD-FIGI` to `POS-FIGI`.
   - Display relevant details.
3. Execute SQL SELECT statement to fetch `POS_BALANCE`.
4. If `SQLCODE` is not 0:
   - Set `TRD-SAC-BOOKING-FAILED` to TRUE.
   - Move `SQLCODE` to `WS-SQLCODE`.
   - Set `DATA-EXCEPTION` to TRUE.
   - Move SPACES to `WS-EXCEPTION`.
   - Concatenate error message and store in `WS-EXCEPTION`.
   - Perform `9000-REPORT-EXCEPTION`.
   - Go to `1000-EXIT`.
5. Fetch `POS_BOOKING` into `POS-BALANCE`.
6. If `SQLCODE` is 0:
   - If `DEBIT-SELLER` is TRUE:
     - Compute `POS-BALANCE` by subtracting `TRD-ORDER-QTY`.
   - Else:
     - Compute `POS-BALANCE` by adding `TRD-ORDER-QTY`.
   - Update `TBTRDPOS` with new `POS-BALANCE`.
   - If `SQLCODE` is 0:
     - Set `TRD-SAC-BOOKING-DONE` to TRUE.
   - Else:
     - Set `TRD-SAC-BOOKING-FAILED` to TRUE.
     - Move `SQLCODE` to `WS-SQLCODE`.
     - Set `DATA-EXCEPTION` to TRUE.
     - Move SPACES to `WS-EXCEPTION`