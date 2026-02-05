# 📋 All Transformations - Complete Guide with Examples

This document explains all **17 transformations** in your ETL framework with real input/output examples.

---

## 1️⃣ **TRIM_STRINGS**

**Purpose:** Remove leading and trailing whitespace from string columns

**YAML Configuration:**
```yaml
{ order: 0, trim_strings: { 
    enabled: true, 
    columns: [first_name, last_name, email_raw] 
} }
```

**Input:**
| first_name | last_name | email_raw |
|------------|-----------|-----------|
| " John " | "Doe " | " john@example.com " |
| "Alice" | " Smith" | "alice@test.com " |

**Output:**
| first_name | last_name | email_raw |
|------------|-----------|-----------|
| "John" | "Doe" | "john@example.com" |
| "Alice" | "Smith" | "alice@test.com" |

---

## 2️⃣ **REGEX_CLEAN**

**Purpose:** Clean data using regex patterns (remove unwanted characters)

### **For Number Columns:**

**YAML Configuration:**
```yaml
{ order: 1, regex_clean: { 
    enabled: true,
    number_columns: [
      { name: balance_raw, out: balance_clean, 
        pattern_remove: "[^0-9\\.\\-]", trim: true }
    ],
    email_columns: []
} }
```

**Input:**
| balance_raw |
|-------------|
| "$1,234.56" |
| " 9,876 " |
| "+-500.00" |

**Output (adds balance_clean column):**
| balance_raw | balance_clean |
|-------------|---------------|
| "$1,234.56" | "1234.56" |
| " 9,876 " | "9876" |
| "+-500.00" | "-500.00" |

### **For Email Columns:**

**YAML Configuration:**
```yaml
{ order: 1, regex_clean: { 
    enabled: true,
    number_columns: [],
    email_columns: [
      { name: email_raw, out: email, trim: true, lowercase: true }
    ]
} }
```

**Input:**
| email_raw |
|-----------|
| " John@Example.COM " |
| "ALICE@TEST.com" |

**Output (adds email column):**
| email_raw | email |
|-----------|-------|
| " John@Example.COM " | "john@example.com" |
| "ALICE@TEST.com" | "alice@test.com" |

---

## 3️⃣ **PARSE_NUMERIC**

**Purpose:** Convert cleaned string numbers to numeric types (int, double, float)

**YAML Configuration:**
```yaml
{ order: 2, parse_numeric: { 
    enabled: true,
    columns: [
      { name: balance_clean, out: balance, cast_type: double }
    ]
} }
```

**Input:**
| balance_clean |
|---------------|
| "1234.56" |
| "9876" |
| "-500.00" |

**Output (adds balance column):**
| balance_clean | balance |
|---------------|---------|
| "1234.56" | 1234.56 (double) |
| "9876" | 9876.0 (double) |
| "-500.00" | -500.0 (double) |

---

## 4️⃣ **PARSE_DATE**

**Purpose:** Parse date/timestamp strings into proper date/timestamp types

**YAML Configuration:**
```yaml
{ order: 3, parse_date: { 
    enabled: true,
    columns: [
      { name: opened_date_raw, out: opened_date,
        formats: ["yyyy-MM-dd HH:mm:ss", "yyyy-MM-dd", "dd/MM/yyyy"] }
    ]
} }
```

**Input:**
| opened_date_raw |
|-----------------|
| "2023-01-15 10:30:00" |
| "2023-01-15" |
| "15/01/2023" |

**Output (adds opened_date column):**
| opened_date_raw | opened_date |
|-----------------|-------------|
| "2023-01-15 10:30:00" | 2023-01-15 (date) |
| "2023-01-15" | 2023-01-15 (date) |
| "15/01/2023" | 2023-01-15 (date) |

---

## 5️⃣ **NORMALIZE_STATUS**

**Purpose:** Standardize categorical values using a mapping dictionary

