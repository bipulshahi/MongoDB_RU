1. MongoDB Querying Related Documents
2. MongoDB Aggregation Methods
3. MongoDB Aggregation Pipeline
4. Putting It All Together (Advanced Aggregations + Real Use Cases)
---

# STEP 1 — MongoDB Querying Related Documents

Goal: Learn how to query:

* Embedded documents
* Arrays
* Nested arrays
* Match specific values inside arrays or sub-documents
* Projection
* Filtering
* Sorting
* Query patterns for your dataset

We will use the **students** collection.

---

# 1.1 Querying Simple Fields

Find the student whose name is “Riya”:

```js
db.students.find({ name: "Riya" });
```

Find all students older than 22:

```js
db.students.find({ age: { $gt: 22 } });
```

Find students whose major is “Computer Science”:

```js
db.students.find({ major: "Computer Science" });
```

---

# 1.2 Querying Array Fields (subjects)

Find all students who study “Math”:

```js
db.students.find({ subjects: "Math" });
```

MongoDB automatically checks inside arrays.

Find students taking **both** “Math” and “Physics”:

```js
db.students.find({ subjects: { $all: ["Math", "Physics"] } });
```

Find students taking **any of** Math, AI, ML:

```js
db.students.find({ subjects: { $in: ["Math", "AI", "ML"] } });
```

---

# 1.3 Querying Embedded Documents (scores array)

We want students who scored **above 90 in Math**:

```js
db.students.find({
  scores: {
    $elemMatch: { subject: "Math", score: { $gt: 90 } }
  }
});
```

Find all students who have a score entry for “AI”:

```js
db.students.find({ "scores.subject": "AI" });
```

---

# 1.4 Querying Nested Fields

Find only student name and their Math score:

```js
db.students.find(
  { "scores.subject": "Math" },
  { name: 1, scores: 1 }
);
```

Find students whose graduation date is before 2024:

```js
db.students.find({
  graduation_Date: { $lt: ISODate("2024-01-01") }
});
```
---

# 1.5 Sorting / Limiting

Top 5 oldest students:

```js
db.students.find().sort({ age: -1 }).limit(5);
```

Alphabetical order:

```js
db.students.find().sort({ name: 1 });
```

---

# 1.6 Combining Filters (AND / OR)

Find students:

* age < 22
* who study Math

```js
db.students.find({
  age: { $lt: 22 },
  subjects: "Math"
});
```

Find students who are either:

* major = Physics
* OR age > 23

```js
db.students.find({
  $or: [
    { major: "Physics" },
    { age: { $gt: 23 } }
  ]
});
```

---

# 1.7 Real Case Study Query Examples

### A. Find all Computer Science students scoring above 90 in any subject

```js
db.students.find({
  major: "Computer Science",
  "scores.score": { $gt: 90 }
});
```

### B. Students who completed graduation (field exists)

```js
db.students.find({ graduation_Date: { $exists: true } });
```

### C. Students who have scored in exactly 3 subjects

(using array length)

```js
db.students.find({
  $expr: { $eq: [ { $size: "$scores" }, 3 ] }
});
```

---

# STEP 1 Assignments
---

## **(Easy)**

**Question:**
Write a query to find all students whose **major is Physics**.
Display only name and age.

**Solution:**

```js
db.students.find(
  { major: "Physics" },
  { name: 1, age: 1, _id: 0 }
);
```

**Expected Output Explanation:**
It will return one document:

```
{ name: "Riya", age: 17 }
```

---

## **(Medium)**

**Question:**
Find all students who study **both** “Math” AND “Physics”.

**Solution:**

```js
db.students.find({
  subjects: { $all: ["Math", "Physics"] }
});
```

**Expected Students Returned:**

* Riya
* Rahul

Because their subjects contain both Math and Physics.

---

## **(Hard)**

**Question:**
Find all students who scored **more than 90 in ANY subject**, and show:

* name
* subject where they scored > 90
* score

**Solution (using projection + $filter):**

```js
db.students.aggregate([
  {
    $project: {
      name: 1,
      high_scores: {
        $filter: {
          input: "$scores",
          as: "s",
          cond: { $gt: ["$$s.score", 90] }
        }
      }
    }
  },
  { $match: { "high_scores.0": { $exists: true } } }
]);
```

**Explanation:**

* `$filter` extracts only score entries where score > 90
* `$match` ensures at least one such score exists

**Expected Students (based on your data):**

* Aniket (Math 95)
* Sourav (Math 91)
* Karan (Deep Learning 94, NLP 91)
* Arjun (Algebra 92, Calculus 90 → Note: 90 doesn't count, but Algebra 92 counts)
* Diya (Organic 90 → does NOT count, needs > 90)
* Manish (Ethical Hacking 91)

---

