## Error Category: Pattern Match

**Date:** 02 Feb 2024
**Context:** Student_id pattern

**Error:**
```python
Error: Pattern not matching STU/00/00/0000
```

**Code:**
```python
match = re.match(value, pattern)  # Wrong order of parameters
```
**Solution:**

```python
match = re.match(pattern, value)  # Pattern comes first
```

**Explanation:**  
The patterns are a match but `re.match` has a parameter order.

First parameter - the pattern to match against
Second parameter - the string to be matched




## Error Category: Pattern Match

**Date:** 02 Feb 2024
**Context:** Student_id pattern

**Error:**
```python
re.error: global flags not at the start of the expression at position 1
```

**Code:**
```python
pattern = r'^(?i)STU/(\d{2})/(\d{2})/([0-9]{4})$'  # (?i) after ^
```
**Solution:**

```python
pattern = r'(?i)^STU/(\d{2})/(\d{2})/([0-9]{4})$'  # (?i) before ^
```
