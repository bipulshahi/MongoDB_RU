## MongoDB Relationships (Using Your `students` Data)

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