**YAML Configuration:**
```yaml
{ order: 4, normalize_status: { 
    enabled: true,
    columns: [
      { name: status_raw, out: status,
        mapping: { 
          a: "Active", 
          active: "Active", 
          closed: "Closed", 
          c: "Closed" 
        } }
    ]
} }
```

**Input:**
| status_raw |
|------------|
| "a" |
| "ACTIVE" |
| " Active " |
| "Closed" |
| "c" |
| " C " |

**Output (adds status column):**
| status_raw | status |
|------------|--------|
| "a" | "Active" |
| "ACTIVE" | "Active" |
| " Active " | "Active" |
| "Closed" | "Closed" |
| "c" | "Closed" |
| " C " | "Closed" |

**Note:** Trims whitespace and converts to lowercase before matching!

---

## 6️⃣ **NULL_HANDLING**

**Purpose:** Drop rows that have null values in critical columns

**YAML Configuration:**
```yaml
{ order: 4, null_handling: { 
    enabled: true, 
    drop_if_null: [customer_id, account_id] 
} }
```

**Input:**
| customer_id | account_id | balance |
|-------------|------------|---------|
| "CUST_1" | "ACC_1" | 1000.00 |
| null | "ACC_2" | 2000.00 |
| "CUST_3" | null | 3000.00 |
| "CUST_4" | "ACC_4" | null |

**Output:**
| customer_id | account_id | balance |
|-------------|------------|---------|
| "CUST_1" | "ACC_1" | 1000.00 |
| "CUST_4" | "ACC_4" | null |

**Removed:**
- Row 2: null customer_id ❌
- Row 3: null account_id ❌
- Row 4: null balance is OK (not in critical list) ✅

---

## 7️⃣ **DUPLICATE_HANDLING**

**Purpose:** Remove duplicate rows based on unique key columns

**YAML Configuration:**
```yaml
{ order: 5, duplicate_handling: { 
    enabled: true, 
    unique_keys: [customer_id] 
} }
```

**Input:**
| customer_id | first_name | balance |
|-------------|------------|---------|
| "CUST_1" | "John" | 1000.00 |
| "CUST_2" | "Alice" | 2000.00 |
| "CUST_1" | "John" | 1000.00 |
| "CUST_3" | "Bob" | 3000.00 |
| "CUST_2" | "Alice" | 2500.00 |

**Output:**
| customer_id | first_name | balance |
|-------------|------------|---------|
| "CUST_1" | "John" | 1000.00 |
| "CUST_2" | "Alice" | 2000.00 |
| "CUST_3" | "Bob" | 3000.00 |

**Note:** Keeps first occurrence, drops subsequent duplicates

---

## 8️⃣ **RENAME_COLUMNS**

**Purpose:** Rename columns using a mapping dictionary

**YAML Configuration:**
```yaml
{ order: 1, rename_columns: { 
    enabled: true,
    mapping: { 
      old_name: new_name,
      cust_id: customer_id,
      amt: amount
    }
} }
```

**Input:**
| cust_id | amt | old_name |
|---------|-----|----------|
| "CUST_1" | 100 | "John" |

**Output:**
| customer_id | amount | new_name |
|-------------|--------|----------|
| "CUST_1" | 100 | "John" |

---

## 9️⃣ **STANDARDIZE_COLUMN_CASE**

**Purpose:** Convert column NAMES (not values) to snake_case or lowercase

**YAML Configuration:**
```yaml
{ order: 4, standardize_column_case: { 
    enabled: true, 
    format: snake_case 
} }
```

**Input (column names):**
| FirstName | LastName | Email-Address | Phone Number |
|-----------|----------|---------------|--------------|
| "John" | "Doe" | "john@test.com" | "123-456" |

**Output (column names changed):**
| first_name | last_name | email_address | phone_number |
|------------|-----------|---------------|--------------|
| "John" | "Doe" | "john@test.com" | "123-456" |

**Note:** This currently only renames COLUMNS, not values!

---

## 🔟 **FILL_MISSING**

