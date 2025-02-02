# SQLAlchemy Errors

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



