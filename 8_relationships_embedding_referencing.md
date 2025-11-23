## MongoDB Relationships (Using `students` Data)

### 1.1 What is a "relationship" in MongoDB?

In SQL, relationships = joins between tables (one-to-one, one-to-many, many-to-many).

In MongoDB, we have **documents** in **collections**. There are no built-in “joins” like SQL, but data can still be related in 3 main ways:

1. **One-to-One (1–1)**
   One student → one identity card
2. **One-to-Many (1–M)**
   One student → many subjects or many scores
3. **Many-to-Many (M–M)**
   Many students ↔ many courses (each student can take many courses, each course has many students)

MongoDB supports these using:

* **Embedding** (put documents inside another document)
* **Referencing** (store an `_id` or some reference to another document)

We will use your `students` documents to see these.

---

### 1.2 Existing relationships inside your `students` document

Take this example:

```js
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
}
```

We can see several relationships just inside this one document.

#### Relationship A: Student → Subjects (One-to-Many)

* One student (Aniket)
* Many subjects: `["Programming", "Database", "AI"]`

This is a **1–M relationship**, modeled as an **array of values**.

We could imagine another collection `subjects`, but here you chose to **embed just the names** as an array of strings.

---

#### Relationship B: Student → Scores (One-to-Many with embedded documents)

* One student (Aniket)
* Many score entries:

  ```js
  scores: [
    { subject: "Math", score: 95 },
    { subject: "AI", score: 88 },
    { subject: "Database", score: 92 }
  ]
  ```

This is also **1–M**, but now each item is a **subdocument** (`{ subject, score }`), not just a string.

So we already see **MongoDB embedding** being used.

---

#### Relationship C: Student → Graduation details (One-to-One)

For those students who have `graduation_Date`:

```js
graduation_Date: ISODate("2024-07-01")
```

We can think of this as:

* One student → one graduation record (simple date field now, but we could later expand it).

This is a **1–1** relationship modeled as a **field**.

---

### 1.3 Imagining more collections for our case study

Right now we only have one collection: `students`.

We will **imagine** we are designing more collections for a **“University Student Portal”**:

* `students` – already exists (your data)
* `courses` – information about each course
* `departments` – departmental information
* `enrollments` – who enrolled in which course (to handle many-to-many)

We will not implement all of them now, but we will use them conceptually in later steps when we discuss **embedding vs referencing**.

---

### 1.4 Quick mental map of relationships in our system

Think about possible relationships:

1. **Department ↔ Student**

   * One Department has many Students
   * One Student belongs to one Department
     → 1–M (Department → Students)

2. **Student ↔ Course**

   * One Student can enroll in many Courses
   * One Course can have many Students
     → M–M (Students ↔ Courses)

3. **Student ↔ Scores**

   * One Student has many Scores
     → 1–M (Student → Scores)

4. **Student ↔ Graduation Record**

   * One Student has at most one Graduation detail
     → 1–1

We will use these in Steps 2 and 3 to design **embedded** and **referenced** schemas.

---

### Easy – Identify Relationship Types in Your Existing Data

**Question (1e):**
Look at this document (Priya):

```js
{
   name: "Priya",
   age: 26,
   major: "Computer Science",
   subjects: ["Programming", "AI", "ML"],
   scores: [
     { subject: "Math", score: 80 },
     { subject: "ML", score: 92 }
   ],
   graduation_Date: ISODate("2023-05-20")
}
```

Q1. What is the relationship between **Priya** and her `subjects` array?
Q2. What is the relationship between **Priya** and her `scores` array?
Q3. What is the relationship between **Priya** and `graduation_Date`?

**Solution (1e):**

1. **Priya → subjects**:

   * One student (Priya) → many subjects.
   * Relationship type: **One-to-Many (1–M)**.
   * Implemented as: **Array of strings**.

2. **Priya → scores**:

   * One student (Priya) → many score records.
   * Relationship type: **One-to-Many (1–M)**.
   * Implemented as: **Array of embedded documents**.

3. **Priya → graduation_Date**:

   * One student (Priya) → one graduation record (date).
   * Relationship type: **One-to-One (1–1)**.
   * Implemented as: **Simple field in the same document**.

---

### Medium – Design a New Relationship Using Text

**Question (1m):**
We want to add **“Student Identity Card”** information. Each student will have exactly one ID card with these fields:

* `id_card_number` (string, unique)
* `issue_date` (date)
* `valid_till` (date)

