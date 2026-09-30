# Dockerfile Usage

A simple hands-on demonstration of how a `Dockerfile` is used to build a Docker image and run an application inside a container.

---

## 1. Project Structure

Create a project directory:

```bash
mkdir dockerfile-demo
cd dockerfile-demo
```

The project contains:

```text
dockerfile-demo/
├── Dockerfile
└── index.html
```

Check the files:

```bash
ls -la
```

![Dockerfile Demo Project Files](screenshots/Screenshot%202026-09-30%20115624.png)

---

## 2. Create the Application

Create `index.html` containing the demo webpage.

The page demonstrates the basic Dockerfile instructions used in this project:

- `FROM`
- `COPY`
- `CMD`

---

## 3. Create the Dockerfile

Create a file named `Dockerfile`:

```dockerfile
FROM nginx:alpine

WORKDIR /usr/share/nginx/html

COPY index.html .

EXPOSE 80

CMD ["nginx", "-g", "daemon off;"]
```

### Dockerfile Instructions

| Instruction | Purpose |
|---|---|
| `FROM` | Defines the base image |
| `WORKDIR` | Sets the working directory inside the image |
| `COPY` | Copies application files into the image |
| `EXPOSE` | Documents the port used by the application |
| `CMD` | Defines the default command executed when the container starts |

---

## 4. Build the Docker Image

Build the image using:

```bash
sudo docker build -t dockerfile-demo .
```

Here:

- `docker build` builds an image from the Dockerfile.
- `-t dockerfile-demo` gives the image its name.
- `.` specifies the current directory as the build context.
- Docker automatically looks for a file named `Dockerfile`.

![Docker Build and Image](screenshots/Screenshot%20(338).png)

### Verify the Image

```bash
sudo docker image ls
```

The image should appear as:

```text
dockerfile-demo:latest
```

---

## 5. Run a Container

Run the image as a container:

```bash
sudo docker run -d \
  --name dockerfile-demo-container \
  -p 8080:80 \
  dockerfile-demo
```

The port mapping:

```text
8080:80
```

means:

```text
Host Port 8080 → Container Port 80
```

---

## 6. Verify the Running Container

Check the running containers:

```bash
sudo docker ps
```

The container should show a port mapping similar to:

```text
0.0.0.0:8080->80/tcp
```

![Running Dockerfile Container](screenshots/Screenshot%202026-09-30%20120749.png)

---

## 7. Open the Application

Open the VM IP address in a browser:

```text
http://YOUR-VM-IP:8080
```

Example:

```text
http://192.168.1.16:8080
```

The webpage should be served by the Nginx container.

![Dockerfile Demo Website](screenshots/Screenshot%202026-09-30%20120844.png)

---

## 8. How It Works

The process can be summarized as:

```text
index.html
     │
     ▼
Dockerfile
     │
     │ docker build
     ▼
Docker Image
     │
     │ docker run
     ▼
Docker Container
     │
     │ Port 8080 → 80
     ▼
Web Browser
```

The `Dockerfile` defines how the image is created, while the container runs the application from that image.

---

## 9. Important Commands

### Build the image

```bash
sudo docker build -t dockerfile-demo .
```

### List images

```bash
sudo docker image ls
```

### Run the container

```bash
sudo docker run -d \
  --name dockerfile-demo-container \
  -p 8080:80 \
  dockerfile-demo
```

### List running containers

```bash
sudo docker ps
```

### Stop the container

```bash
sudo docker stop dockerfile-demo-container
```

### Remove the container

```bash
sudo docker rm dockerfile-demo-container
```

### Remove the image

```bash
sudo docker rmi dockerfile-demo
```

---

## 10. Key Learning

This demonstration shows the basic Dockerfile workflow:

```text
Dockerfile
    ↓
docker build
    ↓
Docker Image
    ↓
docker run
    ↓
Docker Container
    ↓
Application
```

A `Dockerfile` provides a repeatable way to package an application and its runtime environment into a Docker image.