# 63 - Run Portfolio Using Custom Docker Images

This project demonstrates how to run a PHP portfolio application using **two custom Docker images**:

- Custom Web image using PHP and Apache
- Custom PostgreSQL database image
- Separate Web and Database containers
- Custom Docker network
- PHP-to-PostgreSQL connectivity
- Database-driven project information

The containers are created manually without Docker Compose.

---

## Project Architecture

```text
                    Browser
                       |
                       | :8080
                       v
              +-------------------+
              |   portfolio-web   |
              |-------------------|
              | Custom PHP/Apache |
              | Docker Image      |
              +---------+---------+
                        |
                        | Docker Network
                        | portfolio-network-new
                        |
                        v
              +-------------------+
              |   portfolio-db    |
              |-------------------|
              | Custom PostgreSQL |
              | Docker Image      |
              +-------------------+
                        |
                        v
                  PostgreSQL DB
                     portfolio
```

---

## Project Structure

```text
63-run-portfolio-using-custom-portfolio-image/
├── db/
│   ├── Dockerfile
│   └── init.sql
├── screenshots/
└── web/
    ├── Dockerfile
    └── index.php
```

### Project Structure Screenshot

![Project Structure](<screenshots/Screenshot 2026-09-30 153310.png>)

---

# 1. Create Project Directories

Create the project directory and required folders:

```bash
mkdir -p 63-run-portfolio-using-custom-portfolio-image/{web,db,screenshots}

cd 63-run-portfolio-using-custom-portfolio-image
```

Create the required files:

```bash
touch web/Dockerfile
touch web/index.php
touch db/Dockerfile
touch db/init.sql
```

---

# 2. Create the Web Dockerfile

The Web image is based on the official PHP Apache image.

It installs the PostgreSQL PHP extensions so that PHP can communicate with PostgreSQL.

### `web/Dockerfile`

```dockerfile
FROM php:8.2-apache

RUN apt-get update \
    && apt-get install -y libpq-dev \
    && docker-php-ext-install pdo pdo_pgsql \
    && rm -rf /var/lib/apt/lists/*

COPY index.php /var/www/html/index.php

EXPOSE 80

CMD ["apache2-foreground"]
```

### Explanation

| Instruction | Purpose |
|---|---|
| `FROM php:8.2-apache` | Uses PHP 8.2 with Apache |
| `apt-get install libpq-dev` | Installs PostgreSQL development libraries |
| `docker-php-ext-install pdo pdo_pgsql` | Enables PostgreSQL support in PHP |
| `COPY index.php` | Copies the portfolio into Apache's web directory |
| `EXPOSE 80` | Documents Apache's HTTP port |
| `CMD` | Starts Apache in the foreground |

---

# 3. Create the Portfolio

The portfolio is created using PHP and HTML/CSS.

The PHP application connects to PostgreSQL using environment variables:

```text
DB_HOST
DB_PORT
DB_NAME
DB_USER
DB_PASSWORD
```

The application retrieves project information from the PostgreSQL `projects` table and displays it dynamically.

---

# 4. Build the Custom Web Image

Build the first version of the custom Web image:

```bash
sudo docker build -t custom-portfolio-web:1.0 ./web
```

Verify the image:

```bash
sudo docker images
```

### Web Image Build

![Build Web Image](<screenshots/Screenshot 2026-09-30 154107.png>)

The custom image is created with the name:

```text
custom-portfolio-web:1.0
```

---

# 5. Create the Database Initialization File

The PostgreSQL database uses an initialization script to create the `projects` table and insert sample project data.

### `db/init.sql`

```sql
CREATE TABLE projects (
    id SERIAL PRIMARY KEY,
    name VARCHAR(100) NOT NULL,
    description TEXT NOT NULL,
    technology VARCHAR(100) NOT NULL
);

INSERT INTO projects (name, description, technology)
VALUES
(
    'Docker Portfolio',
    'Portfolio application running with custom Docker images.',
    'Docker, PHP, PostgreSQL'
),
(
    'Linux Automation',
    'Linux automation and administration practice project.',
    'Linux, Bash, Ansible'
),
(
    'Travel App',
    'Containerized travel application built for Docker practice.',
    'PHP, PostgreSQL, Docker'
);
```

---

# 6. Create the Database Dockerfile

### `db/Dockerfile`

```dockerfile
FROM postgres:15-alpine

COPY init.sql /docker-entrypoint-initdb.d/init.sql
```

The PostgreSQL image automatically executes SQL files placed inside:

```text
/docker-entrypoint-initdb.d/
```

during initial database initialization.

---

# 7. Build the Custom Database Image

Build the custom PostgreSQL image:

```bash
sudo docker build -t custom-portfolio-db:1.0 ./db
```

Then verify the available images:

```bash
sudo docker images
```

### Database Image Build

![Build Database Image](<screenshots/Screenshot 2026-09-30 154540.png>)

The custom database image is:

```text
custom-portfolio-db:1.0
```

The Web image is:

```text
custom-portfolio-web:1.0
```

