# **Core Concept: What is a Container?**

A container is an isolated, lightweight, and portable unit for packaging and running software. It includes the application code, its dependencies, libraries, and configuration files. Crucially, it is not a full virtual machine.

*   **The Kernel:** This is the core of the operating system that manages hardware (CPU, memory, devices), handles processes, and provides security. **All containers on a single host share the exact same underlying host machine's Linux kernel.**
*   **The Userland (Userspace):** This is the environment where applications run, including the filesystem, package managers (`apt`, `yum`), system libraries (e.g., `glibc`), and utilities (`ls`, `bash`). **A container provides its own isolated userland.**

**Key Takeaway:** When you run `docker run ubuntu`, you are not booting a new OS. You are starting a process that uses the host's kernel but has an isolated filesystem (the Ubuntu userland) provided by its base image.

**Note on macOS/Windows:** Since these systems do not run a Linux kernel natively, Docker Desktop creates a lightweight Linux Virtual Machine (VM). The containers run inside this VM, sharing its kernel, while their userland is still provided by their base images.

---

# **Docker Images**

*   **Definition:** A Docker image is a read-only template containing instructions for creating a container. It is built from a `Dockerfile`.
*   **Layers:** Images are composed of multiple read-only layers. Each instruction in a `Dockerfile` creates a new layer.
    *   **Benefit:** Layers are cached and reusable. If you download two different versions of an app, Docker will only download the layers that differ, saving time and bandwidth.
*   **Registry:** Images are stored and shared in registries. **Docker Hub** is the default public registry (similar to GitHub for code).

---

# **The Dockerfile**

A `Dockerfile` is a text document containing all the commands a user could call on the command line to assemble an image.

**Typical Dockerfile Structure & Instructions:**

```dockerfile
# Use an official Node.js runtime as the parent (base) image.
# The node:19 image is itself likely based on a minimal OS like Alpine.
FROM node:19

# Create a directory in the container to hold the application code.
RUN mkdir -p /var/www/master_analytics

# Set the working directory for any subsequent instructions.
WORKDIR /var/www/master_analytics

# Copy the package.json and package-lock.json files to the working directory.
# Doing this before copying the rest of the code allows for better layer caching.
COPY package*.json ./

# Install the application dependencies inside the container.
RUN npm install
# Alternatively, for a clean, reproducible install in CI environments:
# RUN npm ci --only=production

# Copy the rest of the application's source code from the host to the container.
# It's recommended to have a `.dockerignore` file to exclude files like `node_modules`.
COPY . .

# Inform Docker that the container listens on the specified network port at runtime.
# (This is informational, not a command to open the port).
EXPOSE 8080

# Define the default command to run when the container starts.
# This starts the application.
CMD ["npm", "run", "start"]
```

**Layer Caching Strategy:** Dependencies (`npm install`) are installed before copying the application code. This means if only the application code changes, the `node_modules` layer can be reused from the cache, making subsequent builds much faster.

**Building the Image:**
```bash
docker build -t my-app:latest .
# -t : Tags the image with a name (`my-app`) and optional tag (`latest`)
# .  : The build context (path to the directory containing the Dockerfile)
```

---

# **Running Containers**

*   **Basic Run:**
    ```bash
    docker run -it ubuntu:22.04 /bin/bash
    # -it : Interactive mode with a pseudo-TTY (lets you interact with the shell)
    # /bin/bash : The command to run inside the container
    ```
*   **Detaching:** To exit a container without stopping it, use the detach sequence: `Ctrl+P` followed by `Ctrl+Q`.
*   **Container Lifespan:** A container lives only as long as the main process inside it is running. If you run `/bin/bash` and type `exit`, the bash process ends, and the container stops.

**`ENTRYPOINT` vs. `CMD`:**
*   **`ENTRYPOINT`:** Configures a container to run as an executable. It is the command that is *always* executed.
*   **`CMD`:** Provides default arguments for the `ENTRYPOINT`. If no `ENTRYPOINT` is set, `CMD` is the command that runs. It can be easily overridden at runtime.
*   **They are often used together:** `ENTRYPOINT ["myapp"]` and `CMD ["--help"]`. Running the container without arguments would execute `myapp --help`.

---

# **Docker Compose**

Docker Compose is a tool for defining and running multi-container Docker applications. You configure your application’s services, networks, and volumes in a `docker-compose.yml` file.

