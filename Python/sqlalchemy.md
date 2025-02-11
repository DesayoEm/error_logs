# SQLAlchemy Errors


### 
## Error Category: Category

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






