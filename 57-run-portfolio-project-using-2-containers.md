# Docker Portfolio Project Using 2 Containers

This project runs a simple dynamic portfolio using **two Docker containers**:

- **Apache + PHP** — serves the portfolio website
- **PostgreSQL** — stores the portfolio project data

Both containers communicate through a custom Docker network.

> **Note:** This project does not use a Dockerfile or Docker Compose.

---

## 1. Project Structure

```text
portfolio-two-containers/
├── src/
│   └── index.php
├── db/
│   └── init.sql
└── README.md
```

---

## 2. Create the Docker Network

Create a custom network for communication between the two containers.

```bash
sudo docker network create portfolio-two-container-network
```

Check the network:

```bash
sudo docker network ls
```

![Docker network](<screenshots/Screenshot 2026-09-29 115341.png>)

---

## 3. Create the PostgreSQL Container

Run the PostgreSQL container:

```bash
sudo docker run -d \
  --name portfolio-two-container-db \
  --network portfolio-two-container-network \
  -e POSTGRES_DB=portfolio \
  -e POSTGRES_USER=portfolio_user \
  -e POSTGRES_PASSWORD='Portfolio@123' \
  postgres:15
```

### Network Error

During the setup, the container initially failed because the custom Docker network did not exist.

![Docker network error](<screenshots/Screenshot (315).png>)

The error was:

```text
network portfolio-two-container-network not found
```

After recreating the network, the PostgreSQL container was started again.

---

## 4. Copy the Database SQL File

From the `db` directory:

```bash
sudo docker cp init.sql portfolio-two-container-db:/tmp/init.sql
```

![Copy init.sql](<screenshots/Screenshot 2026-09-29 115341.png>)

---

## 5. Initialize the PostgreSQL Database

Execute the SQL file inside the PostgreSQL container:

```bash
sudo docker exec -it portfolio-two-container-db \
psql -U portfolio_user -d portfolio -f /tmp/init.sql
```

The output confirms that the table was created and two records were inserted.

```text
CREATE TABLE
INSERT 0 2
```

![Database initialization](<screenshots/Screenshot 2026-09-29 115921.png>)

---

## 6. Create the Apache + PHP Container

Pull the PHP-Apache image automatically while creating the container:

```bash
sudo docker run -d \
  --name portfolio-two-container-web \
  --network portfolio-two-container-network \
  -p 8090:80 \
  -e PGHOST=portfolio-two-container-db \
  -e PGDATABASE=portfolio \
  -e PGUSER=portfolio_user \
  -e PGPASSWORD='Portfolio@123' \
  -e PGPORT=5432 \
  php:8.2-apache
```

![Create PHP Apache container](<screenshots/Screenshot 2026-09-29 120432.png>)

---

## 7. Install PostgreSQL PHP Dependencies

The standard `php:8.2-apache` image does not have the PostgreSQL development libraries required to build the PostgreSQL PHP extensions.

Update the package list:

```bash
sudo docker exec -it portfolio-two-container-web apt-get update
```

Install `libpq-dev`:

```bash
sudo docker exec -it portfolio-two-container-web apt-get install -y libpq-dev
```

![Install libpq-dev](<screenshots/Screenshot (316).png>)

Install the PostgreSQL PHP extensions:

```bash
sudo docker exec -it portfolio-two-container-web \
docker-php-ext-install pgsql pdo_pgsql
```

![Install PHP PostgreSQL extensions](<screenshots/Screenshot (317).png>)

These extensions allow PHP to connect to PostgreSQL using PDO.

---

## 8. Copy the Portfolio PHP File

Go to the `src` directory:

```bash
cd ~/portfolio-two-containers/src
```

Copy `index.php` into Apache's document root:

```bash
sudo docker cp index.php portfolio-two-container-web:/var/www/html/
```

![Copy index.php](<screenshots/Screenshot 2026-09-29 121643.png>)

The file is placed at:

```text
/var/www/html/index.php
```

---

## 9. Verify the PHP File

The portfolio uses environment variables to connect to the PostgreSQL container.

The important database settings are:

```php
DB_HOST
DB_NAME
DB_USER
DB_PASSWORD
DB_PORT
```

The database hostname is the PostgreSQL container name:

```text
portfolio-two-container-db
```

![Portfolio PHP file](<screenshots/Screenshot (318).png>)

---

## 10. Restart the Apache Container

After installing the PHP extensions and copying the PHP file:

```bash
sudo docker restart portfolio-two-container-web
```

![Restart Apache container](<screenshots/Screenshot (319).png>)

---

## 11. Access the Portfolio

The Apache container maps:

```text
Host Port 8090 → Container Port 80
```

Open the following address in a browser:

```text
http://192.168.1.16:8090
```

![Portfolio website](<screenshots/Screenshot (319).png>)

The portfolio displays the project information stored in PostgreSQL.

---

## 12. Container Architecture

```text
                    Host Machine
                         |
                    Port 8090
                         |
                         v
        +--------------------------------+
        |  portfolio-two-container-web  |
        |       PHP + Apache             |
        +--------------------------------+
                         |
                         | Docker Network
                         | portfolio-two-container-network
                         v
        +--------------------------------+
        |  portfolio-two-container-db   |
        |       PostgreSQL 15            |
        +--------------------------------+
                         |
                    portfolio DB
                         |
                      projects
```

---

## 13. Useful Commands

### Check running containers

```bash
sudo docker ps
```

### Check all containers

```bash
sudo docker ps -a
```

### Check the network

```bash
sudo docker network inspect portfolio-two-container-network
```

### View Apache container logs

```bash
sudo docker logs portfolio-two-container-web
```

### View PostgreSQL container logs

```bash
sudo docker logs portfolio-two-container-db
```

### Stop the containers

```bash
sudo docker stop portfolio-two-container-web
sudo docker stop portfolio-two-container-db
```

### Start the containers again

```bash
sudo docker start portfolio-two-container-db
sudo docker start portfolio-two-container-web
```

---

## Result

The project demonstrates a basic **two-container Docker architecture** without using Dockerfile or Docker Compose:

- **Container 1:** PHP + Apache web application
- **Container 2:** PostgreSQL database
- **Custom Docker network:** allows container-to-container communication
- **PHP PDO:** connects the web application to PostgreSQL
- **Port forwarding:** exposes the Apache application on host port `8090`
