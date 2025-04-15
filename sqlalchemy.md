
# SQLAlchemy Errors


**Date:** 15 Apr 2025
**Context:** Attempting to delete an entity only if its set null on delete,
expecting ON DELETE SET NULL constraints to pass.

**Error:**
```python
WARNING:kademia:RoleDeletionConstraintError | d9ebeb39-535a-403e-8db3-4d42fec35c23 |
Deletion blocked: Foreign key constraint fk_staff_roles_staff_archived_by not be SET NULL for safe deletion

# This is a custom error triggered when a required ON DELETE SET NULL constraint is either missing or misreported.


```

**Code:**
```python
for fk in inspector.get_foreign_keys(table_name):
    fk_name = fk['name']
    if fk.get('ondelete', '').upper() != 'SET NULL':
        raise ForeignKeyConstraintMisconfiguredError(fk_name=fk_name)

```

**Solution:**

```python
def get_fk_delete_rules_from_info_schema(session, table_name: str) -> dict:
    query = text("""
        SELECT tc.constraint_name, rc.delete_rule
        FROM information_schema.table_constraints tc
        JOIN information_schema.referential_constraints rc
          ON tc.constraint_name = rc.constraint_name
        WHERE tc.constraint_type = 'FOREIGN KEY'
          AND tc.table_name = :table_name
    """)
    result = session.execute(query, {'table_name': table_name}).fetchall()
    return {row.constraint_name: row.delete_rule.upper() for row in result}


```

**Explanation:**  
SQLAlchemy's inspector.get_foreign_keys() does not reliably return the ondelete rule for foreign key constraints, particularly with PostgreSQL. 
This results in false negatives, where constraints that are actually ON DELETE SET NULL in the database appear missing or invalid in the code.

Using information_schema.referential_constraints via parameterized raw SQL provides an accurate and secure way to verify delete behavior, ensuring validation checks reflect the actual DB state.
###


**Date:** 08 Apr 2025
**Context:** Deleting a Parent entity that has related Child records (CASCADE).

**Error:**
```python
Error: psycopg2.errors.NotNullViolation: null value in column "educator_id" of relation "educator_qualifications" violates not-null constraint
DETAIL: Failing row contains (..., NULL, ...).

```

**Code:**
```python
class Educator(Staff):
    __tablename__ = 'educators'

    id: Mapped[UUID] = mapped_column(
        ForeignKey('staff.id', ondelete='CASCADE', name='fk_educators_staff_id'),
        primary_key=True
    )

    qualifications: Mapped[List['EducatorQualification']] = relationship(
        back_populates='educator'
    )
    
    class EducatorQualification(Base, AuditMixins, TimeStampMixins, ArchiveMixins):
    educator_id: Mapped[UUID] = mapped_column(ForeignKey('educators.id',
            ondelete='CASCADE', name='fk_educator_qualifications_educators_educator_id')
        )

    # Relationships
    educator: Mapped['Educator'] = relationship(
        'Educator', back_populates='qualifications',
        foreign_keys="[EducatorQualification.educator_id]"
    )


```


**Solution:**

```python
class Educator(Staff):
    __tablename__ = 'educators'

    id: Mapped[UUID] = mapped_column(
        ForeignKey('staff.id', ondelete='CASCADE', name='fk_educators_staff_id'),
        primary_key=True
    )

    qualifications: Mapped[List['EducatorQualification']] = relationship(
        back_populates='educator',
        cascade="all, delete-orphan"  #Added cascade behavior at ORM level
    )

)

class EducatorQualification(Base, AuditMixins, TimeStampMixins, ArchiveMixins):
    educator_id: Mapped[UUID] = mapped_column(ForeignKey('educators.id',
            ondelete='CASCADE', name='fk_educator_qualifications_educators_educator_id')
        )

    # Relationships
    educator: Mapped['Educator'] = relationship(
        'Educator', back_populates='qualifications',
        foreign_keys="[EducatorQualification.educator_id]", passive_deletes=True
    )

```

**Explanation:**  
By adding cascade=\"all, delete-orphan\" to the parent Educator.qualifications relationship, SQLAlchemy ensures that when an Educator is deleted, all associated EducatorQualification records are also deleted automatically before the database constraints are triggered.

This solves the NOT NULL violation by cleaning up dependent records in the ORM session.

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






