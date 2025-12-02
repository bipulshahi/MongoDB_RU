# Database Setup
---

# **1. Database Creation + Sample Data**

We create a fresh database: `universityDB`

```js
use universityDB
```

Create a `students` collection with **simple structured data**:

```js
db.students.insertMany([
  {
    name: "Aniket",
    age: 22,
    major: "Computer Science",
    credits_earned: 82,
    scores: [
      { subject: "Math", score: 92 },
      { subject: "AI", score: 88 }
    ]
  },
  {
    name: "Riya",
    age: 19,
    major: "Physics",
    credits_earned: 58,
    scores: [
      { subject: "Math", score: 85 },
      { subject: "Physics", score: 89 }
    ]
  },
  {
    name: "Tanya",
    age: 24,
    major: "Electronics",
    credits_earned: 40,
    scores: [
      { subject: "Circuits", score: 71 }
    ]
  },
  {
    name: "Arjun",
    age: 20,
    major: "Mathematics",
    credits_earned: 96,
    scores: [
      { subject: "Algebra", score: 94 },
      { subject: "Calculus", score: 90 }
    ]
  }
]);
```

This dataset is intentionally compact so that we can understand the operations clearly.

---

# **CASE STUDY 1 — Querying Related Documents**

Goal: Learn querying techniques on nested fields and arrays.

### **Problem Statement**

You are asked to retrieve student performance details for reporting.
Queries needed:

1. Students who studied "Math"
2. Students whose Math score is above 90
3. Students aged below 22 and having Physics subject
4. Show only: name + major + Math score
---

### **Query 1 — Students who studied Math**

```js
db.students.find({ "scores.subject": "Math" });
```

Why it works:
`scores.subject` checks inside array of objects.

---

### **Query 2 — Math score > 90**

```js
db.students.find({
  scores: { $elemMatch: { subject: "Math", score: { $gt: 90 } } }
});
```

---

### **Query 3 — Age < 22 AND Physics subject**

```js
db.students.find({
  age: { $lt: 22 },
  "scores.subject": "Physics"
});
```

---

### **Query 4 — Show name + major + Math score only**

Aggregation required because `.find()` cannot extract filtered array items.

```js
db.students.aggregate([
  { $unwind: "$scores" },
  { $match: { "scores.subject": "Math" } },
  {
    $project: {
      name: 1,
      major: 1,
      math_score: "$scores.score",
      _id: 0
    }
  }
]);
```

This output **cannot be produced using find()**.

---

**one simple case study** that uses only:

* `$unwind`
* `$group`
* `$addFields`
* `$round`
* `$project`

And it includes **performance_category** based on rounded average score.

---

# **CASE STUDY — Student Performance Categorization**

### **Problem Statement**

The university wants a very simple report showing:

* Student Name
* Average Score (rounded to nearest whole number)
* Performance Category (Low / Medium / High)

Performance rules:

| Rounded Avg Score | Category |
| ----------------- | -------- |
| 0–70              | Low      |
| 71–85             | Medium   |
| > 85              | High     |

---

# **Solution Pipeline (Very Simple)**

```js
db.students.aggregate([
  { $unwind: "$scores" },

  // Step 1: Calculate average score per student
  {
    $group: {
      _id: "$name",
      avg_score: { $avg: "$scores.score" }
    }
  },

  // Step 2: Add rounded score
  {
    $addFields: {
      avg_score_rounded: { $round: ["$avg_score", 0] }
    }
  },

  // Step 3: Add performance category
  {
    $addFields: {
      performance_category: {
        $cond: [
          { $lte: ["$avg_score_rounded", 70] }, "Low",
          {
            $cond: [
              { $lte: ["$avg_score_rounded", 85] }, "Medium",
              "High"
            ]
          }
        ]
      }
    }
  },

  // Step 4: Clean output
  {
    $project: {
      name: "$_id",
      avg_score_rounded: 1,
      performance_category: 1,
      _id: 0
    }
  }
]);
```

---

# **Expected Output (Sample)**

