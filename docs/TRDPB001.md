# Documentation for COBOL Program `TRDPB001`

## 1. High-Level Overview

### Purpose and Functionality
The COBOL program `TRDPB001` is designed to handle order settlement processes. It reads from a settlement queue, books settlement and money accounts, updates orders, and logs summaries. The program supports multi-threading by running multiple batch jobs for settling trades based on currency.

### Key Inputs, Processes, and Outputs
- **Inputs:**
  - Settlement Queue (`TBTRDSTQ`)
  - Trade Data
- **Processes:**
  - Initialize variables and open the settlement queue.
  - Read and process trade records from the settlement queue.
  - Perform accounting operations including booking settlement and money accounts.
  - Update order statuses.
  - Log summaries and handle exceptions.
- **Outputs:**
  - Updated order records.
  - Logged summaries.
  - Exception reports.

## 2. Data Structures

### Input Data Formats

| Name            | Source       | Fields                                                                 |
|-----------------|--------------|------------------------------------------------------------------------|
| Trade Record    | TBTRDSTQ     | `STQ-ID`, `STQ-TRADE-DATA`, `STQ-CURRENCY` |

#### Fields Description
- **STQ-ID:** Identifier for the settlement queue entry.
- **STQ-TRADE-DATA:** Data related to the trade.
- **STQ-CURRENCY:** Currency of the trade.

### Output Data Formats

| Name            | Source       | Fields                                                                 |
|-----------------|--------------|------------------------------------------------------------------------|
| Order Record    | TBTRDORD     | `ORD-STATUS`, `ORD-CURRENCY`, `ORD-TRADING-XCHNG`, `ORD-TRADEID`, `ORD-FIGI`, `ORD-BUY-SELL-IND` |
| Summary Record  | TBTRDSUM     | `SUM-CUSTOMER-ID`, `SUM-OVERDUE`, `SUM-REJECTED`, `SUM-SETTLED`, `SUM-CURRENCY`, `SUM-OPEN-BALANCE`, `SUM-TOTAL-DEBIT`, `SUM-TOTAL-CREDIT`, `SUM-CLOSE-BALANCE` |
| Exception Record| TRDPBEXC     | `EXCEPTION-DESC`, `EXCEPTION-RECORD-LEN`, `EXCEPTION-TYPE` |

#### Fields Description
- **ORD-STATUS:** Status of the order.
- **ORD-CURRENCY:** Currency of the order.
- **ORD-TRADING-XCHNG:** Trading exchange of the order.
- **ORD-TRADEID:** Trade identifier.
- **ORD-FIGI:** Financial Instrument Global Identifier.
- **ORD-BUY-SELL-IND:** Indicator for buy or sell.
- **SUM-CUSTOMER-ID:** Customer identifier for the summary.
- **SUM-OVERDUE:** Number of overdue trades.
- **SUM-REJECTED:** Number of rejected trades.
- **SUM-SETTLED:** Number of settled trades.
- **SUM-CURRENCY:** Currency for the summary.
- **SUM-OPEN-BALANCE:** Opening balance for the summary.
- **SUM-TOTAL-DEBIT:** Total debit amount for the summary.
- **SUM-TOTAL-CREDIT:** Total credit amount for the summary.
- **SUM-CLOSE-BALANCE:** Closing balance for the summary.
- **EXCEPTION-DESC:** Description of the exception.
- **EXCEPTION-RECORD-LEN:** Length of the exception record.
- **EXCEPTION-TYPE:** Type of the exception.

## 3. Functional Decomposition

### Functions and Modules

#### 0000-MAINLINE
- **Inputs:** None
- **Outputs:** None
- **Description:** Main entry point of the program. Initializes variables, opens files, reads records, writes records, and performs housekeeping.

#### 1000-TRADE-SETTLEMENT
- **Inputs:** Trade data from `TBTRDSTQ`
- **Outputs:** None
- **Description:** Handles the settlement of trades by reading from the settlement queue and performing accounting operations.

#### 2000-ACCOUNTING
- **Inputs:** Trade data
- **Outputs:** None
- **Description:** Performs accounting operations including booking settlement and money accounts.

#### 2001-SAC-BOOKING
- **Inputs:** Trade data
- **Outputs:** None
- **Description:** Books settlement accounts.

#### 2002-MAC-BOOKING
- **Inputs:** Trade data
- **Outputs:** None
- **Description:** Books money accounts.

#### 5000-READ-FROM-STLMT-QUEUE
- **Inputs:** Settlement queue data
- **Outputs:** Trade data
- **Description:** Reads trade data from the settlement queue.

#### 8000-UPDATE-ORDER
- **Inputs:** Order data
- **Outputs:** Updated order records
- **Description:** Updates the status of orders.

#### 8001-LOG-SUMMARY
- **Inputs:** Summary data
- **Outputs:** Logged summaries
- **Description:** Logs summary information for the processed trades.

#### 9000-REPORT-EXCEPTION
- **Inputs:** Exception data
- **Outputs:** Exception reports
- **Description:** Reports exceptions encountered during processing.

## 4. Program Logic

### Pseudocode Representation

```pseudocode
BEGIN PROGRAM TRDPB001

    PERFORM 1000-INITIALIZE THRU 1000-EXIT
    MOVE 1 TO WS-PAGE-NO
    PERFORM 1100-WRITE-HEADER THRU 1100-EXIT
    MOVE SPACES TO EOF-FLAG
    OPEN INPUT TBTRDSTQ
    MOVE 'ERROR OPENING TBTRDSTQ' TO ERR-MSG
    PERFORM 9999-VSAM-ERR-CHK THRU 9999-EXIT
    MOVE 0 TO STQ-ID
    START TBTRDSTQ KEY >= STQ-ID
    MOVE 'ERROR POSITIONING TBTRDSTQ' TO ERR-MSG
    PERFORM 9999-VSAM-ERR-CHK THRU 9999-EXIT
    PERFORM 1200-READ-BOOK THRU 1200-EXIT
    PERFORM 2000-READ-ALL-BOOKS THRU 2000-EXIT UNTIL END-OF-FILE
    CLOSE TBTRDSTQ
    MOVE 'ERROR CLOSING TBTRDSTQ' TO ERR-MSG
    PERFORM 9999-VSAM-ERR-CHK THRU 9999-EXIT
    PERFORM 3000-HOUSEKEEPING THRU 3000-EXIT
    STOP RUN

END PROGRAM TRDPB001
```

### Detailed Steps

1. **Initialization:**
   - Perform initialization by calling `1000-INITIALIZE`.
   - Set the page number to 1.
   - Write the header to the output file by calling `1100-WRITE-HEADER`.
   - Set the EOF flag to spaces.
   - Open the settlement queue file `TBTRDSTQ` for input.
   - Handle any errors during file opening by calling `9999-VSAM-ERR-CHK`.

2. **Reading and Processing Records:**
   - Set the settlement queue ID to 0.
   - Start reading the settlement queue file from the beginning.
   - Handle any errors during file positioning by calling `9999-VSAM-ERR-CHK`.
   - Read the first trade record by calling `1200-READ-BOOK`.
   - Loop through all trade records by calling `2000-READ-ALL-BOOKS` until the end of the file is reached.

3. **Closing Files and Housekeeping:**
   - Close the settlement queue file `TBTRDSTQ`.
   - Handle any errors during file closing by calling `9999-VSAM-ERR-CHK`.
   - Perform housekeeping tasks by calling `3000-HOUSEKEEPING`.
   - Terminate the program.

This documentation provides a comprehensive guide to understanding and re-implementing the functionality of the COBOL program `TRDPB001` in a modern programming language.