# Demo: Bind Mount on Apache Container

This project demonstrates how a **bind mount** can be used to serve website files from the host system using an Apache container.

The host directory is mounted directly into the Apache container.

---

## 1. Create the Host Directory

Create a directory on the host:

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

![Create Host HTML File](screenshots/Screenshot%20(329).png)

---

## 2. Run Apache Container with Bind Mount

Run an Apache container and mount the host directory to Apache's document root:

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

![Apache Container with Bind Mount](screenshots/Screenshot%20(327).png)

---

## 3. Access the Website

Open the following address in a browser:

```text
http://192.168.1.16:8080
```

The webpage is being served by Apache from the host directory through the bind mount.

![Apache Website](screenshots/Screenshot%20(326).png)

---

## 4. Modify the Host File

Change the content of the `index.html` file on the host:

```bash
nano index.html
```

For example:

```html
<h1>HIII!!! Updated from the host</h1>
```

Save the file and refresh the browser.

The updated content is immediately available because the host directory is bind-mounted into the Apache container.

![Updated Apache Website](screenshots/Screenshot%20(330).png)

---

## 5. Verify the Bind Mount

The bind mount can be verified using:

```bash
sudo docker inspect apache-bind-demo
```

The `Mounts` section shows:

```text
"Type": "bind"
"Source": "/home/moin/apache-bind-data"
"Destination": "/usr/local/apache2/htdocs"
```

![Verify Bind Mount](screenshots/Screenshot%20(331).png)

---

## How It Works

```text
Host
/home/moin/apache-bind-data
          │
          │ Bind Mount
          ▼
Container
/usr/local/apache2/htdocs
          │
          ▼
       Apache
          │
          ▼
      Web Browser
```

The files remain on the host while Apache serves them from inside the container.

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

- A host directory was created for the website files.
- The directory was bind-mounted into an Apache container.
- Apache served the HTML file from the mounted directory.
- Changes made to the host file were reflected in the running container.
- `docker inspect` was used to verify the bind mount.