**Key Concepts in the `docker-compose.yml` Example:**
*   **`services`:** Define the different application components (e.g., a web app, a database, a message queue).
*   **`ports`:** Map ports from the host machine to the container (`host_port:container_port`).
*   **`environment`:** Set environment variables inside the container. **Important:** To change an environment variable, you must recreate the container (`docker-compose up -d --force-recreate`).
*   **`volumes`:** Persist data generated by and used by Docker containers. They can be named (e.g., `mongo-db`) or bind mounts (e.g., `./host_dir:/container_dir`).
*   **Named Volumes:** Data is managed by Docker and persists even if the container is removed. Defined in a top-level `volumes:` section.
*   **Bind Mounts:** Map a specific file or directory on the host machine into the container. Changes on the host are *not* always reflected in the container in real-time for all systems and **may** require a container restart. **Warning:** If the host file does not exist when the container starts, Docker will create a *directory* with that name.

**Example Compose Commands:**
```bash
# Start all services in detached mode
docker-compose up -d

# Stop and remove containers, networks (but not volumes)
docker-compose down

# Recreate containers even if their configuration hasn't changed (e.g., to apply new env vars)
docker-compose up -d --force-recreate

# Use a custom Compose file
docker-compose -f custom-compose.yml up -d
```

```yml
name: rudraksh

services:
#default user/pass = guest
  my-rabbit:
    image: rabbitmq:3-management
    container_name: 'rabbitmq'
    ports:
      - 5672:5672
      - 15672:15672
    volumes:
      - rmq-data:/var/lib/rabbitmq/

  some-mongo:
    image: mongo
    # restart: always

    ports:
      - 27017:27017
    environment:
      MONGO_INITDB_ROOT_USERNAME: mongoadmin
      MONGO_INITDB_ROOT_PASSWORD: secret

    volumes:
      - mongo-db:/data/db


  db:
    image: mysql
    restart: always

    command: --default-authentication-plugin=mysql_native_password

    ports:
      - 3306:3306
    
    environment:
      MYSQL_ALLOW_EMPTY_PASSWORD: yes
      MYSQL_USER: 'admin'
      MYSQL_ROOT_PASSWORD: root
      MYSQL_ROOT_HOST: '%'
      MY_TEST_ENV_VAR: test2  #printenv after exec, composeup reassin env variables

    volumes:
      - mysql7:/var/lib/mysql


  my-docker-postgres:
    image: postgres
    ports:
      - 5432:5432
    environment:
      POSTGRES_HOST_AUTH_METHOD: trust
    volumes:
      - postgres-volume:/var/lib/postgresql/data


# https://github.com/TimWolla/docker-adminer/issues/17#issuecomment-346016456
  adminer:
    image: adminer

    ports:
      - 8080:8080


volumes:
  mongo-db:
  postgres-volume:
  mysql7:
  rmq-data:
```

---

# **Essential Docker Commands**

*   **Containers:**
    *   `docker ps` : List running containers.
    *   `docker ps -a` : List all containers (including stopped ones).
    *   `docker rm <container_id>` : Remove a stopped container.
    *   `docker stop <container_id>` : Stop a running container gracefully.
    *   `docker logs <container_id>` : View the logs of a container.
    *   `docker exec -it <container_name> /bin/bash` : Run a command (like a shell) inside a running container.

*   **Images:**
    *   `docker images` : List images.
    *   `docker rmi <image_id>` : Remove an image.

*   **Networking:**
    *   `docker network ls` : List all networks.
    *   `docker network create --driver bridge alpine-net` : Create a new user-defined bridge network named `alpine-net`. (The `--driver bridge` flag is often optional as it's the default).
    *   `docker network inspect <network_name>` : Get detailed information (IP addresses, connected containers, etc.) about a network.
    *   `docker network connect alpine-net alpine2` : Connect the running container named `alpine2` to the `alpine-net` network.
    *   `docker network disconnect bridge alpine2` : Disconnect the container `alpine2` from the default `bridge` network. A Docker container can be connected to multiple networks simultaneously. It will get different ip address for different networks.
    *   `docker inspect <container_id>` : Get low-level information about a container (JSON format). You can use Go templates to filter, e.g., `docker inspect c1 -f "{{json .NetworkSettings.Networks }}"` to see only network details.

*   **Why these commands are useful:** The default `bridge` network provides basic isolation but doesn't offer DNS resolution between containers (you must use `--link`, which is legacy). **User-defined networks** (like `alpine-net`) provide automatic DNS resolution, allowing containers to find each other by name. This is a fundamental best practice for multi-container applications. The commands you listed are the manual process for moving a container from the default network to a custom, more functional one.

*   **Inspecting Containers:**
    *   `docker inspect <container_id>` : Get low-level information about a container (JSON format). You can use Go templates to filter, e.g., `docker inspect c1 -f "{{json .NetworkSettings.Networks }}"`.

---