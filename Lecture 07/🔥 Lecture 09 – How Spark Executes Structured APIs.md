
**(DataFrame, Dataset, SQL → One Execution Path)**

One of the most powerful ideas in Spark is also one of the most misunderstood:

> **It does NOT matter whether you write DataFrame, Dataset, or SQL code.**  
> Spark executes all of them through the **same internal pipeline**.

Different syntax.  
Same optimizer.  
Same execution engine.

Let’s walk through what actually happens — using the diagrams.

---

## 1️⃣ You Write Structured Code (User Code)

You start by writing **structured APIs**:

- DataFrame API
    
- Dataset API
    
- Spark SQL
    

This is shown clearly in **Figure 4-1**:

- SQL
    
- DataFrames
    
- Datasets
    

All of them feed into the **Catalyst Optimizer**.

At this point:

❌ No data is read  
❌ No executors are used  
❌ No computation happens

Spark is only validating:  
👉 _Is this syntactically and logically correct?_


![[Figure 4-1.jpeg]]

---

## 2️⃣ Spark Builds an _Unresolved Logical Plan_

If the code is valid, Spark converts it into an **Unresolved Logical Plan**.

This is shown in **Figure 4-2**:

`User Code → Unresolved Logical Plan`

Why “unresolved”?

Because Spark still doesn’t know:

- What exact columns mean
    
- What tables exist
    
- What data types are involved
    

At this stage:

- Column names are just strings
    
- Tables are just references
    

Still **no execution**.


![[Figure 4-2.jpeg]]

---

## 3️⃣ Analysis Phase – Resolving the Plan Using the Catalog

Next comes **Analysis**.

Spark uses the **Catalog** (metastore, schemas, table definitions) to:

- Resolve column names
    
- Resolve table references
    
- Validate data types
    

This transforms the plan into a:

👉 **Resolved Logical Plan**

You can see this clearly in **Figure 4-2**:

`Unresolved Logical Plan         ↓ (Analysis + Catalog) Resolved Logical Plan`

If something is wrong here (missing column, wrong type):  
👉 Spark fails **before execution**.

---

## 4️⃣ Logical Optimization – Making the Plan Smarter

Now Spark applies **logical optimizations**.

Still no execution.

Spark:

- Pushes filters down
    
- Removes unused columns
    
- Reorders operations
    
- Simplifies expressions
    

This results in an:

👉 **Optimized Logical Plan**

Still:  
❌ No cluster work  
❌ No RDDs  
❌ No tasks

This is **what** Spark wants to do — not **how**.

---

## 5️⃣ Physical Planning – Choosing _How_ to Execute

Now we move to **Figure 4-3** (Physical Planning).

Spark takes the optimized logical plan and asks:

> “What are the possible physical ways to run this?”

Spark:

- Generates **multiple physical plans**
    
- Compares them using a **cost model**
    
- Chooses the **best physical plan**
    

Examples of decisions here:

- Broadcast join vs shuffle join
    
- Hash aggregation vs sort aggregation
    
- Scan strategy
    

This is where **performance is decided**.

---
![[Figure 4-3.jpeg]]
## 6️⃣ Spark Executes the Physical Plan (RDD-Level Execution)

Here’s the key truth many people miss:

> **All Structured APIs are executed as RDD operations internally.**

Once the physical plan is chosen:

- Spark breaks it into stages
    
- Creates tasks per partition
    
- Executes work on executors
    

This is:  
👉 **RDD manipulation on the cluster**

That’s why:

- Shuffles still exist
    
- Partitions still matter
    
- Spark UI shows tasks and stages
    

Structured APIs are **optimized abstractions**, not magic.

