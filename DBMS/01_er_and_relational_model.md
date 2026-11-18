# ER Model & Relational Model — Normalization

---

## 1. ER Diagram Components

| Symbol | Meaning |
|--------|---------|
| Rectangle | Entity |
| Ellipse | Attribute |
| Double Ellipse | Multivalued attribute |
| Dashed Ellipse | Derived attribute |
| Diamond | Relationship |
| Double Rectangle | Weak entity |
| Double Diamond | Identifying relationship |
| Underlined attribute | Primary key |

**Cardinality:**
- 1:1 — one employee has one parking spot
- 1:N — one department has many employees
- M:N — many students enroll in many courses

---

## 2. ER to Relational Mapping

| ER Construct | Relational Table |
|--------------|-----------------|
| Entity E with key K | Table E with K as PK |
| 1:1 relationship | Add FK to either side (prefer total participation side) |
| 1:N relationship | Add FK on the N side |
| M:N relationship | Separate junction table with FKs from both sides |
| Multivalued attribute | Separate table with entity PK as FK |
| Weak entity | Table with partial key + owner PK as composite PK |

---

## 3. Functional Dependencies

`A → B` means: knowing the value of A uniquely determines B.

**Armstrong's Axioms:**
1. **Reflexivity:** If B ⊆ A, then A → B
2. **Augmentation:** If A → B, then AC → BC
3. **Transitivity:** If A → B and B → C, then A → C

**Derived rules:** Union, decomposition, pseudotransitivity.

**Closure of attribute set A+:** All attributes determined by A.

---

## 4. Normal Forms

### 1NF (First Normal Form)
- All attributes are atomic (no multivalued or composite attributes)
- Each column has a single value
- Rows are uniquely identified

**Violation:** `{Student: "Alice", Courses: "Math, CS"}` → split courses into separate rows.

### 2NF (Second Normal Form)
- Must be 1NF
- No **partial dependency** — every non-key attribute is fully dependent on the entire primary key
- Only relevant when PK is composite

**Example violation:**
```
Order(OrderID, ProductID, ProductName, Quantity)
PK = {OrderID, ProductID}
ProductName depends only on ProductID (partial dep) → 2NF violation

Fix: Split into Order(OrderID, ProductID, Quantity) + Product(ProductID, ProductName)
```

### 3NF (Third Normal Form)
- Must be 2NF
- No **transitive dependency** — non-key attribute doesn't depend on another non-key attribute

**Example violation:**
```
Employee(EmpID, DeptID, DeptName)
EmpID → DeptID, DeptID → DeptName (transitive: EmpID →→ DeptName via DeptID)

Fix: Employee(EmpID, DeptID) + Department(DeptID, DeptName)
```

**3NF Test:** For every FD X → Y, either X is a superkey, or Y is part of some candidate key.

### BCNF (Boyce-Codd Normal Form)
- Stricter than 3NF
- For every FD X → Y, X must be a superkey

**3NF vs BCNF:** 3NF allows Y to be prime (part of candidate key); BCNF does not.

```
Example: Teaches(Student, Course, Teacher)
FDs: Teacher → Course (each teacher teaches one course)
     Student, Course → Teacher

Candidate keys: {Student, Teacher}, {Student, Course}
FD: Teacher → Course — Teacher is NOT a superkey → BCNF violation
But Teacher → Course, Teacher is part of CK {Student,Teacher} → 3NF is OK

BCNF decomposition:
  TeacherCourse(Teacher, Course) + StudentTeacher(Student, Teacher)
  Loses the FD Student,Course → Teacher (may lose some dependency preservation)
```

---

## 5. Normalization Summary

| Normal Form | Requirement | What It Eliminates |
|-------------|-------------|-------------------|
| 1NF | Atomic values, unique rows | Repeating groups |
| 2NF | Full functional dependency on entire PK | Partial dependencies |
| 3NF | No transitive dependencies | Transitive dependencies |
| BCNF | Every determinant is a superkey | Anomalies from non-superkey determinants |
| 4NF | No multivalued dependencies | Multivalued dependencies |

---

## 6. Update / Insert / Delete Anomalies

```
BadTable(EmpID, EmpName, DeptID, DeptName, DeptLocation)

Insert anomaly: Can't add a department without an employee
Delete anomaly: Deleting last employee in a dept loses dept info
Update anomaly: Changing DeptLocation requires updating all employee rows
```

Normalization eliminates these anomalies.

---

## 7. Practice Questions

1. Convert the ER diagram: Student (StudentID, Name) —< enrolls >— Course (CourseID, Title) with grade attribute on enrollment
2. Identify all FDs and normal form violations in `Orders(OID, CustID, CustName, CustCity, ProdID, ProdName, Qty, Price)`
3. Decompose to BCNF: `R(A, B, C, D)` with FDs `A → B`, `B → C`, `C → D`
