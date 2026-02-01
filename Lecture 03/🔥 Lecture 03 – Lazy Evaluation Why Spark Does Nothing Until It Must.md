

One of the most confusing things for new Spark users is this feeling:

> “I wrote code… but Spark didn’t do anything.”

That’s not a bug.  
That’s **Lazy Evaluation** — and it’s one of Spark’s biggest strengths.

---

## 🧠 What Lazy Evaluation Really Means

In Spark:

👉 **Transformations do NOT execute immediately**  
👉 Spark only builds a _logical plan_  
👉 Actual execution happens **only when Spark must return a result**

Until then, Spark is just _planning_.

---

## Example (Real Life)

`df = spark.read.parquet("transactions") df2 = df.filter("amount > 100") df3 = df2.groupBy("customer_id").sum("amount")`

At this point:

❌ No data read  
❌ No filtering  
❌ No aggregation

Spark is doing **nothing** — intentionally.

---

## When Does Spark Finally Execute?

Only when you call an **action**, such as:

- `show()`
    
- `count()`
    
- `collect()`
    
- `write()`
    

Example:

`df3.show()`

💥 Now Spark:

- Builds the execution plan
    
- Optimizes it
    
- Launches jobs
    
- Runs tasks across the cluster
    

---

## Why Lazy Evaluation Is Powerful

Lazy evaluation allows Spark to:

- Optimize the full query (not step by step)
    
- Reorder operations
    
- Push filters down
    
- Remove unnecessary work
    

Spark doesn’t rush.  
It waits until it sees **the whole picture**.

---

## 🎯 Key Takeaway

> Spark doesn’t execute code line by line.  
> Spark executes **results**, not intentions.

If nothing happens — ask yourself:  
👉 _Did I trigger an action?_

---

## 🎨 Visual Prompt

_A Spark DAG forming gradually with transformations in gray, then lighting up only when an action is triggered._

---

---

# 🔥 Lecture 04 – Transformations vs Actions (Why Jobs Suddenly Explode)

Ever noticed this?

You write **one small line of code**…  
and suddenly Spark launches **dozens of jobs**.

That’s not random.  
That’s **Transformations vs Actions**.

---

## 🧩 Transformations (Spark Is Still Calm)

Transformations:

- Define _what_ you want
    
- Do NOT execute
    
- Are lazily evaluated
    

Examples:

- `select`
    
- `filter`
    
- `withColumn`
    
- `groupBy`
    
- `join`
    

They only modify the **logical plan**.

---

## ⚡ Actions (Spark Goes to Work)

Actions:

- Force execution
    
- Return data or write results
    
- Trigger jobs, stages, tasks
    

Examples:

- `show`
    
- `count`
    
- `collect`
    
- `write`
    
- `foreach`
    

This is where Spark **must deliver a result**.

---

## Why Jobs “Explode”

One action can trigger:

- Multiple stages
    
- Multiple shuffles
    
- Hundreds of tasks
    

Because Spark executes **everything needed** to reach that action.

Not just the last line —  
**the entire lineage**.

---

## Common Mistake 🚨

Calling multiple actions:

`df.count() df.show() df.write.parquet("output")`

That’s **three executions** — unless cached.

---

## 🎯 Key Takeaway

> Transformations describe the plan.  
> Actions force Spark to pay the bill.

Know where your actions are — they define **performance, cost, and runtime**.