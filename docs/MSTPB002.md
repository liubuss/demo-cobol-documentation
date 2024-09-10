# Documentation for COBOL Program MSTPB002

---

## 1. High-Level Overview

**Purpose and Functionality:**
The COBOL program MSTPB002 is designed to retrieve security data. It performs operations to fetch and process security-related information from a database.

**Key Inputs, Processes, and Outputs:**
- **Inputs:**
  - Security-related parameters such as `SEC-FIGI`, `SEC-CURRENCY`, `SEC-DESCRIPTION`, etc.
- **Processes:**
  - Mainline processing
  - Security data retrieval
  - Error handling and status setting
- **Outputs:**
  - Security status and error messages
  - Retrieved security data fields

---

## 2. Data Structures

**Input Data Formats:**

| Name                  | Source         | Fields                                                                 |
|-----------------------|----------------|------------------------------------------------------------------------|
| Security Parameters   | Database       | `SEC-FIGI`, `SEC-CURRENCY`, `SEC-DESCRIPTION`, `SEC-SYMBOL`, `SEC-SHARE-CLASS-FIGI`, `SEC-TYPE` |
| SQL Parameters        | Internal       | `SQL-INIT-FLAG`, `SQL-INIT-DONE`, `SQLCODE`                            |

**Output Data Formats:**

| Name                  | Fields                                                                 |
|-----------------------|------------------------------------------------------------------------|
| Security Status       | `88-SECURITY-EXCEPTION`, `88-SECURITY-ACTIVE`, `88-SECURITY-NOT-FOUND` |
| Security Error Message| `LS-SECURITY-ERROR-MSG`                                                |
| Security Data         | `LS-CURRENCY`, `LS-DESCRIPTION`, `LS-SYMBOL`, `LS-SHARE-CLASS-FIGI`, `LS-TYPE` |

---

## 3. Functional Decomposition

**Mainline Processing (0000-MAINLINE):**
- **Inputs:** None
- **Outputs:** Security status and error messages
- **Description:**
  - Sets `88-SECURITY-EXCEPTION` to true.
  - Initializes `LS-SECURITY-ERROR-MSG` with spaces.
  - Calls `1000-GET-SECURITY` paragraph.
  - Returns control to the caller using `GOBACK`.

**Get Security Data (1000-GET-SECURITY):**
- **Inputs:** Security parameters and SQL parameters
- **Outputs:** Security data and status
- **Description:**
  - Copies `LS-FIGI` to `SEC-FIGI`.
  - Executes SQL SELECT statement to fetch security data.
  - Copies `SQLCODE` to `WS-SQLCODE`.
  - Evaluates `SQLCODE`:
    - When `0`: Copies fetched security data to respective fields and sets `88-SECURITY-ACTIVE` to true.
    - When `100`: Sets `88-SECURITY-NOT-FOUND` to true and initializes `LS-SECURITY-ERROR-MSG` with "SECURITY Not Found".
    - When others: Sets `88-SECURITY-EXCEPTION` to true, initializes `LS-SECURITY-ERROR-MSG` with spaces, and concatenates error message with `WS-SQLCODE`.

---

## 4. Program Logic

**Pseudocode:**

```
MAINLINE:
    SET 88-SECURITY-EXCEPTION TO TRUE
    MOVE SPACES TO LS-SECURITY-ERROR-MSG
    PERFORM 1000-GET-SECURITY THRU 1000-EXIT
    GOBACK

1000-GET-SECURITY:
    MOVE LS-FIGI TO SEC-FIGI
    EXECUTE SQL SELECT FROM TBTRDSEC WHERE SEC_FIGI = :SEC-FIGI
    MOVE SQLCODE TO WS-SQLCODE
    EVALUATE SQLCODE
        WHEN 0:
            MOVE SEC-CURRENCY TO LS-CURRENCY
            MOVE SEC-DESCRIPTION TO LS-DESCRIPTION
            MOVE SEC-SYMBOL TO LS-SYMBOL
            MOVE SEC-SHARE-CLASS-FIGI TO LS-SHARE-CLASS-FIGI
            MOVE SEC-TYPE TO LS-TYPE
            SET 88-SECURITY-ACTIVE TO TRUE
        WHEN 100:
            SET 88-SECURITY-NOT-FOUND TO TRUE
            MOVE 'SECURITY Not Found' TO LS-SECURITY-ERROR-MSG
        WHEN OTHER:
            SET 88-SECURITY-EXCEPTION TO TRUE
            MOVE SPACES TO LS-SECURITY-ERROR-MSG
            STRING 'SECURITY Access failed. SQLCODE = ' WS-SQLCODE INTO LS-SECURITY-ERROR-MSG
```
