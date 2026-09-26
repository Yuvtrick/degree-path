DegreePath — Master Project Specification

1. Project Overview
   DegreePath is a full-stack academic degree planning platform designed initially for students at the Open University of Israel.
   The system helps students understand, plan, track, and optimize their entire academic degree.
   A student should be able to:

- Select their degree program.
- Mark courses they have already completed.
- See exactly which degree requirements remain.
- See which courses they are currently eligible to take.
- Understand why a course is unavailable.
- Build future semesters.
- Validate that a semester plan is academically legal.
- Track progress toward graduation.
- Receive intelligent course recommendations.
- Automatically generate possible degree plans.
- Understand how changing one course affects future semesters.
- Optimize their plan according to personal constraints.
  The project should NOT be treated as a simple course-tracking website.
  The core of the project is a Degree Planning Engine that models academic requirements and course dependencies and performs graph-based reasoning, constraint validation, planning, and optimization.

2. The Problem
   Planning a degree at the Open University can be surprisingly difficult.
   Information exists, but the student has to combine multiple pieces manually:
   Degree requirements

- Mandatory courses
- Elective requirements
- Prerequisites
- Recommended prerequisites
- Credits
- Course availability
- Already completed courses
- Future semester workload
- Personal preferences

A student may know:
"I still need Algorithms."

But Algorithms may require Data Structures.
Data Structures may itself require another course.
And delaying Data Structures may indirectly delay several advanced courses.
Currently, students often need to manually inspect course pages, degree requirements, spreadsheets, university websites, Facebook groups, WhatsApp groups, or ask other students.
DegreePath turns this into a computational problem. 3. Main Product Philosophy
The platform should answer four fundamental questions:
Where am I?
Degree progress: 47%

Mandatory courses:
8 / 15

Elective credits:
12 / 30

Total credits:
58 / 120

What can I take next?
Available next semester:

✓ Data Structures
✓ Probability
✓ Computer Organization

Unavailable:

✗ Algorithms
Missing prerequisite:
Data Structures

What should I take next?
Example:
Recommended:

1. Data Structures
2. Linear Algebra II
3. Probability

with explanations.
How should I finish my degree?
Example:
Goal:
Finish as quickly as possible

Constraints:
Maximum 3 courses / semester
Maximum 2 during summer
Balanced workload

Generated plan:

2027A
Data Structures
Probability
Linear Algebra II

2027B
Algorithms
Computer Organization
Elective A

2027C
Elective B
Elective C

...

4. Target Users
   Initially:
   Open University of Israel — B.Sc. Computer Science students.
   This is intentional.
   Do NOT initially build a generic platform supporting every university.
   A focused real product is preferable to an overly generic architecture with no users.
   However, the internal data model should eventually allow:
   University
   ↓
   DegreeProgram
   ↓
   DegreeVersion
   ↓
   Requirements
   ↓
   Courses

This allows future expansion. 5. Core Technical Objective
This project exists both as a useful product and as a strong Software Engineering portfolio project.
Therefore it should demonstrate naturally:
Java
Spring Boot

REST APIs

PostgreSQL

React
TypeScript

Algorithms
Data Structures

Graphs

Constraint solving / optimization

Database design

Backend architecture

Testing

Docker

Git / GitHub

CI/CD

Cloud deployment

Caching when justified

Performance measurement

Technologies must NEVER be added solely to increase the number of technologies on the CV.
Every technology should solve an actual engineering problem.
For example:
Do NOT add Kafka simply because Kafka is popular.
Do NOT split the project into microservices simply because microservices sound impressive.
Start with a well-designed modular monolith. 6. Proposed Technology Stack
Backend
Java
Spring Boot
Spring Web
Spring Data JPA
Hibernate

Possible later additions:
Spring Security
JWT / OAuth

7. Frontend
   React
   TypeScript

Possible supporting tools:
Vite
React Router
TanStack Query

UI framework can be decided later.
Frontend complexity is secondary to the backend/planning engine. 8. Database
Primary database:
PostgreSQL

