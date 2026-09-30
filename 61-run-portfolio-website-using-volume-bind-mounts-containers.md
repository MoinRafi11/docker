# Run Portfolio Website Using Volume, Bind Mounts & Containers

A simple portfolio application running with two Docker containers:

- **PHP + Apache** → Web container
- **PostgreSQL** → Database container
- **Docker Volume** → Persistent PostgreSQL data
- **Bind Mount** → Host project files mounted into the web container
- **Docker Network** → Communication between both containers

---

## 1. Project Structure

The existing project structure is:

```text
portfolio-two-containers/
├── db/
│   └── init.sql
├── src/
│   └── index.php
└── README.md
```

![Project structure](screenshots/Screenshot%202026-09-30%20095139.png)

---

## 2. Create Docker Network

Create a dedicated network for the two containers:

```bash
sudo docker network create portfolio-network
```

Verify:

```bash
sudo docker network ls
```

![Docker network](screenshots/Screenshot%202026-09-30%20095139.png)

---

## 3. Create Docker Volume

Create a named Docker volume for PostgreSQL data:

```bash
sudo docker volume create portfolio-db-data
```

Verify:

```bash
sudo docker volume ls
```

![Docker volume](screenshots/Screenshot%202026-09-30%20095334.png)

The volume is used for persistent PostgreSQL data.

---

## 4. Create PostgreSQL Container

Run the PostgreSQL container on the same Docker network:

```bash
sudo docker run -d \
  --name portfolio-db \
  --network portfolio-network \
  -e POSTGRES_USER=postgres \
  -e POSTGRES_PASSWORD=postgres@123 \
  -e POSTGRES_DB=portfolio \
  -v portfolio-db-data:/var/lib/postgresql/data \
  postgres:15-alpine
```

Check the running container:

```bash
sudo docker ps
```

![PostgreSQL container](screenshots/Screenshot%202026-09-30%20100155.png)

The important volume mapping is:

```text
portfolio-db-data
        ↓
/var/lib/postgresql/data
```

This stores PostgreSQL data in a Docker-managed volume.

---

## 5. Load Database Data

The SQL file is located at:

```text
db/init.sql
```

Copy it into the PostgreSQL container:

```bash
sudo docker cp init.sql portfolio-db:/tmp/init.sql
```

Execute the SQL file:

```bash
sudo docker exec -it portfolio-db \
psql -U postgres -d portfolio -f /tmp/init.sql
```

![Copy and execute init.sql](screenshots/Screenshot%202026-09-30%20101225.png)

The SQL file creates the `projects` table and inserts sample portfolio projects.

---

## 6. Verify PostgreSQL Data

Enter PostgreSQL:

```bash
sudo docker exec -it portfolio-db psql -U postgres -d portfolio
```

List the tables:

```sql
\dt
```

View the project records:

```sql
SELECT * FROM projects;
```

Example table:

```text
 id | title              | description                         | technology
----+--------------------+-------------------------------------+-------------------
 1  | Docker Storage Lab | Practical Docker projects...        | Docker
 2  | Linux Automation   | Linux administration...              | Linux / Ansible
 3  | Web Development    | Responsive web applications...      | HTML / CSS / JS
```

The database can contain additional project records added later.

---

## 7. Create the Web Container

The portfolio files are stored on the host in:

```text
/home/moin011/portfolio-two-containers/src
```

Run the PHP + Apache container:

```bash
sudo docker run -d \
  --name portfolio-web \
  --network portfolio-network \
  -p 8080:80 \
  -v /home/moin011/portfolio-two-containers/src:/var/www/html \
  php:8.2-apache
```

Check both containers:

```bash
sudo docker ps
```

![Web container with bind mount](screenshots/Screenshot%202026-09-30%20101534.png)

The bind mount connects:

```text
Host:
~/portfolio-two-containers/src

        ↓

Container:
/var/www/html
```

Any changes made to the files in the host `src` directory are immediately available inside the web container.

---

## 8. Access the Portfolio

Find the VM IP:

```bash
hostname -I
```

Open the application:

```text
http://YOUR_VM_IP:8080
```

Example:

```text
http://192.168.1.16:8080
```

![Portfolio homepage](screenshots/Screenshot%20(332).png)

---

## 9. Portfolio Projects

The portfolio contains a Projects section where project information can be displayed from the PostgreSQL database.

![Portfolio projects section](screenshots/Screenshot%20(333).png)

The PHP application connects to the PostgreSQL container using the Docker network:

```text
portfolio-web
      │
      │ portfolio-network
      ▼
portfolio-db
```

The database query retrieves project records:

```php
$stmt = $pdo->query(
    "SELECT title, description, technology FROM projects ORDER BY id"
);

$projects = $stmt->fetchAll(PDO::FETCH_ASSOC);
```

The records are displayed using:

```php
<?php foreach ($projects as $project): ?>

    <div class="project">

        <div class="project-tag">
            <?= htmlspecialchars($project["technology"]) ?>
        </div>

        <h3>
            <?= htmlspecialchars($project["title"]) ?>
        </h3>

        <p>
            <?= htmlspecialchars($project["description"]) ?>
        </p>

    </div>

<?php endforeach; ?>
```

The `foreach` loop displays every project returned by PostgreSQL.

---

## 10. Storage Used in This Project

### Docker Volume

PostgreSQL uses a Docker volume:

```text
portfolio-db-data
        ↓
/var/lib/postgresql/data
```

Purpose:

- Stores database data
- Keeps PostgreSQL data separate from the container
- Provides persistent storage

### Bind Mount

The web container uses a bind mount:

```text
/home/moin011/portfolio-two-containers/src
        ↓
/var/www/html
```

Purpose:

- Uses files directly from the host
- Changes to `index.php` are immediately reflected in the container
- Useful during development

---

## 11. Final Architecture

```text
                         Browser
                            │
                            │ :8080
                            ▼
                  ┌───────────────────┐
                  │   portfolio-web   │
                  │   PHP + Apache    │
                  └─────────┬─────────┘
                            │
                    portfolio-network
                            │
                            ▼
                  ┌───────────────────┐
                  │   portfolio-db    │
                  │    PostgreSQL     │
                  └─────────┬─────────┘
                            │
                            ▼
                  ┌───────────────────┐
                  │ portfolio-db-data │
                  │   Docker Volume   │
                  └───────────────────┘


Host Files
/home/moin011/portfolio-two-containers/src
                    │
                    │ Bind Mount
                    ▼
              /var/www/html
              portfolio-web
```

---

## 12. Key Commands

### Check containers

```bash
sudo docker ps
```

### Check network

```bash
sudo docker network ls
```

### Check volume

```bash
sudo docker volume ls
```

### Inspect web container mounts

```bash
sudo docker inspect portfolio-web
```

### Inspect database container mounts

```bash
sudo docker inspect portfolio-db
```

### Enter PostgreSQL

```bash
sudo docker exec -it portfolio-db psql -U postgres -d portfolio
```

### View database records

```sql
SELECT * FROM projects;
```

---

## Conclusion

This project demonstrates how Docker storage and containers can be combined in a practical application.

- **Bind mount** → Website source files
- **Docker volume** → PostgreSQL persistent data
- **Docker network** → Communication between containers
- **PHP + Apache** → Web application
- **PostgreSQL** → Database
```