## Understanding Distributed Processing (With Real Code)

Most Spark problems don’t come from Spark itself.

They come from **thinking locally while running distributed**.

Let’s fix that.

---

## 🧠 The Core Idea of Distributed Processing

Spark does **not** run your code once.

Spark runs your code:

- On **many partitions**
    
- In **parallel**
    
- Across **multiple executors**
    

If you don’t understand this, you’ll:

- Break performance
    
- Break correctness
    
- Break scalability
    

---

## 1️⃣ Data Is Split First, Logic Comes Second

When Spark reads data, it **splits it into partitions**.

Example:

`df = spark.read.parquet("/data/transactions") print(df.rdd.getNumPartitions())`

Spark now sees:  
👉 _Many small chunks of data_  
👉 Not “one DataFrame”

Each partition will be processed **independently**.

---

## 2️⃣ Your Code Runs Per Partition (Not Once)

This is the biggest mindset shift.

Example:

`df.filter(df.amount > 100)`

Spark does NOT:

- Loop over rows centrally
    

Spark DOES:

- Send this filter logic to **each executor**
    
- Apply it on **each partition**
    

Same code.  
Many parallel executions.

---

## 3️⃣ Actions Trigger Distributed Execution

Until you call an action:

`df.count()`

Nothing runs.

Once you do:

- Spark creates jobs
    
- Splits work into tasks
    
- Executes tasks per partition
    

Each task:

- Runs on one executor core
    
- Processes one partition
    

---

## 4️⃣ Common Distributed Mistake 🚨

Trying to use driver-side logic.

❌ Wrong:

`total = 0 for row in df.collect():     total += row.amount`

✔ Correct:

`df.groupBy().sum("amount")`

Why?

- `collect()` pulls data to the driver
    
- Distributed systems die when centralized
    

---

## 5️⃣ Partitioning Controls Parallelism

Parallelism ≈ number of partitions.

`df.repartition(100)`

Too few partitions:

- CPUs sit idle
    

Too many partitions:

- Overhead dominates
    

Distributed processing is a **balancing act**.