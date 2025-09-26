# DBMS Concepts Complete Cheat Sheet

## 1. Database Management System (DBMS) Fundamentals

### What is DBMS?
- **Definition**: Software that manages databases and provides interface between users and database
- **Purpose**: Store, retrieve, and manage data efficiently and securely

### DBMS Components
- **Database Engine**: Core service for storing, processing, and securing data
- **Database Schema**: Structure that represents logical view of database
- **Query Processor**: Interprets and executes database queries
- **Transaction Manager**: Ensures database consistency during concurrent access
- **Storage Manager**: Manages data storage on physical media

### Types of DBMS
1. **Hierarchical DBMS**: Tree-like structure (IMS)
2. **Network DBMS**: Graph structure with multiple parent-child relationships
3. **Relational DBMS**: Tables with rows and columns (MySQL, PostgreSQL, Oracle)
4. **Object-Oriented DBMS**: Stores data as objects
5. **NoSQL DBMS**: Non-relational (MongoDB, Cassandra, Redis)

---

## 2. Database Schema

### Definition
- **Schema**: Logical structure/blueprint of database
- **Instance**: Actual data stored in database at specific time

### Types of Schema
1. **Physical Schema**: How data is stored physically on storage media
2. **Logical Schema**: Structure of entire database for community of users
3. **View Schema (External)**: Structure of database for specific user groups

### Three-Schema Architecture
- **External Level**: User views and application programs
- **Conceptual Level**: Community user view (logical schema)
- **Internal Level**: Storage structure and access paths

### Schema Components
- **Tables/Relations**: Basic storage units
- **Attributes/Columns**: Properties of entities
- **Domains**: Set of allowable values for attributes
- **Constraints**: Rules that data must follow
- **Indexes**: Structures to speed up data retrieval

---

## 3. Entity-Relationship (ER) Model

### Basic Concepts
- **Entity**: Real-world object or thing (Student, Course, Department)
- **Attribute**: Property of entity (Name, Age, ID)
- **Relationship**: Association between entities

### Types of Attributes
1. **Simple**: Cannot be divided (Age, Name)
2. **Composite**: Can be divided into sub-parts (Full Name → First Name + Last Name)
3. **Single-valued**: Has one value (Date of Birth)
4. **Multi-valued**: Has multiple values (Phone Numbers)
5. **Derived**: Calculated from other attributes (Age from Date of Birth)
6. **Key Attribute**: Uniquely identifies entity instances

### Types of Entities
1. **Strong Entity**: Has primary key, exists independently
2. **Weak Entity**: No primary key, depends on strong entity
3. **Associative Entity**: Represents many-to-many relationship

### Relationship Types
1. **One-to-One (1:1)**: Each entity instance relates to one instance of another
2. **One-to-Many (1:M)**: One instance relates to many instances of another
3. **Many-to-Many (M:N)**: Many instances relate to many instances

### ER Diagram Symbols
- **Rectangle**: Entity
- **Ellipse**: Attribute
- **Diamond**: Relationship
- **Double Rectangle**: Weak Entity
- **Double Ellipse**: Multi-valued Attribute
- **Dashed Ellipse**: Derived Attribute
- **Underlined**: Key Attribute

### Extended ER Features
- **Specialization**: Top-down approach, general entity → specialized entities
- **Generalization**: Bottom-up approach, specialized entities → general entity
- **Aggregation**: Relationship treated as higher-level entity
- **Composition**: Strong form of aggregation

---

## 4. Relational Model

### Basic Concepts
- **Relation**: Table with rows and columns
- **Tuple**: Row in a table
- **Attribute**: Column in a table
- **Domain**: Set of allowable values for attribute
- **Degree**: Number of attributes in relation
- **Cardinality**: Number of tuples in relation

### Relational Model Constraints

#### Domain Constraints
- **Definition**: Restricts attribute values to specific domain
- **Example**: Age must be between 0 and 150