**Purpose:** Fill null values with default values

**YAML Configuration:**
```yaml
{ order: 9, fill_missing: { 
    enabled: true,
    columns: [
      { name: currency, value: "UNKNOWN" },
      { name: status, value: "Pending" }
    ]
} }
```

**Input:**
| account_id | currency | status |
|------------|----------|--------|
| "ACC_1" | "USD" | "Active" |
| "ACC_2" | null | "Closed" |
| "ACC_3" | "EUR" | null |
| "ACC_4" | null | null |

**Output:**
| account_id | currency | status |
|------------|----------|--------|
| "ACC_1" | "USD" | "Active" |
| "ACC_2" | "UNKNOWN" | "Closed" |
| "ACC_3" | "EUR" | "Pending" |
| "ACC_4" | "UNKNOWN" | "Pending" |

---

## 1️⃣1️⃣ **SURROGATE_KEY**

**Purpose:** Generate a unique UUID key for each row

**YAML Configuration:**
```yaml
{ order: 5, surrogate_key: { 
    enabled: true, 
    column: customer_sk, 
    method: uuid 
} }
```

**Input:**
| customer_id | first_name |
|-------------|------------|
| "CUST_1" | "John" |
| "CUST_2" | "Alice" |

**Output (adds customer_sk column):**
| customer_id | first_name | customer_sk |
|-------------|------------|-------------|
| "CUST_1" | "John" | "a1b2c3d4-e5f6-7890-abcd-ef1234567890" |
| "CUST_2" | "Alice" | "b2c3d4e5-f6a7-8901-bcde-f12345678901" |

---

## 1️⃣2️⃣ **OUTLIER_CAPPING**

**Purpose:** Cap numeric values to a min/max range

**YAML Configuration:**
```yaml
{ order: 11, outlier_capping: { 
    enabled: true,
    columns: [
      { name: balance, min: -50000, max: 10000000 }
    ]
} }
```

**Input:**
| account_id | balance |
|------------|---------|
| "ACC_1" | 5000.00 |
| "ACC_2" | -100000.00 |
| "ACC_3" | 15000000.00 |
| "ACC_4" | 0.00 |

**Output:**
| account_id | balance |
|------------|---------|
| "ACC_1" | 5000.00 |
| "ACC_2" | -50000.00 (capped to min) |
| "ACC_3" | 10000000.00 (capped to max) |
| "ACC_4" | 0.00 |

---

## 1️⃣3️⃣ **FUZZY_NORMALIZE**

**Purpose:** Normalize text by removing punctuation, collapsing spaces, and lowercasing

**YAML Configuration:**
```yaml
{ order: 1, fuzzy_normalize: { 
    enabled: true,
    columns: [
      { name: company_name, out: company_normalized }
    ],
    rules: {
      remove_punctuation: true,
      collapse_spaces: true,
      lowercase: true
    }
} }
```

**Input:**
| company_name |
|--------------|
| "ABC Corp., Inc." |
| "XYZ   Company!!!" |
| "Test-Company (2023)" |

**Output (adds company_normalized column):**
| company_name | company_normalized |
|--------------|-------------------|
| "ABC Corp., Inc." | "abc corp inc" |
| "XYZ   Company!!!" | "xyz company" |
| "Test-Company (2023)" | "testcompany 2023" |

---

## 1️⃣4️⃣ **FLATTEN_JSON**

**Purpose:** Flatten nested JSON structures (structs and arrays) into flat columns

**YAML Configuration:**
```yaml
{ order: 0, flatten_json: { 
    enabled: true 
} }
```

**Input (nested structure):**
```
root
 |-- event_id: string
 |-- session_info: struct
 |    |-- session_id: string
 |    |-- page_url: string
 |-- device: struct
 |    |-- type: string
 |    |-- os: string
```

| event_id | session_info | device |
|----------|--------------|--------|
| "EVT001" | {session_id: "sess_123", page_url: "https://..."} | {type: "mobile", os: "iOS"} |