It stores:
Users
Universities
Degree programs
Degree versions
Courses
Course prerequisites
Degree requirements
Completed courses
Semester plans
User preferences
Course metadata

9. Infrastructure
   Development:
   Docker
   Docker Compose

Eventually:
Frontend container
Backend container
PostgreSQL container

Production deployment can later use an appropriate cloud provider. 10. Architecture
Initial architecture:
┌───────────────────┐
│ React Client │
│ TypeScript │
└─────────┬─────────┘
│
HTTPS
│
┌─────────▼─────────┐
│ Spring Boot │
│ REST API │
└─────────┬─────────┘
│
┌─────────────────────┼─────────────────────┐
│ │ │
▼ ▼ ▼

User Module Degree Module Planning Module

       │                     │                     │
       │                     │              ┌──────┴──────┐
       │                     │              │             │
       │                     │           Graph Engine   Optimizer
       │                     │
       └─────────────────────┼──────────────────────┐
                             │                      │
                             ▼                      ▼
                       PostgreSQL              Cache
                                                later

11. Backend Modules
    Keep modules separated by responsibility.
    Suggested structure:
    backend/

user/
course/
degree/
progress/
semester/
planning/
recommendation/
analytics/
common/

Later possibly:
auth/
admin/

12. Course Model
    A course should contain information such as:
    Course

id
courseCode
name
credits
level
category
description

difficultyEstimate
workloadEstimate

availableSemesters

prerequisites
recommendedPrerequisites

Example:
Data Structures

credits: 6

prerequisites:
Introduction to Computer Science

recommended:
Discrete Mathematics

difficulty: 8
workload: 8

Difficulty/workload must have a documented source or methodology if shown publicly. 13. Degree Model
Example:
B.Sc Computer Science

Required credits: X

Mandatory courses
Elective requirements
Seminar requirements
Workshop requirements
General requirements

Do NOT hard-code these rules inside Java.
They should exist as structured data.
That distinction is important.
Bad:
if (degree.equals("computer-science")) {
requiredCredits = 120;
}

Better:
DegreeProgram
DegreeRequirement
RequirementGroup
CourseRequirement
CreditRequirement

The engine operates on degree definitions. 14. Degree Versions
This is an important real-world problem.
Degree requirements can change.
Therefore:
Computer Science
│
├── Curriculum 2025
├── Curriculum 2026
└── Curriculum 2027

Students should belong to an appropriate curriculum/version.
This avoids changing historical plans whenever requirements change. 15. The Course Dependency Graph
This is one of the central CS elements.
Represent courses as vertices:
V = courses

and prerequisites as directed edges.
Example:
Calculus I
↓
Calculus II

Intro CS
↓
Data Structures
↓
Algorithms

Conceptually:
Intro CS ──► Data Structures ──► Algorithms
│
└──► Advanced Course

This allows us to perform graph algorithms naturally. 16. Graph Operations
The engine should eventually support:
Prerequisite traversal
Given course X:
What must be completed before X?

Reverse dependency traversal
Given course X:
What future courses does X unlock?

Eligibility
Given completed set C:
Which courses are currently available?

Dependency depth
Determine how far a course is from courses already completed.
Critical courses
Identify courses whose delay potentially blocks many future courses.
Topological ordering
Generate valid course ordering respecting prerequisites. 17. Eligibility Engine
Function conceptually:
getEligibleCourses(
degree,
completedCourses,
semester
)

Example:
Completed:

✓ Intro CS
✓ Calculus I

Course:
Data Structures

Required:
Intro CS ✓

Result:
ELIGIBLE

Another:
Course:
Algorithms

Required:
Data Structures ✗

Result:
NOT ELIGIBLE

But instead of returning just false, return an explanation:
{
"eligible": false,
"missingRequirements": [
"Data Structures"
]
}

Explainability should be an important design principle. 18. Progress Engine
Calculate multiple types of progress.
Not merely:
completedCourses / totalCourses

because degree requirements can be credit-based.
Track:
Total degree credits
Mandatory credits
Elective credits
Advanced credits
Seminar requirements
Other requirement groups