#### Key Constraints
1. **Super Key**: Set of attributes that uniquely identifies tuples
2. **Candidate Key**: Minimal super key
3. **Primary Key**: Selected candidate key
4. **Alternate Key**: Candidate keys not chosen as primary key
5. **Foreign Key**: References primary key of another relation

#### Referential Integrity Constraints
- **Definition**: Foreign key must match primary key value in referenced relation
- **Actions on Violation**:
  - **Restrict**: Reject operation
  - **Cascade**: Propagate changes
  - **Set Null**: Set foreign key to null
  - **Set Default**: Set foreign key to default value

### Codd's Rules (12 Rules for Relational Model)
1. **Information Rule**: All data stored as values in tables
2. **Guaranteed Access**: Every value accessible through table name, column name, and primary key
3. **Systematic Treatment of Nulls**: Uniform representation of missing information
4. **Dynamic On-line Catalog**: Metadata stored as ordinary data
5. **Comprehensive Data Sublanguage**: Support for DDL, DML, integrity constraints
6. **View Updating**: All theoretically updatable views are updatable
7. **High-level Insert, Update, Delete**: Set-at-a-time operations
8. **Physical Data Independence**: Changes to physical storage don't affect applications
9. **Logical Data Independence**: Changes to logical structure don't affect applications
10. **Integrity Independence**: Integrity constraints stored in catalog
11. **Distribution Independence**: Database distribution transparent to users
12. **Non-subversion**: Direct access cannot bypass integrity rules

---

## 5. Relational Algebra & Calculus

### Relational Algebra Operations

#### Basic Operations
1. **Selection (σ)**: Selects tuples satisfying condition
   - `σ(condition)(Relation)`
   - Example: `σ(age > 25)(Student)`

2. **Projection (π)**: Selects specific columns
   - `π(attribute list)(Relation)`
   - Example: `π(name, age)(Student)`

3. **Union (∪)**: Combines tuples from two relations
   - `R ∪ S`
   - Relations must be union-compatible

4. **Set Difference (-)**: Tuples in R but not in S
   - `R - S`

5. **Cartesian Product (×)**: All possible combinations
   - `R × S`

#### Derived Operations
1. **Intersection (∩)**: Common tuples
   - `R ∩ S = R - (R - S)`

2. **Join (⋈)**: Combines related tuples
   - **Natural Join**: `R ⋈ S`
   - **Theta Join**: `R ⋈θ S`
   - **Equi Join**: Special case of theta join with equality

3. **Division (÷)**: Find tuples in R associated with all tuples in S
   - `R ÷ S`

#### Additional Operations
- **Rename (ρ)**: Changes relation/attribute names
- **Assignment (←)**: Assigns result to relation variable
- **Outer Joins**: Left, Right, Full outer joins

### Relational Calculus

#### Tuple Relational Calculus (TRC)
- **Format**: `{t | P(t)}`
- **Meaning**: Set of all tuples t such that predicate P(t) is true
- **Example**: `{t | Student(t) ∧ t.age > 20}`

#### Domain Relational Calculus (DRC)
- **Format**: `{x₁, x₂, ..., xₙ | P(x₁, x₂, ..., xₙ)}`
- **Meaning**: Set of domain values where predicate P is true
- **Example**: `{name, age | Student(name, age, dept) ∧ dept = 'CS'}`

#### Quantifiers
- **Existential (∃)**: There exists
- **Universal (∀)**: For all

---

## 6. Normalization

### Purpose of Normalization
- **Eliminate Redundancy**: Reduce data duplication
- **Prevent Anomalies**: Insert, update, delete anomalies
- **Ensure Data Integrity**: Maintain consistency
- **Optimize Storage**: Reduce storage space

### Functional Dependencies (FD)
- **Definition**: X → Y means X functionally determines Y
- **Types**:
  - **Trivial**: Y ⊆ X (e.g., AB → A)
  - **Non-trivial**: Y ⊄ X
  - **Partial**: Non-key attribute depends on part of primary key
  - **Transitive**: X → Y and Y → Z, then X → Z