```
{
  name: "Aniket",
  avg_score_rounded: 90,
  performance_category: "High"
}
{
  name: "Riya",
  avg_score_rounded: 87,
  performance_category: "High"
}
{
  name: "Tanya",
  avg_score_rounded: 71,
  performance_category: "Medium"
}
{
  name: "Arjun",
  avg_score_rounded: 92,
  performance_category: "High"
}
```

---


# **CASE STUDY 2 — Tag Levels: Small, Medium, High**

We will assign tags based on **credits_earned**:

* Credits ≤ 50  → "Small"
* 51–80         → "Medium"
* > 80          → "High"

Also use `$round()` to create “normalized score”.

### **Problem Statement**

Generate a report showing:

* Student name
* credits_earned
* level_tag (Small/Medium/High)
* average_score (rounded to 2 decimal places)

This involves:

* `$unwind`
* `$group`
* `$addFields`
* `$cond`
* `$round`

### **Solution Pipeline**

```js
db.students.aggregate([
  { $unwind: "$scores" },

  // Step 1: compute average score
  {
    $group: {
      _id: "$name",
      major: { $first: "$major" },
      credits_earned: { $first: "$credits_earned" },
      avg_score: { $avg: "$scores.score" }
    }
  },

  // Step 2: add categories + rounded score
  {
    $addFields: {

      // Tag level based on credits
      level_tag: {
        $cond: [
          { $lte: ["$credits_earned", 50] }, "Small",
          {
            $cond: [
              { $lte: ["$credits_earned", 80] }, "Medium",
              "High"
            ]
          }
        ]
      },

      // Rounded average score
      avg_score_rounded: { $round: ["$avg_score", 2] }
    }
  },

  // Optional: project final output
  {
    $project: {
      name: "$_id",
      major: 1,
      credits_earned: 1,
      level_tag: 1,
      avg_score_rounded: 1,
      _id: 0
    }
  }
]);
```

### Expected Output:

```
{
  name: "Aniket",
  major: "Computer Science",
  credits_earned: 82,
  level_tag: "High",
  avg_score_rounded: 90
}
{
  name: "Riya",
  major: "Physics",
  credits_earned: 58,
  level_tag: "Medium",
  avg_score_rounded: 87
}
{
  name: "Tanya",
  major: "Electronics",
  credits_earned: 40,
  level_tag: "Small",
  avg_score_rounded: 71
}
{
  name: "Arjun",
  major: "Mathematics",
  credits_earned: 96,
  level_tag: "High",
  avg_score_rounded: 92
}
```

---
* Querying related documents
* Aggregation methods
* Full aggregation pipelines
* `$match`, `$project`, `$group`, `$unwind`, `$addFields`, `$round`, `$sort`
* Tagging (Small / Medium / High / Outstanding)
* Simple business-like reporting

All examples will continue using your existing **universityDB.students** dataset.

---
---

# **CASE STUDY 3 — Score-Based Student Classification Report**

Goal:
Classify students into categories based on **their average subject score**.

Levels:

| Average Score | Category    |
| ------------- | ----------- |
| 0–70          | Weak        |
| 71–85         | Average     |
| 86–95         | Strong      |
| >95           | Outstanding |

We will generate a report:

* name
* avg_score (rounded to 2 decimals)
* performance_category
* subject_count

This is a perfect use of:

* `$unwind`
* `$group`
* `$round`
* `$addFields`
* `$cond` (nested conditions)
* `$project`

---

## **Solution Pipeline**

```js
db.students.aggregate([
  { $unwind: "$scores" },

  // Step 1 — Calculate average score and subject count
  {
    $group: {
      _id: "$name",
      avg_score: { $avg: "$scores.score" },
      subject_count: { $sum: 1 }
    }
  },

  // Step 2 — Round average score
  {
    $addFields: {
      avg_score_rounded: { $round: ["$avg_score", 2] }
    }
  },

  // Step 3 — Assign categories based on score
  {
    $addFields: {
      performance_category: {
        $cond: [
          { $lte: ["$avg_score_rounded", 70] }, "Weak",
          {
            $cond: [
              { $lte: ["$avg_score_rounded", 85] }, "Average",
              {
                $cond: [
                  { $lte: ["$avg_score_rounded", 95] }, "Strong",
                  "Outstanding"
                ]
              }
            ]
          }
        ]
      }
    }
  },

  // Step 4 — Final projection
  {
    $project: {
      name: "$_id",
      subject_count: 1,
      avg_score_rounded: 1,
      performance_category: 1,
      _id: 0
    }
  }
]);
```