Task:
Q1. Is “Student → ID Card” a 1–1, 1–M, or M–M relationship?
Q2. For a beginner-friendly design, would you **embed** the ID card inside the `students` document or create a separate `id_cards` collection and **reference** it?
Q3. Write a sample MongoDB document for student `"Riya"` with an embedded `id_card` object.

**Solution (1m):**

1. **Relationship type**

   * Each student has one ID card.
   * Relationship: **One-to-One (1–1)**.

2. **Embed or reference?**

   * Data is tightly related to the student and used almost always with the student.
   * Size is small, not many nested levels.
   * Beginner-friendly, simpler queries if embedded.
     → **Embedding is a good choice**.

3. **Sample document with embedded `id_card`**

```js
db.students.updateOne(
  { name: "Riya" },
  {
    $set: {
      id_card: {
        id_card_number: "ID2025RIYA001",
        issue_date: ISODate("2025-01-10"),
        valid_till: ISODate("2029-01-10")
      }
    }
  }
);
```

After this update, Riya’s document will look like:

```js
{
  name: "Riya",
  age: 17,
  major: "Physics",
  subjects: ["Math", "Physics", "Chemistry"],
  scores: [
    { subject: "Math", score: 85 },
    { subject: "Physics", score: 89 }
  ],
  id_card: {
    id_card_number: "ID2025RIYA001",
    issue_date: ISODate("2025-01-10"),
    valid_till: ISODate("2029-01-10")
  }
}
```

---

### Hard – Think About Many-to-Many (Students ↔ Courses)

**Question (1h):**
Assume we now create a new collection `courses`:

```js
db.courses.insertMany([
  {
    _id: ObjectId("66c1c1111111111111111111"),
    code: "CS101",
    title: "Intro to Programming",
    credits: 3
  },
  {
    _id: ObjectId("66c1c1222222222222222222"),
    code: "AI201",
    title: "Introduction to AI",
    credits: 4
  }
]);
```

We know:

* One student can enroll in many courses.
* One course can have many students.

Q1. What type of relationship is “Students ↔ Courses”?
Q2. Propose **two possible ways** to model this relationship in MongoDB (just explain in words and with small structure examples, not full code):

* Option A: store course references inside `students`
* Option B: use a separate `enrollments` collection
  Q3. For a system with **thousands of students and courses**, which option is more flexible and why?

**Solution (1h):**

1. **Relationship type**

* Many students can enroll in the same course.
* One student can enroll in many courses.
  → This is a **Many-to-Many (M–M)** relationship.

---

2. **Two modeling options**

**Option A – Store course references inside `students`**

Each student document keeps an array of course IDs:

```js
{
  name: "Aniket",
  // other fields...
  enrolled_course_ids: [
    ObjectId("66c1c1111111111111111111"), // CS101
    ObjectId("66c1c1222222222222222222")  // AI201
  ]
}
```

Pros:

* Easy to find which courses a given student is enrolled in.
* Fewer collections to manage.

Cons:

* Harder to answer questions like “list all students in course CS101” without scanning many student documents.
* If we also want per-course information (like semester, marks per course, dropped/active status), this array can become very complex.

---

**Option B – Use an `enrollments` collection (recommended for large systems)**

Create a separate collection:

```js
db.enrollments.insertOne({
  student_id: ObjectId("...AniketId..."),
  course_id: ObjectId("66c1c1111111111111111111"), // CS101
  semester: "2025-Spring",
  status: "active",
  grade: "A"
});
```

There will be one `enrollments` document per student–course pair.

Pros:

* Very flexible: we can easily add more fields (`semester`, `status`, `grade`).
* Easy to query either direction:

  * All courses of a student → filter `student_id`
  * All students in a course → filter `course_id`
* Better for large-scale systems with many relations.

Cons:

* More collections (`students`, `courses`, `enrollments`).
* Slightly more complex queries (may require multiple find calls or `$lookup`).

---

3. **Which is more flexible for thousands of students and courses?**

For large scale, **Option B: `enrollments` collection** is more flexible because:

* Handles growth (many enrollments) without making student documents too big.
* Easier to maintain, index, and query from both sides.
* Supports additional attributes (grades, semester, status) without cluttering the student or course documents.

So for a **real university portal**, a separate `enrollments` collection is a better long-term design.

---
---

# **MongoDB Embedding Documents**

Focus area: *When to embed, how to embed, how to query embedded fields, plus assignments.*

