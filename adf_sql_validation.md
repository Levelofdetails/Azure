


# ⭐ 1. ADF SQL Validation Best‑Practices Checklist  
Use this before running any Lookup / dynamic SQL pipeline.

## ✅ **SQL Syntax Validation**
- Always test the final SQL in **SSMS** or **Azure Query Editor**.  
- Replace dynamic parameters with sample values and run the query manually.  
- Validate schema + table existence using `INFORMATION_SCHEMA.TABLES`.  
- Avoid reserved keywords as table names (use `[Order]`, `[User]`, etc.).  

## ✅ **ADF Expression Validation**
- Use `concat()` instead of string concatenation to avoid missing spaces.  
- Always add a space before dynamic table names.  
- Validate dynamic expressions using the ADF **Expression Builder**.  
- Use a **Set Variable** activity to print the final SQL for debugging.  

## ✅ **Pipeline Debugging**
- Run Lookup alone in **Debug mode** and inspect the **Output** tab.  
- Ensure Lookup returns the expected JSON structure (`firstRow`, `value`).  
- Validate that parameters are not null or empty.  

## ✅ **Data Quality & Governance**
- Validate row counts before processing.  
- Validate schema drift if using Mapping Data Flows.  
- Log SQL queries, table names, and timestamps for audit.  

---

# ⭐ 2. ADF Dynamic SQL Validator (Ready-to-Use)

### **Lookup Query (validates table existence)**
```sql
SELECT TABLE_SCHEMA, TABLE_NAME
FROM INFORMATION_SCHEMA.TABLES
WHERE TABLE_NAME = '@{pipeline().parameters.TableName}'
```

### **Dynamic SQL Builder**
Use a Set Variable activity:

```json
@concat(
    'SELECT * FROM ',
    pipeline().parameters.SchemaName,
    '.',
    pipeline().parameters.TableName
)
```

### **Debug SQL Printer**
Set Variable → `DebugSQL`:

```json
@activity('Lookup_TableCheck').output.firstRow.TableName
```

### **Fail Pipeline if Table Not Found**
Add an If Condition:

```json
@equals(activity('Lookup_TableCheck').output.count, 0)
```

Inside True → Fail activity:

```
Table does not exist. Validation failed.
```

---

# ⭐ 3. ADF SQL Logging Framework (Enterprise-Ready)

### **Log Structure (JSON)**
```json
{
  "PipelineName": "@{pipeline().Pipeline}",
  "ActivityName": "@{activity().Activity}",
  "SQLQuery": "@{variables('DebugSQL')}",
  "TableName": "@{pipeline().parameters.TableName}",
  "Timestamp": "@{utcNow()}",
  "Status": "Validated"
}
```

### **Sink: ADLS Gen2 → Raw Logs Folder**
Path example:

```
/logs/sql-validation/year=/month=/day=/hour=
```

### **ADF Activity to Write Logs**
Use **Copy Activity**  
Source: Inline JSON  
Sink: ADLS → Append Blob or JSON file

---

# ⭐ 4. ADF Validation Pipeline Template (Step-by-Step)

## **Pipeline: Validate_SQL_And_Table**

### **Parameters**
- `SchemaName`
- `TableName`

### **Activities**
#### **1. Lookup → Validate Table Exists**
Runs:

```sql
SELECT TABLE_SCHEMA, TABLE_NAME
FROM INFORMATION_SCHEMA.TABLES
WHERE TABLE_NAME = '@{pipeline().parameters.TableName}'
```

#### **2. If Condition → Table Exists?**
Expression:

```json
@equals(activity('Lookup_TableCheck').output.count, 1)
```

**False branch → Fail Pipeline**

#### **3. Set Variable → Build SQL**
```json
@concat(
    'SELECT * FROM ',
    pipeline().parameters.SchemaName,
    '.',
    pipeline().parameters.TableName
)
```

#### **4. Set Variable → Debug SQL Printer**
Stores final SQL for logging.

#### **5. Copy Activity → Log SQL to ADLS**
Writes JSON log.

#### **6. Success → Trigger Main Pipeline**
Use Execute Pipeline activity.

---

# ⭐ 5. PDF‑Ready Best Practices Summary (Brochure Style)

Below is a clean, formatted version you can paste into Word/Canva and export as PDF.

---

## **Azure Data Factory – SQL Validation Best Practices**  
### *KamalaTeck | Enterprise Data Engineering Standards*

---

### **1. SQL Syntax Validation**
- Validate SQL in SSMS before using in ADF  
- Replace dynamic parameters with sample values  
- Validate schema and table existence  
- Avoid reserved keywords  

---

### **2. ADF Expression Best Practices**
- Use `concat()` for safe string building  
- Always include spaces in dynamic SQL  
- Validate expressions using Expression Builder  
- Print SQL using Set Variable for debugging  

---

### **3. Pipeline Debugging**
- Run Lookup in Debug mode  
- Inspect Output JSON  
- Validate parameter values  
- Ensure Lookup returns expected structure  

---

### **4. Data Quality & Governance**
- Validate row counts  
- Validate schema drift  
- Log SQL queries and metadata  
- Maintain audit logs in ADLS  

---

### **5. Validation Pipeline Framework**
- Lookup → Validate table  
- If Condition → Fail early  
- Build SQL dynamically  
- Log SQL to ADLS  
- Trigger main pipeline  

---



Just tell me what you want next.
