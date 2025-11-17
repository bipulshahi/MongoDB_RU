**MongoDB UPDATE Methods**

1. updateOne()
2. updateMany()
3. replaceOne()
4. Important update operators:

   * **$set**, **$unset**, **$inc**, **$mul**, **$rename**
   * **$push**, **$addToSet**, **$pull**, **$pop**

5. Array update operations
6. Upserts
7. Case study examples
8. Hands-on tasks
9. Mini quiz

# **MongoDB UPDATE Methods**

Assume we are using your travelDB and the `trips`, `users`, `bookings`, `partners` collections.

---

# **1. updateOne() — Update a Single Matching Document**

### Syntax:

```js
db.collection.updateOne(filter, update)
```

---

### **Case Study Example — Update price of a Goa trip**

```js
db.trips.updateOne(
  { destination: "Goa" },
  { $set: { price: 15999 } }
)
```

### Update availability after one booking:

```js
db.trips.updateOne(
  { _id: ObjectId("...") },
  { $inc: { availability: -1 } }
)
```

---

# **2. updateMany() — Update Multiple Documents**

Used when many trips need price correction, availability update, or new feature addition.

### Example — Increase price of all beach trips by ₹500:

```js
db.trips.updateMany(
  { category: "beach" },
  { $inc: { price: 500 } }
)
```

### Offer: Add “free breakfast” to all mountain trips:

```js
db.trips.updateMany(
  { category: "mountain" },
  { $addToSet: { features: "free breakfast" } }
)
```

---

# **3. replaceOne() — Replace Entire Document (Full Replacement)**

Important: **Everything is overwritten except `_id`.**

### Example — Replace a trip completely

```js
db.trips.replaceOne(
  { title: "Old Trip" },
  {
    title: "Updated Trip",
    category: "adventure",
    price: 24999,
    destination: "Rishikesh",
    availability: 30
  }
)
```

Warning:
Do **not** use this unless you want to replace the entire document.

---

# **4. Key Update Operators (Very Important)**

---

## **A. $set — Add or update fields**

```js
db.trips.updateOne(
  { destination: "Manali" },
  { $set: { featured: true, discount: 1000 } }
)
```

---

## **B. $unset — Remove a field**

```js
db.trips.updateOne(
  { destination: "Goa" },
  { $unset: { discount: "" } }
)
```

---

## **C. $inc — Increment or decrement a number**

Increase price by 10%:

```js
db.trips.updateOne(
  { destination: "Goa" },
  { $inc: { price: price * 0.10 } }   // Usually you calculate percentage in app
)
```

Reduce availability by 1:

```js
$inc: { availability: -1 }
```

---

## **D. $mul — Multiply values**

Example: Holiday sale → 20% off:

```js
db.trips.updateMany(
  { category: "beach" },
  { $mul: { price: 0.8 } }
)
```

---

## **E. $rename — Rename fields**

Rename “start_date” to “startDate”:

```js
db.trips.updateMany(
  {},
  { $rename: { start_date: "startDate" } }
)
```

---

# **5. Updating Array Fields**

Travel app commonly uses arrays: `features`, `tags`, etc.

---

## **$push — Add item to array**

```js
db.trips.updateOne(
  { destination: "Goa" },
  { $push: { features: "sunset cruise" } }
)
```

---

## **$addToSet — Add only if value doesn’t exist**

```js
db.trips.updateOne(
  { destination: "Goa" },
  { $addToSet: { tags: "popular" } }
)
```

---

## **$pull — Remove specific value**

Remove “camping” from features:

```js
db.trips.updateOne(
  { destination: "Manali" },
  { $pull: { features: "camping" } }
)
```

---

## **$pop — Remove first or last element**

```js
$pop: { features: -1 }  // remove first
$pop: { features: 1 }   // remove last
```

---

# **6. Upserts — Update if exists, Insert if not**

Add a new partner, or update if exists:

```js
db.partners.updateOne(
  { name: "HolidayHotels" },
  { $set: { rating: 4.3 } },
  { upsert: true }
)
```

MongoDB will:

* update if exists
* insert if not

---

# **7. Travel-App Based Case Study Updates**

### **A. Apply a festival discount**

```js
db.trips.updateMany(
  { category: "beach" },
  { $inc: { price: -500 } }
)
```

### **B. Add a new feature to all trips going to Goa**

```js
db.trips.updateMany(
  { destination: "Goa" },
  { $addToSet: { features: "hotel upgrade" } }
)
```

### **C. Mark a trip as “Sold Out”**

```js
db.trips.updateOne(
  { _id: ObjectId("...") },
  { $set: { availability: 0, status: "sold out" } }
)
```

### **D. Clean-up: Remove unused “discountCode” field**

```js
db.trips.updateMany({}, { $unset: { discountCode: "" } })
```

---

# **8. Hands-On Class Activities**

### **Activity A: Simple Updates**

1. Increase price of all mountain trips by ₹1,000
2. Decrease availability of Goa trips by 2

### **Activity B: Array Updates**

1. Add `"guided tour"` to all Manali trips
2. Remove `"breakfast"` from all beach trips

### **Activity C: Upserts**

Update/create a partner:

```js
{ name: "SkyFly Airlines", rating: 4.7 }
```

### **Activity D: ReplaceOne Demo**

Pick a single trip and replace with a new structure.

### **Activity E: Real-world Task**

Write a query to:

* Update all trips starting after June 2026
* Add `earlyBirdOffer: true`

---

# **9. Mini Quiz (Update Methods)**

**Q1 (Theory):**
What is the difference between `updateOne()` and `replaceOne()`?

**Q2 (Theory):**
What is the purpose of `$unset`? Give an example.

**Q3 (Practical):**
Write a command to add `"snorkeling"` to the features of Andaman trips, without duplicates.

**Q4 (Practical):**
Increase price of all adventure trips by 15%.

**Q5 (Hybrid):**
A student tries to update multiple trips but only one updates:

```js
db.trips.updateOne({ category: "beach" }, { $set: { featured: true } })
```

Explain why, and correct the command.
