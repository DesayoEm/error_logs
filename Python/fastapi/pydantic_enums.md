# Test Errors

###
## Error Category: Category

**Date:** 08 Mar 2025
**Context:** FastAPI endpoint test failing when testing archive functionality

**Error:**
```python
AssertionError: Expected 'archive_department' to be called once. Called 0 times.
```

**Code:**
```python
@router.patch("/{department_id}", status_code=204)
def archive_department(department_id: UUID, reason: ArchiveReason,
                       db: Session = Depends(get_db)):
    staff_departments_crud = StaffDepartmentCrud(db)
    return staff_departments_crud.archive_department(department_id, reason)

# Test code:
def test_archive_department(mock_crud):
    mock_instance, test_uuid = mock_crud
    archive_data = {"reason": "ADMINISTRATIVE"}
    response = client.patch(f"/api/v1/staff/departments/{test_uuid}", json={"reason": "ADMINISTRATIVE"})
    reason = ArchiveReason.ADMINISTRATIVE
    mock_instance.archive_department.assert_called_once_with(test_uuid, reason)
```
**Solution:**

```python
# Create a Pydantic model to handle the request body
class ArchiveRequest(BaseModel):
    reason: ArchiveReason

# Update the router endpoint to use the model
@router.patch("/{department_id}", status_code=204)
def archive_department(
        department_id: UUID,
        archive_request: ArchiveRequest,
        db: Session = Depends(get_db)
):
    staff_departments_crud = StaffDepartmentCrud(db)
    return staff_departments_crud.archive_department(department_id, archive_request.reason)
```

**Explanation:**

When a FastAPI endpoint defines a parameter without specifying its source (not a path parameter and not from Depends()), FastAPI tries to extract it directly from the request body. The test was sending a JSON object with a nested "reason" field, but the endpoint was expecting the reason value directly. By creating a Pydantic model that matches the structure of the JSON body, FastAPI can correctly parse the request and extract the nested fields, ensuring the mock is called with the correct parameters during testing