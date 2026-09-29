# Demo: Bind Mount on Apache Container

This project demonstrates how a **bind mount** can be used to serve website files from a host directory using an Apache container.

---

## 1. Create the Host Directory and HTML File

Create a directory for the website files:

```bash
mkdir apache-bind-data
```

Move into the directory:

```bash
cd apache-bind-data
```

Create the HTML file:

```bash
nano index.html
```

Example content:

```html
<!DOCTYPE html>
<html>
<head>
    <title>Docker Bind Mount</title>
</head>
<body>
    <h1>Hello from Bind Mount!</h1>
    <p>This page is served from a host directory.</p>
</body>
</html>
```

Verify the file:

```bash
cat index.html
```

![Create Host HTML File](screenshots/Screenshot%20(326).png)

---

## 2. Run the Apache Container

Run an Apache container and bind mount the host directory to Apache's document root:

```bash
sudo docker run -d \
  --name apache-bind-demo \
  -p 8080:80 \
  -v /home/moin/apache-bind-data:/usr/local/apache2/htdocs \
  httpd:2.4
```

Check the running container:

```bash
sudo docker ps
```

The bind mount connects:

```text
Host:
/home/moin/apache-bind-data

Container:
/usr/local/apache2/htdocs
```

![Apache Container with Bind Mount](screenshots/Screenshot%20(327).png)

---

## 3. Access the Website

Open the Apache website in a browser:

```text
http://192.168.1.16:8080
```

The HTML file from the host directory is served by Apache inside the container.

![Apache Website](screenshots/Screenshot%20(330).png)

---

## 4. Modify the File on the Host

Because the host directory is bind-mounted, changes made to the host file are reflected inside the container.

Modify the file:

```bash
nano index.html
```

For example, change the heading to:

```html
<h1>HIII!!! Updated from the host</h1>
```

Verify the updated file:

```bash
cat index.html
```

![Updated Host HTML File](screenshots/Screenshot%20(329).png)

---

## 5. Verify the Change in the Browser

Refresh the browser:

```text
http://192.168.1.16:8080
```

The updated content is displayed without recreating the container.

![Updated Apache Website](screenshots/Screenshot%20(331).png)

---

## 6. Verify the Bind Mount

The bind mount can also be verified using:

```bash
sudo docker inspect apache-bind-demo
```

Look for the `Mounts` section. It will show the host directory and the container destination.

Expected values:

```text
Type: bind
Source: /home/moin/apache-bind-data
Destination: /usr/local/apache2/htdocs
```

---

## How It Works

```text
Host Directory
/home/moin/apache-bind-data
          │
          │ Bind Mount
          ▼
Apache Container
/usr/local/apache2/htdocs
          │
          ▼
     Web Browser
```

Changes made to the files in the host directory are available to Apache through the bind mount.

---

## Useful Commands

Check the container:

```bash
sudo docker ps
```

Inspect the container:

```bash
sudo docker inspect apache-bind-demo
```

Stop the container:

```bash
sudo docker stop apache-bind-demo
```

Remove the container:

```bash
sudo docker rm apache-bind-demo
```

---

## Summary

- Created a host directory containing an HTML file.
- Mounted the directory into an Apache container using a bind mount.
- Accessed the website through Apache.
- Modified the HTML file directly on the host.
- Verified that the changes were immediately reflected in the browser.
- Used `docker inspect` to verify the bind mount.