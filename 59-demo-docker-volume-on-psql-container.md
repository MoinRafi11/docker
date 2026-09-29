# Demo: Docker Volume on PostgreSQL Container

This project demonstrates how a Docker volume can be used to persist PostgreSQL data.

The same Docker volume is attached to a new PostgreSQL container after the original container is removed. The data remains available because it is stored in the volume.

---

## 1. Create a Docker Volume

Create a volume named `psql-data`:

```bash
sudo docker volume create psql-data
```

Check the available volumes:

```bash
sudo docker volume ls
```

![Create PostgreSQL Volume](screenshots/Screenshot%20(322).png)

---

## 2. Create PostgreSQL Container

Create a PostgreSQL container and mount the `psql-data` volume to the PostgreSQL data directory:

```bash
sudo docker run -d \
  --name psql-demo \
  -e POSTGRES_PASSWORD=postgres \
  -e POSTGRES_DB=testdb \
  -v psql-data:/var/lib/postgresql/data \
  postgres:15-alpine
```

The volume is mounted at:

```text
/var/lib/postgresql/data
```

---

## 3. Create Sample Data

Access PostgreSQL inside the container:

```bash
sudo docker exec -it psql-demo psql -U postgres -d testdb
```

Create a sample table:

```sql
CREATE TABLE students (
    id SERIAL PRIMARY KEY,
    name VARCHAR(100),
    course VARCHAR(100)
);
```

Insert sample data:

```sql
INSERT INTO students (name, course) VALUES
('Moin', 'BCA'),
('Ayan', 'BCA'),
('Rahul', 'MCA');
```

Verify the data:

```sql
SELECT * FROM students;
```

![PostgreSQL Sample Data](screenshots/Screenshot%20(323).png)

Exit PostgreSQL:

```sql
\q
```

---

## 4. Remove the PostgreSQL Container

Remove the original container:

```bash
sudo docker rm -f psql-demo
```

The container is removed, but the `psql-data` volume remains.

Create a new PostgreSQL container using the **same volume**:

```bash
sudo docker run -d \
  --name psql-demo-new \
  -e POSTGRES_PASSWORD=postgres \
  -e POSTGRES_DB=testdb \
  -v psql-data:/var/lib/postgresql/data \
  postgres:15-alpine
```

![Remove and Recreate PostgreSQL Container](screenshots/Screenshot%20(324).png)

---

## 5. Verify the Persistent Data

Connect to PostgreSQL in the new container:

```bash
sudo docker exec -it psql-demo-new psql -U postgres -d testdb
```

Check the tables:

```sql
\dt
```

Then check the stored data:

```sql
SELECT * FROM students;
```

The `students` table and its data are still available even though the original PostgreSQL container was removed.

![Verify Persistent PostgreSQL Data](screenshots/Screenshot%20(325).png)

---

## How It Works

```text
             Docker Volume
                psql-data
                    │
                    ▼
        ┌─────────────────────┐
        │   psql-demo         │
        │   PostgreSQL        │
        └─────────────────────┘
                    │
              Container removed
                    │
                    ▼
        ┌─────────────────────┐
        │   psql-demo-new     │
        │   PostgreSQL        │
        └─────────────────────┘
                    │
                    ▼
              Same psql-data
                    │
                    ▼
          Previous data remains
```

---

## Important Point

Removing a container does **not** remove the Docker volume automatically.

The PostgreSQL data remains available as long as the `psql-data` volume is preserved.

To remove the volume manually:

```bash
sudo docker volume rm psql-data
```

---

## Summary

- Created a Docker volume named `psql-data`.
- Mounted the volume to a PostgreSQL container.
- Created and stored sample PostgreSQL data.
- Removed the original PostgreSQL container.
- Created a new PostgreSQL container using the same volume.
- Verified that the original data was still available.

This demonstrates how Docker volumes provide **persistent storage for PostgreSQL containers**.