# 56 - Docker Networking Types with Commands

Docker networking allows containers to communicate with each other, the host system, and external networks.

In this lab, the main Docker networking types were explored:

- **Bridge**
- **Host**
- **None**
- **Custom Bridge Network**

---

## 1. List Docker Networks

First, check the networks available by default:

```bash
sudo docker network ls
```

By default, Docker provides networks such as:

- `bridge`
- `host`
- `none`

![Docker network ls](<screenshots/Screenshot (200).png>)

---

## 2. Bridge Network

The **bridge** network is Docker's default network for containers.

Inspect the default bridge network:

```bash
sudo docker network inspect bridge
```

The output shows details such as:

- Network driver: `bridge`
- Subnet: `172.17.0.0/16`
- Gateway: `172.17.0.1`
- Connected containers

![Inspect default bridge network](<screenshots/Screenshot (201).png>)

### Run a Container Using the Bridge Network

Containers use the default bridge network when no other network is specified.

Example:

```bash
sudo docker run --rm alpine ip addr
```

---

## 3. Host Network

With the **host** network, the container shares the host's network namespace instead of getting its own isolated network interface.

Run an Alpine container using the host network:

```bash
sudo docker run --rm --network host alpine ip addr
```

The container can see the host's network interfaces directly.

![Docker host network](<screenshots/Screenshot (202).png>)

---

## 4. None Network

The **none** network provides a completely isolated networking environment.

Run an Alpine container using the none network:

```bash
sudo docker run --rm --network none alpine ip addr
```

Only the loopback interface (`lo`) is available.

![Docker none network](<screenshots/Screenshot (203).png>)

---

## 5. Create a Custom Bridge Network

Docker also allows us to create our own bridge networks.

Create a custom network:

```bash
sudo docker network create my-custom-network
```

Docker returns the ID of the newly created network.

![Create custom Docker network](<screenshots/Screenshot (204).png>)

---

## 6. Inspect the Custom Network

Verify the configuration of the newly created network:

```bash
sudo docker network inspect my-custom-network
```

The custom network uses the bridge driver and has its own subnet and gateway.

Example configuration shown in the output:

```text
Driver: bridge
Subnet: 172.18.0.0/16
Gateway: 172.18.0.1
```

![Inspect custom Docker network](<screenshots/Screenshot (205).png>)

---

## Quick Comparison

| Network Type | Description |
|---|---|
| `bridge` | Default isolated Docker network |
| `host` | Container shares the host network |
| `none` | No external network connectivity |
| Custom bridge | User-created isolated Docker network |

---

## Useful Commands

```bash
# List networks
sudo docker network ls

# Inspect a network
sudo docker network inspect bridge

# Run using host networking
sudo docker run --rm --network host alpine ip addr

# Run using no networking
sudo docker run --rm --network none alpine ip addr

# Create a custom network
sudo docker network create my-custom-network

# Inspect custom network
sudo docker network inspect my-custom-network
```

---

## Key Takeaways

- Docker provides `bridge`, `host`, and `none` networks by default.
- The **bridge** network provides container network isolation with Docker-managed networking.
- The **host** network removes the normal network isolation between the container and host.
- The **none** network provides maximum network isolation.
- Custom bridge networks can be created for more controlled container networking.