---

# 8. Create the Docker Network

Create a dedicated Docker bridge network:

```bash
sudo docker network create portfolio-network-new
```

Verify the network:

```bash
sudo docker network ls
```

The custom network should appear in the list:

```text
portfolio-network-new
```

### Docker Network

![Docker Network](<screenshots/Screenshot 2026-09-30 154755.png>)

---

# 9. Run the PostgreSQL Container

Start the database container first:

```bash
sudo docker run -d \
  --name portfolio-db \
  --network portfolio-network-new \
  -e POSTGRES_DB=portfolio \
  -e POSTGRES_USER=portfolio_user \
  -e POSTGRES_PASSWORD=portfolio_pass \
  custom-portfolio-db:1.0
```

### Container Configuration

| Setting | Value |
|---|---|
| Container name | `portfolio-db` |
| Image | `custom-portfolio-db:1.0` |
| Network | `portfolio-network-new` |
| Database | `portfolio` |
| User | `portfolio_user` |
| Password | `portfolio_pass` |
| PostgreSQL port | `5432` |

---

# 10. Check the Database Container

Check running containers:

```bash
sudo docker ps
```

The database container should be running:

```text
portfolio-db
```

### Database Container

![Database Container](<screenshots/Screenshot 2026-09-30 155130.png>)

---

# 11. Check PostgreSQL Logs

Check the database logs:

```bash
sudo docker logs portfolio-db
```

The initialization process should show:

```text
CREATE DATABASE
CREATE TABLE
INSERT 0 3
```

and eventually:

```text
database system is ready to accept connections
```

### PostgreSQL Initialization

![PostgreSQL Initialization](<screenshots/Screenshot 2026-09-30 155208.png>)

This confirms that:

- PostgreSQL started successfully.
- The database was created.
- `init.sql` was executed.
- The `projects` table was created.
- Three project records were inserted.

---

# 12. Verify the Database Table

Enter PostgreSQL inside the database container:

```bash
sudo docker exec -it portfolio-db psql -U portfolio_user -d portfolio
```

List the tables:

```sql
\dt
```

The `projects` table should be displayed.

Then query the table:

```sql
SELECT * FROM projects;
```

The three inserted projects should appear.

Exit PostgreSQL:

```sql
\q
```

### Database Table and Data

![Database Table](<screenshots/Screenshot 2026-09-30 155322.png>)

The query confirms that PostgreSQL contains:

```text
Docker Portfolio
Linux Automation
Travel App
```

---

# 13. Database Data Verification

The database records can also be verified directly through PostgreSQL.

```sql
SELECT * FROM projects;
```

Expected records:

| ID | Name | Technology |
|---:|---|---|
| 1 | Docker Portfolio | Docker, PHP, PostgreSQL |
| 2 | Linux Automation | Linux, Bash, Ansible |
| 3 | Travel App | PHP, PostgreSQL, Docker |

### PostgreSQL Query Output

![Project Data](<screenshots/Screenshot 2026-09-30 155335.png>)

---

# 14. Update the Web Application

The PHP application uses PDO to connect to PostgreSQL.

The important connection values are obtained from environment variables:

```php
$dbHost = getenv('DB_HOST') ?: 'portfolio-db';
$dbPort = getenv('DB_PORT') ?: '5432';
$dbName = getenv('DB_NAME') ?: 'portfolio';
$dbUser = getenv('DB_USER') ?: 'portfolio_user';
$dbPass = getenv('DB_PASSWORD') ?: 'portfolio_pass';
```

The database connection uses:

```php
$dsn = "pgsql:host=$dbHost;port=$dbPort;dbname=$dbName";

$pdo = new PDO($dsn, $dbUser, $dbPass, [
    PDO::ATTR_ERRMODE => PDO::ERRMODE_EXCEPTION
]);
```

The projects are retrieved using:

```sql
SELECT name, description, technology
FROM projects
ORDER BY id
```

---

# 15. Rebuild the Web Image

After updating `index.php`, create a new version of the Web image:

```bash
sudo docker build -t custom-portfolio-web:1.1 ./web
```

Verify the image:

```bash
sudo docker images
```

### Updated Web Image

![Updated Web Image](<screenshots/Screenshot 2026-09-30 160145.png>)

The updated image is:

```text
custom-portfolio-web:1.1
```

---

# 16. Run the Web Container

Run the Web container and connect it to the same Docker network as PostgreSQL:

```bash
sudo docker run -d \
  --name portfolio-web \
  --network portfolio-network-new \
  -p 8080:80 \
  -e DB_HOST=portfolio-db \
  -e DB_PORT=5432 \
  -e DB_NAME=portfolio \
  -e DB_USER=portfolio_user \
  -e DB_PASSWORD=portfolio_pass \
  custom-portfolio-web:1.1
```

### Important Configuration

```text
Web Container
     |
     | DB_HOST=portfolio-db
     |
     v
PostgreSQL Container
```

The Web container does **not** use `localhost` for the database.

Instead, it uses:

```text
portfolio-db
```

