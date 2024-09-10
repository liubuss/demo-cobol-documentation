# TRDPB003 Documentation

## High-Level Overview
The `TRDPB003` COBOL program is designed to handle money booking transactions. The main functionalities include:
- Debiting the seller's money account.
- Crediting the buyer's money account.

## Data Structures

### Input Data Formats

| Name                  | Source        | Fields                                                                 |
|-----------------------|---------------|------------------------------------------------------------------------|
| Transaction Record    | Input File    | `TRD-BUYER-MONEY-ACC-NUM`, `TRD-SELLER-MONEY-ACC-NUM`, `TRD-CURRENCY`, `TRD-ORDER-AMOUNT` |
| SQL Initialization    | Database      | `SQL-INIT-DONE`, `SQLCODE`, `SQL-INIT-FLAG`                            |

### Output Data Formats

| Name                  | Destination   | Fields                                                                 |
|-----------------------|---------------|------------------------------------------------------------------------|
| Exception Report      | Output File   | `WS-EXCEPTION`, `EXCEPTION-DESC`, `EXCEPTION-RECORD-LEN`, `EXCEPTION-TYPE` |
| Account Balance Update| Database      | `MAC-BALANCE`, `MAC-CURRENCY`, `MAC-NUMBER`                            |

## Functional Decomposition

### Main Functions

1. **0000-MAINLINE**
   - **Inputs:** None
   - **Outputs:** `TRD-MAC-BOOKING-FAILED`, `DEBIT-BUYER`, `TRD-MAC-BOOKING-DONE`, `TRD-STATUS`
   - **Description:** 
     - Sets initial flags for booking failure and debit buyer.
     - Calls the `1000-MAC-BOOKING` paragraph.
     - If booking is done, sets the credit seller flag and calls `1000-MAC-BOOKING` again.
     - Ends with a `GOBACK`.

2. **1000-MAC-BOOKING**
   - **Inputs:** `CREDIT-SELLER`, `TRD-BUYER-MONEY-ACC-NUM`, `TRD-SELLER-MONEY-ACC-NUM`, `TRD-CURRENCY`, `TRD-ORDER-AMOUNT`, `SQLCODE`
   - **Outputs:** `WS-EXCEPTION`, `WS-SQLCODE`, `TRD-MAC-BOOKING-DONE`, `TRD-STATUS`, `MAC-BALANCE`, `MAC-CURRENCY`, `MAC-NUMBER`
   - **Description:** 
     - Determines whether to credit seller or debit buyer.
     - Executes SQL statements to fetch and update account balances.
     - Handles SQL exceptions and reports them.

3. **9000-REPORT-EXCEPTION**
   - **Inputs:** `WS-EXCEPTION`, `APPL-EXCEPTION`, `DATA-EXCEPTION`, `SYSTEM-EXCEPTION`, `WS-EXCP-TYPE`
   - **Outputs:** `EXCEPTION-DESC`, `EXCEPTION-RECORD-LEN`, `EXCEPTION-TYPE`
   - **Description:** 
     - Moves exception details to the exception report.
     - Calls the exception handler program.

## Program Logic

The program logic is broken down into the following steps:

1. **0000-MAINLINE**
   ```
   SET TRD-MAC-BOOKING-FAILED TO TRUE
   SET DEBIT-BUYER TO TRUE
   PERFORM 1000-MAC-BOOKING THRU 1000-EXIT
   IF TRD-MAC-BOOKING-DONE THEN
       SET TRD-MAC-BOOKING-FAILED TO TRUE
       SET CREDIT-SELLER TO TRUE
       PERFORM 1000-MAC-BOOKING THRU 1000-EXIT
   END-IF
   GOBACK
   ```

2. **1000-MAC-BOOKING**
   ```
   IF CREDIT-SELLER
       MOVE TRD-CURRENCY TO MAC-CURRENCY
       MOVE TRD-SELLER-MONEY-ACC-NUM TO MAC-NUMBER
   ELSE
       MOVE TRD-CURRENCY TO MAC-CURRENCY
       MOVE TRD-BUYER-MONEY-ACC-NUM TO MAC-NUMBER
   END-IF

   SELECT MAC_BALANCE FROM TBTRDMAC WHERE MAC_CURRENCY = :MAC-CURRENCY AND MAC_NUMBER = :MAC-NUMBER FOR UPDATE OF MAC_BALANCE

   IF SQLCODE NOT = 0
       SET TRD-MAC-BOOKING-FAILED TO TRUE
       MOVE SQLCODE TO WS-SQLCODE
       SET DATA-EXCEPTION TO TRUE
       MOVE SPACES TO WS-EXCEPTION
       STRING 'Open MAC_BOOKING Cursor failed : SQLCODE = ' WS-SQLCODE DELIMITED BY SIZE INTO WS-EXCEPTION
       PERFORM 9000-REPORT-EXCEPTION THRU 9000-EXIT
       GO TO 1000-EXIT
   END-IF

   FETCH MAC_BOOKING INTO :MAC-BALANCE

   IF SQLCODE = 0
       IF CREDIT-SELLER
           COMPUTE MAC-BALANCE = MAC-BALANCE + TRD-ORDER-AMOUNT
       ELSE
           COMPUTE MAC-BALANCE = MAC-BALANCE - TRD-ORDER-AMOUNT
       END-IF

       UPDATE TBTRDMAC SET MAC_BALANCE = :MAC-BALANCE WHERE MAC_CURRENCY = :MAC-CURRENCY AND MAC_NUMBER = :MAC-NUMBER

       IF SQLCODE = 0
           SET TRD-MAC-BOOKING-DONE TO TRUE
       ELSE
           SET TRD-MAC-BOOKING-FAILED TO TRUE
           MOVE SQLCODE TO WS-SQLCODE
           SET DATA-EXCEPTION TO TRUE
           MOVE SPACES TO WS-EXCEPTION
           STRING 'Update using MAC_BOOKING Cursor failed : SQLCODE = ' WS-SQLCODE DELIMITED BY SIZE INTO WS-EXCEPTION
           PERFORM 9000-REPORT-EXCEPTION THRU 9000-EXIT
       END-IF
   ELSE
       SET TRD-MAC-BOOKING-FAILED TO TRUE
       MOVE SQLCODE TO WS-SQLCODE
       SET DATA-EXCEPTION TO TRUE
       MOVE SPACES TO WS-EXCEPTION
       STRING 'Fetch MAC_BOOKING Cursor failed : SQLCODE = ' WS-SQLCODE DELIMITED BY SIZE INTO WS-EXCEPTION
       PERFORM 9000-REPORT-EXCEPTION THRU 9000-EXIT
   END-IF

   CLOSE MAC_BOOKING
   ```

3. **9000-REPORT-EXCEPTION**
   ```
   MOVE WS-EXCEPTION TO EXCEPTION-DESC
   MOVE LENGTH OF WS-EXCEPTION TO EXCEPTION-RECORD-LEN
   EVALUATE TRUE
       WHEN SYSTEM-EXCEPTION
           MOVE 'SYSTEM' TO EXCEPTION-TYPE
       WHEN APPL-EXCEPTION
           MOVE 'APPLICATION' TO EXCEPTION-TYPE
       WHEN DATA-EXCEPTION
           MOVE 'DATA' TO EXCEPTION-TYPE
   END-EVALUATE
   ADD 46 TO EXCEPTION-RECORD-LEN
   CALL WS-EXCEPTION-HANDLER USING EXCEPTION-RECORD-LEN, EXCEPTION-RECORD
   ```