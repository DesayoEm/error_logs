# SQLAlchemy Errors



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
import datetime
```python
class StaffAvailability(Base, TimeStampMixins):
    # ...
    date: Mapped[datetime.date] = mapped_column(Date)
    # ...
```

**Explanation:**  
The error occured because of a naming conflict between the column name date and the Python type date used in the type annotation. 

When SQLAlchemy processes type annotations, it needs to determine if the type includes None to establish nullability.

The ambiguity caused SQLAlchemy to try evaluating the clause itself as a boolean expression rather than checking the type, resulting in the error. 
The solution is to either rename the column to something other than date or rename the imported type to avoid the collision.






**Date:** 04 Feb 2024
**Context:** TraKademik

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
The error occured because session.query() returns a Query object, 
I needed to actually execute the query to get the Student object before I could  delete it.

Added .first() after the query to actually execute it and get the Student object


### 
## Error Category: Category

**Date:** 02 Feb 2024
**Context:** TraKademik

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
Its a keyword arg not a comparison therefore the column name comes first.






