# Case study: Student records (MongoDB)

**Topics covered in this step:** MongoDB **Insert** methods (insertOne, insertMany, bulkWrite), best-practice tips, how to verify inserted data.

We’ll use the `students` collection. Assume the shell/URI is connected and the database is selected, e.g.:

```js
use yourDatabaseName
```

---

## 1. Concept — What is an insert?

An **insert** adds new documents to a collection. MongoDB accepts documents in BSON (binary JSON). Common insert methods:

* `insertOne(document)` — inserts a single document.
* `insertMany([doc1, doc2, ...], options)` — inserts multiple documents at once.
* `bulkWrite([...])` — for mixed operations (insert/update/delete) or performance when doing many operations.

**Note:** If a document lacks `_id`, MongoDB creates one automatically.

---

## 2. Example: using your dataset with `insertMany`

You already provided a ready-to-insert array. In mongo shell:

```js
db.students.insertMany([
  {
    name: "Aniket",
    age: 22,
    major: "Computer Science",
    subjects: ["Programming", "Database", "AI"],
    scores: [
      { subject: "Math", score: 95 },
      { subject: "AI", score: 88 },
      { subject: "Database", score: 92 }
    ],
    graduation_Date: ISODate("2024-07-01")
  },
  /* ... (rest of the 15 documents exactly as you provided) ... */
  {
    name: "Manish",
    age: 27,
    major: "Cybersecurity",
    subjects: ["Ethical Hacking", "Networks", "Cryptography"],
    scores: [
      { subject: "Ethical Hacking", score: 91 },
      { subject: "Cryptography", score: 87 }
    ],
    graduation_Date: ISODate("2022-05-12")
  }
])
```

**What happens:** MongoDB will insert each document and return an object with `insertedCount` and `insertedIds`.

---

## 3. Options and behaviour

* `ordered` (default `true`): if `true`, MongoDB stops at first error (e.g., duplicate key); if `false`, it attempts to insert remaining docs.

  ```js
  db.students.insertMany(myDocs, { ordered: false })
  ```
* `writeConcern`: controls acknowledgment behavior (we’ll keep defaults for beginners).
* Duplicate `_id` causes an error — avoid manually setting duplicate `_id`s.

---

## 4. Quick verification (Find a few documents)

After inserting, verify:

```js
// count documents
db.students.countDocuments()

// show first 5 documents
db.students.find().limit(5).pretty()

// find a specific student
db.students.findOne({ name: "Aniket" })
```

---

## 5. Exercises

### Task 1 (Easy)

**Instruction:** Insert a single new student `Rohit` with these fields:

* `name`: "Rohit"
* `age`: 20
* `major`: "Computer Science"
* `subjects`: ["Programming", "OS", "Math"]
* `scores`: [{subject: "Programming", score: 86}]
  Do it with `insertOne`, then show the inserted document using `findOne`.

**Solution:**

```js
// Insert
db.students.insertOne({
  name: "Rohit",
  age: 20,
  major: "Computer Science",
  subjects: ["Programming", "OS", "Math"],
  scores: [{ subject: "Programming", score: 86 }]
})

// Verify
db.students.findOne({ name: "Rohit" })

/* Expected: returns the inserted document with an _id field added by MongoDB. */
```

---

### Task 2 (Medium)

**Instruction:** Insert two students at once (`insertMany`): `Asha` and `Vivek`. Provide their minimal fields: `name`, `age`, `major`. Use `insertMany` and show how to handle if one of them accidentally shares an `_id` already present (explain `ordered:false`).

**Solution (teacher answer):**

```js
// Normal insertMany
db.students.insertMany([
  { name: "Asha", age: 19, major: "Physics" },
  { name: "Vivek", age: 22, major: "Mechanical Engineering" }
])

// If there is a risk of duplicate _id and you want to insert others even on error:
db.students.insertMany([
  { _id: ObjectId("64a..."), name: "Asha", age: 19, major: "Physics" },
  { _id: ObjectId("64b..."), name: "Vivek", age: 22, major: "Mechanical Engineering" }
], { ordered: false })
```

**Explanation:** With `ordered:false` MongoDB attempts to insert every doc and reports which failed.

---

### Task 3 (Hands-on understanding)

**Instruction:** After inserting all provided documents, run a count and answer: **How many documents are in `students`?** (Assume only the original provided list was inserted and no other documents.)

**Solution:**
If you inserted the 16 documents you provided plus any additional ones inserted in Tasks above (e.g., Rohit, Asha, Vivek), use:

```js
db.students.countDocuments()
```

* If only the original 16 were inserted: `16`.
* If you also inserted Rohit, Asha, Vivek from Tasks 1 and 2, the count will be `19`.

*(This teaches students to check actual counts rather than assume.)*

---

### Short takeaway:

* Use `insertOne` for single documents, `insertMany` for multiple.
* Use `ordered:false` for resilience when inserting many docs.
* Always verify using `find()` or `countDocuments()`.

 
# MongoDB Find Methods + Query Operators