Example:
Overall
████████████░░░░░░░░
61%

Mandatory
███████████████░░░░░
74%

Electives
████████░░░░░░░░░░░░
38%

19. Semester Planner
    Users manually create semesters.
    Example:
    2027A

Data Structures
Probability
Linear Algebra II

When adding a course, validate it.
Possible warnings:
BLOCKED

Missing prerequisite:
Calculus I

or:
WARNING

This semester has unusually high estimated workload.

Important distinction:
Hard constraints
Cannot be violated.
Soft constraints
Can be violated but produce warnings / affect optimization score. 20. Automatic Degree Planner
This is one of the flagship features.
Input:
Completed courses

Maximum courses per semester

Maximum summer courses

Target completion date

Preferred workload

Preferred / avoided courses

Degree requirements

Output:
Possible degree plan

21. Constraint Model
    Example hard constraints:
    Prerequisites must be satisfied.

Course must be offered during semester.

Required degree credits must eventually be reached.

Mandatory courses must be completed.

Maximum courses per semester cannot be exceeded.

Soft constraints:
Balance workload.

Avoid multiple extremely difficult courses together.

Prefer prerequisite-unlocking courses earlier.

Prefer user's selected interests.

Minimize number of semesters.

22. Optimization Goals
    Eventually support different strategies.
    Fastest Graduation
    Objective:
    minimize number of semesters

Balanced
Objective:
minimize workload variance

Working Student
Example:
max 2 courses / semester
avoid workload > threshold

Custom
User specifies constraints. 23. Planning Algorithm — Evolution
Do not immediately attempt the mathematically perfect optimizer.
Build incrementally.
Version 1
Greedy planning.
Prioritize:
mandatory courses

- courses unlocking many dependencies
- available courses

Version 2
Graph-aware heuristic.
Score:
courseScore =
dependencyImportance

- requirementImportance
- availabilityImportance

* workloadPenalty

Version 3
Search/optimization.
Possible techniques:
Backtracking
Branch & Bound
Constraint Satisfaction
Dynamic Programming where applicable

Potential later investigation:
Integer Programming
Constraint Programming

Only if justified. 24. "What If?" Engine
This should eventually become one of DegreePath's signature features.
Example:
User has:
2027A
Data Structures
Probability
Linear Algebra II

They ask:
What happens if I move Data Structures to 2027B?

System computes affected dependency chain.
Data Structures
2027A → 2027B
│
▼
Algorithms
2027B → 2028A
│
▼
Advanced Algorithms
2028A → 2028B

Then:
Estimated graduation impact:
+1 semester

This is a genuinely interesting graph problem. 25. Dependency Impact Analysis
For every course:
Directly unlocks: 3
Indirectly unlocks: 8

Example:
Data Structures

Direct:
Algorithms
Course A

Indirect:
Machine Learning
Advanced Algorithms
Course B
Course C
...

This can contribute to recommendation scoring. 26. Course Recommendation Engine
Recommendations should initially be deterministic and explainable.
Not:
"AI recommends Algorithms."

Instead:
Recommended: Data Structures

Why?

• Mandatory course
• Available this semester
• Unlocks 4 future courses
• Required for Algorithms
• Fits your workload target

This is much more technically defensible. 27. Recommendation Score
Potential model:
score(course) =

mandatoryWeight

- unlockWeight
- degreeProgressWeight
- availabilityWeight
- userPreferenceWeight

*

workloadPenalty

Weights should be configurable.
Later we can experiment with recommendation quality. 28. Alternative Plans
Instead of generating only one answer:
Recommended plans:

FASTEST
Graduation: 2028A
Average workload: High

BALANCED
Graduation: 2028B
Average workload: Medium

LIGHT
Graduation: 2029A
Average workload: Low

This is a particularly good product feature. 29. Degree Requirement Engine
One challenge is that degrees aren't simply lists of courses.
Examples:
Complete all courses from Group A

Choose 3 from Group B

Complete at least 24 credits from Group C

Complete one seminar

Choose either:
A + B
OR
C + D

