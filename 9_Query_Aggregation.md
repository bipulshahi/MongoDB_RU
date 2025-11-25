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
---

# STEP 2 — MongoDB Aggregation Methods

Most important MongoDB aggregation operators:

1. `$match`
2. `$project`
3. `$group`
4. `$sort`
5. `$limit`
6. `$unwind`
7. `$count`
8. `$addFields`

All examples will be based on your **students** collection.

---

# 2.1 What is Aggregation?

Aggregation is used when you want to:

* Summarize data
* Compute statistics
* Group students
* Calculate averages
* Count documents
* Transform complex documents

Think of aggregation as **SQL GROUP BY + WHERE + HAVING + JOIN + SELECT** combined in a pipeline.

Aggregation is done using:

```js
db.students.aggregate([...])
```

Where `[...]` is a sequence of stages.

---

# 2.2 Stage 1 — $match (filtering)

Works like `find()`, but inside aggregation.

### Example 1 — Find only Computer Science students:

```js
db.students.aggregate([
  { $match: { major: "Computer Science" } }
]);
```

### Example 2 — Students scoring above 90 in any subject:

```js
db.students.aggregate([
  { $match: { "scores.score": { $gt: 90 } } }
]);
```

---

# 2.3 Stage 2 — $project (select fields / reshape documents)

Used to:

* Include or exclude fields
* Transform fields
* Create new fields

### Example 1 — Show only name and major:

```js
db.students.aggregate([
  { $project: { name: 1, major: 1, _id: 0 } }
]);
```

### Example 2 — Add a fullname field:

```js
db.students.aggregate([
  { 
    $project: {
      name: 1,
      major: 1,
      name_upper: { $toUpper: "$name" }
    }
  }
]);
```

### Example 3 — Add a new field: name in uppercase:

```js
db.students.aggregate([
  { $match: { major: "Computer Science" } },
  { 
    $project: { 
      name: 1, 
      subjects: 1, 
      name_upper: { $toUpper: "$name" }, 
      _id: 0 
    } 
  }
]);
```

### Example 4 — Project only Math score:

```js
db.students.aggregate([
  {
    $project: {
      name: 1,
      math_score: {
        $filter: {
          input: "$scores",
          as: "s",
          cond: { $eq: ["$$s.subject", "Math"] }
        }
      }
    }
  }
]);
```

---

# 2.4 Stage 3 — $group (summaries, totals, averages)

**This is the most important operator in aggregation.**

Typical uses:

* average score
* count students per major
* highest score per subject

### Example 1 — Count students by major

```js
db.students.aggregate([
  {
    $group: {
      _id: "$major",
      student_count: { $sum: 1 }
    }
  }
]);
```

### Example 2 — Average age of students

```js
db.students.aggregate([
  {
    $group: {
      _id: null,
      avg_age: { $avg: "$age" }
    }
  }
]);
```

### Example 3 — Highest score achieved in the whole dataset

```js
db.students.aggregate([
  { $unwind: "$scores" },
  {
    $group: {
      _id: null,
      highest_score: { $max: "$scores.score" }
    }
  }
]);
```

---

# 2.5 Stage 4 — $sort

Sort in ascending (1) or descending (-1):

### Example 1 — Sort by age (descending)

```js
db.students.aggregate([
  { $sort: { age: -1 } }
]);
```

### Example 2 — Sort by name:

```js
db.students.aggregate([
  { $sort: { name: 1 } }
]);
```

---

# 2.6 Stage 5 — $limit

Return only first n documents.

```js
db.students.aggregate([
  { $limit: 5 }
]);
```

Often used after sorting.

---

# 2.7 Stage 6 — $unwind (explode arrays)

This is the most powerful operator after `$group`.

It converts each array element into a separate document.

### Example — Unwind scores array:

```js
db.students.aggregate([
  { $unwind: "$scores" }
]);
```

This transforms:

```
{
  name: "Aniket",
  scores: [ {Math}, {AI}, {DB} ]
}
```

into three documents:

```
{name: "Aniket", scores:{Math}}
{name: "Aniket", scores:{AI}}
{name: "Aniket", scores:{DB}}
```

### Example — Find highest score per subject

```js
db.students.aggregate([
  { $unwind: "$scores" },
  {
    $group: {
      _id: "$scores.subject",
      highest_score: { $max: "$scores.score" }
    }
  }
]);
```

---

# 2.8 Stage 7 — $count

Simple count of documents:

```js
db.students.aggregate([
  { $count: "total_students" }
]);
```

---

# 2.9 Stage 8 — $addFields

Add calculated fields.

Example: Add pass/fail based on Math score:

```js
db.students.aggregate([
  {
    $addFields: {
      pass_in_math: {
        $gt: [
          {
            $max: {
              $map: {
                input: "$scores",
                as: "s",
                in: {
                  $cond: [
                    { $eq: ["$$s.subject", "Math"] },
                    "$$s.score",
                    0
                  ]
                }
              }
            }
          },
          40
        ]
      }
    }
  }
]);
```

---

# MINI CASE STUDY:

"Department-Level Score Summary"

Let’s say we add department info (referenced earlier). Even without real references, we can simulate using `$match` and `$unwind`.

### Goal:

Find **average subject score** across all students for each subject.

### Query:

```js
db.students.aggregate([
  { $unwind: "$scores" },
  {
    $group: {
      _id: "$scores.subject",
      avg_score: { $avg: "$scores.score" }
    }
  },
  { $sort: { avg_score: -1 } }
]);
```

### Meaning:

* First explode scores array
* Then group by subject
* Calculate average
* Sort by average (highest first)

---

# STEP 2 Assignments

### **(Easy)**

**Question:**
Count how many students have “Math” as a subject.

**Solution:**

```js
db.students.aggregate([
  { $match: { subjects: "Math" } },
  { $count: "total_students_with_math" }
]);
```

Expected result:

```
{ total_students_with_math: 9 }
```

(Your dataset has 9 students with Math.)

---

### **(Medium)**

**Question:**
Find the **average score in each subject**.

**Hint:** Use `$unwind` + `$group`.

**Solution:**

```js
db.students.aggregate([
  { $unwind: "$scores" },
  {
    $group: {
      _id: "$scores.subject",
      avg_score: { $avg: "$scores.score" }
    }
  }
]);
```

Expected output (approx):

| Subject       | Avg Score    |
| ------------- | ------------ |
| Math          | approx 85–90 |
| AI            | approx 88    |
| NLP           | 91           |
| Deep Learning | 94           |
| etc           |              |

---

### **(Hard)**

**Question:**
Find the **top 3 students** with the **highest total score across all their subjects**.

**Solution:**

```js
db.students.aggregate([
  { $unwind: "$scores" },
  {
    $group: {
      _id: "$name",
      total_score: { $sum: "$scores.score" }
    }
  },
  { $sort: { total_score: -1 } },
  { $limit: 3 }
]);
```

What this does:

* Unwinds scores
* Sums score of each student
* Sorts in descending order
* Takes top 3

Expected winners (based on your dataset):

1. Sneha
2. Karan
3. Aniket

---
---

# STEP 3 — MongoDB Aggregation Pipeline

The Aggregation Pipeline allows you to process documents through **multiple stages**, just like water flowing through pipes.

Think of it as a sequence:

```
Data → Filter → Transform → Group → Sort → Output
```

A real pipeline might look like:

```js
db.students.aggregate([
  { $match: {...} },
  { $unwind: "$scores" },
  { $group: {...} },
  { $sort: {...} }
]);
```

Each stage takes input from the previous stage.

We will build pipelines step-by-step.

---

# 3.1 Pipeline Structure Basics

A full aggregation pipeline looks like this:

```js
db.students.aggregate([
  { $match: { age: { $gt: 20 } } },
  { $project: { name: 1, age: 1 } },
  { $sort: { age: -1 } }
]);
```