In this step, we cover:

1. Basic `find()` usage
2. Projecting specific fields
3. Common query operators: `$gt`, `$lt`, `$in`, `$nin`, `$and`, `$or`, `$exists`, `$elemMatch`
4. Array querying basics
5. Tasks for students with answers

---

# 1. Basic `find()`

`find()` returns all documents matching a filter.
If you pass an empty filter, you get everything.

### Examples

```js
// All students
db.students.find()
```

```js
// Student named "Aniket"
db.students.find({ name: "Aniket" })
```

---

# 2. Limiting, Sorting, Pretty

```js
db.students.find().limit(3).pretty()
```

```js
db.students.find().sort({ age: 1 })      // ascending
db.students.find().sort({ age: -1 })     // descending
```

---

# 3. Projection (select only some fields)

Second argument selects which fields to display.

```js
// Show only name and major
db.students.find({}, { name: 1, major: 1 })
```

```js
// Hide _id
db.students.find({}, { name: 1, major: 1, _id: 0 })
```

---

# 4. Query Operators

---

## `$gt`, `$lt`, `$gte`, `$lte`

Greater-than, less-than.

```js
// Students older than 22
db.students.find({ age: { $gt: 22 } })
```

```js
// Students age 18 or younger
db.students.find({ age: { $lte: 18 } })
```

---

## `$in`, `$nin`

```js
// Students majoring in these fields
db.students.find({ major: { $in: ["Computer Science", "AI", "Data Science"] } })
```

```js
// Students NOT in Engineering fields
db.students.find({ major: { $nin: ["Electrical Engineering", "Mechanical Engineering", "Civil Engineering"] } })
```

---

## `$and`, `$or`

```js
// Age < 22 AND major is Physics
db.students.find({
  $and: [
    { age: { $lt: 22 } },
    { major: "Physics" }
  ]
})
```

```js
// Age < 20 OR major is Chemistry
db.students.find({
  $or: [
    { age: { $lt: 20 } },
    { major: "Chemistry" }
  ]
})
```

---

## `$exists`

```js
// Students who have a graduation_Date field
db.students.find({ graduation_Date: { $exists: true } })
```

---

## `$elemMatch` (important for scores array)

Useful when filtering inside array of objects.

```js
// Students who scored > 90 in any subject
db.students.find({
  scores: { $elemMatch: { score: { $gt: 90 } } }
})
```

```js
// Students who have AI subject with score >= 88
db.students.find({
  scores: { $elemMatch: { subject: "AI", score: { $gte: 88 } } }
})
```

---

# 5. Array queries (basic)

### Check if a value exists inside an array

```js
// Students who study "Math"
db.students.find({ subjects: "Math" })
```

### Check multiple values required in array

```js
// Students who study BOTH Programming AND AI
db.students.find({ subjects: { $all: ["Programming", "AI"] } })
```

---

# 6. Useful classroom examples using your dataset

### Example 1: All students older than 24

```js
db.students.find({ age: { $gt: 24 } })
```

### Example 2: Students who have taken “Statistics”

```js
db.students.find({ subjects: "Statistics" })
```

### Example 3: Students who have graduation date before 2024

```js
db.students.find({
  graduation_Date: { $lt: ISODate("2024-01-01") }
})
```

### Example 4: Students who scored at least 90 in Math

```js
db.students.find({
  scores: { $elemMatch: { subject: "Math", score: { $gte: 90 } } }
})
```

---

# 7. Tasks

## Task 1 (Easy)

**Find all students whose age is greater than 20. Display only name and age.**

**Answer:**

```js
db.students.find(
  { age: { $gt: 20 } },
  { name: 1, age: 1, _id: 0 }
)
```

---

## Task 2 (Medium)

**Find all students who study "AI" as part of their subjects array. Only show name and subjects.**

**Answer:**

```js
db.students.find(
  { subjects: "AI" },
  { name: 1, subjects: 1, _id: 0 }
)
```

---

## Task 3 (Medium)

**Find all students who have graduation_Date (i.e., field exists) AND age >= 23.**

**Answer:**

```js
db.students.find({
  $and: [
    { graduation_Date: { $exists: true } },
    { age: { $gte: 23 } }
  ]
})
```

---

## Task 4 (Hard)

**Find all students who scored more than 85 in any subject.
Use `$elemMatch`.**

**Answer:**

```js
db.students.find({
  scores: { $elemMatch: { score: { $gt: 85 } } }
})
```

---

## Task 5 (Hard)

**Find students whose major is NOT Engineering (Electrical, Mechanical, Civil). Use `$nin`.**

**Answer:**

```js
db.students.find({
  major: { $nin: ["Electrical Engineering", "Mechanical Engineering", "Civil Engineering"] }
})
```

---

# Summary

You now know:

* How to fetch data with `find()`
* How to filter using query operators
* How to project specific fields
* How to search inside arrays and nested structures


