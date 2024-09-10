# Test Cases for MSTPB002 Migration

To ensure the successful migration of the COBOL program MSTPB002 to a modern programming language, we need to create comprehensive test cases. These test cases will verify that the migrated program behaves as expected and produces the correct outputs based on various input scenarios.

## Test Case 1: Successful Security Data Retrieval

**Description:** Verify that the program successfully retrieves security data when the security FIGI exists in the database.

**Input:**
- `SQL-INIT-FLAG`: 'Y'
- `SEC-FIGI`: '123456'
- `SEC-SYMBOL`: 'AAPL'
- `SEC-CURRENCY`: 'USD'
- `SEC-SHARE-CLASS-FIGI`: '1234567890'
- `SEC-DESCRIPTION`: 'Apple Inc.'
- `SEC-TYPE`: 'Equity'

**Expected Output:**
- `LS-CURRENCY`: 'USD'
- `LS-DESCRIPTION`: 'Apple Inc.'
- `LS-SYMBOL`: 'AAPL'
- `LS-SHARE-CLASS-FIGI`: '1234567890'
- `LS-TYPE`: 'Equity'
- `88-SECURITY-ACTIVE`: TRUE
- `88-SECURITY-EXCEPTION`: FALSE
- `88-SECURITY-NOT-FOUND`: FALSE
- `LS-SECURITY-ERROR-MSG`: ''

## Test Case 2: Security Data Not Found

**Description:** Verify that the program handles the case where the security FIGI does not exist in the database.

**Input:**
- `SQL-INIT-FLAG`: 'Y'
- `SEC-FIGI`: '999999'
- `SEC-SYMBOL`: ''
- `SEC-CURRENCY`: ''
- `SEC-SHARE-CLASS-FIGI`: ''
- `SEC-DESCRIPTION`: ''
- `SEC-TYPE`: ''

**Expected Output:**
- `LS-CURRENCY`: ''
- `LS-DESCRIPTION`: ''
- `LS-SYMBOL`: ''
- `LS-SHARE-CLASS-FIGI`: ''
- `LS-TYPE`: ''
- `88-SECURITY-ACTIVE`: FALSE
- `88-SECURITY-EXCEPTION`: FALSE
- `88-SECURITY-NOT-FOUND`: TRUE
- `LS-SECURITY-ERROR-MSG`: 'SECURITY Not Found'

## Test Case 3: SQL Exception Handling

**Description:** Verify that the program handles SQL exceptions correctly.

**Input:**
- `SQL-INIT-FLAG`: 'Y'
- `SEC-FIGI`: '123456'
- `SEC-SYMBOL`: 'AAPL'
- `SEC-CURRENCY`: 'USD'
- `SEC-SHARE-CLASS-FIGI`: '1234567890'
- `SEC-DESCRIPTION`: 'Apple Inc.'
- `SEC-TYPE`: 'Equity'

**Expected Output:**
- `LS-CURRENCY`: ''
- `LS-DESCRIPTION`: ''
- `LS-SYMBOL`: ''
- `LS-SHARE-CLASS-FIGI`: ''
- `LS-TYPE`: ''
- `88-SECURITY-ACTIVE`: FALSE
- `88-SECURITY-EXCEPTION`: TRUE
- `88-SECURITY-NOT-FOUND`: FALSE
- `LS-SECURITY-ERROR-MSG`: 'SECURITY Access failed. SQLCODE = <SQLCODE>'
