
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