| Pipeline Stage | Meaning                                  |
| -------------- | ---------------------------------------- |
| `$match`       | Filter documents                         |
| `$project`     | Select / reshape fields                  |
| `$unwind`      | Break arrays into rows                   |
| `$group`       | Summarize                                |
| `$sort`        | Order                                    |
| `$limit`       | Take top N                               |
| `$lookup`      | Join other collections (later if needed) |

---

# 3.2 Example 1 — Compute Average Math Score Across All Students

Goal:

* Explode scores
* Match only Math
* Calculate average

Pipeline:

```js
db.students.aggregate([
  { $unwind: "$scores" },
  { $match: { "scores.subject": "Math" } },
  {
    $group: {
      _id: "Math",
      avg_math_score: { $avg: "$scores.score" }
    }
  }
]);
```

Explanation:

1. `$unwind` creates one document per score
2. `$match` keeps only Math
3. `$group` calculates the average

---

# 3.3 Example 2 — Find Highest Score Per Subject (End-to-End Pipeline)

Pipeline:

```js
db.students.aggregate([
  { $unwind: "$scores" },
  {
    $group: {
      _id: "$scores.subject",
      highest: { $max: "$scores.score" }
    }
  },
  { $sort: { highest: -1 } }
]);
```

Output example:

| Subject       | Highest Score |
| ------------- | ------------- |
| Deep Learning | 94            |
| NLP           | 91            |
| Math          | 95            |
| Security      | 90            |
| ML            | 92            |

---

# 3.4 Example 3 — Total Score of Each Student

Pipeline:

```js
db.students.aggregate([
  { $unwind: "$scores" },
  {
    $group: {
      _id: "$name",
      total_score: { $sum: "$scores.score" }
    }
  },
  { $sort: { total_score: -1 } }
]);
```

Top scorers likely:

1. Sneha
2. Karan
3. Aniket

---

# 3.5 Example 4 — Students With Their Highest Single Score

Pipeline:

```js
db.students.aggregate([
  { $unwind: "$scores" },
  {
    $group: {
      _id: "$name",
      highest_score: { $max: "$scores.score" }
    }
  },
  { $sort: { highest_score: -1 } }
]);
```

---

# 3.6 Example 5 — Count Students By Number of Subjects

Pipeline:

```js
db.students.aggregate([
  {
    $project: {
      name: 1,
      no_of_subjects: { $size: "$subjects" }
    }
  },
  {
    $group: {
      _id: "$no_of_subjects",
      student_count: { $sum: 1 }
    }
  },
  { $sort: { _id: 1 } }
]);
```

---

# 3.7 Real Case Study

**"Identify top performers by department and subject"**

Even though we didn't add department references earlier, we can simulate them.

Goal:

1. Unwind scores
2. Sort by score
3. Group by subject
4. Take highest scorer in each subject

Pipeline:

```js
db.students.aggregate([
  { $unwind: "$scores" },
  { $sort: { "scores.score": -1 } },
  {
    $group: {
      _id: "$scores.subject",
      topper: { $first: "$name" },
      highest_score: { $first: "$scores.score" }
    }
  },
  { $sort: { _id: 1 } }
]);
```

This outputs:

| Subject       | Topper | Score |
| ------------- | ------ | ----- |
| AI            | Aniket | 88    |
| Algebra       | Arjun  | 92    |
| Deep Learning | Karan  | 94    |
| Math          | Aniket | 95    |
| ML            | Priya  | 92    |
| NLP           | Karan  | 91    |

---

# 3.8 Example 6 — Apply Multiple Filters

Goal:

* Students above 20
* Score more than 90 in any subject
* Sort by age

Pipeline:

```js
db.students.aggregate([
  { $match: { age: { $gt: 20 } } },
  { $unwind: "$scores" },
  { $match: { "scores.score": { $gt: 90 } } },
  {
    $project: {
      name: 1,
      age: 1,
      subject: "$scores.subject",
      score: "$scores.score"
    }
  },
  { $sort: { age: -1 } }
]);
```

---

# STEP 3 Assignments

---

## **(Easy)**

**Question:**
Build a pipeline to show:

* Student name
* Total number of subjects they study

**Solution:**

```js
db.students.aggregate([
  {
    $project: {
      name: 1,
      number_of_subjects: { $size: "$subjects" }
    }
  }
]);
```

**What students learn:** `$project` + `$size`.

---

## **(Medium)**

**Question:**
Find **average score per student**, and sort in descending order.

**Solution:**

```js
db.students.aggregate([
  { $unwind: "$scores" },
  {
    $group: {
      _id: "$name",
      avg_score: { $avg: "$scores.score" }
    }
  },
  { $sort: { avg_score: -1 } }
]);
```

**What students learn:** `$unwind`, `$group`, `$avg`, `$sort`.

---

## **(Hard)**

**Question:**
Find **the top student in each subject**, including:

* subject
* student name
* highest score

**Solution:**

```js
db.students.aggregate([
  { $unwind: "$scores" },
  { $sort: { "scores.score": -1 } },
  {
    $group: {
      _id: "$scores.subject",
      topper: { $first: "$name" },
      highest_score: { $first: "$scores.score" }
    }
  },
  { $sort: { _id: 1 } }
]);
```
---
---

# STEP 4 — COMPLETE CASE STUDY

**University Analytics Dashboard using MongoDB Aggregation Pipelines**

Goal: Build a set of meaningful analytics reports that a university might need, using your `students` dataset.

We will produce:

1. Student Performance Dashboard
2. Subject-Level Analytics
3. Department-Level Insights
4. Ranking & Score Distribution
5. Graduation Analytics
6. Advanced Multi-Stage Pipelines
7. Assignments (1e, 1m, 1h) with full solutions

---

# 4.1 Report 1 — Student Performance Dashboard

### A. Total Score per Student

```js
db.students.aggregate([
  { $unwind: "$scores" },
  {
    $group: {
      _id: "$name",
      total_score: { $sum: "$scores.score" }
    }
  },
  { $sort: { total_score: -1 } }
]);
```

### B. Average Score per Student

```js
db.students.aggregate([
  { $unwind: "$scores" },
  {
    $group: {
      _id: "$name",
      avg_score: { $avg: "$scores.score" }
    }
  },
  { $sort: { avg_score: -1 } }
]);
```

### C. Highest Single Score per Student

```js
db.students.aggregate([
  { $unwind: "$scores" },
  {
    $group: {
      _id: "$name",
      highest: { $max: "$scores.score" }
    }
  },
  { $sort: { highest: -1 } }
]);
```

---

# 4.2 Report 2 — Subject-Level Analytics

### A. Highest Score in Each Subject

```js
db.students.aggregate([
  { $unwind: "$scores" },
  { $sort: { "scores.score": -1 } },
  {
    $group: {
      _id: "$scores.subject",
      topper: { $first: "$name" },
      highest_score: { $first: "$scores.score" }
    }
  },
  { $sort: { _id: 1 } }
]);
```

### B. Average Score in Each Subject

```js
db.students.aggregate([
  { $unwind: "$scores" },
  {
    $group: {
      _id: "$scores.subject",
      avg_score: { $avg: "$scores.score" }
    }
  },
  { $sort: { avg_score: -1 } }
]);
```

### C. Number of Students Taking Each Subject

```js
db.students.aggregate([
  { $unwind: "$subjects" },
  {
    $group: {
      _id: "$subjects",
      student_count: { $sum: 1 }
    }
  },
  { $sort: { student_count: -1 } }
]);
```

---

# 4.3 Report 3 — Department-Level Insights (Simulated)

Since we do not have departments linked, we simulate by grouping by major.

### A. Number of Students per Major

```js
db.students.aggregate([
  {
    $group: {
      _id: "$major",
      count: { $sum: 1 }
    }
  },
  { $sort: { count: -1 } }
]);
```

### B. Average Score per Major

```js
db.students.aggregate([
  { $unwind: "$scores" },
  {
    $group: {
      _id: "$major",
      avg_score: { $avg: "$scores.score" }
    }
  },
  { $sort: { avg_score: -1 } }
]);
```

