**MongoDB DELETE Methods**

1. `deleteOne()`
2. `deleteMany()`
3. Query-based deletes
4. Deleting embedded fields vs whole documents
5. TTL (Time-To-Live) Indexes (very useful for temporary data)
6. Soft Deletes (recommended for production apps)
7. Travel-app case study examples
8. Hands-on student tasks
9. Mini quiz

---

# **DELETE Methods in MongoDB**

Deletes in MongoDB are powerful and (dangerously) simple.
You must teach students **filter carefully** and never delete without conditions.

---

# **1. deleteOne() — Deletes the FIRST matching document**

### Syntax:

```js
db.collection.deleteOne(filter)
```

### Case Study Example:

Delete one beach trip priced below ₹5,000:

```js
db.trips.deleteOne({ category: "beach", price: { $lt: 5000 } })
```

### Delete one user by email:

```js
db.users.deleteOne({ email: "tempuser@example.com" })
```

---

# **2. deleteMany() — Deletes ALL matching documents**

### Syntax:

```js
db.collection.deleteMany(filter)
```

### Case Study Example — Remove all expired trips:

```js
db.trips.deleteMany({
  endDate: { $lt: ISODate() }
})
```

### Remove test users:

```js
db.users.deleteMany({ email: /test/i })
```

### Remove all bookings for a deleted trip:

```js
db.bookings.deleteMany({ tripId: "TRIP001" })
```

---

# **3. Delete ALL documents (Dangerous)**

Do *not* teach this lightly — explain the danger.

```js
db.trips.deleteMany({})
```

This clears the collection but keeps the collection itself.

---

# **4. Drop a Collection (More Dangerous)**

Deletes the entire collection structure + all data.

```js
db.trips.drop()
```

---

# **5. Drop a Database (Most Dangerous)**

```js
db.dropDatabase()
```

Use this only for dev/testing.

---

# **6. Deleting Embedded Fields (Not Whole Document)**

You can remove specific fields without deleting the document.

### Remove discount field from Goa trips:

```js
db.trips.updateMany(
  { destination: "Goa" },
  { $unset: { discount: "" } }
)
```

---

# **7. TTL (Time-To-Live) Indexes — Automatic Expiry**

Perfect for:

* temporary bookings
* OTPs
* login sessions
* temp carts
* limited-time offers

### Step 1 — Add a timestamp field

```js
db.sessions.insertOne({
  userId: "USR1001",
  createdAt: new Date()
})
```

### Step 2 — Create TTL index (expire after 30 minutes)

```js
db.sessions.createIndex(
  { createdAt: 1 },
  { expireAfterSeconds: 1800 }
)
```

MongoDB automatically deletes documents after the expiry time — **no delete queries needed**.

---

# **8. Soft Delete (Recommended in Real Apps)**

Instead of deleting, mark as deleted.

### Add deleted flag:

```js
db.trips.updateOne(
  { _id: ObjectId("...") },
  { $set: { deleted: true, deletedAt: new Date() } }
)
```

### Benefits:

* Data recovery
* Auditing
* No accidental loss
* Logs remain intact

### Your app will filter out:

```js
db.trips.find({ deleted: { $ne: true } })
```

---

# **9. Travel-App Case Study: Real Examples**

### **A. Delete trips ending before today**

```js
db.trips.deleteMany({
  endDate: { $lt: ISODate() }
})
```

---

### **B. Delete users who never booked anything**

Assume:

* bookings collection has `userId`

Find users without bookings using `$nin`:

```js
db.users.deleteMany({
  _id: { $nin: db.bookings.distinct("userId") }
})
```

---

### **C. Delete a specific partner**

```js
db.partners.deleteOne({ name: "SkyHigh Flights" })
```

---

### **D. Delete reviews older than 1 year**

```js
db.reviews.deleteMany({
  createdAt: { $lt: ISODate("2024-01-01") }
})
```

---

### **E. Create TTL index for temporary offers**

```js
db.offers.createIndex(
  { expiresAt: 1 },
  { expireAfterSeconds: 0 }
)
```

This deletes the document exactly at `expiresAt`.

---

# **10. Hands-On Activities for Students**

### **Activity A: Practice deleteOne()**

Delete a single adventure trip costing more than ₹30,000.

---

### **Activity B: Use deleteMany()**

Delete all trips where availability is 0.

---

### **Activity C: Regex Delete**

Delete all users whose email ends in `.temp`.

---

### **Activity D: Implement a Soft Delete**

Mark a trip with `_id = X` as deleted.

---

### **Activity E: TTL Task**

Create a `tempCart` collection where items should expire after 10 minutes.

Steps:

1. Insert sample data
2. Add `createdAt`
3. Create TTL index

---

# **11. Mini Quiz (DELETE Methods)**

**Q1 (Theory):**
What is the difference between `deleteOne()` and `deleteMany()`?

**Q2 (Theory):**
What is a TTL index used for?

**Q3 (Practical):**
Write a delete query to remove all Manali trips that cost above ₹20,000.

**Q4 (Practical):**
You inserted 50 test users with email ending `"@dummy.com"`.
Write a query to delete all of them.

**Q5 (Hybrid):**
A student ran:

```js
db.trips.drop()
```

thinking it would only delete expired trips.
Explain the mistake and write the correct command.
