# Academic Transfer Credit Planner
### Midterm Project — Introduction to Artificial Intelligence
**Student Program:** Computer Science | **GTU**

---

## Table of Contents

1. [Project Overview](#project-overview)
2. [Application Structure](#application-structure)
3. [Course Data](#course-data)
4. [Equivalency Algorithm](#equivalency-algorithm)
5. [How to Use](#how-to-use)
6. [Improvement Plan — AI-Powered Automatic Syllabus Comparison](#improvement-plan)

---

## 1. Project Overview

This single-file web application helps a **Computer Science** student plan an academic program transfer by identifying which of their completed courses have equivalents in the target program. The tool supports transfers to three target programs:

- Computer Engineering
- Civil Engineering
- Electrical and Electronics Engineering

The application is built with **pure HTML, CSS, and JavaScript** — no frameworks, no external libraries, no build tools.

---

## 2. Application Structure

The application is contained in a **single `.html` file** with embedded CSS and JavaScript. The UI is divided into four logical sections:

| Section | Description |
|---|---|
| **Current Program** | Displays the student's enrolled program and course statistics |
| **My Courses** | A full table of all courses (completed, in-progress, and additional) |
| **Transfer Controls** | Dropdown to select the target program + Transfer button |
| **Equivalency Table** | Appears after clicking Transfer; shows matched and unmatched courses |

### Key design decisions

- All course data is stored as a JavaScript array (`MY_COURSES`) — easy to maintain
- Target program catalogs are stored as a dictionary (`PROGRAM_COURSES`) keyed by program name
- A secondary lookup table (`TOPIC_EQUIV`) handles cases where courses cover the same topic but have different codes

---

## 3. Course Data

The student's course list reflects a third-year Computer Science student who has completed Semesters 1–4 and is currently in Semester 5.

### Passed courses (Semesters 1–4)

```
MATH150  Calculus I
CS50     Introduction to Programming
P432     Introduction to Modern Thought I
CS212    Cyberlaw and Ethics
Q185     English Language Course C1-1
MATH151  Calculus II
CS211    Object Oriented Programming
Q955     English Language Course C1-2
DE112    Mathematical Foundation of Computing
DE111    Computer Organization
MATH252  Calculus III
CS311    Programming Languages
MATH254  Introduction to Linear Algebra
PHYS195  Basics of Physics I (Including Lab)
EE300    Computational and Statistical Methods
CS312    Computer and Data Networks
CS221    Algorithms and Data Structures
MATH311  Principles of Numerical Methods
```

### In-progress courses (Semester 5, counted as "No")

```
CS223  Introduction to Cybersecurity
CS313  Elements of Computational Complexity
DE101  Introduction to Operating Systems
P737   Academic Techniques
```

### Additional courses (always counted as "Yes")

Five courses chosen from the CS program catalog, representing advanced subjects relevant for transfer evaluation:

```
CS321  Introduction to Machine Learning
CS322  Databases and Information Systems
CS323  Design Patterns
CS314  Distributed and Parallel Computing
CS324  Artificial Neural Networks
```

---

## 4. Equivalency Algorithm

### Overview

The algorithm determines course equivalencies in **two passes**:

1. **Exact code match** — check if the student's course code exists in the target program's catalog
2. **Topic-based fallback** — if no exact match, consult a manually curated mapping table for courses that cover equivalent topics under different codes

If neither pass finds a match, the result is **N/A**.

### Pass 1: Exact Code Match

```javascript
function findEquivalent(course, targetProgram) {
  const targetCourses = PROGRAM_COURSES[targetProgram];
  const targetMap = {}; // code → course object
  targetCourses.forEach(c => targetMap[c.code] = c);

  // Pass 1: exact code
  if (targetMap[course.code]) {
    return { code: course.code, name: targetMap[course.code].name, type: 'exact' };
  }
  // ... Pass 2 below
}
```

This works well for shared foundational courses like `MATH150`, `MATH151`, `MATH252`, `DE112`, `DE111`, `EE300`, `CS50`, etc., which appear identically across multiple programs.

### Pass 2: Topic-Based Fallback

Some courses cover equivalent content but have different codes across programs. These are manually defined in `TOPIC_EQUIV`:

```javascript
const TOPIC_EQUIV = {
  "Computer Engineering": {
    // CS211 (OOP in CS) ↔ CE111 (Procedural C in CompEng)
    // Both are core programming foundation courses
    "CS211": { code: "CE111", name: "Procedural Programming with C" }
  },
  // ...
};
```

**Example:** `CS211 — Object Oriented Programming` (CS program) is mapped to `CE111 — Procedural Programming with C` (Computer Engineering), because both courses serve the same curriculum role: teaching the student a structured programming paradigm as the foundation for more advanced software development.

### Decision rules applied

| Rule | Rationale |
|---|---|
| Same course code → direct equivalent | Shared foundational courses are identical across engineering programs |
| Different code but same domain → topic equivalent | e.g., OOP and Procedural C both teach core programming skills |
| CS-specific courses (e.g., Design Patterns, ML) with no counterpart → N/A | Specialized CS electives rarely have 1:1 equivalents in EE or Civil Eng |
| Language courses (Q185, Q955) → matched only if target program has the same code | Language tracks differ between programs |
| Civil Engineering → only math/physics/chemistry courses match | CE is a hardware-physical discipline; CS courses have no CE equivalents |

### Coverage summary (approximate)

| Transfer Target | Expected Match Rate |
|---|---|
| Computer Engineering | ~80% — shares most CS core + electives |
| Electrical and Electronics Engineering | ~50% — shares math/physics/hardware foundation |
| Civil Engineering | ~30% — only math and natural science courses match |

---

## 5. How to Use

1. Open `transfer_planner.html` in any modern web browser
2. The course table loads automatically on page open
3. Select a **target program** from the dropdown menu
4. Click the **Transfer** button
5. The equivalency table appears below, showing:
   - Which of your passed courses have equivalents
   - The match type (Exact or Topic-based)
   - A summary of the overall credit recognition percentage

---

## 6. Improvement Plan

> **Future Chapter: AI-Powered Automatic Syllabus Comparison**

### 6.1 Motivation

The current system relies on **manual equivalency mappings** — a curated lookup table built by hand from reading course catalogs. This approach has significant limitations:

- It does not scale to large programs with hundreds of courses
- It requires human expertise and is prone to errors or gaps
- It cannot detect subtle equivalencies between courses with very different names
- It requires manual updates whenever curricula change

An AI-powered system would automate and improve this process by comparing the **actual content** of courses, not just their names or codes.

---

### 6.2 Proposed Architecture

```
┌─────────────────────────────────────────────────────┐
│                   INPUT LAYER                       │
│  Source Program Syllabus  │  Target Program Syllabus │
└───────────┬───────────────────────────┬─────────────┘
            │                           │
            ▼                           ▼
┌─────────────────────────────────────────────────────┐
│               SYLLABUS PARSER                        │
│  Extract: course name, objectives, topics, outcomes │
└─────────────────────────────┬───────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────┐
│             EMBEDDING & SIMILARITY ENGINE            │
│  Convert each course description → vector embedding │
│  Compute cosine similarity between all course pairs │
└─────────────────────────────┬───────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────┐
│              LLM REASONING MODULE                    │
│  For high-similarity pairs: LLM evaluates context,  │
│  credit hours, prerequisites, and learning outcomes │
│  to make a final equivalency decision               │
└─────────────────────────────┬───────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────┐
│              OUTPUT: EQUIVALENCY TABLE               │
│  Course A (source) → Course B (target) | Confidence │
└─────────────────────────────────────────────────────┘
```

---

### 6.3 Step-by-Step Algorithm

#### Step 1 — Syllabus Collection

Load the syllabus documents for each course in both programs. Each syllabus typically contains:

- Course title and code
- Course description
- Learning objectives (what the student will be able to do)
- Weekly topic list
- Textbooks and references
- Prerequisites and credit hours

```python
def load_syllabi(program_name: str) -> list[dict]:
    """
    Returns a list of course dicts, each with:
    { code, title, description, objectives, topics[], credits }
    """
```

#### Step 2 — Text Embedding

Convert each course's combined text (description + objectives + topics) into a dense vector representation using a pre-trained language model:

```python
from sentence_transformers import SentenceTransformer

model = SentenceTransformer('all-MiniLM-L6-v2')  # lightweight, fast

def embed_course(course: dict) -> list[float]:
    text = f"{course['title']}. {course['description']}. Topics: {', '.join(course['topics'])}"
    return model.encode(text).tolist()
```

Each course is now represented as a vector in a high-dimensional semantic space. Courses that cover similar topics will have vectors that point in similar directions.

#### Step 3 — Similarity Matrix

Compute the **cosine similarity** between every course in Program A and every course in Program B:

```python
import numpy as np
from sklearn.metrics.pairwise import cosine_similarity

def compute_similarity_matrix(courses_a, courses_b):
    vectors_a = [embed_course(c) for c in courses_a]
    vectors_b = [embed_course(c) for c in courses_b]
    return cosine_similarity(vectors_a, vectors_b)
    # Result: matrix of shape [len(courses_a), len(courses_b)]
```

A similarity score of **1.0** means identical content; **0.0** means completely unrelated.

#### Step 4 — Candidate Pair Selection

For each course in Program A, identify the top-k most similar courses from Program B as candidate equivalencies:

```python
def get_candidates(similarity_matrix, courses_b, threshold=0.65, top_k=3):
    candidates = []
    for i, row in enumerate(similarity_matrix):
        top_indices = np.argsort(row)[::-1][:top_k]
        for j in top_indices:
            if row[j] >= threshold:
                candidates.append({
                    "source_idx": i,
                    "target_idx": j,
                    "similarity": float(row[j])
                })
    return candidates
```

#### Step 5 — LLM Final Decision

For each candidate pair, send the course details to an LLM (e.g., Claude) and ask it to make a final equivalency judgment:

```python
def llm_evaluate_equivalency(course_a: dict, course_b: dict) -> dict:
    prompt = f"""
You are an academic transfer evaluator.

COURSE A (source program):
Title: {course_a['title']}
Description: {course_a['description']}
Learning Objectives: {course_a['objectives']}
Credits: {course_a['credits']}

COURSE B (target program):
Title: {course_b['title']}
Description: {course_b['description']}
Learning Objectives: {course_b['objectives']}
Credits: {course_b['credits']}

Decide: Can Course A be accepted as an equivalent to Course B for transfer purposes?
Respond with JSON: {{ "equivalent": true/false, "confidence": 0.0-1.0, "reason": "..." }}
"""
    response = call_claude_api(prompt)
    return parse_json(response)
```

The LLM can reason about nuances that pure vector similarity cannot capture — for example, two courses might both mention "neural networks" but one is a deep theory course and the other is a practical tools course, making them poor equivalents despite high textual similarity.

#### Step 6 — Output Table

Assemble the final equivalency table with confidence scores:

```python
def build_equivalency_table(courses_a, courses_b, candidates, llm_results):
    table = []
    for result in llm_results:
        if result['equivalent']:
            table.append({
                "source_course": courses_a[result['source_idx']]['code'],
                "target_course": courses_b[result['target_idx']]['code'],
                "confidence":    result['confidence'],
                "reason":        result['reason'],
            })
        else:
            table.append({
                "source_course": courses_a[result['source_idx']]['code'],
                "target_course": "N/A",
                "confidence":    0,
                "reason":        result['reason'],
            })
    return table
```

---

### 6.4 Why AI Helps

| Capability | Manual Approach | AI-Powered Approach |
|---|---|---|
| Scale | Limited to small catalogs | Handles hundreds of courses instantly |
| Accuracy | Human error, gaps in knowledge | Semantic understanding of content |
| Updates | Manual revision required | Re-run when syllabi change |
| Cross-language | Requires translation | Multilingual embedding models available |
| Nuance | Surface-level name matching | Deep reasoning about learning outcomes |
| Transparency | No explanation given | LLM produces a written justification |

The most important insight is that **course titles are unreliable** — two courses titled "Introduction to Programming" at two universities might cover completely different content, while "Discrete Mathematics" and "Mathematical Foundations of Computing" might be nearly identical. Only by reading and comparing the actual syllabi — which is what the AI does — can accurate equivalency decisions be made.

---

*End of documentation.*
