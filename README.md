# DevOps Practical 6: Introduction to Ansible

**LYCS DevOps Sem 7, Q6: Introduction to Ansible (30 marks)**

Repository: https://github.com/show-star/aa6

## Aim

With **1 control node and 2 managed nodes**: install Ansible on the control node, create an inventory with the 2 servers grouped as `[webservers]`, test them with `ansible all -m ping`, write a playbook `install-packages.yml` that updates the apt cache, installs `apache2`, `curl`, `git` and `vim`, starts and enables Apache and copies a custom `index.html` to `/var/www/html/`, run it, check Apache in the browser, and run it again to prove idempotency (`changed=0`).

## What this practical is about

- **Ansible** is an automation tool for setting up servers. You write *what* the servers should look like, and Ansible makes it so.
- **Control node:** the machine where Ansible is installed and run from.
- **Managed nodes:** the servers Ansible configures. They need only **SSH** and **Python**, with no extra software. This is why Ansible is called **agentless**.
- **Inventory:** a file listing the managed nodes, in groups such as `[webservers]`.
- **Playbook:** a YAML file listing the **tasks** to run on a group of hosts.
- **Module:** a built-in unit of work used by a task, such as `apt`, `service` or `copy`.
- **Idempotency:** running the same playbook again changes nothing if the servers are already correct (`changed=0`).

### How this lab gives you 3 machines on one Windows PC

There aren't three real machines, so this lab runs **three Ubuntu containers** on one Docker network. The control node connects to the two managed nodes over **SSH**, exactly as it would to real servers.

| Container | Role | Login | Browser |
| --- | --- | --- | --- |
| `control` | Control node (Ansible is installed here) | None | None |
| `node1` | Managed node 1 (web server) | `root` / `ansible123` over SSH | http://localhost:8091 |
| `node2` | Managed node 2 (web server) | `root` / `ansible123` over SSH | http://localhost:8092 |

## Files in this repo

```
aa6/
├── Dockerfile.node            Image for node1 and node2: Ubuntu + SSH server + Python
├── compose.yaml               Starts control, node1 and node2
├── lab/
│   ├── inventory.ini          The [webservers] group: node1 and node2
│   ├── ansible.cfg            Uses inventory.ini and skips the SSH host key prompt
│   ├── index.html             The custom web page Ansible copies to both servers
│   └── install-packages.yml   The playbook
└── README.md                  These steps
```

You don't need to download anything: every file is created by the commands below.

---

## Before you start

