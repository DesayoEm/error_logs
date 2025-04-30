# Mongo DB Errors

###
## Error Category: Empty Results from MongoDB Query

**Date:** 30 Apr 2024
**Context:**  Endpoint was returning an empty list even though data existed in the database.

**Error:**
```python
Error: XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX
```

**Code:**

In the logging function: ```product_id``` was converted to a string representation of an ObjectId


```python
data = {
    "product_id": str(ObjectId(product_id)),
    ...
```

In the query, the same ID was converted to an actual ```ObjectId``` object

```python
    def validate_obj_id(id_str: str, entity_name: str = "Document")-> ObjectId:
        """Validate and convert string ID to ObjectId."""
        try:
            return ObjectId(id_str)
        except InvalidId as e:
            raise InvalidIdError(entity =entity_name, detail=str(e))
```

```python
obj_id = self.validate_obj_id(product_id, "Product")
cursor = self.price_logs.find({"product_id": obj_id})
```


**Solution:**
Made the storage and query formats consistent by querying the product as a string and not a serialized ```objectID```

```python
        try:
            skip = (page - 1) * per_page if page > 0 else 0
            cursor = self.price_logs.find({"product_id":product_id}) \
                .sort("date_checked", pymongo.ASCENDING) \
                .skip(skip).limit(per_page)

            yield from self.yield_documents(cursor)
```

 