We will continue using your `students` dataset and expand it logically as if this were a **University Student Portal**.

Our goal here is to understand embedding **not just as arrays**, but *structured sub-documents* that model real-world entities.

---

## 2.1 What is Embedding in MongoDB?

**Embedding** means storing related data *inside* a document rather than separating it into another collection.

Example (simple):

```js
{
  name: "Riya",
  address: {
    city: "Delhi",
    pincode: 110001
  }
}
```

This is different from referencing where we would store:

```js
address_id: ObjectId("...reference here...")
```

---

## 2.2 When is Embedding Recommended?

Embed when:

1. Data belongs to the parent entity naturally ("is part of").
2. Data is *read together* most of the time.
3. The number of subdocuments is small and bounded.
4. Data is mostly static or does not require cross-document updates.
5. You want fewer collections and simpler queries.

Use embedding for:

* Student identity card info
* Student address
* List of scores
* Contact details

NOT ideal for:

* Many-to-many relations
* Data frequently updated independently
* Data shared across multiple parents

---

## 2.3 Real Embedding Example (Add Address to Student)

Let’s embed **address information** inside student documents.

### Insert Example

We add address for "Aniket":

```js
db.students.updateOne(
  { name: "Aniket" },
  {
    $set: {
      address: {
        street: "Sector 10 Road",
        city: "Noida",
        state: "UP",
        pincode: 201301
      }
    }
  }
);
```

### What the updated document looks like

```js
{
  name: "Aniket",
  age: 22,
  major: "Computer Science",
  subjects: ["Programming", "Database", "AI"],
  scores: [...],
  address: {
    street: "Sector 10 Road",
    city: "Noida",
    state: "UP",
    pincode: 201301
  }
}
```

This is embedding because `address` becomes *a part of student*.

---

## 2.4 Querying Embedded Fields

### Find students by city

```js
db.students.find({ "address.city": "Noida" });
```

### Show only name and city (projection)

```js
db.students.find(
  { "address.city": "Noida" },
  { name: 1, "address.city": 1, _id: 0 }
);
```

### Find with nested conditions

Students in UP with pincode above 200000:

```js
db.students.find({
  "address.state": "UP",
  "address.pincode": { $gt: 200000 }
});
```

---

## 2.5 Larger Embedding Example — Full Student Identity Details

Let's embed a richer structure:

```js
db.students.updateOne(
  { name: "Sneha" },
  {
    $set: {
      identity: {
        id_card_no: "IT2025SNE001",
        issued_by: "Registrar Office",
        valid_till: ISODate("2029-01-01"),
        emergency_contact: {
          name: "Ramesh Kumar",
          relation: "Father",
          phone: "9876543210"
        }
      }
    }
  }
);
```

This demonstrates **multi-level embedding**.

---

## 2.6 When NOT to Embed (Important Theory)

Do **NOT** embed if:

| Case                                       | Reason                            | Example                                    |
| ------------------------------------------ | --------------------------------- | ------------------------------------------ |
| Document may grow unbounded                | MongoDB limit = 16MB per document | Student → books borrowed over many years   |
| Data reused across entities                | Duplication issues                | Course reused across many students         |
| Heavy update operations needed on children | Updating parent becomes costly    | Scores updated often across many semesters |
| Many-to-many relationships                 | Natural relational structure      | Students ↔ Courses                         |

Example of bad embedding:

```js
// Wrong design — embedding entire course documents into students
{ name: "Aniket", courses: [{ full_course_object_here }, ...] }
```

This leads to duplication and inconsistent updates.

---

### **(Easy)**

Add a simple address field (city + state only) to **"Diya"** using embedding.

**Task:**
Write an update query to store:
City: Jaipur
State: Rajasthan

**Solution:**

```js
db.students.updateOne(
  { name: "Diya" },
  {
    $set: {
      address: {
        city: "Jaipur",
        state: "Rajasthan"
      }
    }
  }
);
```

Verify:

```js
db.students.find({ name: "Diya" }, { address: 1 });
```

---

### **(Medium)**

Add *educational history* as embedded documents (multiple entries).

Data to insert for `"Arjun"`:

| Field     | Value            |
| --------- | ---------------- |
| Course    | BSc Mathematics  |
| Year      | 2022             |
| Institute | Delhi University |

Also add:

| Field     | Value        |
| --------- | ------------ |
| Course    | Class 12     |
| Year      | 2020         |
| Institute | DPS RK Puram |

**Solution:**

