

**When Spark Saves Work (and When It Backfires)**

One of the most dangerous Spark myths:

> “Caching always makes Spark faster.”

Sometimes it does.  
Sometimes it makes things worse.

---

## 🧠 What Caching Really Does

Caching tells Spark:

👉 “If this DataFrame is reused, don’t recompute it.”

Instead of recomputing the entire lineage:

- Spark stores the data
    
- Reuses it for future actions
    

---

## Example

`df = spark.read.parquet("transactions") \      .filter("amount > 100")  df.cache()  df.count() df.groupBy("customer_id").sum("amount").show()`

Without caching:

- Filter runs twice
    

With caching:

- Filter runs once
    
- Results reused
    

---

## Storage Levels (Important)

Spark can store cached data in:

- Memory only
    
- Memory + disk
    
- Disk only
    
- Serialized formats
    

Choosing the wrong level can:

- Evict useful data
    
- Cause spills
    
- Increase GC pressure
    

---

## When Caching Helps ✅

- Reused DataFrames
    
- Iterative algorithms
    
- Multiple actions on same dataset
    

---

## When Caching Hurts ❌

- One-time queries
    
- Huge datasets that don’t fit in memory
    
- Cached data never reused
    

⚠️ Cached data competes with:

- Execution memory
    
- Shuffle buffers
    

---

## Common Mistake 🚨

Caching **everything** “just in case”.

Spark is lazy — but memory is not infinite.

---

## 🎯 Key Takeaway

> Cache **intentionally**, not emotionally.

If it’s reused → consider caching  
If it’s not → don’t touch it