We will cover:

1. `findOne()`
2. `find()`
3. Projections (include/exclude fields)
4. Comparision operators ($gt, $lt, $eq…)
5. Logical operators ($and, $or, $not, $nor)
6. Array queries ($in, $all, $elemMatch)
7. Regex searches
8. Sorting, Pagination, Limiting, Skipping
9. Querying Nested & Embedded Documents
10. Travel-app-based hands-on tasks
11. Mini quiz

---

# **MongoDB FIND Methods**

We'll use the **travelDB** we’ve already built:

* `trips`
* `users`
* `bookings`

---

# **1. findOne() — Get Exactly One Document**

### Syntax:

```js
db.collection.findOne(query)
```

### Case Study Example:

Find any one beach trip:

```js
db.trips.findOne({ category: "beach" })
```

---

# **2. find() — Retrieve Multiple Documents**

### Syntax:

```js
db.collection.find(query)
```

### Example: Find all Goa trips

```js
db.trips.find({ destination: "Goa" })
```

---

# **3. Projections — Select Only Required Fields**

### Syntax:

```js
db.collection.find(query, projection)
```

### Include fields:

```js
db.trips.find(
  { destination: "Goa" },
  { title: 1, price: 1, _id: 0 }
)
```

### Exclude fields:

```js
db.trips.find(
  { category: "beach" },
  { features: 0 }
)
```

---

# **4. Comparison Operators**

| Operator | Meaning                  |
| -------- | ------------------------ |
| `$gt`    | greater than             |
| `$lt`    | less than                |
| `$gte`   | greater than or equal    |
| `$lte`   | less than or equal       |
| `$ne`    | not equal                |
| `$eq`    | equal (default behavior) |

### **Examples (Travel App):**

1. Trips cheaper than ₹15,000:

```js
db.trips.find({ price: { $lt: 15000 } })
```

2. Trips priced between ₹10,000 and ₹20,000:

```js
db.trips.find({
  price: { $gte: 10000, $lte: 20000 }
})
```

3. Trips not in Manali:

```js
db.trips.find({ destination: { $ne: "Manali" } })
```

---

# **5. Logical Operators**

### `$and`

```js
db.trips.find({
  $and: [
    { category: "mountain" },
    { price: { $lt: 15000 } }
  ]
})
```

### `$or`

```js
db.trips.find({
  $or: [
    { category: "beach" },
    { category: "adventure" }
  ]
})
```

### `$nor` (none of the conditions)

```js
db.trips.find({
  $nor: [
    { destination: "Goa" },
    { category: "adventure" }
  ]
})
```

---

# **6. Array Queries**

### `$in`

Find all trips whose category is in a list:

```js
db.trips.find({
  category: { $in: ["beach", "mountain"] }
})
```

### `$all`

Find trips that include **all** these features:

```js
db.trips.find({
  features: { $all: ["breakfast", "airport pickup"] }
})
```

### `$elemMatch`

Find trips where a feature matches a condition (useful for embedded arrays of objects).

Example (if ratings were stored as an array of rating objects):

```js
db.trips.find({
  ratingsList: {
    $elemMatch: { stars: { $gte: 4 } }
  }
})
```

---

# **7. Sorting, Limiting, Skipping, Pagination**

### **Sort**

```js
db.trips.find().sort({ price: 1 })   // ascending
db.trips.find().sort({ price: -1 })  // descending
```

### **Limit**

```js
db.trips.find().limit(5)
```

### **Skip**

```js
db.trips.find().skip(10)
```

### **Pagination Formula**

```js
db.trips.find().skip((page - 1) * limit).limit(limit)
```

---

# **8. Querying Embedded Documents**

### Embedded Object Example:

```js
ratings: { avg: 4.5, count: 150 }
```

Find trips with rating above 4.6:

```js
db.trips.find({ "ratings.avg": { $gt: 4.6 } })
```

Find trips with more than 200 reviews:

```js
db.trips.find({ "ratings.count": { $gt: 200 } })
```

---

# **10. Hands-On Activities for Students**

### **Activity A: Simple Filtering**

Find:

1. All adventures
2. All trips under ₹20,000
3. All Goa trips with availability > 10

### **Activity B: Projection Tasks**

1. Show only `title` and `price` for all beach trips
2. Hide `_id` field and show only destination names

### **Activity C: Sorting & Pagination**

1. Show top 5 cheapest trips
2. Show most expensive trips first
3. Display page 2 with limit 3

### **Activity D: Regex**

1. Find all trips containing “Island”
2. Find all trips whose destination starts with “M”

### **Activity E: Combined Query**

Find all beach trips:

* priced between 10k and 20k
* availability > 0
* sorted by price low-to-high
* only title and price

---

# **11. Mini Quiz (FIND Methods)**

**Q1 (Theory):**
What is the difference between `findOne()` and `find()`?

**Q2 (Theory):**
Explain the difference between `$in` and `$all` with examples.

**Q3 (Practical):**
Write a query to find all mountain trips costing less than ₹18,000.

**Q4 (Practical):**
Write a query to sort all beach trips by price descending and show only title and price.

**Q5 (Hybrid):**
A student runs this query but gets no results:

```js
db.trips.find({ "ratings.avg": 4.5 })
```

Explain why, and rewrite the correct query to find trips with rating ≥ 4.5.

