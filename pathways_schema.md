# Pathways Database Schema (Version 6)

## ICS499 Capstone Project

**Metro State University**

---

# Overview

The **Pathways** application helps middle and high school students discover educational opportunities including:

* Competitions
* Scholarships
* Summer Programs
* Research Opportunities
* Internships
* Camps
* Clubs
* Volunteer Activities
* Leadership Programs
* Career Exploration

Students will build a mobile application using **Thunkable** backed by a relational database.

The supplied Excel spreadsheet contains the initial data set.

The goal of this project is to design a **production-quality backend database** that can scale to tens of thousands of opportunities and support future AI-powered search and recommendations.

---

# System Architecture

```
                 +------------------+
                 |  Thunkable App   |
                 +--------+---------+
                          |
                     REST API
                          |
                 +--------+---------+
                 |  MySQL Database  |
                 +--------+---------+
                          |
                 AI Search / RAG Engine
                          |
              Vector Database (Future)
```

---

# Database Design

The database follows **Third Normal Form (3NF)**.

Advantages:

* Eliminates duplicate data
* Improves maintainability
* Easier searching
* Better scalability
* Supports AI-powered recommendations

---

# Main Tables

```
Opportunity
Category
Field
State
GradeBand
Tag

Bridge Tables

OpportunityCategory
OpportunityField
OpportunityGradeBand
OpportunityTag
```

---

# Opportunity Table

| Column          | Type         |
| --------------- | ------------ |
| OpportunityID   | CHAR(36)     |
| Title           | VARCHAR(250) |
| Description     | TEXT         |
| Organization    | VARCHAR(200) |
| Website         | VARCHAR(500) |
| Cost            | VARCHAR(50)  |
| Deadline        | DATE         |
| DeliveryMode    | ENUM         |
| NationalFlag    | BOOLEAN      |
| LastVerified    | DATE         |
| AISummary       | TEXT         |
| EmbeddingText   | TEXT         |
| SearchKeywords  | TEXT         |
| PopularityScore | INT          |

---

# Category Table

| Column     | Type         |
| ---------- | ------------ |
| CategoryID | INT          |
| Name       | VARCHAR(100) |

Examples:

* Competition
* Scholarship
* Camp
* Internship
* Summer Program
* Research
* Volunteering

---

# Field Table

| Column  | Type         |
| ------- | ------------ |
| FieldID | INT          |
| Name    | VARCHAR(100) |

Examples:

* Artificial Intelligence
* Computer Science
* Engineering
* Mathematics
* Biology
* Chemistry
* Physics
* Business
* Arts
* Humanities

---

# GradeBand Table

Examples:

```
K-2
3-5
6-8
9-10
11-12
9-12
College
All Ages
```

---

# State Table

Examples:

```
National
California
Texas
Minnesota
Florida
Georgia
```

---

# Tag Table

Tags improve searching.

Examples:

```
AI
Coding
STEM
Leadership
Scholarship
Competition
Free
Virtual
Summer
Research
```

---

# Many-to-Many Relationships

One opportunity can have:

* Multiple categories
* Multiple fields
* Multiple grade bands
* Multiple tags

Therefore bridge tables are required.

```
OpportunityCategory
-----------------------
OpportunityID
CategoryID

OpportunityField
-----------------------
OpportunityID
FieldID

OpportunityGradeBand
-----------------------
OpportunityID
GradeBandID

OpportunityTag
-----------------------
OpportunityID
TagID
```

---

# MySQL Schema