Therefore requirements should eventually support logical structures.
Conceptually:
AND
OR
CHOOSE_N
MIN_CREDITS
REQUIRED

Example:
AND
├── Intro CS
├── Data Structures
├── Algorithms
└── MIN_CREDITS(24)
└── Electives

This becomes a requirement tree.
That's another interesting data-structure component of the project. 30. Degree Requirement Validation
The engine takes:
Student completed courses

- Degree requirement tree

and calculates:
Satisfied
Partially satisfied
Unsatisfied

with explanation. 31. User Dashboard
Potential dashboard:
Good evening, Yuval

Computer Science B.Sc.

Overall Progress
██████████████░░░░░░
68%

Credits
72 / 108

Mandatory
11 / 16

Electives
18 / 30

────────────────────────

Next semester

Data Structures
Probability
Computer Organization

Estimated workload:
MEDIUM-HIGH

────────────────────────

Recommended next:

Algorithms
Linear Algebra II
...

32. Degree Graph Visualization
    A visual dependency map could become a standout UI feature.
    Example:
    ┌── Algorithms ── Advanced Algorithms
    │
    Intro CS ── Data Structures
    │
    └── Course X ── Course Y

Colors/statuses can represent:
Completed
Available
Planned
Locked

Clicking a node shows course details.
This makes the underlying graph tangible to users. 33. Critical Path
Potential advanced feature:
Academic Critical Path
Identify chains that determine the earliest possible graduation.
Example:
Intro
↓
Data Structures
↓
Algorithms
↓
Advanced Course
↓
Seminar

If each is offered only during particular semesters, delaying one can delay graduation.
The system can warn:
Critical course

Delaying this course may postpone graduation.

This is an excellent combination of algorithms and actual product value. 34. Course Availability
Courses may not be offered every semester.
Model:
CourseOffering

course
academicYear
semester

For example:
Algorithms

2027A ✓
2027B ✓
2027C ✗

This makes planning significantly more interesting. 35. Workload Model
Eventually each course can have:
difficultyScore
workloadScore

Sources need to be considered carefully.
Potential sources:

- official data,
- anonymous student input,
- aggregated historical feedback.
  Do not fabricate ratings.

36. Community Data — Future Feature
    Students could anonymously report:
    Weekly hours
    Difficulty
    Exam difficulty
    Assignment workload

Then aggregate:
Data Structures

Difficulty:
8.1 / 10

Weekly workload:
11.3 hours

Based on:
143 reports

This would add a crowdsourcing component. 37. Personalized Workload
Eventually the user could specify:
Works full time
Available study hours: 25/week

Planner tries to keep estimated workload within that limit. 38. Course Combination Analysis
Interesting future feature:
Determine combinations students report as particularly difficult.
Example:
Calculus II

- Linear Algebra II
- Data Structures

Estimated workload:
VERY HIGH

Could later be based on aggregate data rather than arbitrary rules. 39. Search
Course search should support:
name
course code
keywords
category

Do not build Elasticsearch initially.
PostgreSQL is sufficient until evidence says otherwise. 40. Authentication
MVP may initially work without accounts.
Later:
Register
Login
Logout
Password hashing
JWT/session

Potential OAuth:
Google

But authentication should not delay the planning engine. 41. Persistence
Logged-in users can save:
Completed courses
Current plan
Preferences
Generated plans

42. Plan Versioning
    Interesting advanced feature.
    When a user changes their plan:
    Plan v1
    Plan v2
    Plan v3

They can compare versions.
Example:
v1 → Graduation 2028A
v2 → Graduation 2028B

Difference:
Algorithms moved one semester.

43. REST API Examples
    Potential endpoints:
    GET /api/courses

GET /api/courses/{id}

GET /api/degrees

GET /api/degrees/{id}

GET /api/users/{id}/progress

GET /api/users/{id}/eligible-courses

POST /api/plans

POST /api/plans/generate

POST /api/plans/validate

POST /api/plans/simulate-change

These are conceptual, not frozen API contracts. 44. Database Relationships
Initial conceptual ER model:
University
│
└── DegreeProgram
│
└── DegreeVersion
│
└── DegreeRequirement

