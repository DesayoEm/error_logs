#Docker Errors

## Error Category: Port mapping issues in Docker config

**Date:** 11 Feb 2024
**Context:** Context

**Error:**
```python
Error: sqlalchemy.exc.OperationalError: (psycopg2.OperationalError) 
connection to server at "localhost" (::1), port 5433 failed: Connection refused (0...
```

**Code:**
```python
container_config = {
    'name': container_name,
    'image': 'postgres:16.1-alpine3.19',
    'detach': True,
    'ports':{'5432':5433},
    'environment':{
        'POSTGRES_USER': 'postgres',
        'POSTGRES_PASSWORD': TEST_DB_PWD,
        'POSTGRES_DB': 'testtrakademik'
    },
    'volumes':[f"{scripts_dir}:/docker-entrypoint-initdb.d"]
}
```
**Solution:**

```python
    container_config = {
    'name': container_name,
    'image': 'postgres:16.1-alpine3.19',
    'detach': True,
    'ports': {'5432/tcp': ('127.0.0.1', 5433)},
    'environment':{
        'POSTGRES_USER': 'postgres',
        'POSTGRES_PASSWORD': TEST_DB_PWD,
        'POSTGRES_DB': 'testtrakademik'
    },
    'volumes':[f"{scripts_dir}:/docker-entrypoint-initdb.d"]

}
container = client.containers.run(**container_config)

```

**Explanation:**  
Properly specifying the host interface (127.0.0.1 instead of 0.0.0.0) and the explicit TCP protocol specification
Using the correct container port (5432) mapped to the host port (5433)




###
## Error Category: Docker

**Date:** 11 Feb 2024
**Context:** Containerising Postgres database

**Error:**
```python
RuntimeError: Container did not stabilize on time
```

**Code:**

*Inside setup.sh:*
```python
export PGUSER='postgres'

psql -c 'CREATE DATABASE testtrakademik'
psql testtrakademik -c  'CREATE EXTENSION IF NOT EXISTS \'uuid-ossp'\';

```

*Inside docker_utils:*
```python
def is_container_ready(container):
    container.reload()
    return container.status =='running'

def wait_for_stable_status(container, duration = 3, interval = 1):
    start_time = time.time()
    stable_count = 0
    while time.time() - start_time < duration:
        if is_container_ready(container):
            stable_count +=1
        else:
            stable_count = 0
        if stable_count >= duration/interval:
            return True
        time.sleep(interval)
    return False


def start_database_container():
    client = docker.from_env()
    scripts_dir = os.path.abspath('./scripts')
    container_name = 'testtrakademik'

    try:
        existing_container = client.containers.get(container_name)
        print(f"Container {container_name} exists. stopping and removing...")
        existing_container.stop()
        existing_container.remove()
        print(f"Container {container_name} stopped and removed")
    except docker.errors.NotFound:
        print(f"Container {container_name} does not exist")


    container_config = {
        'name': container_name,
        'image': 'postgres:16.1-alpine3.19',
        'detach': True,
        'ports': {'5432/tcp': ('127.0.0.1', 5433)},
        'environment':{
            'POSTGRES_USER': 'postgres',
            'POSTGRES_PASSWORD': TEST_DB_PWD,
            'POSTGRES_DB': 'testtrakademik'
        },
        'volumes':[f"{scripts_dir}:/docker-entrypoint-initdb.d"]

    }
    container = client.containers.run(**container_config)

    while not is_container_ready(container):
        time.sleep(2)
    if not wait_for_stable_status(container):
        raise RuntimeError('Container did not stabilize on time')
    return container

```


**Solution:**
*1. Used debug print statements in wait function:*

```python
def wait_for_stable_status(container, duration = 3, interval = 1):
    start_time = time.time()
    stable_count = 0
    while time.time() - start_time < duration:
        if is_container_ready(container):
            stable_count +=1
        else:
            stable_count = 0
        if stable_count >= duration/interval:
            return True
        time.sleep(interval)
    print("Container logs:")
    print(container.logs().decode('utf-8'))
    return False
```
*got a more explicit error message:*
```python
2025-02-11 01:01:00.915 UTC [50] ERROR:  database "testtrakademik" already exists
2025-02-11 01:01:00.915 UTC [50] STATEMENT:  CREATE DATABASE testtrakademik
ERROR:  database "testtrakademik" already exists
2025-02-11 01:01:00.926 UTC [52] ERROR:  syntax error at or near "\" at character 32
2025-02-11 01:01:00.926 UTC [52] STATEMENT:  CREATE EXTENSION IF NOT EXISTS \uuid-ossp
    ERROR:  syntax error at or near "\"
```

*fixed the uuid-ossp extension syntax*
```python
#!/bin/bash
set -e

psql -v ON_ERROR_STOP=1 --username "$POSTGRES_USER" --dbname "$POSTGRES_DB" <<-EOSQL
CREATE EXTENSION IF NOT EXISTS "uuid-ossp";
EOSQL
```
Ensured it had Unix line endings (LF instead of CRLF) by running

```python

(Get-Content scripts/setup.sh -Raw).Replace("`r`n", "`n") | Set-Content scripts/setup.sh -NoNewline

```
in powershell.

**Explanation:**  
Fixed the uuid-ossp extension syntax to use proper quotes.
Added proper psql connection using Docker's environment variables.