---

## **Expected Output**

```
{
  name: "Aniket",
  subject_count: 2,
  avg_score_rounded: 90,
  performance_category: "Strong"
}
{
  name: "Riya",
  subject_count: 2,
  avg_score_rounded: 87,
  performance_category: "Strong"
}
{
  name: "Tanya",
  subject_count: 1,
  avg_score_rounded: 71,
  performance_category: "Average"
}
{
  name: "Arjun",
  subject_count: 2,
  avg_score_rounded: 92,
  performance_category: "Strong"
}
```

---

## **What We Will Learn Here**

* Conditional classification
* Multi-stage pipeline
* Using `$round` effectively
* Using `$addFields` multiple times
* Grouping + projection

---

# **CASE STUDY 4 — Subject Difficulty Analysis Report**

Goal:
Identify which subjects appear to be “harder” based on **overall performance**.

Difficulty logic:

| Average Score | Difficulty Level |
| ------------- | ---------------- |
| > 90          | Easy             |
| 80–90         | Moderate         |
| < 80          | Hard             |

We will compute:

* subject
* average score (rounded)
* highest score
* lowest score
* number of students
* difficulty_level

This requires:

* `$unwind`
* `$group`
* `$round`
* `$addFields`
* `$project`
* `$sort`

---

## **Solution Pipeline**

```js
db.students.aggregate([
  { $unwind: "$scores" },

  // Step 1 — Group by subject
  {
    $group: {
      _id: "$scores.subject",
      avg_subject_score: { $avg: "$scores.score" },
      max_subject_score: { $max: "$scores.score" },
      min_subject_score: { $min: "$scores.score" },
      student_count: { $sum: 1 }
    }
  },

  // Step 2 — Round the average
  {
    $addFields: {
      avg_subject_score_rounded: { $round: ["$avg_subject_score", 2] }
    }
  },

  // Step 3 — Add difficulty level
  {
    $addFields: {
      difficulty_level: {
        $cond: [
          { $gt: ["$avg_subject_score_rounded", 90] }, "Easy",
          {
            $cond: [
              { $gte: ["$avg_subject_score_rounded", 80] }, "Moderate",
              "Hard"
            ]
          }
        ]
      }
    }
  },

  // Step 4 — Final projection
  {
    $project: {
      subject: "$_id",
      avg_subject_score_rounded: 1,
      max_subject_score: 1,
      min_subject_score: 1,
      student_count: 1,
      difficulty_level: 1,
      _id: 0
    }
  },

  // Step 5 — Sort by avg score
  { $sort: { avg_subject_score_rounded: -1 } }
]);
```

---

## **Expected Output** (based on our dataset)

```
{
  subject: "Algebra",
  avg_subject_score_rounded: 94,
  max_subject_score: 94,
  min_subject_score: 94,
  student_count: 1,
  difficulty_level: "Easy"
}
{
  subject: "AI",
  avg_subject_score_rounded: 88,
  difficulty_level: "Moderate"
}
{
  subject: "Math",
  avg_subject_score_rounded: 88.5,
  difficulty_level: "Moderate"
}
{
  subject: "Physics",
  avg_subject_score_rounded: 89,
  difficulty_level: "Moderate"
}
{
  subject: "Circuits",
  avg_subject_score_rounded: 71,
  difficulty_level: "Hard"
}
```

---

## **What We Learn Here**

* Multi-metric grouping
* Difficulty classification using conditions
* Using `$round`
* Sorting results
* Reshaping output with `$project`

---
Great — moving to the final section.

---

# **CASE STUDY 5 — Student Risk & Performance Analytics Dashboard**

**Goal:**
The University Academic Council wants a dashboard showing each student's:

1. Total score
2. Average score (rounded)
3. Highest and Lowest subject score
4. Performance tag (High / Medium / Low)
5. Risk level based on credits_earned
6. Final ranking based on total score

This is a **comprehensive pipeline** touching most core aggregation concepts.

---

# **Problem Breakdown**

We will compute:

### A. total_score