Course
│
├── CoursePrerequisite
│
└── CourseOffering

User
│
├── CompletedCourse
│
└── StudyPlan
│
└── SemesterPlan
│
└── PlannedCourse

45. Data Integrity
    Important rules should be enforced at multiple levels where appropriate.
    Examples:
    Course code unique within university.

No duplicate completed course.

No duplicate course in same semester.

Credits cannot be negative.

Prerequisite cannot reference nonexistent course.

46. Cycles
    Academic prerequisites should normally form a DAG.
    But imported/bad data could accidentally produce:
    A → B
    B → C
    C → A

The system should detect prerequisite cycles.
This gives us a natural use for cycle detection algorithms. 47. Testing Strategy
Testing is a major part of the project.
Unit tests
Especially for:
EligibilityEngine
ProgressEngine
RequirementEngine
GraphEngine
PlanningEngine

Example:
Given:

A → B → C

Completed:
A

Expected eligible:
B

Expected unavailable:
C

48. Integration Tests
    Test:
    REST API

- Service
- Database

Potentially use Testcontainers later. 49. Algorithm Tests
Create synthetic degree graphs.
Examples:
Linear chain

A → B → C → D

Diamond

    B

/ \
 A D
\ /
C

Large generated graphs can be used for performance tests. 50. Performance
Eventually benchmark the planner.
Example:
100 courses
300 dependencies

Planning time:
14 ms

Then larger synthetic datasets:
1,000 courses
5,000 edges

Planning:
XX ms

This gives measurable engineering results for the README/CV. 51. Caching
Only introduce caching when justified.
Potential cache candidates:
Degree definitions
Course graph
Course metadata
Common requirement calculations

Initially an in-memory cache may be enough.
Redis can be considered later if deployment/scale makes it useful. 52. Concurrency
Later consider concurrent users generating plans.
Questions:
Are planner calculations stateless?

Can calculations run concurrently?

What state needs synchronization?

How are simultaneous plan updates handled?

Optimistic locking may eventually be useful for plan edits. 53. Observability
Later production version:
structured logs
request IDs
error tracking
metrics

Potential metrics:
planner.execution.time
plans.generated
eligibility.requests
API latency
DB query latency

54. Error Handling
    Backend should use structured errors.
    Example:
    {
    "code": "PREREQUISITE_NOT_MET",
    "message": "Course cannot be added.",
    "details": {
    "missingCourses": ["20407"]
    }
    }

Not random HTTP 500 responses. 55. Security
Eventually:
password hashing
input validation
authorization
rate limiting if necessary
secure secrets
HTTPS

Never store plaintext passwords. 56. Data Import
This is going to be a real engineering challenge.
Do NOT initially build a complicated scraper.
Start with a curated dataset.
Potential:
data/
courses.json
degree-cs.json
offerings.json

Build an importer.
Later investigate reliable methods of keeping data synchronized with official university information. 57. Admin Tools
Eventually create internal functionality for:
Add course
Update prerequisites
Create degree version
Update course offering
Validate degree graph

This prevents database editing by hand. 58. Data Validation Pipeline
Whenever curriculum data changes:
Import
↓
Schema validation
↓
Reference validation
↓
Cycle detection
↓
Requirement validation
↓
Publish

Very good engineering feature. 59. Explainability
One of the defining principles of DegreePath:
Never return only an answer when we can return the reasoning.
Instead of:
Recommended:
Data Structures

return:
Recommended:
Data Structures

Because:

✓ Mandatory
✓ Prerequisites satisfied
✓ Offered next semester
✓ Unlocks Algorithms
✓ Unlocks 3 additional courses
✓ Fits selected workload

This makes the system much more trustworthy. 60. AI — Optional, Not Core
AI can eventually provide natural-language interaction.
Example:
"אני עובד ארבע פעמים בשבוע ורוצה לסיים תוך שנתיים וחצי בלי לקחת יותר משלושה קורסים."

