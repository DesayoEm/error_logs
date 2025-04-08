
# SQLAlchemy Errors

## Error Category: Category

**Date:** 08 Apr 2025
**Context:** Deleting a parent entity (Educator) with children (EducatorQualification) having ondelete='CASCADE' on their foreign key.

**Error:**
```python
Error: psycopg2.errors.NotNullViolation: null value in column "educator_id" of relation "educator_qualifications" violates not-null constraint
DETAIL: Failing row contains (..., NULL, ...).

```

**Code:**
```python
educator_id: Mapped[UUID] = mapped_column(
    ForeignKey('educators.id', ondelete='CASCADE', name='fk_educator_qualifications_educator_id')
)
educator: Mapped['Educator'] = relationship(
    'Educator', 
    back_populates='qualifications',
    foreign_keys="[EducatorQualification.educator_id]"
)

```


**Solution:**

```python
educator: Mapped['Educator'] = relationship(
    'Educator',
    back_populates='qualifications',
    foreign_keys="[EducatorQualification.educator_id]",
    passive_deletes=True
)

```

**Explanation:**  
Even though the database foreign key uses ondelete='CASCADE', SQLAlchemy ORM by default tries to NULL the child foreign key before the parent is deleted.

Since educator_id is a NOT NULL column, this attempt fails and raises a constraint error.

By adding passive_deletes=True to the relationship(), SQLAlchemy trusts the database to handle the deletion, avoiding manual NULLing, and the cascade works properly.




###
## Error Category: Type Annotation Conflict

**Date:** 10 Mar 2025
**Context:**  SQLAlchemy ORM Model Definition

**Error:**
```python
TypeError: Boolean value of this clause is not defined
```
**Code:**
```python
class StaffAvailability(Base, TimeStampMixins):
    # ...
    date: Mapped[date] = mapped_column(Date)
    # ...
```
**Solution:**

```python
import datetime

class StaffAvailability(Base, TimeStampMixins):
    # ...
    date: Mapped[datetime.date] = mapped_column(Date)
    # ...

ALTERNATIVELY,

import datetime as pydate

class StaffAvailability(Base, TimeStampMixins):
    # ...
    date: Mapped[pydate] = mapped_column(Date)
    # ...
```

**Explanation:**  
The error occured because of a naming conflict between the column name date and the Python type date used in the type annotation. 

When SQLAlchemy processes type annotations, it needs to determine if the type includes None to establish nullability.

The ambiguity caused SQLAlchemy to try evaluating the clause itself as a boolean expression rather than checking the type, resulting in the error. 
The solution is to either rename the column to something other than date or rename the imported type to avoid the collision.


###
## Error Category: Type Annotation Conflict
**Date:** 04 Feb 2024
**Context:** Query syntax

**Error:**
```python
Error: Class 'sqlalchemy.orm.query.Query' is not mapped
```

**Code:**
```python
user = session.query(Students).filter_by(student_id = 'STU/00/01/0000')
```
**Solution:**

```python
user = session.query(Students).filter_by(student_id = 'STU/00/01/0000').first()
```

**Explanation:**  
The error occurred because session.query() returns a Query object, 
I needed to actually execute the query to get the Student object before I could  delete it.

Added .first() after the query to actually execute it and get the Student object


### 
## Error Category: Category

**Date:** 02 Feb 2024
**Context:** Query arguments

**Error:**
```python
NameError: name 'student_id' is not defined. Did you mean: 'studentId'?
```

**Code:**
```python
student = self.db.query(Students).filter_by(studentId==student_id).first()
```
**Solution:**

```python
student = self.db.query(Students).filter_by(student_id=studentId).first()
```

**Explanation:**  
It's a keyword arg not a comparison therefore the column name comes first.