### Armstrong's Axioms
1. **Reflexivity**: If Y ⊆ X, then X → Y
2. **Augmentation**: If X → Y, then XZ → YZ
3. **Transitivity**: If X → Y and Y → Z, then X → Z

### Additional Rules
- **Union**: If X → Y and X → Z, then X → YZ
- **Decomposition**: If X → YZ, then X → Y and X → Z
- **Pseudo-transitivity**: If X → Y and WY → Z, then WX → Z

### Normal Forms

#### First Normal Form (1NF)
- **Requirement**: All attributes must have atomic (indivisible) values
- **Violations**: Multi-valued attributes, repeating groups
- **Example Issue**: Student table with multiple phone numbers in single cell

#### Second Normal Form (2NF)
- **Requirement**: 1NF + No partial dependencies on primary key
- **Condition**: Non-key attributes fully functionally dependent on primary key
- **Example Issue**: (StudentID, CourseID) → StudentName (partial dependency)

#### Third Normal Form (3NF)
- **Requirement**: 2NF + No transitive dependencies
- **Condition**: Non-key attributes not dependent on other non-key attributes
- **Example Issue**: StudentID → DeptID → DeptName

#### Boyce-Codd Normal Form (BCNF)
- **Requirement**: 3NF + Every determinant is a candidate key
- **Stricter than 3NF**
- **Condition**: For every FD X → Y, X must be a super key

#### Fourth Normal Form (4NF)
- **Requirement**: BCNF + No multi-valued dependencies
- **Multi-valued Dependency**: X →→ Y means for each X value, there's a set of Y values independent of other attributes

#### Fifth Normal Form (5NF)
- **Requirement**: 4NF + No join dependencies
- **Also called**: Project-Join Normal Form (PJNF)
- **Condition**: Cannot be decomposed without loss of information

### Denormalization
- **Purpose**: Improve query performance by introducing controlled redundancy
- **Trade-offs**: Performance vs. storage and consistency
- **Techniques**: Duplicate data, pre-computed values, materialized views

---

## 7. Transactions

### Transaction Definition
- **Transaction**: Unit of work performed against database
- **Properties**: Must maintain database consistency
- **Example**: Bank transfer (debit one account, credit another)

### ACID Properties

#### Atomicity
- **Definition**: Transaction is all-or-nothing
- **Guarantee**: Either all operations complete or none do
- **Implementation**: Using commit/rollback mechanisms

#### Consistency
- **Definition**: Transaction brings database from one consistent state to another
- **Guarantee**: All integrity constraints maintained
- **Example**: Sum of all account balances remains constant after transfer

#### Isolation
- **Definition**: Concurrent transactions don't interfere with each other
- **Guarantee**: Each transaction appears to execute in isolation
- **Levels**: Different isolation levels provide different guarantees

#### Durability
- **Definition**: Committed transaction changes are permanent
- **Guarantee**: Changes survive system failures
- **Implementation**: Using logs and recovery mechanisms

### Transaction States
1. **Active**: Transaction is executing
2. **Partially Committed**: Transaction completed but not yet committed
3. **Committed**: Transaction successfully completed
4. **Failed**: Transaction cannot be completed normally
5. **Aborted**: Transaction rolled back to initial state
6. **Terminated**: Transaction has left system

### Concurrency Control Problems

#### Lost Update Problem
- **Scenario**: Two transactions update same data, one update is lost
- **Example**: T1 and T2 both read balance=100, T1 adds 50, T2 adds 30, final result is 130 instead of 180

#### Dirty Read Problem
- **Scenario**: Transaction reads uncommitted data from another transaction
- **Example**: T1 updates balance, T2 reads new balance, T1 rolls back

#### Unrepeatable Read Problem
- **Scenario**: Transaction reads same data twice, gets different values
- **Example**: T1 reads balance twice, T2 updates balance between reads

