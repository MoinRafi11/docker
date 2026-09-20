# 54 - Docker HTTPD Container with Port Forwarding

This exercise demonstrates Docker port mapping by running an Apache HTTPD container and serving a custom animated HTML page through a forwarded host port.

The port mapping used in this setup is:

```text
Host Port 8081 → Container Port 80
```

---

## 1. Pull the Apache HTTPD Image

First, pull the official Apache HTTPD image.

### Command

```bash
sudo docker pull httpd
```

The `httpd` image provides the Apache HTTP Server used in this exercise.

### Screenshot

![Docker Pull HTTPD](screenshots/Screenshot(181).png)

---

## 2. Run the Apache Container with Port Mapping

Create and run the Apache HTTPD container in detached mode.

### Command

```bash
sudo docker run -d --name apache-port-container -p 8081:80 httpd
```

### Command Breakdown

| Option | Purpose |
|---|---|
| `docker run` | Creates and starts a new container |
| `-d` | Runs the container in detached mode |
| `--name apache-port-container` | Assigns a name to the container |
| `-p 8081:80` | Maps host port `8081` to container port `80` |
| `httpd` | Uses the Apache HTTPD image |

The important part is:

```text
8081:80
```

which means:

```text
Host Port 8081 → Container Port 80
```

### Screenshot

![Run Apache with Port Mapping](screenshots/Screenshot(182).png)

---

## 3. Verify the Port Mapping

Check the running container.

### Command

```bash
sudo docker ps
```

The `PORTS` column confirms that port `8081` on the host is forwarded to port `80` inside the container.

Expected mapping:

```text
0.0.0.0:8081->80/tcp
```

### Screenshot

![Docker PS Port Mapping](screenshots/Screenshot(183).png)

---

## 4. Copy the Custom HTML Page into the Container

Instead of using Apache's default `It works!` page, a custom animated HTML page was created.

Copy the HTML file into Apache's document root:

```bash
sudo docker cp sample.html apache-port-container:/usr/local/apache2/htdocs/index.html
```

The official Apache HTTPD image serves web files from:

```text
/usr/local/apache2/htdocs/
```

### Screenshot

![Copy Custom HTML Page](screenshots/Screenshot(184).png)

---

## 5. Verify the HTML File Inside the Container

Check that the custom page exists inside the Apache document root.

### Command

```bash
sudo docker exec apache-port-container ls -l /usr/local/apache2/htdocs/
```

The output confirms that `index.html` is present.

### Screenshot

![HTML File Inside Container](screenshots/Screenshot(185).png)

---

## 6. Check the Port Mapping Directly

Docker can display the published port of a specific container.

### Command

```bash
sudo docker port apache-port-container
```

The output confirms the mapping:

```text
80/tcp -> 0.0.0.0:8081
```

This means:

```text
Container Port 80 → Host Port 8081
```

### Screenshot

![Docker Port Mapping](screenshots/Screenshot(186).png)

---

## 7. Test the Custom Page Using curl

Test the Apache server through the mapped host port.

### Command

```bash
curl http://localhost:8081
```

The response contains the custom HTML page served by Apache.

This verifies that the request is successfully travelling through the Docker port mapping.

### Screenshot

![curl Custom Page](screenshots/Screenshot(187).png)

---

## 8. Access the Custom Page from a Web Browser

The Apache server can also be accessed through the host machine's IP address.

In this setup, the host IP address is:

```text
192.168.1.16
```

Open:

```text
http://192.168.1.16:8081
```

The custom animated Docker port-mapping page is displayed.

The page demonstrates:

```text
8081 → 80
```

and indicates that the Apache HTTP Server is running inside the Docker container.

### Screenshot

![Custom Docker Port Mapping Page](screenshots/Screenshot(191).png)

---

## Port Mapping Flow

The complete request flow is:

```text
Browser
   │
   │ http://192.168.1.16:8081
   ▼
Host Port 8081
   │
   │ Docker Port Mapping
   ▼
Container Port 80
   │
   ▼
Apache HTTPD
   │
   ▼
/usr/local/apache2/htdocs/index.html
```

The Docker configuration can be represented as:

```text
192.168.1.16:8081
        │
        ▼
   Docker 8081:80
        │
        ▼
 Apache HTTPD :80
        │
        ▼
    index.html
```

---

## Key Concept: Docker Port Mapping

Docker port mapping follows this syntax:

```text
-p HOST_PORT:CONTAINER_PORT
```

In this exercise:

```bash
-p 8081:80
```

means:

```text
8081 → Host Port
80   → Container Port
```

The Apache server continues to listen on port `80` inside the container, while Docker makes it accessible through port `8081` on the host.

Therefore, a request to:

```text
http://192.168.1.16:8081
```

is forwarded to:

```text
Apache Container:80
```

---

## Commands Used

### Pull the HTTPD Image

```bash
sudo docker pull httpd
```

### Run the Container

```bash
sudo docker run -d --name apache-port-container -p 8081:80 httpd
```

### Check Running Containers

```bash
sudo docker ps
```

### Copy the Custom HTML Page

```bash
sudo docker cp sample.html apache-port-container:/usr/local/apache2/htdocs/index.html
```

### Verify the HTML File

```bash
sudo docker exec apache-port-container ls -l /usr/local/apache2/htdocs/
```

### Check Port Mapping

```bash
sudo docker port apache-port-container
```

### Test the Page

```bash
curl http://localhost:8081
```

---

## Summary

In this exercise:

1. The official `httpd` image was pulled.
2. An Apache HTTPD container was created.
3. Host port `8081` was mapped to container port `80`.
4. A custom HTML page was copied into the Apache document root.
5. The port mapping was verified using `docker ps` and `docker port`.
6. The custom page was tested using `curl`.
7. The page was accessed through the host IP address in a web browser.

The final port mapping was:

```text
Host:      192.168.1.16:8081
                │
                ▼
Docker:       8081:80
                │
                ▼
Container:       :80
                │
                ▼
          Apache HTTPD
                │
                ▼
            index.html
```

---

## Conclusion

Docker port mapping allows services running inside containers to be accessed through ports on the host machine.

In this example, Apache HTTPD listens on port `80` inside the container, while Docker forwards host port `8081` to it.

The custom animated HTML page was successfully served through:

```text
http://192.168.1.16:8081
```

demonstrating the complete flow from the host port to the Apache web server running inside the Docker container.