1. **Open Docker Desktop** (Windows key, type *Docker Desktop*, Enter). Wait until the bottom left says **Engine running**.
2. **Open PowerShell** (Windows key, type *PowerShell*, Enter). Use PowerShell, **not Command Prompt**.
3. **Check Docker works:**
   ```powershell
   docker --version
   ```
   If you get **'docker' is not recognized**, see [Error 1](#error-1-docker-is-not-recognized) and fix it first.

**Where to do this:** this practical uses **two different places**.

| Part | Where | Prompt looks like |
| --- | --- | --- |
| Part A | **PowerShell**, in `Desktop\ansible-lab` | `PS C:\Users\HP\Desktop\ansible-lab>` |
| Parts B to F | **Inside the control node** (a Linux shell) | `root@control:/#` or `root@control:~/lab#` |
| Browser checks | **Chrome** on Windows | `http://localhost:8091` and `http://localhost:8092` |

The Parts B to F commands are Linux commands. They **won't work in PowerShell**.

---

## Part A: Start the 3 machines (PowerShell)

```powershell
cd $HOME\Desktop
mkdir ansible-lab
cd ansible-lab
```

**Dockerfile.node** (the managed nodes: Ubuntu with an SSH server and Python):

```powershell
@'
FROM ubuntu:22.04
ENV DEBIAN_FRONTEND=noninteractive
RUN apt-get update && apt-get install -y openssh-server python3 sudo && mkdir -p /var/run/sshd && echo "root:ansible123" | chpasswd && sed -i "s/^#\?PermitRootLogin.*/PermitRootLogin yes/" /etc/ssh/sshd_config
EXPOSE 22 80
CMD ["/usr/sbin/sshd", "-D"]
'@ | Set-Content -Encoding ascii Dockerfile.node
```

**compose.yaml** (starts all three):

```powershell
@'
services:
  control:
    image: ubuntu:22.04
    container_name: control
    hostname: control
    command: sleep infinity
  node1:
    build:
      context: .
      dockerfile: Dockerfile.node
    container_name: node1
    hostname: node1
    ports:
      - "8091:80"
  node2:
    build:
      context: .
      dockerfile: Dockerfile.node
    container_name: node2
    hostname: node2
    ports:
      - "8092:80"
'@ | Set-Content -Encoding ascii compose.yaml
```

**Start them:**

```powershell
dir
docker compose up -d --build
docker compose ps
```

`dir` shows `compose.yaml` and `Dockerfile.node`, neither of them 0 bytes. The first build takes a few minutes. `docker compose ps` shows **control**, **node1** and **node2**, all **Up**.

📸 **Screenshot:** `docker compose ps` with the three containers.

---

## Part B: Install Ansible on the control node (Q6.1, 4 marks)

Go inside the control node:

```powershell
docker exec -it control bash
```

The prompt changes to `root@control:/#`. You're now in Linux. Run:

```bash
export DEBIAN_FRONTEND=noninteractive
apt update
apt install -y ansible sshpass curl
ansible --version
```

`sshpass` lets Ansible log in with a password. `ansible --version` prints the Ansible version and its config file path.

📸 **Screenshot:** `ansible --version`.

---

## Part C: Inventory and connectivity test (Q6.2, 5 marks)

Still inside the control node:

```bash
mkdir -p /root/lab && cd /root/lab

cat > inventory.ini << 'EOF'
[webservers]
node1 ansible_host=node1
node2 ansible_host=node2

[webservers:vars]
ansible_user=root
ansible_password=ansible123
ansible_python_interpreter=/usr/bin/python3
EOF

cat > ansible.cfg << 'EOF'
[defaults]
inventory = inventory.ini
host_key_checking = False
EOF

cat inventory.ini
ansible all -m ping
```

Both nodes reply in green:

```
node1 | SUCCESS => {
    "changed": false,
    "ping": "pong"
}
node2 | SUCCESS => {
    "changed": false,
    "ping": "pong"
}
```

| Line | Meaning |
| --- | --- |
| `[webservers]` | The group name used by the playbook |
| `node1 ansible_host=node1` | Host name in Ansible, and the address to connect to |
| `[webservers:vars]` | Settings for every host in the group |
| `ansible_user` / `ansible_password` | The SSH login |
| `inventory = inventory.ini` | Use this inventory without typing `-i` every time |
| `host_key_checking = False` | Don't stop to ask "Are you sure you want to continue connecting?" |

📸 **Screenshot:** `cat inventory.ini` and `ansible all -m ping`.

---

## Part D: Write install-packages.yml (Q6.3, 12 marks)

The custom web page:

```bash
cat > index.html << 'EOF'
<!DOCTYPE html>
<html>
<body style="font-family:Arial;text-align:center;margin-top:100px;">
  <h1>Deployed by Ansible</h1>
  <p>Your Name - Your Roll Number</p>
</body>
</html>
EOF
```

The playbook:

```bash
cat > install-packages.yml << 'EOF'
- name: Install and configure web servers
  hosts: webservers
  become: yes
  tasks:
    - name: Update apt cache
      apt:
        update_cache: yes
        cache_valid_time: 3600

    - name: Install packages
      apt:
        name:
          - apache2
          - curl
          - git
          - vim
        state: present

    - name: Ensure apache2 is started and enabled
      service:
        name: apache2
        state: started
        enabled: yes

    - name: Copy custom index.html
      copy:
        src: index.html
        dest: /var/www/html/index.html
EOF

ls
cat install-packages.yml
```

`ls` shows `ansible.cfg  index.html  install-packages.yml  inventory.ini`.

| Part | Module | What it does |
| --- | --- | --- |
| `hosts: webservers` | | Run on every host in the `[webservers]` group |
| `become: yes` | | Run the tasks with admin (sudo) rights |
| Update apt cache | `apt` | Same as `apt update`. `cache_valid_time: 3600` skips it if it ran in the last hour |
| Install packages | `apt` | Installs apache2, curl, git and vim. `state: present` means "make sure it's installed" |
| Start and enable apache2 | `service` | `started` = running now, `enabled` = starts on every boot |
| Copy custom index.html | `copy` | Copies `index.html` from the control node to `/var/www/html/` on each server |

📸 **Screenshot:** `cat install-packages.yml`.

---

## Part E: Run the playbook (Q6.4, 5 marks)

```bash
ansible-playbook install-packages.yml
```

The first run takes a few minutes, while both servers download Apache. Each task shows **changed** (yellow) or **ok** (green) for node1 and node2. It ends with:

```
PLAY RECAP *********************************************************************
node1   : ok=5    changed=4    unreachable=0    failed=0    skipped=0
node2   : ok=5    changed=4    unreachable=0    failed=0    skipped=0
```

`failed=0` for both nodes means it worked.

📸 **Screenshot:** the playbook output with PLAY RECAP.

---

## Part F: Check in the browser and prove idempotency (Q6.5, 4 marks)

**1. Browser.** On Windows, open in Chrome:

- **http://localhost:8091**, which is node1
- **http://localhost:8092**, which is node2

Both show **Deployed by Ansible**.

**2. The servers' own IP addresses.** Inside the control node:

```bash
ansible webservers -m command -a "hostname -I"
curl http://node1
curl http://node2
```

The first command prints each server's IP on the Docker network. `curl` prints the HTML of each server's page.

**3. Idempotency.** Run the same playbook again:

```bash
ansible-playbook install-packages.yml
```

This time every task shows **ok** and PLAY RECAP shows **changed=0**:

```
node1   : ok=5    changed=0    unreachable=0    failed=0    skipped=0
node2   : ok=5    changed=0    unreachable=0    failed=0    skipped=0
```

Everything is already in the correct state, so Ansible changes nothing. This is **idempotency**.

📸 **Screenshots:** both browser pages (address bar visible), `hostname -I`, and the second run with `changed=0`.

**4. Finish.** Leave the control node and stop the lab:

```bash
exit
```

```powershell
docker compose down
```

---

## Errors and fixes

These are the errors that came up while doing this practical, with the exact fix for each.

### Error 1: 'docker' is not recognized

```
docker : The term 'docker' is not recognized as the name of a cmdlet, function, script file, or operable program.
```

**Why:** Docker Desktop is installed for one user only (in `AppData\Local\Programs\DockerDesktop`), so a new PowerShell window doesn't know where `docker.exe` is. A `$env:Path` fix only lasts until that window is closed, so the error comes back in every new window.

**Fix for this window:**

```powershell
$env:Path += ";$env:LOCALAPPDATA\Programs\DockerDesktop\resources\bin"
docker --version
docker compose up -d --build
```

**Permanent fix:** run this once:

```powershell
[Environment]::SetEnvironmentVariable("Path", [Environment]::GetEnvironmentVariable("Path","User") + ";$env:LOCALAPPDATA\Programs\DockerDesktop\resources\bin", "User")
```

Then close PowerShell and open a **new** window. `docker --version` now works without the extra line. If it still fails, restart the PC once.

If `Test-Path "$env:LOCALAPPDATA\Programs\DockerDesktop\resources\bin\docker.exe"` says **False**, use `C:\Program Files\Docker\Docker\resources\bin` instead. If both are False, Docker Desktop isn't installed.

### Other errors

| Error | Where | Why | Fix |
| --- | --- | --- | --- |
| `error during connect` / `open //./pipe/docker_engine` | PowerShell | Docker Desktop isn't running | Open Docker Desktop and wait for **Engine running** |
| `no configuration file provided: not found` | PowerShell | Not in the `ansible-lab` folder | `cd $HOME\Desktop\ansible-lab` |
| `Set-Content` / `@'` is not recognized | PowerShell | You're in Command Prompt | Open **PowerShell** |
| `Error response from daemon: No such container: control` | PowerShell | The lab isn't running | Run `docker compose up -d` in `ansible-lab` |
| `bash: ansible: command not found` | Control node | Ansible isn't installed, or you're in the wrong container | Run Part B again inside `control` |
| `The term 'ansible' is not recognized` | PowerShell | You ran a Linux command in PowerShell | First run `docker exec -it control bash` |
| `to use the 'ssh' connection type with passwords, you must install the sshpass program` | Control node | `sshpass` is missing | `apt install -y sshpass` |
| `UNREACHABLE! ... Connection refused` | Control node | node1/node2 aren't running, or SSH hasn't started | In PowerShell: `docker compose ps`, then `docker compose up -d` |
| `UNREACHABLE! ... Could not resolve hostname node1` | Control node | Containers weren't started from the same `compose.yaml` | Run `docker compose down`, then `docker compose up -d --build` in `ansible-lab` |
| `Host key verification failed` | Control node | `ansible.cfg` isn't being used | `cd /root/lab` (it must contain `ansible.cfg`) |
| `Invalid/incorrect password` / `Permission denied` | Control node | Wrong password in `inventory.ini` | It must be `ansible_password=ansible123` |
| `Could not find or access 'index.html'` | Control node | `index.html` isn't in `/root/lab` | `cd /root/lab`, then create it again (Part D) |
| `found character that cannot start any token` / YAML error | Control node | A Tab or broken spacing in the playbook | Paste the whole `cat > install-packages.yml << 'EOF'` block again |
| A `cat > ... << 'EOF'` block seems stuck (prompt shows `>`) | Control node | The `EOF` line wasn't pasted | Type `EOF` and press **Enter** |
| Browser shows "This site can't be reached" on 8091/8092 | Chrome | The playbook hasn't run yet, or failed | Run Part E and check PLAY RECAP has `failed=0` |
| `port is already allocated` (8091/8092) | PowerShell | Another container uses the port | `docker ps`, then `docker stop <name>` |

**If you close the window by mistake:** open PowerShell, run `docker exec -it control bash`, then `cd /root/lab`. Ansible and all your files are still there, as long as you didn't run `docker compose down`.

---

## Viva points

| Question | Answer |
| --- | --- |
| What is Ansible? | An open-source, agentless automation tool for configuring servers, using SSH and YAML playbooks |
| Why "agentless"? | Managed nodes need only SSH and Python, with no Ansible software installed on them |
| Control node vs managed node? | The control node runs Ansible. Managed nodes are the servers it configures |
| What is an inventory? | A file listing managed hosts and their groups (here `[webservers]`) |
| What is a playbook? | A YAML file of plays and tasks to run on hosts |
| What is a task? | One step in a playbook that uses a module |
| What is a module? | A built-in unit of work, such as `apt` (packages), `service` (services) or `copy` (files) |
| What does `become: yes` do? | Runs tasks with admin (sudo) rights |
| What does `ansible all -m ping` do? | Checks Ansible can log in to every host and run Python. It's not a network ping |
| What is idempotency? | Running the playbook again makes no changes if the servers are already correct (`changed=0`) |
| Ad-hoc command vs playbook? | Ad-hoc (`ansible ... -m ...`) runs one task once. A playbook is a saved, repeatable set of tasks |
| Ansible vs shell scripts? | Ansible is idempotent, works on many servers at once, and describes the end state instead of every step |

## Conclusion

We set up one control node and two managed nodes, installed Ansible on the control node, and grouped the two servers as `[webservers]` in an inventory. `ansible all -m ping` confirmed the connection. The playbook `install-packages.yml` updated the apt cache, installed apache2, curl, git and vim, started and enabled Apache, and copied a custom `index.html` to both servers, which we checked in the browser. Running the playbook a second time gave `changed=0` on both servers, which proves Ansible is idempotent.