Sum of all subject scores.

### B. avg_score_rounded

Average score rounded to 2 decimals.

### C. max_score & min_score

Highest and lowest score per student.

### D. performance_tag

Based on average score:

| Avg Score | Tag    |
| --------- | ------ |
| > 90      | High   |
| 75–90     | Medium |
| < 75      | Low    |

### E. risk_level (based on credits earned)

| credits_earned | risk_level |
| -------------- | ---------- |
| < 50           | Critical   |
| 50–70          | Moderate   |
| > 70           | Safe       |

### F. rank

Rank students by total score (1 = highest).

---

# **Final Aggregation Pipeline (Complete Solution)**

```js
db.students.aggregate([

  // Step 1: Unwind scores array
  { $unwind: "$scores" },

  // Step 2: Group per student
  {
    $group: {
      _id: "$name",
      major: { $first: "$major" },
      credits_earned: { $first: "$credits_earned" },
      total_score: { $sum: "$scores.score" },
      avg_score: { $avg: "$scores.score" },
      max_score: { $max: "$scores.score" },
      min_score: { $min: "$scores.score" }
    }
  },

  // Step 3: Round average score
  {
    $addFields: {
      avg_score_rounded: { $round: ["$avg_score", 2] }
    }
  },

  // Step 4: Add performance tag (High, Medium, Low)
  {
    $addFields: {
      performance_tag: {
        $cond: [
          { $gt: ["$avg_score_rounded", 90] }, "High",
          {
            $cond: [
              { $gte: ["$avg_score_rounded", 75] }, "Medium",
              "Low"
            ]
          }
        ]
      }
    }
  },

  // Step 5: Add risk level based on credits_earned
  {
    $addFields: {
      risk_level: {
        $cond: [
          { $lt: ["$credits_earned", 50] }, "Critical",
          {
            $cond: [
              { $lte: ["$credits_earned", 70] }, "Moderate",
              "Safe"
            ]
          }
        ]
      }
    }
  },

  // Step 6: Ranking using setWindowFields
  {
    $setWindowFields: {
      sortBy: { total_score: -1 },
      output: {
        rank: { $rank: {} }
      }
    }
  },

  // Step 7: Final projection for clean output
  {
    $project: {
      name: "$_id",
      major: 1,
      credits_earned: 1,
      total_score: 1,
      avg_score_rounded: 1,
      max_score: 1,
      min_score: 1,
      performance_tag: 1,
      risk_level: 1,
      rank: 1,
      _id: 0
    }
  }
]);
```

---

# **Expected Output (Based on Your Dataset)**

```
{
  name: "Arjun",
  major: "Mathematics",
  credits_earned: 96,
  total_score: 184,
  avg_score_rounded: 92,
  max_score: 94,
  min_score: 90,
  performance_tag: "High",
  risk_level: "Safe",
  rank: 1
}
{
  name: "Aniket",
  major: "Computer Science",
  credits_earned: 82,
  total_score: 180,
  avg_score_rounded: 90,
  max_score: 92,
  min_score: 88,
  performance_tag: "Medium",
  risk_level: "Safe",
  rank: 2
}
{
  name: "Riya",
  major: "Physics",
  credits_earned: 58,
  total_score: 174,
  avg_score_rounded: 87,
  max_score: 89,
  min_score: 85,
  performance_tag: "Medium",
  risk_level: "Moderate",
  rank: 3
}
{
  name: "Tanya",
  major: "Electronics",
  credits_earned: 40,
  total_score: 71,
  avg_score_rounded: 71,
  performance_tag: "Low",
  risk_level: "Critical",
  rank: 4
}
```

---

# **Why This Case Study Is Perfect**

It uses nearly everything:

| Aggregation Feature          | Used? |
| ---------------------------- | ----- |
| Querying related docs        | ✔     |
| $match                       | ✔     |
| $project                     | ✔     |
| $group                       | ✔     |
| $unwind                      | ✔     |
| $addFields                   | ✔     |
| Conditional tagging          | ✔     |
| $round                       | ✔     |
| $setWindowFields (ranking)   | ✔     |
| Multi-stage pipeline         | ✔     |
| Realistic analytics scenario | ✔     |

---



