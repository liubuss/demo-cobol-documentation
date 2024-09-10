# MSTPB002 Documentation

## High-Level Overview

The COBOL program MSTPB002 is designed to retrieve security data. It performs operations to fetch and process security-related information from a database.

- [Indicative Test Cases ](MSTPB002_test_cases.md)  

## Data Structures

### Input Data Formats:

| Name                  | Source          | Fields                                                                 |
|-----------------------|-----------------|------------------------------------------------------------------------|
| Security Input Record | Internal        | `SQL-INIT-FLAG`, `SEC-SYMBOL`, `LS-FIGI`, `SEC-CURRENCY`, `SQL-INIT-DONE`, `SEC-SHARE-CLASS-FIGI`, `SQLCODE`, `SEC-DESCRIPTION`, `SEC-TYPE` |

### Output Data Formats:

| Name                  | Source          | Fields                                                                 |
|-----------------------|-----------------|------------------------------------------------------------------------|
| Security Output Record| Internal        | `88-SECURITY-EXCEPTION`, `LS-SECURITY-ERROR-MSG`, `LS-SECURITY-STATUS`, `SEC-CURRENCY`, `LS-CURRENCY`, `WS-SQLCODE`, `LS-DESCRIPTION`, `LS-SYMBOL`, `LS-SHARE-CLASS-FIGI`, `LS-TYPE`, `RETURN-CODE` |

## Functional Decomposition

### Main Functions:

1. **0000-MAINLINE**
   - **Inputs:** None
   - **Outputs:** `88-SECURITY-EXCEPTION`, `LS-SECURITY-ERROR-MSG`, `LS-SECURITY-STATUS`
   - **Description:** Initializes variables and calls the `1000-GET-SECURITY` function to retrieve security data.

2. **1000-GET-SECURITY**
   - **Inputs:** `SQL-INIT-FLAG`, `SEC-SYMBOL`, `LS-FIGI`, `SEC-CURRENCY`, `SQL-INIT-DONE`, `SEC-SHARE-CLASS-FIGI`, `SQLCODE`, `SEC-DESCRIPTION`, `SEC-TYPE`
   - **Outputs:** `88-SECURITY-EXCEPTION`, `LS-SECURITY-ERROR-MSG`, `SEC-SYMBOL`, `88-SECURITY-ACTIVE`, `LS-CURRENCY`, `WS-SQLCODE`, `SQLCODE`, `SEC-TYPE`, `SEC-FIGI`, `LS-SHARE-CLASS-FIGI`, `LS-FIGI`, `LS-SYMBOL`, `LS-SECURITY-STATUS`, `RETURN-CODE`, `SEC-CURRENCY`, `LS-DESCRIPTION`, `SEC-SHARE-CLASS-FIGI`, `88-SECURITY-NOT-FOUND`, `SEC-DESCRIPTION`, `LS-TYPE`
   - **Description:** Retrieves security data from the database and processes it based on the SQLCODE.

## Program Logic

### Pseudocode:

```pseudocode
PROCEDURE DIVISION.

0000-MAINLINE.
    SET 88-SECURITY-EXCEPTION TO TRUE
    MOVE SPACES TO LS-SECURITY-ERROR-MSG
    PERFORM 1000-GET-SECURITY THRU 1000-EXIT
    GOBACK

1000-GET-SECURITY.
    MOVE LS-FIGI TO SEC-FIGI
    EXEC SQL
        SELECT SEC_CURRENCY, SEC_DESCRIPTION, SEC_SYMBOL, SEC_SHARE_CLASS_FIGI, SEC_TYPE
        INTO :SEC-CURRENCY, :SEC-DESCRIPTION, :SEC-SYMBOL, :SEC-SHARE-CLASS-FIGI, :SEC-TYPE
        FROM TBTRDSEC
        WHERE SEC_FIGI = :SEC-FIGI
    END-EXEC
    MOVE SQLCODE TO WS-SQLCODE
    EVALUATE SQLCODE
        WHEN 0
            MOVE SEC-CURRENCY TO LS-CURRENCY
            MOVE SEC-DESCRIPTION TO LS-DESCRIPTION
            MOVE SEC-SYMBOL TO LS-SYMBOL
            MOVE SEC-SHARE-CLASS-FIGI TO LS-SHARE-CLASS-FIGI
            MOVE SEC-TYPE TO LS-TYPE
            SET 88-SECURITY-ACTIVE TO TRUE
        WHEN 100
            SET 88-SECURITY-NOT-FOUND TO TRUE
            MOVE 'SECURITY Not Found' TO LS-SECURITY-ERROR-MSG
        WHEN OTHER
            SET 88-SECURITY-EXCEPTION TO TRUE
            MOVE SPACES TO LS-SECURITY-ERROR-MSG
            STRING 'SECURITY Access failed. SQLCODE = ' WS-SQLCODE DELIMITED BY SIZE INTO LS-SECURITY-ERROR-MSG
    END-EVALUATE
1000-EXIT.
    EXIT.
```