```sql
CREATE TABLE Opportunity(

OpportunityID CHAR(36) PRIMARY KEY,

Title VARCHAR(250) NOT NULL,

Description TEXT,

Organization VARCHAR(200),

Website VARCHAR(500),

Cost VARCHAR(50),

Deadline DATE,

DeliveryMode ENUM('Virtual','In-Person','Hybrid'),

NationalFlag BOOLEAN,

LastVerified DATE,

AISummary TEXT,

EmbeddingText TEXT,

SearchKeywords TEXT,

PopularityScore INT DEFAULT 0

);

CREATE TABLE Category(

CategoryID INT AUTO_INCREMENT PRIMARY KEY,

Name VARCHAR(100) UNIQUE

);

CREATE TABLE Field(

FieldID INT AUTO_INCREMENT PRIMARY KEY,

Name VARCHAR(100) UNIQUE

);

CREATE TABLE GradeBand(

GradeBandID INT AUTO_INCREMENT PRIMARY KEY,

Name VARCHAR(50) UNIQUE

);

CREATE TABLE State(

StateID INT AUTO_INCREMENT PRIMARY KEY,

Name VARCHAR(100) UNIQUE

);

CREATE TABLE Tag(

TagID INT AUTO_INCREMENT PRIMARY KEY,

Name VARCHAR(100) UNIQUE

);

CREATE TABLE OpportunityCategory(

OpportunityID CHAR(36),

CategoryID INT,

PRIMARY KEY(OpportunityID,CategoryID),

FOREIGN KEY(OpportunityID)

REFERENCES Opportunity(OpportunityID),

FOREIGN KEY(CategoryID)

REFERENCES Category(CategoryID)

);

CREATE TABLE OpportunityField(

OpportunityID CHAR(36),

FieldID INT,

PRIMARY KEY(OpportunityID,FieldID),

FOREIGN KEY(OpportunityID)

REFERENCES Opportunity(OpportunityID),

FOREIGN KEY(FieldID)

REFERENCES Field(FieldID)

);

CREATE TABLE OpportunityGradeBand(

OpportunityID CHAR(36),

GradeBandID INT,

PRIMARY KEY(OpportunityID,GradeBandID),

FOREIGN KEY(OpportunityID)

REFERENCES Opportunity(OpportunityID),

FOREIGN KEY(GradeBandID)

REFERENCES GradeBand(GradeBandID)

);

CREATE TABLE OpportunityTag(

OpportunityID CHAR(36),

TagID INT,

PRIMARY KEY(OpportunityID,TagID),

FOREIGN KEY(OpportunityID)

REFERENCES Opportunity(OpportunityID),

FOREIGN KEY(TagID)

REFERENCES Tag(TagID)

);
```

---

# Sample Data

```sql
INSERT INTO Category VALUES
(1,'Competition'),
(2,'Scholarship'),
(3,'Research');

INSERT INTO Field VALUES
(1,'Computer Science'),
(2,'Artificial Intelligence'),
(3,'Engineering');

INSERT INTO GradeBand VALUES
(1,'6-8'),
(2,'9-12');

INSERT INTO State VALUES
(1,'National'),
(2,'California');

INSERT INTO Tag VALUES
(1,'Coding'),
(2,'AI'),
(3,'Free');

INSERT INTO Opportunity
VALUES
(
'OPP-000001',
'Congressional App Challenge',
'National coding competition for students',
'Congressional App Challenge',
'https://www.congressionalappchallenge.us',
'Free',
'2026-11-01',
'Virtual',
TRUE,
'2026-06-15',
NULL,
NULL,
'coding,app,competition',
0
);

INSERT INTO OpportunityCategory
VALUES
('OPP-000001',1);

INSERT INTO OpportunityField
VALUES
('OPP-000001',1);

INSERT INTO OpportunityGradeBand
VALUES
('OPP-000001',2);

INSERT INTO OpportunityTag
VALUES
('OPP-000001',1);
```

---

# Example Queries

## Find all AI opportunities

```sql
SELECT o.Title

FROM Opportunity o

JOIN OpportunityField ofld

ON o.OpportunityID=ofld.OpportunityID

JOIN Field f

ON ofld.FieldID=f.FieldID

WHERE f.Name='Artificial Intelligence';
```

---

## Find free competitions

```sql
SELECT Title

FROM Opportunity

WHERE Cost='Free';
```

---

## Find opportunities for grades 9-12

```sql
SELECT o.Title

FROM Opportunity o

JOIN OpportunityGradeBand g

ON o.OpportunityID=g.OpportunityID

JOIN GradeBand gb

ON g.GradeBandID=gb.GradeBandID

WHERE gb.Name='9-12';
```

---

## Find national opportunities

```sql
SELECT Title

FROM Opportunity

WHERE NationalFlag=TRUE;
```

---

# Future AI Enhancements

The database is designed to support future AI features:

* Semantic Search
* Personalized Recommendations
* AI Chat Assistant
* Career Path Suggestions
* Similar Opportunity Discovery
* LLM-powered Summaries
* Vector Database Integration
* Embedding Generation

Suggested future columns:

* VectorID
* RecommendationScore
* SimilarityScore
* OpenAIEmbedding
* SourceURL
* ApplicationStatus

---

# Stretch Goals

Students who finish early may implement:

* Favorites
* User Profiles
* Saved Opportunities
* Notifications
* Deadline Reminders
* AI Search
* Natural Language Queries
* Recommendation Engine
* Firebase Synchronization
* Supabase Backend
* Admin Portal
* Analytics Dashboard

---

# Capstone Deliverables

1. MySQL Database
2. Cleaned Data Import
3. Thunkable Mobile App
4. Search Screen
5. Filter Screen
6. Opportunity Details Screen
7. Favorites
8. Submission Documentation
9. Demonstration Video
10. Final Presentation

---

**End of Document**
