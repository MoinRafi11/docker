# 53 — Docker Container Lifecycle

This exercise demonstrates the basic lifecycle of a Docker container using an Ubuntu container.

The main lifecycle operations covered are:

```text
Create → Run → Stop → Start → Restart → Pause → Unpause → Remove
```

---

## 1. Create and Run an Ubuntu Container

Create a new Ubuntu container and start it interactively.

### Command

```bash
sudo docker run -it --name lifecycle-container ubuntu
```

The `-it` option starts the container interactively and attaches a terminal.

The container is named:

```text
lifecycle-container
```

### Screenshot

![Run Ubuntu Container](<screenshots/Screenshot (172).png>)

---

## 2. Check the Ubuntu Container

Once inside the container, verify the operating system information.

### Command

```bash
cat /etc/os-release
```

The output confirms that the container is running Ubuntu.

The container shell is visible as:

```text
root@4e6551c90609:/#
```

### Screenshot

![Ubuntu Container OS Information](<screenshots/Screenshot (172).png>)

---

## 3. Check All Containers

After exiting the interactive container, check all containers on the host.

### Command

```bash
sudo docker ps -a
```

The `-a` option displays both running and stopped containers.

The `lifecycle-container` container appears in the list.

### Screenshot

![Docker PS All](<screenshots/Screenshot (173).png>)

---

## 4. Start the Stopped Container

A stopped container can be started again without creating a new container.

### Command

```bash
sudo docker start lifecycle-container
```

After starting it, verify the running containers:

```bash
sudo docker ps
```

The container appears with a running status.

### Screenshot

![Start Container](<screenshots/Screenshot (174).png>)

---

## 5. Stop the Container

Stop the running container using:

### Command

```bash
sudo docker stop lifecycle-container
```

Then check all containers:

```bash
sudo docker ps -a
```

The container now appears with an `Exited` status.

### Screenshot

![Stop Container](<screenshots/Screenshot (175).png>)

---

## 6. Start and Restart the Container

A stopped container can be started again using:

```bash
sudo docker start lifecycle-container
```

The container can then be restarted using:

```bash
sudo docker restart lifecycle-container
```

Finally, verify its current state:

```bash
sudo docker ps
```

The output shows `lifecycle-container` running.

### Screenshot

![Start and Restart Container](<screenshots/Screenshot (176).png>)

---

## 7. Pause the Running Container

A running container can temporarily be paused.

### Command

```bash
sudo docker pause lifecycle-container
```

Then check the container status:

```bash
sudo docker ps
```

The status changes to:

```text
Up ... (Paused)
```

Pausing a container suspends its processes without stopping or removing the container.

### Screenshot

![Paused Container](<screenshots/Screenshot (177).png>)

---

## 8. Unpause the Container

Resume the paused container using:

### Command

```bash
sudo docker unpause lifecycle-container
```

Then verify its status:

```bash
sudo docker ps
```

The container returns to a normal running state.

### Screenshot

![Unpaused Container](<screenshots/Screenshot (178).png>)

---

## 9. Stop and Remove the Container

Once the lifecycle operations are complete, stop the container:

### Command

```bash
sudo docker stop lifecycle-container
```

Then remove it:

```bash
sudo docker rm lifecycle-container
```

Finally, verify all containers:

```bash
sudo docker ps -a
```

The `lifecycle-container` is no longer present because it has been removed.

### Screenshot

![Stop and Remove Container](<screenshots/Screenshot (179).png>)

---

## Docker Container Lifecycle

The lifecycle demonstrated in this exercise can be represented as:

```text
                 docker run
                     │
                     ▼
                  Created
                     │
                     ▼
                  Running
                 /       \
                /         \
      docker pause       docker stop
             │                │
             ▼                ▼
          Paused           Stopped
             │                │
      docker unpause      docker start
             │                │
             └───────┬────────┘
                     ▼
                  Running
                     │
              docker restart
                     │
                     ▼
                  Running
                     │
                docker stop
                     │
                     ▼
                  Stopped
                     │
                 docker rm
                     │
                     ▼
                  Removed
```

---

## Important Docker Lifecycle Commands

| Command | Purpose |
|---|---|
| `docker run` | Creates and starts a new container |
| `docker start` | Starts an existing stopped container |
| `docker stop` | Stops a running container |
| `docker restart` | Stops and starts a container again |
| `docker pause` | Pauses processes inside a running container |
| `docker unpause` | Resumes a paused container |
| `docker ps` | Lists running containers |
| `docker ps -a` | Lists all containers |
| `docker rm` | Removes a stopped container |

---

## Key Concepts

### `docker run`

```bash
sudo docker run -it --name lifecycle-container ubuntu
```

Creates a new container from the Ubuntu image and starts it.

If a container with the specified name does not already exist, Docker creates it.

---

### `docker start`

```bash
sudo docker start lifecycle-container
```

Starts an existing stopped container.

It does not create a new container.

---

### `docker stop`

```bash
sudo docker stop lifecycle-container
```

Gracefully stops a running container.

The container itself remains available and can be started again.

---

### `docker restart`

```bash
sudo docker restart lifecycle-container
```

Restarts an existing container.

It effectively stops and then starts the container again.

---

### `docker pause`

```bash
sudo docker pause lifecycle-container
```

Pauses the processes running inside the container.

The container remains present, but its processes are suspended.

---

### `docker unpause`

```bash
sudo docker unpause lifecycle-container
```

Resumes a container that was previously paused.

---

### `docker rm`

```bash
sudo docker rm lifecycle-container
```

Removes the container.

A container normally needs to be stopped before it can be removed.

---

## Summary

The complete lifecycle demonstrated in this exercise was:

```text
Create and Run
      ↓
Running
      ↓
Stop
      ↓
Start
      ↓
Restart
      ↓
Pause
      ↓
Unpause
      ↓
Stop
      ↓
Remove
```

The same Docker container was reused throughout the exercise instead of creating a new container for every lifecycle operation.

---

## Conclusion

The Docker container lifecycle was demonstrated using an Ubuntu container named `lifecycle-container`.

The exercise covered creating, running, stopping, starting, restarting, pausing, unpausing, and removing a Docker container.

These lifecycle commands are fundamental for managing containers during development, testing, and deployment workflows.
