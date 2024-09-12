# MSTPB001 COBOL Program Documentation

## 1. High-Level Overview

**Purpose and Functionality:**
The MSTPB001 COBOL program is designed to retrieve customer data from a database. It processes the main logic and then specifically fetches customer details based on a given customer ID.

**Key Inputs, Processes, and Outputs:**
- **Inputs:**
  - Customer ID
- **Processes:**
  - Main logic processing
  - Fetching customer details from the database
- **Outputs:**
  - Customer details including currency, description, and status
  - Error messages if the customer is not found or inactive

## 2. Data Structures

### Input Data Formats

| Name              | Source       | Fields                                                                                  |
|-------------------|--------------|-----------------------------------------------------------------------------------------|
| Customer Input    | User/Input   | **LS-CUSTOMER-ID**: PIC X(10) - The ID of the customer to fetch.                        |

### Output Data Formats

| Name               | Source         | Fields                                                                                               |
|--------------------|----------------|------------------------------------------------------------------------------------------------------|
| Customer Output    | Program Output | **LS-CUSTOMER-CURRENCY**: PIC X(3) - The currency code of the customer.                              |
|                    |                | **LS-CUSTOMER-DESCRIPTION**: PIC X(50) - The description of the customer.                            |
|                    |                | **LS-CUSTOMER-STATUS**: PIC X(1) - The status of the customer (A for active, I for inactive).        |
|                    |                | **LS-CUSTOMER-ERROR-MSG**: PIC X(100) - Error message if the customer is not found or inactive.      |

## 3. Functional Decomposition

### Main Functions

#### 0000-MAINLINE
- **Inputs:** None
- **Outputs:** Sets `88-CUSTOMER-EXCEPTION`, initializes `LS-CUSTOMER-ERROR-MSG`, calls `1000-GET-CUSTOMER`
- **Description:**
  - Sets the exception flag to true.
  - Initializes the error message to spaces.
  - Calls the `1000-GET-CUSTOMER` function.
  - Returns control to the caller.

#### 1000-GET-CUSTOMER
- **Inputs:** `SQL-INIT-DONE`, `SQL-INIT-FLAG`, `CUS-STATUS`, `LS-CUSTOMER-ID`
- **Outputs:** `LS-CUSTOMER-CURRENCY`, `LS-CUSTOMER-DESCRIPTION`, `LS-CUSTOMER-STATUS`, `LS-CUSTOMER-ERROR-MSG`
- **Description:**
  - Checks if SQL is initialized.
  - Fetches customer details from the database.
  - Sets the output fields based on the fetched data.
  - Handles exceptions and sets error messages if needed.

## 4. Detailed Logic

### 0000-MAINLINE
1. **Initialize Exception Flag:**
   - `SET 88-CUSTOMER-EXCEPTION TO TRUE`
2. **Initialize Error Message:**
   - `MOVE SPACES TO LS-CUSTOMER-ERROR-MSG`
3. **Call Get Customer Function:**
   - `PERFORM 1000-GET-CUSTOMER`
4. **Return Control:**
   - `GOBACK`

### 1000-GET-CUSTOMER
1. **Check SQL Initialization:**
   - `IF SQL-INIT-DONE NOT = 'Y'`
     - `MOVE 'SQL NOT INITIALIZED' TO LS-CUSTOMER-ERROR-MSG`
     - `EXIT`
2. **Fetch Customer Details:**
   - `EXEC SQL`
     - `SELECT CURRENCY, DESCRIPTION, STATUS`
     - `INTO :LS-CUSTOMER-CURRENCY, :LS-CUSTOMER-DESCRIPTION, :LS-CUSTOMER-STATUS`
     - `FROM CUSTOMERS`
     - `WHERE CUSTOMER_ID = :LS-CUSTOMER-ID`
   - `END-EXEC`
3. **Handle SQL Errors:**
   - `IF SQLCODE NOT = 0`
     - `MOVE 'CUSTOMER NOT FOUND' TO LS-CUSTOMER-ERROR-MSG`
     - `EXIT`
4. **Check Customer Status:**
   - `IF LS-CUSTOMER-STATUS = 'I'`
     - `MOVE 'CUSTOMER INACTIVE' TO LS-CUSTOMER-ERROR-MSG`
     - `EXIT`

## 5. Error Handling

### Common Errors and Messages

| Error Condition           | Error Message                  |
|---------------------------|--------------------------------|
| SQL Not Initialized       | SQL NOT INITIALIZED            |
| Customer Not Found        | CUSTOMER NOT FOUND             |
| Customer Inactive         | CUSTOMER INACTIVE              |

## 6. Example Usage

### Input Example
```plaintext
LS-CUSTOMER-ID: '1234567890'
```

### Output Example
```plaintext
LS-CUSTOMER-CURRENCY: 'USD'
LS-CUSTOMER-DESCRIPTION: 'John Doe'
LS-CUSTOMER-STATUS: 'A'
LS-CUSTOMER-ERROR-MSG: ''
```

### Error Example
```plaintext
LS-CUSTOMER-ID: '0000000000'
LS-CUSTOMER-ERROR-MSG: 'CUSTOMER NOT FOUND'
```

---

This documentation provides a comprehensive overview of the MSTPB001 COBOL program, detailing its purpose, data structures, functional decomposition, detailed logic, error handling, and example usage. This should enable a developer with no prior COBOL experience to re-implement its functionality in a modern programming language.