---

# 4.4 Report 4 — Ranking & Score Distribution

### A. Rank Students by Total Score

```js
db.students.aggregate([
  { $unwind: "$scores" },
  {
    $group: {
      _id: "$name",
      total_score: { $sum: "$scores.score" }
    }
  },
  { $sort: { total_score: -1 } }
]);
```

### B. Score Distribution Buckets (0–70, 70–80, 80–90, 90+)

```js
db.students.aggregate([
  { $unwind: "$scores" },
  {
    $bucket: {
      groupBy: "$scores.score",
      boundaries: [0, 70, 80, 90, 101],
      default: "Invalid",
      output: {
        count: { $sum: 1 }
      }
    }
  }
]);
```

---

# 4.5 Report 5 — Graduation Insights

### A. Students Who Graduated Before 2024

```js
db.students.aggregate([
  {
    $match: {
      graduation_Date: { $exists: true, $lt: ISODate("2024-01-01") }
    }
  },
  { $project: { name: 1, graduation_Date: 1 } }
]);
```

### B. Students Without Graduation Date (Still Studying)

```js
db.students.aggregate([
  {
    $match: {
      graduation_Date: { $exists: false }
    }
  },
  { $project: { name: 1, major: 1 } }
]);
```

---

# 4.6 Advanced Multi-Stage Report

**"Top Students per Subject and Their Average Performance Compared to Others"**

Goal:

* Find each subject
* Identify topper
* Compute average
* Show gap between topper and average

Pipeline:

```js
db.students.aggregate([
  { $unwind: "$scores" },

  // Step 1: Compute average per subject
  {
    $group: {
      _id: "$scores.subject",
      avg_subject_score: { $avg: "$scores.score" },
      scores: { $push: { name: "$name", score: "$scores.score" } }
    }
  },

  // Step 2: Extract topper from scores array
  {
    $project: {
      avg_subject_score: 1,
      scores: 1,
      topper: {
        $first: {
          $sortArray: {
            input: "$scores",
            sortBy: { score: -1 }
          }
        }
      }
    }
  },

  // Step 3: Add performance gap
  {
    $addFields: {
      performance_gap: {
        $subtract: ["$topper.score", "$avg_subject_score"]
      }
    }
  },

  { $sort: { performance_gap: -1 } }
]);
```

Output fields:

* Subject
* Topper
* Avg score
* Gap between topper and average

---

# STEP 4 Assignments

---

## **(Easy)**

**Question:**
Create a report showing each student’s **highest score**, sorted highest to lowest.

**Solution:**

```js
db.students.aggregate([
  { $unwind: "$scores" },
  {
    $group: {
      _id: "$name",
      highest_score: { $max: "$scores.score" }
    }
  },
  { $sort: { highest_score: -1 } }
]);
```

---

## **(Medium)**

**Question:**
Find the **top 2 subjects** with the **highest average scores**.

**Solution:**

```js
db.students.aggregate([
  { $unwind: "$scores" },
  {
    $group: {
      _id: "$scores.subject",
      avg_score: { $avg: "$scores.score" }
    }
  },
  { $sort: { avg_score: -1 } },
  { $limit: 2 }
]);
```

---

## **(Hard)**

**Question:**
Generate a full report:

For each **student**:

* total score
* highest score
* average score
* number of subjects
* rank based on total score

**Solution (advanced):**

```js
db.students.aggregate([

  // Flatten scores
  { $unwind: "$scores" },

  // Group to compute total, highest, avg
  {
    $group: {
      _id: "$name",
      total_score: { $sum: "$scores.score" },
      highest_score: { $max: "$scores.score" },
      avg_score: { $avg: "$scores.score" },
      subject_count: { $sum: 1 }
    }
  },

  // Sort by total score
  { $sort: { total_score: -1 } },

  // Add rank numbers
  {
    $setWindowFields: {
      sortBy: { total_score: -1 },
      output: {
        rank: { $rank: {} }
      }
    }
  }
]);
```

---
---
