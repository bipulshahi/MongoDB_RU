# **MongoDB Database Operations**

### *(Databases → Collections → Documents → Data Types → Naming → Basic Commands)*

MongoDB Database Operations include:

1. Creating / Switching Databases
2. Creating Collections
3. Viewing Collections
4. Understanding MongoDB Data Types (BSON Types)
5. Dropping Databases / Collections
6. Database-Level Commands
7. Schema & Naming Conventions
8. Case Study Hands-on Activities

---

# **1. Creating & Using a Database**

MongoDB creates the database only when you insert something into it.

### **Create / switch to a DB**

```js
use travelDB
```

### Check current DB:

```js
db
```

### List all databases:

```js
show dbs
```

### When is a DB *actually* created?

Only after first collection insert:

```js
db.trips.insertOne({ name: "Temp Trip" })
```

---

# **2. Creating Collections**

MongoDB creates a collection on first insert, or explicitly using `createCollection()`.

### **Implicit creation**

```js
db.trips.insertOne({ title: "Goa Beach Holiday" })
```

### **Explicit creation**

```js
db.createCollection("bookings")
```

### **View collections**

```js
show collections
```

Travel-app relevance:
Collections we will use soon:

* `trips`
* `users`
* `bookings`
* `reviews`
* `partners`

---

# **3. Checking Collection Statistics**

```js
db.trips.stats()
```

This shows:

* number of documents
* sizes
* storage engine details

Useful for capacity planning in a large travel app.

---

# **4. MongoDB (BSON) Data Types — Key Ones to Teach**

MongoDB stores data as **BSON (Binary JSON)**.

### Important types (with travel-app examples):

| Type                                | Example                     | Use in Travel App   |
| ----------------------------------- | --------------------------- | ------------------- |
| **String**                          | `"Goa"`                     | destination         |
| **NumberInt / NumberLong / Double** | `12999`                     | price               |
| **Boolean**                         | `true`                      | isActive trip       |
| **Date**                            | `ISODate("2026-01-10")`     | travel dates        |
| **ObjectId**                        | `ObjectId("...")`           | default `_id`       |
| **Array**                           | `["breakfast", "pickup"]`   | features            |
| **Embedded Document**               | `{ avg: 4.6, count: 245 }`  | ratings             |
| **Null**                            | `null`                      | missing values      |
| **BinData**                         | binary                      | storing ticket PDFs |
| **Decimal128**                      | `NumberDecimal("12999.50")` | precise pricing     |

---

# **5. Dropping Databases & Collections**

### Drop collection

```js
db.trips.drop()
```

### Drop database

```js
db.dropDatabase()
```

Travel-app example:
You imported the wrong `partners.csv` file → drop the collection and re-import.

---

# **6. Useful Database-Level Commands**

### Check server info

```js
db.serverStatus()
```

### Check DB stats

```js
db.stats()
```

### Get current user's permissions (optional topic)

```js
db.runCommand({ connectionStatus: 1 })
```

### Validate a collection

```js
db.runCommand({ validate: "trips" })
```

---

# **7. Naming Conventions (VERY Important for Beginners)**

### **Database names**

* lowercase
* no spaces
* example: `travelDB`

### **Collection names**

* lowercase
* plural nouns recommended
* example: `trips`, `users`, `bookings`

### **Field names**

* camelCase
* avoid spaces
* avoid starting with `$`
* example: `startDate`, `tripType`, `avgRating`

### **Document structure**

* Always predictable
* Prefer consistent fields
* Avoid very deep nesting


---

## **Activity A — Create the Database Structure**

### Step 1: Switch to DB

```js
use travelDB
```

### Step 2: Create collections

```js
db.createCollection("trips")
db.createCollection("users")
db.createCollection("bookings")
db.createCollection("reviews")
db.createCollection("partners")
```

### Step 3: Verify

```js
show collections
```

---

## **Activity B — Insert Data to Create DB**

(implicit collection creation demo)

```js
db.trips.insertOne({
  title: "Manali Adventure Trek",
  category: "mountain",
  price: 12999,
  destination: "Manali",
  startDate: ISODate("2026-04-10"),
  endDate: ISODate("2026-04-15"),
  features: ["camping", "guide", "meals"],
  ratings: { avg: 4.5, count: 150 },
  availability: 20
})
```

Explain:
**The moment this insert happens, the `travelDB` is officially created.**

---

## **Activity C — Explore Data Types**

```js
db.trips.findOne()
```

Good for understanding BSON.

---

## **Activity D — Drop & Recreate Collections**

1. Drop an incorrect collection:

```js
db.partners.drop()
```

2. Recreate it:

```js
db.createCollection("partners")
```

3. Import CSV using `mongoimport` (from previous topic).

---

# **Mini Quiz (Database Operations)**

**Q1 (Theory):**
Why doesn’t MongoDB create a database immediately when you run `use travelDB`?

- use travelDB only switches your shell’s current database context to travelDB. MongoDB only actually creates a database when you perform a write that persists something (create a collection, insert a document, or create an index). Until a write happens, there’s nothing on disk for MongoDB to create.

**Q2 (Practical):**
Write a command to create the collection `offers` explicitly.

- db.createCollection("offers")

**Q3 (Practical):**
You mistakenly inserted wrong trips into the collection. Write the command to remove the entire collection.

- db.trips.drop()
- db.trips.deleteMany({})
- db.trips.deleteOne({
  title: "Kerala Backwater Cruise",
  price: NumberDecimal("24999.50"),
  startDate: ISODate("2026-08-10")
})


**Q4 (Hybrid):**
Look at this document and identify at least 3 BSON data types used:

```js
{
  title: "Kerala Backwater Cruise",
  price: NumberDecimal("24999.50"),
  startDate: ISODate("2026-08-10"),
  features: ["houseboat", "dinner"],
  ratings: { avg: 4.7, count: 322 }
}
```

- Here are at least 3 BSON data types present in the document:

String
Example: "Kerala Backwater Cruise", "houseboat", "dinner"

Decimal128
Example: NumberDecimal("24999.50")

Date
Example: ISODate("2026-08-10")

Additional BSON types also present:

Array
Example: features: ["houseboat", "dinner"]

Embedded Document (Object)
Example: ratings: { avg: 4.7, count: 322 }

Double
Example: avg: 4.7

Integer (int32/int64)
Example: count: 322