**Output (flattened):**
```
root
 |-- event_id: string
 |-- session_info_session_id: string
 |-- session_info_page_url: string
 |-- device_type: string
 |-- device_os: string
```

| event_id | session_info_session_id | session_info_page_url | device_type | device_os |
|----------|------------------------|----------------------|-------------|-----------|
| "EVT001" | "sess_123" | "https://..." | "mobile" | "iOS" |

---

## 1️⃣5️⃣ **SCHEMA_VALIDATION**

**Purpose:** Validate that required columns exist in the DataFrame

**YAML Configuration:**
```yaml
{ order: 0, schema_validation: { 
    enabled: true,
    required_columns: [customer_id, account_id, balance]
} }
```

**Input (columns):**
```
DataFrame has: customer_id, account_id, status
```

**Output:**
```
❌ Error: Missing required columns: ['balance']
Pipeline stops!
```

**Input (columns):**
```
DataFrame has: customer_id, account_id, balance, status
```

**Output:**
```
✅ All required columns present
Pipeline continues!
```

---

## 1️⃣6️⃣ **JSON_KEY_DEFAULT**

**Purpose:** Add missing columns with default values

**YAML Configuration:**
```yaml
{ order: 1, json_key_default: { 
    enabled: true,
    columns: [
      { name: status, default: "Unknown" },
      { name: priority, default: 0 }
    ]
} }
```

**Input (DataFrame is missing status and priority columns):**
| customer_id | name |
|-------------|------|
| "CUST_1" | "John" |

**Output (adds missing columns with defaults):**
| customer_id | name | status | priority |
|-------------|------|--------|----------|
| "CUST_1" | "John" | "Unknown" | 0 |

---

## 1️⃣7️⃣ **ARRAY_SIZE_VALIDATION**

**Purpose:** Convert empty arrays to null

**YAML Configuration:**
```yaml
{ order: 2, array_size_validation: { 
    enabled: true,
    columns: [tags, categories]
} }
```

**Input:**
| product_id | tags | categories |
|------------|------|------------|
| "PROD_1" | ["electronics", "sale"] | ["tech"] |
| "PROD_2" | [] | ["books"] |
| "PROD_3" | ["new"] | [] |

**Output:**
| product_id | tags | categories |
|------------|------|------------|
| "PROD_1" | ["electronics", "sale"] | ["tech"] |
| "PROD_2" | null | ["books"] |
| "PROD_3" | ["new"] | null |

---

## 📊 **Summary Table**

| # | Transformation | Purpose | Creates New Column? |
|---|----------------|---------|---------------------|
| 1 | trim_strings | Remove spaces | No (modifies existing) |
| 2 | regex_clean | Clean with regex | Yes (creates "out" column) |
| 3 | parse_numeric | String → number | Yes (creates "out" column) |
| 4 | parse_date | String → date/timestamp | Yes (creates "out" column) |
| 5 | normalize_status | Standardize values | Yes (creates "out" column) |
| 6 | null_handling | Drop null rows | No (removes rows) |
| 7 | duplicate_handling | Remove duplicates | No (removes rows) |
| 8 | rename_columns | Rename columns | No (renames existing) |
| 9 | standardize_column_case | Standardize column names | No (renames existing) |
| 10 | fill_missing | Fill nulls | No (modifies existing) |
| 11 | surrogate_key | Generate UUID | Yes (creates new column) |
| 12 | outlier_capping | Cap min/max values | No (modifies existing) |
| 13 | fuzzy_normalize | Normalize text | Yes (creates "out" column) |
| 14 | flatten_json | Flatten nested JSON | Yes (creates flat columns) |
| 15 | schema_validation | Validate columns exist | No (validation only) |
| 16 | json_key_default | Add missing columns | Yes (creates new columns) |
| 17 | array_size_validation | Empty array → null | No (modifies existing) |

---

## ✅ **All 17 Transformations Documented!**

This is your complete transformation library! 🎯