#### Phantom Read Problem
- **Scenario**: Transaction re-executes query, gets different number of rows
- **Example**: T1 counts records, T2 inserts record, T1 recounts and gets different result

### Isolation Levels
1. **Read Uncommitted**: Allows dirty reads
2. **Read Committed**: Prevents dirty reads
3. **Repeatable Read**: Prevents dirty and unrepeatable reads
4. **Serializable**: Prevents all concurrency problems

### Concurrency Control Techniques

#### Lock-Based Protocols
- **Shared Lock (S)**: Multiple transactions can read
- **Exclusive Lock (X)**: Only one transaction can write
- **Two-Phase Locking (2PL)**:
  - **Growing Phase**: Acquire locks, cannot release
  - **Shrinking Phase**: Release locks, cannot acquire

#### Timestamp-Based Protocols
- **Basic Timestamp**: Each transaction has unique timestamp
- **Thomas Write Rule**: Ignore outdated write operations
- **Timestamp Ordering**: Execute transactions in timestamp order

#### Optimistic Concurrency Control
- **Phases**:
  1. **Read Phase**: Transaction reads and computes in private workspace
  2. **Validation Phase**: Check if transaction can be committed
  3. **Write Phase**: Apply changes to database

### Deadlock
- **Definition**: Two or more transactions waiting for each other
- **Example**: T1 locks A and waits for B, T2 locks B and waits for A

#### Deadlock Prevention
- **Wait-Die**: If Ti older than Tj, Ti waits; else Ti dies
- **Wound-Wait**: If Ti older than Tj, Ti wounds Tj; else Ti waits

#### Deadlock Detection and Recovery
- **Wait-for Graph**: Detect cycles in dependency graph
- **Recovery**: Abort one transaction to break deadlock

---

## 8. Database Recovery

### Failure Types
1. **Transaction Failure**: Logical errors, system errors
2. **System Failure**: Hardware/software crash, power failure
3. **Media Failure**: Disk crash, head crash

### Recovery Techniques

#### Log-Based Recovery
- **Write-Ahead Logging (WAL)**: Log records written before database modifications
- **Log Records**: Transaction ID, data item, old value, new value
- **Checkpoints**: Periodic saves of consistent database state

#### Recovery Algorithms
1. **Deferred Database Modification**: Apply changes only after commit
2. **Immediate Database Modification**: Apply changes immediately, use undo/redo

#### ARIES Recovery Algorithm
- **Analysis Phase**: Determine which transactions to undo/redo
- **Redo Phase**: Repeat history to recreate database state at crash
- **Undo Phase**: Undo uncommitted transactions

### Backup and Recovery Strategies
- **Full Backup**: Complete database copy
- **Incremental Backup**: Changes since last backup
- **Differential Backup**: Changes since last full backup
- **Hot Backup**: Backup while database is running
- **Cold Backup**: Backup while database is shut down

---

## 9. Indexing and Hashing

### Indexing Concepts
- **Purpose**: Speed up data retrieval operations
- **Trade-off**: Storage space and update overhead vs. query performance

### Types of Indexes

#### Primary Index
- **Definition**: Index on key field of ordered file
- **Sparse Index**: One entry per block
- **Dense Index**: One entry per record

#### Secondary Index
- **Definition**: Index on non-key field
- **Always Dense**: Must have entry for each record
- **Multiple Levels**: May have multiple index levels

#### Clustering Index
- **Definition**: Index on non-key field used for clustering
- **File Organization**: Records with same key value stored together

### B-Tree Indexes
- **Properties**: Balanced, multi-way tree
- **Node Structure**: Keys and pointers
- **Operations**: Search, insert, delete maintain balance
- **Variants**: B+ Trees (data only in leaves)

### Hash Indexing
- **Hash Function**: Maps key values to hash addresses
- **Collision Resolution**:
  - **Chaining**: Link colliding records
  - **Open Addressing**: Find next available slot
