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


# MongoDB Update Methods + Update Operators

In this step we cover:

1. `updateOne()`
2. `updateMany()`
3. Update operators:

   * `$set`, `$unset`
   * `$inc`
   * `$push`, `$addToSet`
   * `$pull`
   * `$rename`
   * `$max`, `$min`
4. Updating values inside arrays (including positional operator `$`)
5. Student tasks with answers

---

# 1. updateOne()

Updates **only the first matching document**.

Example: Increase Aniket’s age to 23.

```js
db.students.updateOne(
  { name: "Aniket" },          // filter
  { $set: { age: 23 } }        // update
)
```

---

# 2. updateMany()

Updates **all matching documents**.

Example: Add a new field `status: "active"` to all students older than 20.

```js
db.students.updateMany(
  { age: { $gt: 20 } },
  { $set: { status: "active" } }
)
```

---

# 3. Important Update Operators

---

## `$set`

Add or replace a field.

```js
db.students.updateOne(
  { name: "Riya" },
  { $set: { scholarship: true } }
)
```

---

## `$unset`

Remove a field.

```js
db.students.updateOne(
  { name: "Riya" },
  { $unset: { scholarship: 1 } }
)
```

---

## `$inc`

Increment a numeric value.

Example: Increase Sneha’s age by 1.

```js
db.students.updateOne(
  { name: "Sneha" },
  { $inc: { age: 1 } }
)
```

---

## `$push`

Add a new element to an array.

Example: Add "Cloud Computing" to Isha’s subjects.

```js
db.students.updateOne(
  { name: "Isha" },
  { $push: { subjects: "Cloud Computing" } }
)
```

---

## `$addToSet`

Add to array **only if it does not already exist**.

```js
db.students.updateOne(
  { name: "Isha" },
  { $addToSet: { subjects: "Statistics" } }
)
```

Statistics was already present, so nothing is added.

---

## `$pull`

Remove array element(s).

Example: Remove “Math” from Rahul’s subjects.

```js
db.students.updateOne(
  { name: "Rahul" },
  { $pull: { subjects: "Math" } }
)
```

---

## `$rename`

Rename a field.

Example: Rename `graduation_Date` to `graduationDate`.

```js
db.students.updateMany(
  { graduation_Date: { $exists: true } },
  { $rename: { graduation_Date: "graduationDate" } }
)
```

---

## `$max` / `$min`

Set a value only if condition is met.

Example: If a student’s recorded age is less than 18, set it to 18.

```js
db.students.updateMany(
  {},
  { $max: { age: 18 } }
)
```

Example: Lower max value.

```js
db.students.updateMany(
  { name: "Priya" },
  { $min: { age: 25 } }
)
```

---

# 4. Updating Nested Arrays (Very Important)

### Goal: Update a score inside `scores` array.

Use the **positional operator `$`**, which finds the index of the matched array item.

### Example:

Update Aniket’s AI score from 88 to 90.

```js
db.students.updateOne(
  { name: "Aniket", "scores.subject": "AI" },
  { $set: { "scores.$.score": 90 } }
)
```

Explanation:

* MongoDB finds the index where `scores.subject == "AI"`
* `$` plugs in that index
* Only that element is updated

---

# 5. Updating embedded array by adding a new score

Example: Add a new score for “OS” to Rohit.

```js
db.students.updateOne(
  { name: "Rohit" },
  { $push: { scores: { subject: "OS", score: 78 } } }
)
```

---

---

## Task 1 (Easy)

**Increase Riya’s age by 1 using `$inc`. Then verify.**

**Answer:**

```js
db.students.updateOne(
  { name: "Riya" },
  { $inc: { age: 1 } }
)

db.students.findOne({ name: "Riya" })
```

---

## Task 2 (Easy)

**Add a new field `enrolled: true` to all Computer Science students.**

**Answer:**

```js
db.students.updateMany(
  { major: "Computer Science" },
  { $set: { enrolled: true } }
)
```