```js
db.students.updateOne(
  { name: "Arjun" },
  {
    $set: {
      education: [
        { course: "BSc Mathematics", year: 2022, institute: "Delhi University" },
        { course: "Class 12", year: 2020, institute: "DPS RK Puram" }
      ]
    }
  }
);
```

Verify:

```js
db.students.find({ name: "Arjun" }, { education: 1 });
```

---

### **(Hard)**

We want to embed **semester-wise scores** instead of a flat list.
Design a schema for `"Karan"` where scores are grouped like this:

```
Semester 1:
- Deep Learning: 94
Semester 2:
- NLP: 91
```

This means convert:

```js
scores: [
  { subject: "Deep Learning", score: 94 },
  { subject: "NLP", score: 91 }
]
```

Into:

```js
scores: [
  {
    semester: 1,
    subjects: [
      { name: "Deep Learning", score: 94 }
    ]
  },
  {
    semester: 2,
    subjects: [
      { name: "NLP", score: 91 }
    ]
  }
]
```

**Solution:**

```js
db.students.updateOne(
  { name: "Karan" },
  {
    $set: {
      scores: [
        {
          semester: 1,
          subjects: [
            { name: "Deep Learning", score: 94 }
          ]
        },
        {
          semester: 2,
          subjects: [
            { name: "NLP", score: 91 }
          ]
        }
      ]
    }
  }
);
```

Verification query (search nested field):

```js
db.students.find(
  { "scores.subjects.name": "Deep Learning" },
  { name: 1, scores: 1 }
);
```

We covered here:

* Multi-level embedding
* Querying deeply nested structures
* Schema redesign thinking

---
---

# **MongoDB Referencing Documents**

Focus:

* When to *reference instead of embed*
* Designing related collections (`courses`, `departments`, `enrollments`)
* Writing reference-based queries
* 1e, 1m, 1h assignments
* All examples based on your existing `students` data

---

## 3.1 What is Referencing?

Referencing means **storing only a pointer (such as `_id`) to another document**, instead of storing the full object.

Example:

```js
{
  name: "Aniket",
  course_ids: [
    ObjectId("66c1c1111111111111111111"),
    ObjectId("66c1c1222222222222222222")
  ]
}
```

vs. embedding full courses:

```js
{
  name: "Aniket",
  courses: [
    { code: "CS101", title: "Intro to Programming" },
    ...
  ]
}
```

Referencing avoids duplication and keeps collections modular.

---

## 3.2 When Should You Reference Instead of Embed?

Use **Referencing** when:

| Scenario                        | Why                                | Example                                    |
| ------------------------------- | ---------------------------------- | ------------------------------------------ |
| Many-to-Many                    | Avoid duplication & huge documents | Students ↔ Courses                         |
| Large child data                | Document could exceed 16MB         | Student → All books issued over semesters  |
| Frequent independent updates    | Avoid heavy writes                 | Course updated independently from students |
| Shared data across many parents | Same data repeated                 | Departments shared by many students        |

Typical real-world examples:

* Students to Courses
* Students to Departments
* Products to Vendors
* Users to Orders

---

## 3.3 Create Supporting Collections

Now we create additional collections that make referencing meaningful.

### Create `departments` collection

```js
db.departments.insertMany([
  { _id: ObjectId(), code: "CSE", name: "Computer Science" },
  { _id: ObjectId(), code: "PHY", name: "Physics" },
  { _id: ObjectId(), code: "DS", name: "Data Science" },
  { _id: ObjectId(), code: "ME", name: "Mechanical Engineering" }
]);
```

### Create `courses` collection

```js
db.courses.insertMany([
  { _id: ObjectId(), code: "CS101", title: "Intro to Programming", credits: 3 },
  { _id: ObjectId(), code: "AI201", title: "Introduction to AI", credits: 4 },
  { _id: ObjectId(), code: "ML301", title: "Machine Learning", credits: 4 },
  { _id: ObjectId(), code: "PHY111", title: "Physics Fundamentals", credits: 3 }
]);
```

(Sample `_id`s created automatically)

---

## 3.4 Reference Example — Link Student to a Department

Let’s link `"Aniket"` to `"CSE"`.

Step 1: Fetch department `_id`

```js
db.departments.findOne({ code: "CSE" });
```

Suppose it returns:

```js
{ _id: ObjectId("66aaa111aaa111aaa111aaa1"), code: "CSE", name: "Computer Science" }
```

Step 2: Reference this `_id` in student document:

