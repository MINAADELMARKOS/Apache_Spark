**Because Failing Loudly Is Not a Strategy**

Spark jobs don’t fail because data is perfect.

They fail because:

- Schemas drift
    
- Values are malformed
    
- External systems misbehave
    

If your Spark job crashes on bad data, **your design is fragile**.

---

## 🧠 First Principle: Spark Is Distributed

This means:

- One bad record should **not kill the entire job**
    
- Error handling must be **data-driven**, not try/except-driven
    

Classic `try/except` around Spark code ❌  
Partition-aware handling ✅

---

## 1️⃣ Row-Level Error Handling (Safe Parsing)

Bad input example:

- Invalid dates
    
- Corrupt numbers
    
- Broken JSON
    

### ❌ Dangerous approach

`df = spark.read.csv(path, header=True, inferSchema=True)`

If parsing fails → job fails.

---

### ✅ Safer approach (explicit parsing)

`from pyspark.sql.functions import col, to_timestamp  df = spark.read.csv(path, header=True)  df_parsed = df.withColumn(     "event_time",     to_timestamp(col("event_time"), "yyyy-MM-dd HH:mm:ss") )`

Invalid rows now become:  
👉 `NULL` — not crashes.

---

## 2️⃣ Splitting Valid vs Invalid Records (DLQ Pattern)

This is where **Dead Letter Queues** come in.

`valid_df = df_parsed.filter(col("event_time").isNotNull()) invalid_df = df_parsed.filter(col("event_time").isNull())`

You now have:

- ✅ Valid records → main pipeline
    
- ❌ Invalid records → DLQ
    

---

## 3️⃣ Writing to a Dead Letter Queue

A DLQ is just:

- A table
    
- A file path
    
- A Kafka topic
    

Example (DLQ to Parquet):

`invalid_df \   .withColumn("error_reason", lit("Invalid timestamp")) \   .write \   .mode("append") \   .parquet("/dlq/events/")`

Production rule:

> **Never drop bad data silently.**

Bad data is still **business evidence**.

---

## 4️⃣ Partition-Level Error Isolation (Advanced)

For complex logic:

`def safe_process(rows):     for r in rows:         try:             yield ("valid", r)         except Exception as e:             yield ("invalid", str(e), r)  rdd = df.rdd.mapPartitions(safe_process)`

This ensures:

- One broken record ≠ broken partition
    
- One broken partition ≠ broken job