- **Dynamic Hashing**: Handle growing/shrinking files

### Bitmap Indexing
- **Purpose**: Efficient for low-cardinality attributes
- **Structure**: Bit vector for each distinct value
- **Operations**: Fast Boolean operations on bitmaps

---

## 10. Query Processing and Optimization

### Query Processing Steps
1. **Parsing**: Check syntax and semantics
2. **Translation**: Convert to relational algebra
3. **Optimization**: Find efficient execution plan
4. **Evaluation**: Execute optimized plan

### Query Optimization

#### Cost-Based Optimization
- **Cost Factors**: I/O operations, CPU time, memory usage
- **Statistics**: Table cardinalities, attribute selectivities
- **Cost Estimation**: Estimate cost of different plans

#### Rule-Based Optimization
- **Heuristics**: Apply transformation rules
- **Common Rules**:
  - Push selections down
  - Push projections down
  - Combine selections and cross products into joins

### Join Algorithms

#### Nested Loop Join
- **Algorithm**: For each tuple in R, scan S for matches
- **Cost**: |R| × |S| comparisons
- **Optimization**: Block nested loop join

#### Sort-Merge Join
- **Algorithm**: Sort both relations, then merge
- **Cost**: Sort cost + merge cost
- **Requirement**: Relations sorted on join attribute

#### Hash Join
- **Algorithm**: Build hash table on smaller relation, probe with larger
- **Cost**: One pass through each relation
- **Variants**: Grace hash join, hybrid hash join

---

## 11. Concurrency Control (Advanced)

### Serializability
- **Serial Schedule**: No interleaving of operations
- **Serializable Schedule**: Equivalent to some serial schedule
- **Conflict Serializability**: Based on conflicting operations
- **View Serializability**: Based on read-write patterns

### Multi-Version Concurrency Control (MVCC)
- **Concept**: Multiple versions of data items
- **Read Operations**: Read appropriate version
- **Write Operations**: Create new version
- **Advantages**: Readers don't block writers

### Granularity of Locking
- **Database Level**: Entire database
- **Table Level**: Entire table
- **Page Level**: Database page
- **Row Level**: Individual record
- **Field Level**: Individual attribute

#### Multiple Granularity Locking
- **Intention Locks**: Signal intention to lock at finer granularity
- **Lock Types**: IS, IX, S, X, SIX
- **Lock Compatibility Matrix**: Rules for compatible lock modes

---

## 12. Database Security

### Security Threats
1. **Unauthorized Access**: Access without permission
2. **Malicious Attacks**: SQL injection, DoS attacks
3. **Data Breaches**: Unauthorized disclosure
4. **Insider Threats**: Authorized users misusing access

### Access Control

#### Discretionary Access Control (DAC)
- **Principle**: Owner controls access to resources
- **Implementation**: Access control lists, capabilities

#### Mandatory Access Control (MAC)
- **Principle**: System enforces access policy
- **Implementation**: Security labels, clearance levels

#### Role-Based Access Control (RBAC)
- **Principle**: Access based on user roles
- **Components**: Users, roles, permissions
- **Advantages**: Easier administration, principle of least privilege

### Database Security Features
- **Authentication**: Verify user identity
- **Authorization**: Control access to resources
- **Auditing**: Monitor and log access
- **Encryption**: Protect data confidentiality
- **Backup and Recovery**: Ensure data availability

### SQL Injection Prevention
- **Parameterized Queries**: Use bound parameters
- **Input Validation**: Validate and sanitize input
- **Least Privilege**: Limit database permissions
- **Stored Procedures**: Use secure stored procedures

---

## 13. Distributed Databases

### Characteristics
- **Data Distribution**: Data spread across multiple sites
- **Network Communication**: Sites connected via network
- **Local Autonomy**: Each site has local control
- **Transparency**: Distribution hidden from users

