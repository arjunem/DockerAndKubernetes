# 🐳 Docker Learning Journey

> **Welcome to my comprehensive Docker learning documentation!** This document tracks my entire Docker learning journey with hands-on command examples, detailed explanations, and key concepts.

---

## 📚 Table of Contents

- [Getting Started](#getting-started)
- [Docker Basics - Command Entries](#docker-basics)
- [Learning Notes](#learning-notes)
- [Hands-On Testing: Container Isolation](#hands-on-testing)
- [Three-Container Application Exercise](#three-container-exercise)

---

# 🚀 Getting Started

<details>
<summary><b>Entry #1: Docker Help Command</b></summary>

### 📋 Command
```bash
docker --help
```

### 🎯 Purpose
View Docker's help information to understand available commands and capabilities.

### 💡 Key Takeaways
- ✅ Docker has Common Commands, Management Commands, and Swarm Commands
- ✅ Use `docker COMMAND --help` for detailed help on specific commands
- ✅ Modern Docker includes AI Agent, Compose, and Scout features

### 📊 Command Categories

**Common Commands:**
- `run` - Create and run containers
- `ps` - List containers
- `build` - Build images
- `pull/push` - Download/upload images

**Management Commands:**
- `container` - Manage containers
- `image` - Manage images
- `network` - Manage networks
- `volume` - Manage volumes

</details>

<details>
<summary><b>Entry #2: Docker Version</b></summary>

### 📋 Command
```bash
docker --version
```

### 🎯 Purpose
Check installed Docker version.

### ✅ Output
```
Docker version 27.4.1, build b9d17ea
```

### 💡 Key Info
- Version: 27.4.1
- Build: b9d17ea
- Latest stable release installed

</details>

<details>
<summary><b>Entry #3: Docker System Information</b></summary>

### 📋 Command
```bash
docker info
```

### 🎯 Purpose
Display comprehensive system-wide Docker information.

### 📊 Key System Details

| Category | Details |
|----------|---------|
| **Containers** | 0 running, 0 paused, 0 stopped (Total: 0) |
| **Images** | 0 |
| **Server Version** | 27.4.1 |
| **Storage Driver** | overlayfs2 |
| **Operating System** | Docker Desktop |
| **OSType** | linux |
| **Architecture** | x86_64 |
| **CPUs** | 12 |
| **Total Memory** | 7.595 GiB |

### 💡 Important Observations
- ⚠️ Docker is running on Windows but uses Linux containers (via WSL2)
- ✅ 12 CPU cores available to Docker
- ✅ ~7.6GB RAM allocated

</details>

---

# 📝 Learning Notes

## 🔧 Docker Command Structure

<details>
<summary><b>Understanding Global Flags vs Subcommands</b></summary>

### The Question
Why do we use `docker info` instead of `docker --info`?

### The Answer

Docker follows this command structure:
```
docker [GLOBAL FLAGS] COMMAND [COMMAND FLAGS] [ARGUMENTS]
```

### Examples

**Global Flags (use `--`):**
```bash
docker --version        # Global flag
docker --help          # Global flag
docker --debug info    # Global flag + command
```

**Subcommands (no `--`):**
```bash
docker info            # Subcommand
docker ps              # Subcommand
docker container ls    # Subcommand
```

### 📊 Comparison Table

| Type | Syntax | Example | Purpose |
|------|--------|---------|---------|
| **Global Flag** | `--flag` | `docker --version` | Affects Docker itself |
| **Subcommand** | `command` | `docker info` | Docker action/operation |
| **Command Flag** | `command --flag` | `docker ps --all` | Modifies the command |

</details>

<details>
<summary><b>Old vs New Docker CLI Structure</b></summary>

### Evolution of Docker Commands

Docker has evolved from simple commands to organized management commands.

### Old Style (Still Works)
```bash
docker ps              # List containers
docker images          # List images
docker rm              # Remove container
docker rmi             # Remove image
```

### New Style (Recommended)
```bash
docker container ls    # List containers
docker image ls        # List images
docker container rm    # Remove container
docker image rm        # Remove image
```

### 📊 Why the Change?

| Aspect | Old Style | New Style |
|--------|-----------|-----------|
| **Organization** | Flat commands | Grouped by resource type |
| **Clarity** | Sometimes ambiguous | Always clear |
| **Discoverability** | Harder to explore | Easy to find related commands |
| **Consistency** | Varied patterns | Consistent patterns |

### 💡 Best Practice
Use **new style** for clarity, but know **old style** for compatibility with tutorials.

</details>

<details>
<summary><b>Images vs Containers</b></summary>

### The Analogy 🎯

Think of Docker Images and Containers like:
- **Image** = Recipe 📋
- **Container** = Cooked Meal 🍽️

### Detailed Comparison

| Aspect | Image | Container |
|--------|-------|-----------|
| **What is it?** | Blueprint/Template | Running instance |
| **Analogy** | Class in programming | Object/Instance |
| **State** | Static, immutable | Dynamic, running |
| **Can run?** | No | Yes |
| **Storage** | Stored on disk | Running in memory |
| **Modification** | Cannot be changed | Can be modified |
| **Creation** | Built from Dockerfile | Created from image |

### Examples

```bash
# Image operations
docker image ls                    # List all images
docker image pull nginx            # Download image
docker image rm nginx              # Delete image

# Container operations
docker container run nginx         # Create & start container from image
docker container ls                # List running containers
docker container stop <id>         # Stop container
docker container rm <id>           # Remove container
```

### 🔄 Relationship

```
Dockerfile → Build → Image → Run → Container(s)
                      ↓              ↓
                  (1 image)    (many containers)
```

### 💡 Key Point
**One image can create multiple containers!**
```bash
docker run nginx    # Container 1 from nginx image
docker run nginx    # Container 2 from nginx image
docker run nginx    # Container 3 from nginx image
```

</details>

---

## 🏃 Running Containers

<details>
<summary><b>Entry #4: First Container - nginx (Foreground)</b></summary>

### 📋 Command
```bash
docker container run --publish 80:80 nginx
```

### 🎯 Purpose
Run nginx web server container for the first time.

### 📊 What Happened

1. **Image not found locally** → Docker automatically pulled from Docker Hub
2. **Downloaded nginx image** → Multiple layers downloaded
3. **Container started** → nginx web server running on port 80
4. **Foreground mode** → Terminal attached to container output

### 💡 Key Concepts

**Port Publishing:**
```
--publish 80:80
         ↓   ↓
      HOST:CONTAINER
```
- Host port 80 → maps to → Container port 80
- Access via: http://localhost

**Foreground Mode:**
- Terminal stays attached
- See logs in real-time
- Press `Ctrl+C` to stop (but container may still run!)

### ⚠️ Important Discovery
Pressing `Ctrl+C` **interrupts** the terminal but **doesn't stop** the container!

</details>

<details>
<summary><b>Entry #5: Listing Running Containers</b></summary>

### 📋 Command
```bash
docker container ls
```

### 🎯 Purpose
List all running containers.

### ✅ Output Example
```
CONTAINER ID   IMAGE   COMMAND                  PORTS                NAMES
5f0f530e18c7   nginx   "/docker-entrypoint.…"   0.0.0.0:80->80/tcp   quirky_montalcini
```

### 📊 Understanding the Output

| Column | Meaning | Example |
|--------|---------|---------|
| **CONTAINER ID** | Unique identifier (short) | 5f0f530e18c7 |
| **IMAGE** | Source image | nginx |
| **COMMAND** | Entry point command | /docker-entrypoint.sh |
| **PORTS** | Port mappings | 0.0.0.0:80->80/tcp |
| **NAMES** | Auto-generated or custom name | quirky_montalcini |

### 💡 Important Flags
```bash
docker container ls          # Running containers only
docker container ls -a       # All containers (including stopped)
docker container ls -q       # Only container IDs
```

</details>

<details>
<summary><b>Entry #6: Stopping a Container</b></summary>

### 📋 Command
```bash
docker container stop 5f0f530e18c7
```

### 🎯 Purpose
Gracefully stop a running container.

### 📊 How It Works

1. Docker sends **SIGTERM** signal (graceful shutdown)
2. Container has **10 seconds** to shut down cleanly
3. If not stopped, Docker sends **SIGKILL** (force stop)

### 💡 Variations
```bash
docker container stop <ID>          # By ID
docker container stop <NAME>        # By name
docker container stop <ID> <ID>     # Multiple containers
```

</details>

<details>
<summary><b>Entry #7: Running in Detached Mode</b></summary>

### 📋 Command
```bash
docker container run -d --publish 80:80 --name webhost nginx
```

### 🎯 Purpose
Run container in background (detached mode).

### 🎨 New Flags

| Flag | Purpose | Example |
|------|---------|---------|
| `-d` | Detached mode (background) | Container runs in background |
| `--name` | Custom container name | "webhost" instead of random |

### 📊 Detached vs Foreground

| Mode | Terminal | Logs | Stop |
|------|----------|------|------|
| **Foreground** | Attached | Visible | Ctrl+C (unreliable) |
| **Detached** | Free | Use `docker logs` | `docker stop` |

### ✅ Best Practice
**Always use `-d` for services** (web servers, databases, etc.)

</details>

---

## 🗑️ Container Management

<details>
<summary><b>Entry #8: Listing All Containers</b></summary>

### 📋 Command
```bash
docker container ls -a
```

### 🎯 Purpose
List ALL containers (running + stopped).

### 📊 Container States

| Status | Meaning | Can Start? | Can Remove? |
|--------|---------|------------|-------------|
| **Up** | Running | ❌ Already running | ❌ Must stop first |
| **Exited** | Stopped cleanly | ✅ Yes | ✅ Yes |
| **Created** | Created but never run | ✅ Yes | ✅ Yes |

</details>

<details>
<summary><b>Entry #9: Viewing Container Processes</b></summary>

### 📋 Command
```bash
docker container top webhost
```

### 🎯 Purpose
Display running processes inside a container.

### ✅ Example Output
```
PID     USER    TIME    COMMAND
5348    root    0:00    nginx: master process
5391    101     0:00    nginx: worker process
```

### 💡 Use Cases
- Debug what's running inside container
- Check process hierarchy
- Monitor worker processes

</details>

<details>
<summary><b>Entry #10: Removing Containers</b></summary>

### 📋 Commands
```bash
docker container rm 852 dcd 5f0      # Remove multiple stopped containers
docker container rm -f 852            # Force remove running container
```

### 🎯 Rules

| Container State | Can Remove? | Command |
|----------------|-------------|---------|
| **Stopped** | ✅ Yes | `docker rm <id>` |
| **Running** | ❌ No (must stop first) | `docker stop <id>` then `docker rm <id>` |
| **Running** | ✅ Yes (with force) | `docker rm -f <id>` |

### ⚠️ Important
- Stopped containers remain on disk until explicitly removed
- Force remove (`-f`) is quick but doesn't allow graceful shutdown

</details>

---

# 🧪 Hands-On Testing: Container Isolation

> **This section demonstrates the 5 Linux kernel features that make containers isolated and secure**

## 1️⃣ Process Isolation

<details>
<summary><b>Test #1-3: Process Namespace Isolation</b></summary>

### Commands Executed
```bash
# Create test container
docker container run -d --name test nginx

# Try to view processes inside container (failed - ps not installed)
docker container exec test ps aux

# View Windows host processes
Get-Process | Select-Object -First 30
```

### 📊 Key Findings

| Environment | Process Visibility |
|-------------|-------------------|
| **Windows Host** | ✅ Sees ALL processes (100s of processes) |
| **Container** | ❌ Would only see container processes (if ps was available) |

### 💡 What We Learned

**1. Container Minimalism**
- nginx container doesn't include `ps` utility
- Containers only include what's needed to run the app
- Smaller size = better security + faster downloads

**2. Process Isolation**
- Host sees 100+ processes (Chrome, system services, apps)
- Container would only see its own processes (~1-5)
- Complete process namespace isolation

**3. Real-World Impact**
- Compromised container can't see host processes
- Clean shutdown - stopping container removes all its processes
- No process conflicts between containers

### ✅ Answer
**Process Namespace Isolation**: Container processes can't see Windows host processes, even though they run on the same machine!

</details>

## 2️⃣ Network Isolation

<details>
<summary><b>Test #4-6: Network Namespace Isolation</b></summary>

### Commands Executed
```bash
# Check Windows host network adapters
Get-NetAdapter

# Try to check container network (failed - ip not installed)
docker container exec test ip addr

# Check container IP from host
docker container inspect test --format='{{range .NetworkSettings.Networks}}{{.IPAddress}}{{end}}'
```

### 📊 Key Findings

**Windows Host Network:**
- **7 Network Adapters**
  - Wi-Fi: 480 Mbps (physical)
  - Ethernet: Disconnected (physical)
  - vEthernet (WSL): 10 Gbps ← Docker uses this
  - vEthernet (Default Switch): 10 Gbps
  - VPN adapters

**Container Network:**
- **IP Address**: 172.17.0.2
- **Network**: Docker bridge network (172.17.0.0/16)
- **Isolation**: Can't see host's 7 adapters

### 📊 Network Architecture
```
Windows Host
├── Wi-Fi: 480 Mbps (physical)
├── Ethernet: Disconnected
├── vEthernet (WSL): 10 Gbps ← Docker uses this
└── Multiple virtual adapters

Container (isolated)
└── eth0: 172.17.0.2 ← Only sees this
```

### ✅ Answer
**Network Namespace Isolation**: Container has its own IP (172.17.0.2) completely isolated from host's 7 network adapters!

</details>

## 3️⃣ Filesystem Isolation

<details>
<summary><b>Test #7-9: Filesystem Namespace Isolation</b></summary>

### Commands Executed
```bash
# Check Windows host filesystem
Get-ChildItem C:\

# Check container filesystem
docker container exec test ls /

# Detailed container filesystem
docker container exec test ls -la /
```

### 📊 Completely Different Filesystems

**Windows Host (C:\):**
```
├── Program Files
├── Windows
├── Users
├── Python313
└── msys64
```

**Container (/):**
```
├── bin, sbin, usr (Linux binaries)
├── etc (configurations)
├── var (variable data)
├── .dockerenv (Docker marker)
└── docker-entrypoint.sh
```

### 💡 Special Container Features

| Feature | Purpose |
|---------|---------|
| `.dockerenv` | Identifies this is a Docker container |
| `docker-entrypoint.sh` | Container startup script |
| Symlinks (bin→usr/bin) | Modern Linux structure |

### ✅ Answer
**Filesystem Namespace Isolation**: Container has complete Linux filesystem (/, /etc, /usr) separate from Windows (C:\, Program Files)!

</details>

## 4️⃣ Hostname Isolation

<details>
<summary><b>Test #10-12: UTS Namespace Isolation</b></summary>

### Commands Executed
```bash
# Check Windows hostname
$env:COMPUTERNAME

# Check container hostname
docker container exec test hostname

# Check container ID
docker container inspect test --format='{{.Id}}'
```

### 📊 Key Findings

| System | Hostname | Source |
|--------|----------|--------|
| **Windows Host** | TSPL-NB101 | Windows computer name |
| **Container** | 75a3c4438f6e | First 12 chars of container ID |

### 🔍 Hostname Derivation
```
Container ID:  75a3c4438f6ec6c4ea426dc92f0d79af37b35ed70585a55c34767ba706bb1257
               ^^^^^^^^^^^^                                                    
               └── First 12 characters = hostname

Container Hostname: 75a3c4438f6e
```

### 💡 Customization Options
```bash
# Custom hostname
docker run --hostname my-web-server nginx

# Hostname + domain
docker run --hostname web --domainname example.com nginx
# FQDN: web.example.com
```

### ✅ Answer
**UTS Namespace Isolation**: Container has its own hostname (75a3c4438f6e) separate from Windows host (TSPL-NB101)!

</details>

## 5️⃣ Resource Limits (cgroups)

<details>
<summary><b>Test #13-17: Memory and CPU Limits</b></summary>

### Commands Executed
```bash
# Create memory-limited container
docker container run -d --name limited -m 100m nginx

# Create CPU-limited container
docker container run -d --name cpu-limited --cpus="1.5" nginx

# Monitor resource usage
docker container stats --no-stream
```

### 📊 Resource Limits in Action

**Memory Limits:**
| Container | Memory Usage | Memory Limit | % Used |
|-----------|-------------|--------------|--------|
| **limited** | 12.78 MiB | 100 MiB | 12.78% ✅ Limited |
| **test** | 13.46 MiB | 7.595 GiB | 0.17% ⚠️ Unlimited |

**CPU Limits:**
| Container | CPU Limit | Host CPUs | Max Available |
|-----------|-----------|-----------|---------------|
| **cpu-limited** | 1.5 cores | 12 cores | 12.5% |
| **test** | Unlimited | 12 cores | 100% |

### 💡 What Happens When Exceeded?

**Memory Limit:**
- Container using 99MB of 100MB → OK ✅
- Container tries to use 101MB → KILLED ❌ (Exit code 137)

**CPU Limit:**
- Container limited to 1.5 cores
- Tries to use more → THROTTLED ⚠️

### ✅ Answer
**cgroups**: Linux kernel feature that limits memory (100MB) and CPU (1.5 cores). Prevents resource hogging!

</details>

---

## 🤔 Why Are Some Commands Missing?

<details>
<summary><b>Container Minimalism Philosophy</b></summary>

### The Question
Why don't `ps`, `ip`, and other tools work inside containers?

### The Answer: By Design! ✨

**Containers vs Traditional Servers:**

```
Traditional Server          Docker Container
├── Full OS (GB)           ├── Minimal OS (MB)
├── All utilities          ├── Only nginx
│   ├── ps, top           ├── Required libraries
│   ├── ip, netstat       └── Nothing else!
│   ├── vim, curl
│   └── 1000s of tools
└── Application
```

### 📊 Why This Approach?

| Reason | Benefit |
|--------|---------|
| **Smaller Size** | Faster downloads, less storage |
| **Security** | Fewer tools = smaller attack surface |
| **Performance** | Less to load, faster startup |
| **Single Responsibility** | Does ONE thing well |
| **Immutability** | Forces proper logging/monitoring |

### 📊 Image Size Comparison

| Image | Size | What's Included |
|-------|------|----------------|
| **Ubuntu Full** | ~77 MB | Full Ubuntu + utilities |
| **nginx:latest** | ~67 MB | nginx + minimal base |
| **nginx:alpine** | ~16 MB | nginx + Alpine Linux |
| **nginx distroless** | ~10 MB | nginx + ONLY libraries |

### 🔧 How to Debug Without Tools

**Option 1: Use Docker commands from host**
```bash
docker container top test          # View processes
docker container stats test        # Resource usage
docker container inspect test      # All details
docker container logs test         # Container logs
```

**Option 2: Use debugging images**
```bash
docker run -it --rm nicolaka/netshoot    # Network tools
docker run -it --rm alpine sh            # Minimal Linux
```

### ✅ Key Takeaway
**Missing commands are a FEATURE, not a bug!** Containers are minimal for security and speed. Use Docker commands from host to inspect them.

</details>

---

# 🎯 Three-Container Application Exercise

> **Learn multi-container management by running nginx, Apache, and MySQL together**

<details>
<summary><b>Exercise Overview</b></summary>

### 🎯 Objectives
- Run multiple containers simultaneously
- Manage port mappings without conflicts
- Use environment variables for configuration
- Practice lifecycle management (start, stop, remove)
- Understand multi-container cleanup

### 🏗️ Architecture
```
Host Machine
├── nginx (Port 80)      ← Web Server 1
├── Apache (Port 8080)   ← Web Server 2
└── MySQL (Port 3306)    ← Database Server
```

### ✅ Commands Reference

**Start Containers:**
```bash
docker container run -d --name web-server-1 --publish 80:80 nginx
docker container run -d --name web-server-2 --publish 8080:80 httpd
docker container run -d --name database --publish 3306:3306 -e MYSQL_ROOT_PASSWORD=mysecretpassword mysql
```

**Verify:**
```bash
docker container ls                                    # Running containers
Invoke-WebRequest http://localhost                     # Test nginx
Invoke-WebRequest http://localhost:8080                # Test Apache
docker container logs database                         # Check MySQL
```

**Cleanup:**
```bash
docker container stop web-server-1 web-server-2 database
docker container rm web-server-1 web-server-2 database
docker container ls -a                                 # Verify cleanup
```

</details>

<details>
<summary><b>Detailed Steps & Results</b></summary>

### Entry #18: Start nginx (Web Server 1)
```bash
docker container run -d --name web-server-1 --publish 80:80 nginx
```
✅ Container ID: c4852db441be
✅ Accessible at: http://localhost

---

### Entry #19: Start Apache (Web Server 2)
```bash
docker container run -d --name web-server-2 --publish 8080:80 httpd
```
✅ Container ID: b3ed4c36cc3b
✅ Image automatically downloaded (6 layers)
✅ Accessible at: http://localhost:8080

---

### Entry #20: Start MySQL (Database)
```bash
docker container run -d --name database --publish 3306:3306 -e MYSQL_ROOT_PASSWORD=mysecretpassword mysql
```
✅ Container ID: aecfa56e969d
✅ Image downloaded (10 layers)
✅ Environment variable set for root password
✅ Listening on port 3306

---

### Entry #21: Verify All Running
```bash
docker container ls
```

**Output:**
| Container | Image | Port | Status |
|-----------|-------|------|--------|
| database | mysql | 3306:3306 | Up 1 minute |
| web-server-2 | httpd | 8080:80 | Up 2 minutes |
| web-server-1 | nginx | 80:80 | Up 5 minutes |

✅ All 3 containers running successfully!

---

### Entry #22: Test nginx
```bash
Invoke-WebRequest http://localhost
```
✅ Status: 200 OK
✅ Content: nginx welcome page (615 bytes)

---

### Entry #23: Test Apache
```bash
Invoke-WebRequest http://localhost:8080
```
✅ Status: 200 OK
✅ Content: "It works!" (45 bytes)

**Comparison:**
- nginx: 615 bytes (detailed welcome page)
- Apache: 45 bytes (simple "It works!")

---

### Entry #24: Test MySQL (Client Not Installed)
```bash
mysql -h localhost -P 3306 -u root -p
```
❌ MySQL client not installed on Windows
✅ **This is normal** - don't need client on host to run MySQL server in container

---

### Entry #25: Check MySQL Logs
```bash
docker container logs database
```

**Key Findings:**
✅ MySQL 9.4.0 started successfully
✅ Database files initialized
✅ InnoDB engine running
✅ Listening on port 3306
✅ Ready for connections
✅ 12 CPUs detected, 7.6GB RAM available

---

### Entry #26: Stop All Containers
```bash
docker container stop web-server-1 web-server-2 database
```
✅ All containers stopped gracefully
✅ Batch operation successful

---

### Final Cleanup
```bash
docker container rm web-server-1 web-server-2 database
docker container ls -a
```
✅ All containers removed
✅ System clean

</details>

<details>
<summary><b>Key Learnings from Exercise</b></summary>

### 💡 What We Learned

**1. Port Management**
- Each container needs unique host ports
- Container ports can be the same (both nginx and Apache use 80 internally)
- Format: `--publish HOST:CONTAINER`

**2. Environment Variables**
- Use `-e` flag to pass configuration
- Essential for databases (passwords, users)
- Different for dev/test/prod environments

**3. Container Naming**
- `--name` makes management much easier
- Enables batch operations
- Required for multi-container workflows

**4. Multi-Container Operations**
```bash
# Batch stop
docker container stop name1 name2 name3

# Batch remove
docker container rm name1 name2 name3
```

**5. Container Lifecycle**
```
Created → Running → Stopped → Removed
   ↓         ↓         ↓         ↓
docker    docker    docker    docker
run       logs      stop      rm
```

</details>

---

# 📊 Summary & Quick Reference

## Container Isolation Features Tested ✅

| Feature | Host | Container | Isolation Method |
|---------|------|-----------|------------------|
| **Processes** | 100s of processes | Only container processes | Process Namespace |
| **Network** | 7 adapters, multiple IPs | IP: 172.17.0.2 | Network Namespace |
| **Filesystem** | C:\Windows, Program Files | /, /bin, /etc, /usr | Mount Namespace |
| **Hostname** | TSPL-NB101 | 75a3c4438f6e | UTS Namespace |
| **Resources** | 12 CPUs, 7.6GB RAM | Limited (100MB, 1.5 cores) | cgroups |

## Essential Docker Commands

<details>
<summary><b>Container Lifecycle</b></summary>

```bash
# Create & Start
docker container run nginx                    # Start new container
docker container run -d nginx                 # Detached mode
docker container run -d --name web nginx      # With custom name
docker container run -d -p 80:80 nginx        # With port mapping

# List
docker container ls                           # Running containers
docker container ls -a                        # All containers
docker container ls -q                        # Only IDs

# Inspect
docker container top <name>                   # Running processes
docker container stats <name>                 # Resource usage
docker container logs <name>                  # Container logs
docker container inspect <name>               # Detailed info

# Stop & Remove
docker container stop <name>                  # Stop container
docker container rm <name>                    # Remove stopped container
docker container rm -f <name>                 # Force remove running container

# Batch Operations
docker container stop name1 name2 name3       # Stop multiple
docker container rm name1 name2 name3         # Remove multiple
```

</details>

<details>
<summary><b>Resource Limits</b></summary>

```bash
# Memory Limits
docker run -m 512m nginx                      # 512 megabytes
docker run -m 1g nginx                        # 1 gigabyte

# CPU Limits
docker run --cpus="2" nginx                   # 2 cores
docker run --cpus="0.5" nginx                 # Half a core

# Combined
docker run -m 512m --cpus="1.5" nginx         # Memory + CPU
```

</details>

<details>
<summary><b>Networking</b></summary>

```bash
# Port Mapping
docker run -p 80:80 nginx                     # Host:Container
docker run -p 8080:80 nginx                   # Different host port
docker run -p 80:80 -p 443:443 nginx          # Multiple ports

# Network Commands
docker network ls                             # List networks
docker network inspect bridge                 # Inspect default network
```

</details>

---

## 🎓 Key Concepts Mastered

✅ **Docker Architecture** - Client-server model, daemon, CLI
✅ **Images vs Containers** - Blueprint vs running instance
✅ **Container Lifecycle** - Create, run, stop, remove
✅ **Port Mapping** - Host-to-container communication
✅ **Detached Mode** - Background container execution
✅ **Container Isolation** - 5 Linux kernel features (namespaces, cgroups)
✅ **Resource Limits** - Memory and CPU constraints
✅ **Multi-Container Management** - Running multiple services together
✅ **Environment Variables** - Configuration without rebuilding

---

## 📖 How to Use This Document in Notion

### Import Steps:
1. Copy all content from this file
2. In Notion, create a new page
3. Paste the content
4. Notion will auto-convert markdown formatting

### Features You'll Get:
- ✅ Collapsible toggle sections (all `<details>` tags)
- ✅ Formatted tables
- ✅ Code blocks with syntax highlighting
- ✅ Emoji icons for visual navigation
- ✅ Hierarchical organization
- ✅ Inline callouts and highlights

### Customization:
- Add database views for command entries
- Create linked pages for related concepts
- Add your own notes and annotations
- Create a favorites section for frequently used commands

---

**Happy Dockering! 🐳**

