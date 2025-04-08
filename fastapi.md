
# FastAPI Errors

## Error Category: FastAPI dependencies

**Date:** 26 Mar 2025

**Context:** Using Depends() in FastAPI and instantiating dependencies.

**Error:**
```python
Error: NotImplementedError: Please override in child classes

```
**Code:**

```python
class TokenBearer(HTTPBearer):
    def __init__(self, auto_error: bool = True):
        super().__init__(auto_error=auto_error)


    async def __call__(self, request: Request):
        print(f"Called from: {self.__class__.__name__}")
        credentials = await super().__call__(request)
        token = credentials.credentials
        token_data = token_service.decode_token(token)
        self.verify_token_data(token_data)

        return token_data

    def verify_token_data(self, token_data):
        raise NotImplementedError("Please override in child classes")

class AccessTokenBearer(TokenBearer):
    def verify_token_data(self, token_data: dict) -> None:
        if token_data and token_data.get('refresh', False):
                raise AccessTokenRequiredError


class RefreshTokenBearer(TokenBearer):
    def verify_token_data(self, token_data: dict) -> None:
        if token_data and not token_data.get('refresh', False):
                raise RefreshTokenRequiredError
        
        
        
        
        
        
        
        
@router.get('/refresh_token')
def refresh_token(token_details: dict = Depends(RefreshTokenBearer())):
    return {}

```

**Solution:**

```python
# Explicitly instantiate the dependency as a global variable
refresh = RefreshTokenBearer()

@router.get('/refresh_token')
def refresh_token(token_details: dict = Depends(refresh)):
    return {}

```

**Explanation:**  
Problem: FastAPI’s Depends() caches dependencies, which means if classes aren't explicitly instantiated before passing it to Depends(), it could lead to incorrect instantiations (in this case, using TokenBearer instead of RefreshTokenBearer).

Solution: By explicitly creating the dependency (refresh = RefreshTokenBearer()) before passing it to Depends(), FastAPI uses the correct class and avoids using the wrong parent class (which in turn calls verify_token_data incorrectly).




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