LLM converts that into:
{
"maxCourses": 3,
"targetSemesters": 7,
"workloadPreference": "balanced"
}

Then our own planner solves it.
Important architecture:
Natural language
↓
LLM
↓
Structured constraints
↓
OUR Planning Engine
↓
Plan

Not:
Prompt → LLM → random degree plan

The deterministic engine remains authoritative. 61. Possible Unique Feature — Plan Risk Analysis
DegreePath could calculate a plan risk profile.
For example:
Plan Risk

Academic load MEDIUM
Prerequisite risk HIGH
Availability risk LOW
Graduation delay MEDIUM

Why?
Algorithms is only offered once during
the next two planned semesters.

Failing/delaying Data Structures
may postpone 3 downstream courses.

This could become genuinely distinctive. 62. Bottleneck Detection
Analyze degree graph and identify bottlenecks.
Example:
HIGH IMPACT COURSE

Data Structures

Direct dependents: 3
Indirect dependents: 9

Recommendation:
Consider completing it early.

Again: algorithms directly produce useful product behavior. 63. Plan Comparison
Allow:
Plan A
vs
Plan B

Compare:
Graduation date
Average workload
Maximum workload
Summer courses
Critical dependencies
Risk

64. Schedule Resilience
    Potential advanced algorithmic idea:
    Don't optimize only for fastest graduation.
    A plan could be fast but fragile.
    Example:
    Plan A

Expected graduation:
2028A

but one delayed course →
2029A

versus:
Plan B

Expected:
2028B

one delayed course →
still 2028B

This introduces the concept of robust planning.
That's a genuinely interesting direction if we get far enough. 65. MVP — VERY IMPORTANT
Do not attempt everything above at once.
The first usable version should do only:

1. Load Computer Science degree data.

2. Display courses.

3. User marks completed courses.

4. Calculate progress.

5. Build prerequisite graph.

6. Determine eligible courses.

7. Create a semester.

8. Add/remove courses.

9. Validate prerequisites.

10. Save the plan.

If those ten things work properly:
we have a product. 66. MVP 2
Then:
Course recommendations
Dependency visualization
Unlock analysis
Course availability
Workload

67. V2 — Planning Engine
    Then:
    Automatic multi-semester planning

Constraints

Fastest plan

Balanced plan

Alternative plans

68. V3 — Advanced Intelligence
    Then:
    What-if analysis

Critical path

Risk analysis

Bottleneck detection

Plan comparison

Plan resilience

69. V4 — Real Product
    Then:
    Authentication
    Profiles
    Real users
    Community workload data
    Admin system
    Production deployment
    Monitoring
    Analytics

70. Development Roadmap
    Phase 0 — Research
    Understand:
    degree requirements
    course relationships
    prerequisite types
    curriculum versions
    course availability

Produce a small verified dataset.
Phase 1 — Foundation
Create:
Git repository

/backend
/frontend
/data
/docs

Configure:
Spring Boot
PostgreSQL
React/TS
Docker Compose

Phase 2 — Domain Model
Implement:
Course
Prerequisite
Degree
DegreeVersion
Requirement

Phase 3 — Graph Engine
Implement:
graph construction
DFS/BFS
cycle detection
topological sorting
dependency traversal

Unit-test everything.
Phase 4 — Progress Engine
Implement degree requirement evaluation.
Phase 5 — User Planning
Implement:
completed courses
semester creation
course selection
eligibility validation

Phase 6 — Frontend MVP
Build:
Dashboard
Course browser
Progress
Semester planner

Phase 7 — Recommendation Engine
Implement explainable recommendations.
Phase 8 — Automatic Planner
Start simple.
Then improve algorithmically.
Phase 9 — Advanced Analysis
Implement:
What-if
Critical path
Impact analysis
Plan comparison

Phase 10 — Production
Add:
authentication
Docker production build
CI/CD
deployment
logging
monitoring

71. Git Strategy
    Use Git from day one.
    Do not create the entire project and upload one giant commit.
    History should show development.
    Examples:
    feat: add course domain model

feat: implement prerequisite graph

test: add cycle detection tests

