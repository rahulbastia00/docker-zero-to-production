## Frequently Asked Questions and Common Doubts

### 1. What does `COPY . .` mean?

```dockerfile
COPY . .
```

The first `.` represents the **current directory in the Docker build context (source)**, and the second `.` represents the **current working directory inside the container (destination)**.

For example:

```dockerfile
WORKDIR /app
COPY . .
```

is effectively:

```dockerfile
COPY . /app
```

---

### 2. What is the difference between `RUN` and `CMD`?

`RUN` executes commands while **building the Docker image**, while `CMD` specifies the command that runs when the **container starts**.

```dockerfile
RUN pip install -r requirements.txt
```

Installs dependencies during image creation.

```dockerfile
CMD ["python", "app.py"]
```

Starts the application when the container runs.

> **Simple rule:** `RUN` happens at build time, while `CMD` happens at container runtime.

---

### 3. Why can't we use `RUN` to start the application?

`RUN` executes during the image build process. Starting a long-running application using `RUN` would cause the build process to wait instead of creating the final image.

Therefore, `CMD` is used to define the default application that should start when the container launches.

---

### 4. Why does `CMD` use square brackets?

```dockerfile
CMD ["python", "app.py"]
```

This is called the **exec form**. The first value is the executable, and the remaining values are its arguments.

It is generally preferred because Docker runs the application directly and handles system signals more cleanly.

---

### 5. What does `EXPOSE` do?

```dockerfile
EXPOSE 80
```

`EXPOSE` documents that the application inside the container is expected to listen on port `80`.

However, it **does not publish the port to your computer**.

To make the container accessible from the host, use port mapping:

```bash
docker run -p 5000:80 vote-app
```

This maps:

```text
Host Port 5000 → Container Port 80
```

> **Simple rule:** The application listens, `EXPOSE` documents, and `-p` publishes/maps the port.


## Frequently Asked Questions and Common Doubts

### 1. What is the difference between a `Dockerfile` and `docker-compose.yml`?

| Aspect      | `Dockerfile`                                                       | `docker-compose.yml`                                                   |
| ----------- | ------------------------------------------------------------------ | ---------------------------------------------------------------------- |
| **Scope**   | Builds an image for a single service.                              | Runs and configures multiple services together.                        |
| **Purpose** | Defines the environment, dependencies, files, and startup command. | Defines services, networks, volumes, ports, and environment variables. |
| **Example** | Builds the `vote` application image.                               | Runs `vote`, `result`, `worker`, Redis, and PostgreSQL together.       |

A `docker-compose.yml` does **not combine multiple Dockerfiles into one file**. Instead, it coordinates multiple containers as a single application.

> **Simple rule:** A `Dockerfile` builds a service image, while Docker Compose orchestrates multiple services.

---

### 2. Will Python 3.11 inside Docker conflict with Python 3.12 on my computer?

No. Docker containers are isolated from the host environment.

```text
Host Machine
├── Python 3.12
│
└── Docker Container
    └── Python 3.11
```

When you use:

```dockerfile
FROM python:3.11
```

the application inside the container uses Python 3.11 regardless of the Python version installed on the host machine.

> **Simple rule:** The container uses its own runtime environment.

---

### 3. Do I need to install Python, Java, .NET, PostgreSQL, or Redis on my host machine?

No. In most cases, you only need the Docker Engine installed.

Docker can download the required environments using images.

For example:

```dockerfile
FROM python:3.11
```

provides Python inside the container.

Similarly, Docker can pull pre-built service images:

```bash
docker run postgres:15
docker run redis:7
```

The required software runs inside containers instead of being installed directly on the host.

> **Simple rule:** Your host provides Docker, while containers provide the application runtimes and services.

---

### 4. What does `docker build` actually do?

When you run:

```bash
docker build -t vote-app:v1 .
```

Docker performs the following steps:

```text
1. Reads the Dockerfile.
        ↓
2. Downloads required base images if they are not already available.
        ↓
3. Executes Dockerfile instructions such as WORKDIR, COPY, and RUN.
        ↓
4. Creates image layers.
        ↓
5. Produces the final Docker image.
```

For example:

```dockerfile
FROM python:3.11
```

Docker downloads or uses a cached Python 3.11 image.

Then:

```dockerfile
RUN pip install -r requirements.txt
```

runs `pip` inside the container build environment, not using Python from the host machine.

> **Simple rule:** `docker build` converts Dockerfile instructions into a reusable Docker image.

---

### 5. What does the `.` in `docker build -t vote-app:v1 .` mean?

The final `.` represents the **Docker build context**.

```bash
docker build -t vote-app:v1 .
```

means:

> Use the current directory and its files as the context available during the Docker build.

This allows instructions such as:

```dockerfile
COPY . .
```

to copy files from the build context into the image.

> **Simple rule:** The final `.` tells Docker where to find the files needed for the build.

---

### 6. Does `docker build` require Python or other dependencies to be installed on the host?

No.

For example:

```dockerfile
FROM python:3.11
```

provides Python inside the Docker build environment.

Therefore:

```dockerfile
RUN pip install -r requirements.txt
```

uses the Python and `pip` available inside the Docker image.

```text
Host Machine
    ↓
Docker Engine
    ↓
Python Base Image
    ↓
pip install requirements
```

The host Python installation is not required for this build process.

> **Simple rule:** Docker executes build commands inside the image environment, not directly on your host machine.
