
**When (and When NOT) to Use Them**

Broadcast variables and accumulators are powerful.

Misused?  
👉 They silently destroy performance.

---

## 🧠 Broadcast Variables – What They Really Are

A broadcast variable:

- Is **read-only**
    
- Is sent **once per executor**
    
- Avoids repeated data transfer
    

Perfect for:

- Small lookup tables
    
- Reference data
    
- Dimension mappings
    

---

## 1️⃣ Broadcast Variable Example

`lookup = {     "A": "Premium",     "B": "Standard",     "C": "Basic" }  broadcast_lookup = spark.sparkContext.broadcast(lookup)`

Use it inside transformations:

`from pyspark.sql.functions import udf  def map_category(code):     return broadcast_lookup.value.get(code, "UNKNOWN")  map_udf = udf(map_category)  df.withColumn("category_name", map_udf(col("category_code")))`

---

## 2️⃣ When Broadcasting Is a BAD Idea 🚨

❌ Large datasets  
❌ Frequently changing data  
❌ Writing to broadcast (impossible)

Rule of thumb:

> If it doesn’t fit comfortably in executor memory — don’t broadcast it.

---

## 3️⃣ Broadcast vs Broadcast Join (Important Distinction)

Broadcast variable:

- Manual
    
- Used inside UDFs
    

Broadcast join:

- Spark-level optimization
    
- Handled by Catalyst
    

`from pyspark.sql.functions import broadcast  df.join(broadcast(dim_df), "key")`

Let Spark handle joins when possible.

---

## 4️⃣ Accumulators – What They Are (and Aren’t)

Accumulators are:

- Write-only from executors
    
- Readable only on the driver
    
- Used for **metrics**, not logic
    

Example:

`error_counter = spark.sparkContext.accumulator(0)`

---

## 5️⃣ Accumulator Usage Example

`def validate(row):     if row.amount < 0:         error_counter.add(1)     return row  df.foreach(validate)  print("Invalid rows:", error_counter.value)`

This works because:

- Accumulators aggregate across executors
    
- Spark handles retries safely
    

---

## 6️⃣ Common Accumulator Mistakes 🚨

❌ Using accumulator values to control logic  
❌ Assuming exact counts with retries  
❌ Expecting real-time updates