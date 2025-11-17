The goal:
✔ Practice **thinking like a real MongoDB developer**
✔ Use **Insert, Find, Update, Delete**
✔ Work with arrays, nested objects, projections, filters
✔ Perform multi-step operations similar to production use-cases

---

# **ADVANCED CASE STUDY: “TravelMate” — End-to-End CRUD Simulation**

Imagine your students are building **TravelMate**, a dynamic travel app created by a group of friends.

It has:

* `trips`
* `users`
* `bookings`
* `reviews`
* `partners`

Your job in this module:
- Build + Query + Update + Clean + Analyze data like a real application team.

We will go in 5 phases:

---

# **PHASE 1 — Database Setup + Inserts (Create)**

### 1.1 Create travelDB

```js
use travelDB
```

---

### 1.2 Insert sample trips

Ask students to insert at least **5 trips**:

```js
db.trips.insertMany([
  {
    title: "Goa Beach Escape",
    category: "beach",
    price: 14999,
    destination: "Goa",
    availability: 30,
    features: ["breakfast", "pickup"],
    ratings: { avg: 4.5, count: 120 },
    createdAt: ISODate()
  },
  {
    title: "Manali Mountain Trek",
    category: "mountain",
    price: 12999,
    destination: "Manali",
    availability: 15,
    features: ["camping", "guide"],
    ratings: { avg: 4.7, count: 95 },
    createdAt: ISODate()
  },
  {
    title: "Andaman Blue Lagoon",
    category: "beach",
    price: 28999,
    destination: "Andaman",
    availability: 10,
    features: ["snorkeling", "coral tour"],
    ratings: { avg: 4.8, count: 210 },
    createdAt: ISODate()
  }
])
```

---

### 1.3 Insert users

```js
db.users.insertMany([
  { name: "Aditi", email: "aditi@example.com" },
  { name: "Rohit", email: "rohit@example.com" }
])
```

---

### 1.4 Insert bookings

```js
db.bookings.insertOne({
  userEmail: "rohit@example.com",
  tripTitle: "Goa Beach Escape",
  travelers: 2,
  bookingDate: ISODate()
})
```

---

### 1.5 Insert reviews

```js
db.reviews.insertOne({
  tripTitle: "Goa Beach Escape",
  rating: 5,
  review: "Amazing stay and clean beaches!",
  createdAt: ISODate()
})
```

---

# **PHASE 2 — Retrieval (Read / Find Queries)**

Students now act like the **frontend** asking for data.

---

### 2.1 Find all beach trips

```js
db.trips.find({ category: "beach" })
```

---

### 2.2 Find trips under ₹20,000 sorted by price

```js
db.trips.find({ price: { $lt: 20000 } }).sort({ price: 1 })
```

---

### 2.3 Show only `title`, `price`, `destination`

```js
db.trips.find({}, { title: 1, price: 1, destination: 1, _id: 0 })
```

---

### 2.4 Find trips with rating above 4.6

```js
db.trips.find({ "ratings.avg": { $gt: 4.6 } })
```

---

### 2.5 Find trips that include “snorkeling”

```js
db.trips.find({ features: "snorkeling" })
```

---

# **PHASE 3 — Updates (Modify Data)**

Students act like the **admin backend** updating travel packages.

---

### 3.1 Increase price of all beach trips by 10%

```js
db.trips.updateMany(
  { category: "beach" },
  { $mul: { price: 1.10 } }
)
```

---

### 3.2 Add a new feature to Manali trips

```js
db.trips.updateMany(
  { destination: "Manali" },
  { $addToSet: { features: "bonfire night" } }
)
```

---

### 3.3 Set Goa trips as "Featured"

```js
db.trips.updateOne(
  { destination: "Goa" },
  { $set: { isFeatured: true } }
)
```

---

### 3.4 Remove discount field if old

```js
db.trips.updateMany({}, { $unset: { discount: "" } })
```

---

### 3.5 Recalculate availability after booking

Assume 2 travelers booked Goa:

```js
db.trips.updateOne(
  { title: "Goa Beach Escape" },
  { $inc: { availability: -2 } }
)
```

---

# **PHASE 4 — Deletion (Remove Data)**

Students now act like a **data cleanup team**.

---

### 4.1 Delete expired or old trips

```js
db.trips.deleteMany({
  createdAt: { $lt: ISODate("2024-01-01") }
})
```

---

### 4.2 Delete a test user

```js
db.users.deleteOne({ email: "test@example.com" })
```

---

### 4.3 Delete all bookings for a deleted trip

```js
db.bookings.deleteMany({ tripTitle: "Old Trip" })
```

---

### 4.4 Soft delete a trip instead of removing it

```js
db.trips.updateOne(
  { title: "Manali Mountain Trek" },
  {
    $set: {
      deleted: true,
      deletedAt: new Date()
    }
  }
)
```

---

# **PHASE 5 — Realistic Multi-Step Challenges (Advanced Practice)**

These simulate real app tasks.

---

## **Challenge A — "Find, Then Update" Task**

Find the cheapest beach trip and mark it as recommended.

Step 1:

```js
db.trips.find({ category: "beach" }).sort({ price: 1 }).limit(1)
```

Step 2:

```js
db.trips.updateOne(
  { title: "Goa Beach Escape" }, 
  { $set: { recommended: true } }
)
```

---

## **Challenge B — "Insert + Update" Task**

A user books a trip.
You must:

1. Create the booking
2. Reduce availability

```js
db.bookings.insertOne({
  userEmail: "aditi@example.com",
  tripTitle: "Manali Mountain Trek",
  travelers: 3,
  bookingDate: ISODate()
})
```

Update availability:

```js
db.trips.updateOne(
  { title: "Manali Mountain Trek" },
  { $inc: { availability: -3 } }
)
```

---

## **Challenge C — "Find + Delete" Task**

Delete all reviews older than 6 months.

```js
db.reviews.deleteMany({
  createdAt: {
    $lt: new Date(Date.now() - 180 * 24 * 60 * 60 * 1000)
  }
})
```

---

## **Challenge D — "Array Filtering"**

Add `"family-friendly"` tag to all trips with avg rating > 4.7:

```js
db.trips.updateMany(
  { "ratings.avg": { $gt: 4.7 } },
  { $addToSet: { tags: "family-friendly" } }
)
```

---

## **Challenge E — "Smart Search"**

Find **affordable high-rated trips**:

* under ₹15,000
* rating above 4.5
* availability > 0
* sorted by price

```js
db.trips.find({
  price: { $lt: 15000 },
  "ratings.avg": { $gt: 4.5 },
  availability: { $gt: 0 }
})
.sort({ price: 1 })
```

---

# **STUDENT MINI QUIZ (Advanced CRUD)**

**Q1 (Theory):**
What is the benefit of doing “soft deletes” instead of hard deletes?

---

**Q2 (Theory):**
Why should you be careful when using `deleteMany({})`?

---

**Q3 (Practical):**
Write a query to increase the price of all trips to Andaman by 15%.

---

**Q4 (Practical):**
Write a query to find all the cheapest 3 adventure trips and show only `title` and `price`.

---

**Q5 (Hybrid):**
A user books a 4-day trip to Goa.
Write two queries:

1. Insert the booking
2. Reduce availability of that trip by 4

---