feat: calculate eligible courses

feat: add degree progress engine

feat: implement semester validation

refactor: separate planning engine

perf: optimize dependency traversal

This itself demonstrates professional development habits. 72. Branching
Keep it simple.
main

feature/course-model
feature/graph-engine
feature/planner
...

Use pull requests even when working alone for important changes.
This allows documentation of design decisions. 73. Documentation
Repository should eventually contain:
README.md

docs/
architecture.md
data-model.md
planning-engine.md
algorithms.md
api.md

74. README
    The README should eventually contain:
    Problem

Solution

Screenshots

Architecture

Core algorithms

Tech stack

How planning works

Performance results

How to run locally

Tests

Future improvements

Not just installation commands. 75. Architecture Decision Records
Optional but excellent.
Example:
docs/adr/

001-modular-monolith.md
002-postgresql.md
003-graph-representation.md
004-planner-strategy.md

Each explains:
Problem
Options
Decision
Trade-offs

That gives you great material for interviews. 76. Code Quality
Priorities:
Readable code
Clear naming
Small responsibilities
Tests
Separation of concerns
No unnecessary abstractions

Do NOT over-engineer. 77. Important Rule for Codex
Codex should not build huge parts of the system without explaining them.
This is a learning/portfolio project.
When implementing a significant component:

1. Explain the problem.
2. Explain the proposed design.
3. Explain relevant CS concepts.
4. Implement incrementally.
5. Add tests.
6. Explain trade-offs.
7. Let the student understand the implementation.
   The goal is that the owner can explain every major component during an interview.
   This rule is extremely important.
8. Another Rule for Codex
   Do not introduce new frameworks, databases, queues, cloud services or architectural patterns without justification.
   Before adding technology, answer:
   What problem does it solve?

Why is the current solution insufficient?

What trade-off does it introduce?

79. Another Rule — Algorithms
    Prefer implementing core algorithms ourselves when they represent the educational/technical value of the project.
    Libraries may handle infrastructure.
    They should not replace the central planning logic.
80. Success Criteria
    The project is successful when a real CS student can:
    open DegreePath

select their curriculum

enter completed courses

immediately understand degree progress

see what they can take next

build future semesters

receive warnings about invalid choices

ask the planner for a possible completion path

understand why it recommends that path

modify a course

see how that affects the future

81. Portfolio Success Criteria
    For the project to deserve significant CV space, we eventually want evidence such as:
    X real users

Y plans generated

Z courses modeled

N prerequisite relationships

Planner runtime benchmark

Test coverage

Production deployment

Only report real numbers. 82. What Makes This Project Different
It is NOT:
Course CRUD app

It is:
Academic Constraint & Planning Engine

with a real product built around it.
The interesting computer-science problems are:
Graph modeling

Dependency resolution

Topological ordering

Constraint satisfaction

Search

Optimization

Requirement trees

Impact propagation

Critical path analysis

Recommendation ranking

while the engineering side demonstrates:
Java

Spring Boot

PostgreSQL

REST

React

TypeScript

Docker

Testing

CI/CD

Cloud

Software architecture

That combination is the heart of the project. 83. The Story Behind the Project
This should remain part of the project identity.
The project started because an actual Open University Computer Science student repeatedly encountered the same problem:
Planning the degree requires manually combining prerequisite information, degree requirements, course availability and advice from other students.

Instead of continuing to solve the problem manually each semester, the goal is to model the problem computationally and build the tool that should exist. 84. North Star
Whenever there is uncertainty about whether a feature belongs in DegreePath, ask:
Does this feature make it easier for a student to understand, plan, optimize or complete their degree?

If not, it probably doesn't belong.
And whenever there is uncertainty about a technical addition, ask:
Does this technology solve a real problem in DegreePath, or are we adding it because it looks good on a CV?

If it's the latter:
don't add it. 85. Final Development Principle
Build:
Correct
↓
Simple
↓
Tested
↓
Useful
↓
Measurable
↓
Optimized
↓
Scalable

in that order.
Do not design for millions of users before the first student can successfully plan one semester.