---

## Task 3 (Medium)

**Add “Algorithms” to Priya’s subjects.
Use `$addToSet` so it does not duplicate.**

**Answer:**

```js
db.students.updateOne(
  { name: "Priya" },
  { $addToSet: { subjects: "Algorithms" } }
)
```

---

## Task 4 (Medium)

**Remove the subject "Physics" from Rahul’s subjects array.**

**Answer:**

```js
db.students.updateOne(
  { name: "Rahul" },
  { $pull: { subjects: "Physics" } }
)
```

---

## Task 5 (Hard)

**Increase the score of Vikram in Surveying from 81 to 90.
Use array positional operator.**

**Answer:**

```js
db.students.updateOne(
  { name: "Vikram", "scores.subject": "Surveying" },
  { $set: { "scores.$.score": 90 } }
)
```

---

## Task 6 (Hard)

**Rename graduation_Date to graduationDate for all documents.**

**Answer:**

```js
db.students.updateMany(
  { graduation_Date: { $exists: true } },
  { $rename: { graduation_Date: "graduationDate" } }
)
```

---

# Summary

You now know:

* updateOne vs updateMany
* All core update operators
* How to update nested arrays using `$`
* How to control additions and removals in arrays

---
Perfect — here is **Step 4 (final step)**: MongoDB **Delete Methods** plus a clean **end-to-end workflow** that ties Insert → Find → Update → Delete together using your `students` dataset. Everything is beginner-friendly, with classroom tasks and full answers.

---

# MongoDB Delete Methods + End-to-End Workflow

## What we cover

1. `deleteOne()` and `deleteMany()` — usage and safety tips
2. `findOneAndDelete()` — atomic find+delete with return of deleted doc
3. Best-practices before deleting (verify, backup pattern)
4. Conditional deletes and safe patterns
5. Final end-to-end case study (sequence of commands you can run)
6. Student tasks with answers

---

## 1. `deleteOne(filter)`

Removes **the first document** that matches `filter`.

```js
// Delete Riya (age 17) — single delete
db.students.deleteOne({ name: "Riya" })
```

Return value example:

```js
{ "acknowledged" : true, "deletedCount" : 1 }
```

**Note:** If multiple docs match, only the first matching document (according to internal order) is removed.

---

## 2. `deleteMany(filter)`

Removes **all documents** matching the filter.

```js
// Delete all students older than 26
db.students.deleteMany({ age: { $gt: 26 } })
```

Return value:

```js
{ "acknowledged" : true, "deletedCount" : N }
```

---

## 3. `findOneAndDelete(filter, options)`

Finds a document, deletes it, and returns the deleted document in one atomic operation. Useful when you want the removed doc back (for audit/logging).

```js
// Remove and return first student with major "Chemistry"
const removed = db.students.findOneAndDelete({ major: "Chemistry" })
printjson(removed)
```

Options like `sort` or `projection` are supported.

---

## 4. Safety patterns

### a) Verify before deleting

Always run a `find()` with the same filter first so you know exactly what will be deleted.

```js
db.students.find({ major: "Chemistry" }).pretty()
```

### b) Back up matching documents (optional but recommended)

If you want a simple backup copy into another collection:

```js
// Copy documents to backup collection first
db.students.find({ major: "Chemistry" }).forEach(function(doc) {
  db.students_backup.insertOne(doc);
});

// Then delete
db.students.deleteMany({ major: "Chemistry" })
```

### c) Use a limited `deleteOne()` when unsure

Prefer `deleteOne()` instead of `deleteMany()` if you're not certain.

### d) Use transactions (when using replica sets / sharded clusters)

For multi-document multi-collection operations that must be all-or-nothing, use transactions. (Intro mention only — not diving into code for now.)

---

## 5. Conditional deletes examples using your dataset

### Example A — Delete students under 18 (minor cleanup)

```js
db.students.deleteMany({ age: { $lt: 18 } })
```