because Docker's internal DNS allows containers on the same user-defined network to communicate using container names.

---

# 17. Verify Both Containers

Run:

```bash
sudo docker ps
```

Both containers should be running:

```text
portfolio-web
portfolio-db
```

The Web container exposes:

```text
0.0.0.0:8080 -> 80
```

The PostgreSQL container uses:

```text
5432/tcp
```

### Both Containers Running

![Both Containers](<screenshots/Screenshot 2026-09-30 160145.png>)

---

# 18. Access the Portfolio

Open a browser and visit:

```text
http://YOUR_VM_IP:8080
```

Example:

```text
http://192.168.1.16:8080
```

The portfolio should load successfully.

The page displays:

- Custom Docker Portfolio
- Developer information
- PostgreSQL connection status
- Project information retrieved from PostgreSQL

### Running Portfolio

![Running Portfolio](<screenshots/Screenshot 2026-09-30 160242.png>)

The message:

```text
Connected to PostgreSQL
```

confirms that the PHP application successfully connected to the database.

The three project cards demonstrate that the application is retrieving records from PostgreSQL.

---

# 19. How the Application Works

The complete flow is:

```text
Browser
   |
   | HTTP :8080
   v
portfolio-web
   |
   | PHP / PDO
   |
   | DB_HOST=portfolio-db
   v
portfolio-db
   |
   | PostgreSQL :5432
   v
portfolio database
   |
   v
projects table
```

The Web container communicates with the database through:

```text
portfolio-network-new
```

---

# 20. Verify the Docker Network

Check the network:

```bash
sudo docker network inspect portfolio-network-new
```

The network should contain:

```text
portfolio-web
portfolio-db
```

This confirms that both containers are attached to the same Docker network.

---

# 21. Verify Container Name Resolution

The Web container can resolve the database container by its Docker name.

Run:

```bash
sudo docker exec portfolio-web getent hosts portfolio-db
```

The command should return the IP address associated with:

```text
portfolio-db
```

This demonstrates Docker's internal DNS-based container name resolution.

---

# 22. Useful Docker Commands

### List Docker Images

```bash
sudo docker images
```

### List Running Containers

```bash
sudo docker ps
```

### List All Containers

```bash
sudo docker ps -a
```

### List Networks

```bash
sudo docker network ls
```

### Inspect the Network

```bash
sudo docker network inspect portfolio-network-new
```

### View Web Container Logs

```bash
sudo docker logs portfolio-web
```

### View Database Logs

```bash
sudo docker logs portfolio-db
```

### Open a Shell in the Web Container

```bash
sudo docker exec -it portfolio-web bash
```

### Access PostgreSQL

```bash
sudo docker exec -it portfolio-db psql -U portfolio_user -d portfolio
```

---

# 23. Stop the Containers

To stop the application:

```bash
sudo docker stop portfolio-web portfolio-db
```

---

# 24. Start the Containers Again

```bash
sudo docker start portfolio-db portfolio-web
```

Check:

```bash
sudo docker ps
```

---

# 25. Remove the Containers

If the containers are no longer required:

```bash
sudo docker rm -f portfolio-web portfolio-db
```

---

# 26. Remove the Custom Network

After removing the containers:

```bash
sudo docker network rm portfolio-network-new
```

---

# 27. Remove the Custom Images

To remove the images:

```bash
sudo docker rmi custom-portfolio-web:1.1
sudo docker rmi custom-portfolio-web:1.0
sudo docker rmi custom-portfolio-db:1.0
```

---

# 28. Final Project Structure

```text
63-run-portfolio-using-custom-portfolio-image/
├── db/
│   ├── Dockerfile
│   └── init.sql
├── screenshots/
└── web/
    ├── Dockerfile
    └── index.php
```

---

# 29. Technologies Used

- Docker
- Docker Custom Images
- Docker Containers
- Docker Bridge Networks
- PHP
- Apache
- PostgreSQL
- PDO
- HTML
- CSS
- Linux

---

# 30. What This Project Demonstrates

This project demonstrates how to:

- Create custom Docker images.
- Build a PHP/Apache Web image.
- Build a custom PostgreSQL image.
- Initialize PostgreSQL automatically using `init.sql`.
- Run multiple containers manually.
- Create a custom Docker network.
- Connect containers through a user-defined network.
- Use Docker container names for service discovery.
- Pass configuration through environment variables.
- Connect PHP to PostgreSQL using PDO.
- Retrieve database records dynamically.
- Expose a containerized web application through a host port.
- Verify containers, images, networks and database data using Docker commands.

---

## Final Result

The final application consists of two independently running containers:

```text
custom-portfolio-web:1.1
              |
              | portfolio-network-new
              |
custom-portfolio-db:1.0
```

The browser accesses the Web container through:

```text
http://192.168.1.16:8080
```

while the PHP application communicates internally with PostgreSQL using:

```text
DB_HOST=portfolio-db
DB_PORT=5432
```

The portfolio successfully retrieves and displays project information stored in PostgreSQL.