```js
db.students.updateOne(
  { name: "Aniket" },
  { $set: { department_id: ObjectId("66aaa111aaa111aaa111aaa1") } }
);
```

Now, instead of embedding department info, we reference it.

---

## 3.5 Query Using Reference

### Get Aniket’s department details

```js
let student = db.students.findOne({ name: "Aniket" });
db.departments.findOne({ _id: student.department_id });
```

### Using `$lookup` (MongoDB join)

```js
db.students.aggregate([
  {
    $lookup: {
      from: "departments",
      localField: "department_id",
      foreignField: "_id",
      as: "department"
    }
  },
  { $project: { name: 1, department: 1 } }
]);
```

---

## 3.6 Referencing for Many-to-Many — Enrollments Table

Instead of storing course IDs inside students, we create a separate collection:

```js
db.enrollments.insertMany([
  {
    student_name: "Aniket",
    student_id: ObjectId("..."),
    course_id: ObjectId("...CS101..."),
    semester: "2025-Spring",
    status: "active",
    grade: "A"
  },
  {
    student_name: "Aniket",
    student_id: ObjectId("..."),
    course_id: ObjectId("...AI201..."),
    semester: "2025-Spring",
    status: "active",
    grade: "A"
  }
]);
```

Better: Instead of storing `student_name`, rely only on IDs.

---

## 3.7 Query All Courses of Aniket

Using enrollments:

```js
db.enrollments.find({ student_id: ObjectId("...") });
```

Using join:

```js
db.enrollments.aggregate([
  {
    $lookup: {
      from: "courses",
      localField: "course_id",
      foreignField: "_id",
      as: "course_details"
    }
  },
  { $match: { student_id: ObjectId("...") } }
]);
```

---

## ASSIGNMENTS (1e, 1m, 1h)

### **(Easy)**

Add a reference linking `"Sourav"` to `"Mathematics"` department.

Department data (assume):

```
code: "MATH"
_id: 66def999abc999abc999abc9
```

**Solution:**

```js
db.students.updateOne(
  { name: "Sourav" },
  { $set: { department_id: ObjectId("66def999abc999abc999abc9") } }
);
```

Verify:

```js
db.students.find({ name: "Sourav" }, { department_id: 1 });
```

---

### **(Medium)**

Store enrollment details for `"Isha"`:

| Course           | Code  | Semester    |
| ---------------- | ----- | ----------- |
| Machine Learning | ML301 | 2025-Spring |
| Python           | CS101 | 2025-Spring |

Assume:

```
CS101 _id = AAAAA111AAA
ML301 _id = BBBBB222BBB
Isha _id = SSSSS333SSS
```

**Solution:**

```js
db.enrollments.insertMany([
  {
    student_id: ObjectId("SSSSS333SSS"),
    course_id: ObjectId("BBBBB222BBB"),
    semester: "2025-Spring",
    status: "active"
  },
  {
    student_id: ObjectId("SSSSS333SSS"),
    course_id: ObjectId("AAAAA111AAA"),
    semester: "2025-Spring",
    status: "active"
  }
]);
```

Query all enrollments:

```js
db.enrollments.find({ student_id: ObjectId("SSSSS333SSS") });
```

---

### **(Hard)**

Design a fully normalized referencing-based schema for:

* students
* courses
* enrollments
* instructors

Rules:

* Each course has one instructor.
* Students enroll in many courses.
* Courses have many students.
* Instructors teach many courses.

**Your task:**
Provide the schema structure in MongoDB document format (not SQL), but do NOT embed full objects.

**Solution:**

```js
// STUDENTS
{
  _id: ObjectId(),
  name: "Aniket",
  age: 22,
  department_id: ObjectId("..."),
  // No courses embedded
}

// COURSES
{
  _id: ObjectId(),
  code: "AI201",
  title: "Introduction to AI",
  credits: 4,
  instructor_id: ObjectId("...")  // referenced
}

// INSTRUCTORS
{
  _id: ObjectId(),
  name: "Dr. Sharma",
  department_id: ObjectId("...")
}

// ENROLLMENTS (junction collection)
{
  _id: ObjectId(),
  student_id: ObjectId("..."),
  course_id: ObjectId("..."),
  semester: "2025-Spring",
  status: "active",
  grade: "A"
}
```

Why this is correct:

* No duplication of course data across students
* Enrollment holds dynamic attributes
* Instructor assigned once but queryable via `$lookup`
* Works for large systems

---

