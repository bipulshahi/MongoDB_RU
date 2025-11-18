# **MongoDB CRUD Operations: INSERT Methods**

MongoDB provides four main ways to insert data:

1. **insertOne()**
2. **insertMany()**
3. **Bulk Write Inserts (bulkWrite)**
4. **Import-based Insert (mongoimport)** — already covered earlier but included as a fifth method practically

We’ll use your **Travel App Case Study** for all examples.

---

# **1. insertOne() — Insert a Single Document**

### **Syntax**

```js
db.collection.insertOne(document)
```

### **Case Study Example**

Add a new trip package:

```js
db.trips.insertOne({
  title: "Goa Beach Holiday - 5 Days",
  category: "beach",
  price: 14999,
  destination: "Goa",
  availability: 20,
  features: ["breakfast", "airport pickup", "sightseeing"],
  createdAt: ISODate()
})
```

### **Returns**

A result with:

* acknowledged: true
* insertedId: ObjectId(...)

---

# **2. insertMany() — Insert Multiple Documents**

Useful when onboarding many travel packages at once.

### **Syntax**

```js
db.collection.insertMany([doc1, doc2, doc3])
```

### Case Study Example

Insert multiple trip packages:

```js
db.trips.insertMany([
  {
    title: "Manali Snow Adventure",
    category: "mountain",
    price: 12999,
    destination: "Manali",
    availability: 15,
    features: ["skiing", "camping"],
    createdAt: ISODate()
  },
  {
    title: "Andaman Island Escape",
    category: "beach",
    price: 29999,
    destination: "Andaman",
    availability: 10,
    features: ["snorkeling", "coral tour"],
    createdAt: ISODate()
  }
])
```

### Ordered vs Unordered Inserts

By default inserts are **ordered** — if one fails, the rest stop.

Set `ordered: false` to allow skipping errors:

```js
db.trips.insertMany(docs, { ordered: false })
```

---

# **3. bulkWrite() — High-Performance Bulk Inserts**

For large enterprise applications (like a travel site doing large partner imports), bulkWrite is preferred.

### **Syntax**

```js
db.collection.bulkWrite([
  { insertOne: { document: {...} } },
  { insertOne: { document: {...} } }
])
```

### Case Study Example

```js
db.partners.bulkWrite([
  {
    insertOne: {
      document: {
        name: "HolidayHotels",
        type: "hotel",
        rating: 4.2
      }
    }
  },
  {
    insertOne: {
      document: {
        name: "SkyHigh Flights",
        type: "airline partner",
        rating: 4.5
      }
    }
  }
])
```

### Why use bulkWrite?

* Better performance
* Useful for big imports (hotels, airlines, transport partners)

---

# **4. Insert with auto-generated ObjectId**

MongoDB automatically adds `_id` if you don’t provide one.

```js
db.users.insertOne({
  name: "Aditi Sharma",
  email: "aditi@example.com"
})
```

MongoDB adds:

```js
"_id": ObjectId("...")
```

---

# **5. Insert with Custom _id (Important Concept)**

You can define your own unique ID.

### Example:

Unique trip code as `_id`:

```js
db.trips.insertOne({
  _id: "TRIP-GOA-2025-01",
  title: "Goa Beach Chill",
  price: 12999
})
```

Avoid using:

* duplicate IDs
* very long IDs
* changing IDs later

---

# **6. insert with Validation Errors**

If a schema validator is applied, invalid documents will fail.

### Example:

Suppose a validator requires `price` to be a number.
Try inserting incorrect data:

```js
db.trips.insertOne({
  title: "Bad Trip",
  price: "free"
})
```

MongoDB will reject it if validation rules exist.

---

# **7. Using mongoimport for CSV/JSON Inserts (Quick Revision)**

### JSON Import:

```bash
mongoimport --db travelDB --collection trips --file trips.json --jsonArray
```

### CSV Import:

```bash
mongoimport --db travelDB --collection trips --type=csv --headerline --file trips.csv
```

This is internally equivalent to many insert operations.

---

# **8. Hands-On Class Activity (Highly Interactive)**

### **Activity A: Create Trips**

Ask students to add 3 trips using `insertOne()`:

* 1 beach
* 1 mountain
* 1 adventure

### **Activity B: Bulk Insert Trips**

Insert at least 5 trips with:

```js
db.trips.insertMany([...])
```

### **Activity C: Partner Bulk Import**

Students should simulate partner onboarding:

```js
db.partners.bulkWrite([...])
```

### **Activity D: Create a user**

```js
db.users.insertOne({
  _id: "USR1001",
  name: "Rohit Kumar",
  email: "rohit@example.com",
  preferredDestinations: ["Goa", "Manali"]
})
```

---

# **9. Common Mistakes to Teach**

| Mistake                                        | Why It’s Wrong         | Correct Approach        |
| ---------------------------------------------- | ---------------------- | ----------------------- |
| insertOne({}) with missing required fields     | Bad schema consistency | Use schema validation   |
| insertMany([...]) without `{ ordered: false }` | Stops at first error   | Use unordered inserts   |
| Very large documents                           | 16MB document limit    | Break into smaller docs |
| Using wrong types (price as string)            | Breaks queries         | Always validate         |

---

# **10. Mini Quiz (Insert Operations)**

**Q1 (Theory):**
What is the difference between `insertOne()` and `insertMany()` in terms of execution flow?

**Q2 (Theory):**
Why is `bulkWrite()` preferred for large datasets?

**Q3 (Practical):**
Write a command to insert a new user:

* name: "Pooja Singh"
* email: "[pooja@travel.com](mailto:pooja@travel.com)"
* favouriteCategory: "beach"

**Q4 (Practical):**
Write a command to insert 3 trips at once using `insertMany()`.
