# 55 - Run Verventech Website Container with Port Forwarding

This lab demonstrates how to run the Verventech website from its Docker image, expose the container through a host port, access the website from a browser, check the container logs, and verify the port mapping.

## Docker Image

```text
verventech/verventech-website:latest
```

## 1. Pull the Verventech Website Image

```bash
sudo docker pull verventech/verventech-website
```

The image was downloaded successfully.

![Pull Verventech Website Image](<screenshots/Screenshot (193)(1).png>)

## 2. Verify the Docker Image

```bash
sudo docker images
```

The output confirms that `verventech/verventech-website:latest` is available locally.

![Verify Verventech Website Image](<screenshots/Screenshot (194)(1).png>)

## 3. Run the Website Container with Port Forwarding

Run the website in detached mode and map host port `8082` to container port `80`.

```bash
sudo docker run -d --name verventech-website -p 8082:80 verventech/verventech-website
```

The mapping is:

```text
Host Port 8082 → Container Port 80
```

### Command Breakdown

| Option | Purpose |
|---|---|
| `-d` | Runs the container in detached mode |
| `--name verventech-website` | Names the container |
| `-p 8082:80` | Maps host port `8082` to container port `80` |
| `verventech/verventech-website` | Specifies the website image |

![Run Verventech Website Container](<screenshots/Screenshot (195).png>)

## 4. Verify the Running Container

```bash
sudo docker ps
```

The output confirms that `verventech-website` is running with:

```text
0.0.0.0:8082->80/tcp
```

![Verventech Website Container Running](<screenshots/Screenshot (196).png>)

## 5. Access the Website

Open the following address in a browser:

```text
http://192.168.1.16:8082
```

The Verventech Training website loads successfully from the Docker container.

![Verventech Website Running in Browser](<screenshots/Screenshot (197).png>)

## 6. Check the Container Logs

```bash
sudo docker logs verventech-website
```

The logs show Apache starting and handling HTTP requests from the browser.

The Apache `ServerName` message shown in the logs is a configuration warning and does not prevent the website from running.

![Verventech Website Container Logs](<screenshots/Screenshot (198).png>)

## 7. Verify the Port Mapping

```bash
sudo docker port verventech-website
```

The output confirms:

```text
80/tcp -> 0.0.0.0:8082
80/tcp -> [::]:8082
```

Therefore:

```text
Container Port 80 → Host Port 8082
```

![Verventech Website Port Mapping](<screenshots/Screenshot (199).png>)

## Port Forwarding Flow

```text
Browser
   │
   │ http://192.168.1.16:8082
   ▼
Host Port 8082
   │
   │ Docker Port Mapping
   ▼
Container Port 80
   │
   ▼
Apache HTTPD
   │
   ▼
Verventech Website
```

## Commands Used

```bash
sudo docker pull verventech/verventech-website
sudo docker images
sudo docker run -d --name verventech-website -p 8082:80 verventech/verventech-website
sudo docker ps
sudo docker logs verventech-website
sudo docker port verventech-website
```

Website:

```text
http://192.168.1.16:8082
```

## Summary

In this lab:

1. The `verventech/verventech-website` image was pulled.
2. The image was verified locally.
3. A container named `verventech-website` was started.
4. Host port `8082` was mapped to container port `80`.
5. The running container was verified using `docker ps`.
6. The Verventech Training website was accessed through the browser.
7. Container logs were checked using `docker logs`.
8. The port mapping was verified using `docker port`.

## Conclusion

The Verventech website was successfully deployed from a Docker image and made accessible through host port `8082`.

This demonstrates Docker port forwarding for a web application running inside a container.
