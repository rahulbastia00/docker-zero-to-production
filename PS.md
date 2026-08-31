To master every Docker and containerization concept through a problem-driven, evolutionary approach, you do not need 10 separate repositories. A single, polyglot microservices repository contains all the architectural complexity required:

* **Target Repository:** `[https://github.com/dockersamples/example-voting-app](https://github.com/dockersamples/example-voting-app)`
* **Application Services:** Python (Flask web app), Node.js (Results web app), .NET/Java (Background queue worker), Redis (In-memory broker), PostgreSQL (Relational DB).
* **Initial Setup:** Clone the repository and immediately delete all existing `Dockerfile`s, `docker-compose.yml`, and deployment scripts. You will build and evolve the infrastructure through five iterative stages.

---

### Stage 1: The Raw Monolith & Naive Containerization

**The Problem:** The Python `vote` and Node.js `result` apps run locally, but dependency versions clash, manual installations fail, and the local runtime is polluted.

* **Action:** Write naive, basic `Dockerfile`s using large base images (`python:3.11`, `node:20`).
* **Concepts Mastered:**
* **Architecture & Workflow:** Docker Client vs. Docker Daemon interaction, Docker Host execution, `Dockerfile` syntax (`FROM`, `WORKDIR`, `COPY`, `RUN`, `CMD`), Docker Images vs. Containers.
* **Layering & Caching:** Notice how re-running `docker build` invalidates layers when source code is copied before dependencies. Fix it by caching dependency manifests (`package.json`, `requirements.txt`).
* **Port Mapping:** Expose application ports to the host via `-p 5000:80` and observe how host routing connects to the container layer.



---

### Stage 2: Data Loss & Network Disconnects

**The Problem:** You launch the PostgreSQL and Redis containers manually alongside the web apps. When PostgreSQL crashes or restarts, all voting data disappears. Furthermore, the web apps cannot communicate with Redis or PostgreSQL because `localhost` points to the container itself, not the host.

* **Action:** Fix data persistence and configure manual container networks using the Docker CLI.
* **Concepts Mastered:**
* **Storage & Persistence:** Observe the ephemeral Container Layer losing state upon termination. Fix it using **Named Volumes** (`docker volume create db-data`) for PostgreSQL. Experiment with **Bind Mounts** for hot-reloading code during development and **Tmpfs Mounts** for ephemeral scratch buffers.
* **Container Networking:** Understand the limitations of the default `bridge` network (no automatic DNS resolution). Create a user-defined custom bridge network (`docker network create voting-net`) and use **Embedded DNS** to make services discoverable by name (`redis`, `db`) instead of static IPs. Test network isolation using `--network none` and `--network host`.



---

### Stage 3: Orchestration Fatigue & Configuration Drift

**The Problem:** Running 5 independent `docker run` commands with 20+ flags for ports, networks, volumes, and environment variables is error-prone, untracked, and fails if started in the wrong order (e.g., worker boots before Redis/PostgreSQL).

* **Action:** Consolidate the entire multi-service lifecycle into a declarative `docker-compose.yml`.
* **Concepts Mastered:**
* **Multi-Container Coordination:** Compose `services`, custom networks, named volumes, and `.env` files for dynamic runtime configuration.
* **Lifecycle & Boot Order:** Use `depends_on` alongside container **Healthchecks** (`pg_isready`, `redis-cli ping`) so services wait for dependencies to be healthy, not just started.
* **Fault Tolerance:** Implement container **Restart Policies** (`restart: unless-stopped`, `on-failure`).



---

### Stage 4: Bloated Images, Cache Misses & Security Vulnerabilities

**The Problem:** Your images are >1GB, take minutes to push/pull, contain build-time compilers (SDKs/GCC), and run as `root`, exposing the host kernel to privilege escalation exploits.

* **Action:** Refactor all Dockerfiles for production hardening and security.
* **Concepts Mastered:**
* **Multi-Stage Builds & BuildKit:** Separate build environments from runtime artifacts. Use BuildKit cache mounts (`--mount=type=cache`) to accelerate builds.
* **Distroless & Alpine:** Strip all non-essential binaries and shells. Drop image sizes down to <50MB using `gcr.io/distroless` or `alpine`.
* **Security & Linux Primitives:** Run containers under non-root users (`USER 1001`), clean up **Dangling Images**, and apply container resource caps via **Control Groups (cgroups)** (`--memory="256m"`, `--cpus="0.5"`) to prevent OOM cascade failures on the host. Inspect isolation boundaries created by **Linux Namespaces** (PID, NET, MNT).



---

### Stage 5: Multi-Host Scaling & Secret Management

**The Problem:** As traffic increases, a single host cannot handle the load, and hardcoded database passwords in Compose files expose production credentials.

* **Action:** Transition the Compose application into a local multi-node cluster using Docker Swarm.
* **Concepts Mastered:**
* **Docker Swarm & Overlay Networks:** Initialize a cluster (`docker swarm init`), create an **Overlay Network** spanning multiple nodes, and deploy the application stack (`docker stack deploy -c docker-stack.yml vote`).
* **Secrets Management:** Store database passwords securely using native **Docker Secrets** instead of plaintext environment variables.
* **Scaling & Ingress Mesh:** Scale the `vote` frontend to multiple replicas (`docker service scale vote=5`) and observe Docker's internal routing mesh distributing incoming requests.



---

For a step-by-step walkthrough demonstrating how each microservice in this exact repository is connected, configured, and orchestrated, refer to this [Docker Voting App architecture guide](https://www.youtube.com/watch?v=iOGEBj7Ozak).

This walkthrough breaks down the multi-container data flow of the voting application and shows how each service communicates over isolated networks.