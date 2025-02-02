
# FastAPI Errors

## Error Category: Exception Handling

### Non-Serializable Custom Exception

**Date:** 2 Feb 2025

**Context:** TraKademik

**Error:**
```python
TypeError: Object of type StudentIdFormatError is not JSON serializable
```

**Solution:**

```python
try:
    student_id = self.profile_service.validate_student_id(student_id)
except StudentIdFormatError as e:
    raise HTTPException(status_code=400, detail=str(e))
```

**Explanation:**  
Converting the exception to string makes it serializable for JSON responses. 
FastAPI needs to convert all response data to JSON, and custom exceptions aren't automatically serializable. Using `str(e)` converts the exception message to a string, which can be safely serialized.
