
**Arrays, Structs, Maps (Where Real Data Lives)**

Real data is **not flat**.

If your Spark skills stop at simple columns,  
you’ll struggle with:

- JSON
    
- Events
    
- APIs
    
- Logs
    

Let’s fix that.

---

## 🧠 Spark’s Complex Data Types

Spark supports:

- `struct`
    
- `array`
    
- `map`
    

These allow Spark to model **nested data natively**.

No flattening required (unless you choose to).

---

## 1️⃣ Struct Type (Nested Columns)

Example schema:

`{   "user": {     "id": 123,     "country": "DE"   } }`

Spark code:

`df.select("user.id", "user.country")`

Structs behave like:  
👉 Nested objects, not strings

---

## 2️⃣ Array Type (Repeated Values)

Example:

`"items": ["A", "B", "C"]`

### Exploding Arrays

`from pyspark.sql.functions import explode  df.select(explode("items").alias("item"))`

This turns:

- One row → many rows
    

---

## 3️⃣ Map Type (Key-Value Data)

Example:

`"attributes": {   "os": "android",   "version": "14" }`

Access map values:

`df.select(df.attributes["os"].alias("os"))`

Maps are perfect for:

- Dynamic attributes
    
- Semi-structured data
    

---

## 4️⃣ Combining Complex Types (Realistic Example)

`df.select(     "user.id",     explode("orders").alias("order") ).select(     "id",     "order.amount",     "order.currency" )`

This pattern appears **everywhere** in event pipelines.