(With dataset this would delete Riya if she still exists.)

### Example B — Delete students who have graduated before 2023

Note: You might have `graduation_Date` or `graduationDate` depending on previous renames. Use `$or` to cover both field names:

```js
db.students.deleteMany({
  $or: [
    { graduation_Date: { $exists: true, $lt: ISODate("2023-01-01") } },
    { graduationDate: { $exists: true, $lt: ISODate("2023-01-01") } }
  ]
})
```

### Example C — Remove a student by `_id` (safest when you know exact doc)

```js
db.students.deleteOne({ _id: ObjectId("...") })
```

---

## 6. Combined end-to-end case study (complete workflow)

This walks students through a realistic maintenance task. Each step shows commands and expected outcomes.

**Scenario:**
The department wants to:

1. Add a field `enrolled: true` to all current Computer Science & Artificial Intelligence students.
2. Give any Computer Science or AI student a new `scores` entry for "Ethics" with score 80 — but **only** if they don't already have "Ethics" (avoid duplicate).
3. Find and list those updated students.
4. Remove any students who graduated before 2023 after backing them up.

**Commands (run in order):**

### Step A — Verify who will be updated

```js
db.students.find(
  { major: { $in: ["Computer Science", "Artificial Intelligence"] } },
  { name: 1, major: 1, graduation_Date: 1, graduationDate: 1 }
).pretty()
```

### Step B — Add `enrolled: true`

```js
db.students.updateMany(
  { major: { $in: ["Computer Science", "Artificial Intelligence"] } },
  { $set: { enrolled: true } }
)
```

*Expected result:* `modifiedCount` equals number of CS + AI docs.

### Step C — Add "Ethics" score only if not present (use `$addToSet` with `$each` and `$cond` not available directly for arrays of objects, so we use a safe per-document approach)*

Because `scores` is an array of objects, `$addToSet` won't prevent duplicate *objects* unless they exactly match. To avoid duplicate subject entries, run an update that only targets documents that **don't** already have `scores.subject: "Ethics"`:

```js
db.students.updateMany(
  {
    major: { $in: ["Computer Science", "Artificial Intelligence"] },
    "scores.subject": { $ne: "Ethics" }    // ensure no Ethics entry exists
  },
  {
    $push: { scores: { subject: "Ethics", score: 80 } }
  }
)
```

*Explanation:* the filter ` "scores.subject": { $ne: "Ethics" }` ensures only documents without an "Ethics" subject get the new score.

### Step D — Verify updated docs

```js
db.students.find(
  {
    major: { $in: ["Computer Science", "Artificial Intelligence"] },
    enrolled: true
  },
  { name: 1, enrolled: 1, scores: 1, _id: 0 }
).pretty()
```

### Step E — Backup students who graduated before 2023 (safe delete)

```js
// 1) Select those docs (use both field names)
const toRemoveCursor = db.students.find({
  $or: [
    { graduation_Date: { $exists: true, $lt: ISODate("2023-01-01") } },
    { graduationDate: { $exists: true, $lt: ISODate("2023-01-01") } }
  ]
})

// 2) Copy to backup
toRemoveCursor.forEach(function(doc) {
  db.students_archived.insertOne(doc);
})

// 3) Confirm backup count equals found count
// (In shell, count manually)
print("Archived count:", db.students_archived.countDocuments())

// 4) Now remove from primary collection
db.students.deleteMany({
  $or: [
    { graduation_Date: { $exists: true, $lt: ISODate("2023-01-01") } },
    { graduationDate: { $exists: true, $lt: ISODate("2023-01-01") } }
  ]
})
```

### Step F — Final verification

```js
// Remaining students count
db.students.countDocuments()

// See the backup collection
db.students_archived.find().pretty()
```

---

## 7. Tasks

### Task 1 (Easy)

**Find and delete the student named "Diya" using `findOneAndDelete` and show the deleted document.**

**Answer**