### Types of Distribution
1. **Horizontal Fragmentation**: Divide tuples across sites
2. **Vertical Fragmentation**: Divide attributes across sites
3. **Mixed Fragmentation**: Combination of horizontal and vertical
4. **Replication**: Multiple copies at different sites

### Distributed Transaction Management
- **Two-Phase Commit (2PC)**: Ensure atomicity across sites
- **Phase 1**: Prepare phase - all sites vote to commit/abort
- **Phase 2**: Commit phase - coordinator announces decision

### CAP Theorem
- **Consistency**: All nodes see same data simultaneously
- **Availability**: System remains operational
- **Partition Tolerance**: System continues despite network failures
- **Trade-off**: Can achieve only two of three properties

---

## 14. NoSQL Databases

### Types of NoSQL Databases

#### Document Stores
- **Examples**: MongoDB, CouchDB
- **Structure**: Store documents (JSON-like)
- **Use Cases**: Content management, catalogs

#### Key-Value Stores
- **Examples**: Redis, Amazon DynamoDB
- **Structure**: Simple key-value pairs
- **Use Cases**: Caching, session management

#### Column-Family
- **Examples**: Cassandra, HBase
- **Structure**: Column families and columns
- **Use Cases**: Analytics, time-series data

#### Graph Databases
- **Examples**: Neo4j, Amazon Neptune
- **Structure**: Nodes and edges
- **Use Cases**: Social networks, recommendation systems

### BASE Properties
- **Basically Available**: System available most of the time
- **Soft State**: State may change without input
- **Eventual Consistency**: System becomes consistent over time

---

## 15. Data Warehousing and OLAP

### Data Warehouse Concepts
- **Purpose**: Support decision making and analytics
- **Characteristics**: Subject-oriented, integrated, time-variant, non-volatile
- **Architecture**: ETL processes, data marts, OLAP cubes

### OLTP vs. OLAP
| Aspect | OLTP | OLAP |
|--------|------|------|
| Purpose | Day-to-day operations | Analysis and reporting |
| Data | Current, detailed | Historical, summarized |
| Queries | Simple, frequent | Complex, ad-hoc |
| Design | Normalized | Denormalized |

### Star Schema
- **Fact Table**: Contains measures and foreign keys
- **Dimension Tables**: Contain descriptive attributes
- **Structure**: Fact table at center, dimensions around it

### Snowflake Schema
- **Structure**: Normalized dimension tables
- **Advantage**: Reduces redundancy
- **Disadvantage**: More complex joins

### OLAP Operations
- **Roll-up**: Aggregate data (city → state → country)
- **Drill-down**: Show details (year → quarter → month)
- **Slice**: Fix one dimension
- **Dice**: Fix multiple dimensions
- **Pivot**: Rotate cube to see different perspective

---

## Quick Reference Tables

### Normal Forms Summary
| Normal Form | Requirements | Key Point |
|-------------|-------------|-----------|
| 1NF | Atomic values | No multi-valued attributes |
| 2NF | 1NF + No partial dependencies | Full functional dependency |
| 3NF | 2NF + No transitive dependencies | No indirect dependencies |
| BCNF | 3NF + Every determinant is candidate key | Stricter than 3NF |
| 4NF | BCNF + No multi-valued dependencies | Independent multi-values |
| 5NF | 4NF + No join dependencies | Lossless join |

### Join Types Summary
| Join Type | Description | Result |
|-----------|-------------|---------|
| Inner Join | Matching rows only | Common rows |
| Left Join | All left + matching right | Left preserved |
| Right Join | All right + matching left | Right preserved |
| Full Outer | All rows from both | All rows preserved |
| Cross Join | Cartesian product | All combinations |

### Isolation Levels Summary
| Level | Dirty Read | Unrepeatable Read | Phantom Read |
|-------|------------|-------------------|--------------|
| Read Uncommitted | Yes | Yes | Yes |
| Read Committed | No | Yes | Yes |
| Repeatable Read | No | No | Yes |
| Serializable | No | No | No |