```js
const deleted = db.students.findOneAndDelete({ name: "Diya" })
printjson(deleted)
```

*Expected:* `deleted` contains Diya’s document. The primary collection no longer has Diya.

---

### Task 2 (Medium)

**Backup and delete all students whose major is "Civil Engineering". Provide the commands.**

**Answer**

```js
// 1) Backup
db.students.find({ major: "Civil Engineering" }).forEach(function(doc) {
  db.students_backup.insertOne(doc);
});

// 2) Delete
db.students.deleteMany({ major: "Civil Engineering" })
```

---

### Task 3 (Medium)

**Delete all students who do NOT have a `graduation_Date` or `graduationDate` field (i.e., remove incomplete records). First list them, then delete.**

**Answer**

```js
// 1) find them
db.students.find({
  graduation_Date: { $exists: false },
  graduationDate: { $exists: false }
}).pretty()

// 2) delete them
db.students.deleteMany({
  graduation_Date: { $exists: false },
  graduationDate: { $exists: false }
})
```

*Note:* This may delete many records; emphasize backup before running.

---

### Task 4 (Hard)

**You want to delete only one student who has the lowest Math score across all students. Outline steps and provide shell commands.**

**Answer (step-by-step):**

1. Find the student with lowest Math score using aggregation to get minimal score and the student(s).
2. Delete one of those students using `_id`.

```js
// 1) Find min Math score and the student(s)
const minRes = db.students.aggregate([
  { $unwind: "$scores" },
  { $match: { "scores.subject": "Math" } },
  { $sort: { "scores.score": 1 } },
  { $limit: 1 },
  { $project: { _id: 1, name: 1, "scores.score": 1 } }
]).toArray()

printjson(minRes)

// 2) Delete that student by _id (assuming minRes[0]._id exists)
db.students.deleteOne({ _id: minRes[0]._id })
```

*note:* Aggregation is useful when deletions depend on computed results. Always capture `_id` and verify before deleting.

---

# MongoDB Short Quiz

## Q1.

Which command inserts **multiple** documents into a MongoDB collection at once?

A. insertOne
B. insertMany
C. addMany
D. pushMany

**Answer:** B. insertMany

---

## Q2.

Write a query to find all students whose **age is greater than 22**.

**Answer:**

```js
db.students.find({ age: { $gt: 22 } })
```

---

## Q3.

Which projection will display **only name and major**, and hide `_id`?

A. `{ name:1, major:1 }`
B. `{ name:1, major:1, _id:0 }`
C. `{ name:0, major:1, _id:0 }`
D. `{ major:1, _id:1 }`

**Answer:** B. `{ name:1, major:1, _id:0 }`

---

## Q4.

Find all students who study **AI** (inside their subjects array).

**Answer:**

```js
db.students.find({ subjects: "AI" })
```

---

## Q5.

Which operator is used to **increment** a numeric value in a document?

A. $inc
B. $set
C. $addToSet
D. $push

**Answer:** A. $inc

---

## Q6.

Update student **Aniket's AI score** to **90**, assuming “AI” is inside the nested scores array.

**Answer:**

```js
db.students.updateOne(
  { name: "Aniket", "scores.subject": "AI" },
  { $set: { "scores.$.score": 90 } }
)
```

---

## Q7.

Write a query to **delete all students younger than 18**.

**Answer:**

```js
db.students.deleteMany({ age: { $lt: 18 } })
```

---

## Q8.

Which operator **removes** an item from an array?

A. $push
B. $pull
C. $pop
D. $unset

**Answer:** B. $pull

---

## Q9.

Find all students who have a **graduation date field**, regardless of value.

**Answer:**

```js
db.students.find({ graduationDate: { $exists: true } })
```

(or `graduation_Date` depending on naming)

---

## Q10.

Explain the difference between **deleteOne** and **deleteMany** in one line.

**Answer:**
*deleteOne removes only the first matching document, while deleteMany removes all matching documents.*

---






