# DevOps Viva Prep: Easy Concepts and Practicals 1–6

**LYCS DevOps Sem 7**

Easy explanations of the main DevOps ideas, then what each practical does and the viva questions you're likely to get, with short answers you can say out loud.

| Practical | Repo |
| --- | --- |
| 1. Git | https://github.com/show-star/aa1 |
| 2. Jenkins Freestyle + Maven | https://github.com/show-star/aa2 |
| 3. CI/CD pipeline | https://github.com/show-star/aa2 |
| 4. Docker basics | https://github.com/show-star/aa45 |
| 5. Docker Compose | https://github.com/show-star/aa45/tree/main/p5 |
| 6. Ansible | https://github.com/show-star/aa6 |

**Contents:** [The big picture](#the-big-picture-in-easy-words) · [Practical 1](#practical-1-version-control-with-git) · [Practical 2](#practical-2-jenkins-freestyle-job-with-maven) · [Practical 3](#practical-3-cicd-pipeline-with-jenkins-and-maven) · [Practical 4](#practical-4-docker-basics) · [Practical 5](#practical-5-docker-compose-multi-container-app) · [Practical 6](#practical-6-introduction-to-ansible) · [Rapid-fire questions](#rapid-fire-general-questions) · [Viva tips](#viva-tips)

---

## The big picture in easy words

**DevOps = Development + Operations.** Developers write code, and operations people run it on servers. Earlier they worked separately, so releasing software was slow and things broke. DevOps makes them work together and **automates** the steps from writing code to running it, so software goes out **faster, more often and with fewer mistakes**.

**Analogy: a restaurant kitchen.** The recipe book is the code (**Git** keeps every version of it). A head chef checks every dish before it leaves (**Jenkins** builds and tests every change). Food goes out in standard lunch boxes that work at any table (**Docker containers**). A manager makes sure every branch kitchen is set up the same way (**Ansible**). A supervisor keeps the right number of cooks working and replaces anyone who falls ill (**Kubernetes**). CCTV and order logs show what's happening (**monitoring and logging**).

### The DevOps lifecycle

| Stage | What happens | Tool in your practicals |
| --- | --- | --- |
| Plan and code | Write code and keep its history | Git (Practical 1) |
| Build | Turn code into a runnable file | Maven (Practicals 2, 3) |
| Test | Run automatic tests | JUnit via Maven, in Jenkins |
| Integrate | Build and test every change automatically | Jenkins (Practicals 2, 3) |
| Release and deploy | Put the app where it runs | Jenkins Deploy stage, Docker (Practicals 3, 4, 5) |
| Configure | Set up servers the same way every time | Ansible (Practical 6) |
| Operate and scale | Keep apps running, add copies when busy | Kubernetes |
| Monitor | Watch health and logs | Prometheus, Grafana, ELK |

### Key terms

| Term | Easy explanation | Example |
| --- | --- | --- |
| **Version control** | A system that remembers every version of your files, like unlimited Ctrl + Z with notes | Git |
| **Git** | The version control tool on your PC | `git commit` |
| **GitHub** | A website that stores Git repos online so teams can share them | github.com/show-star/aa1 |
| **CI (Continuous Integration)** | Every time someone pushes code, it is **automatically built and tested**, so bugs are caught the same day | Jenkins runs `mvn clean package` on each commit |
| **CD (Continuous Delivery)** | After CI, the app is **always ready to release**: one click puts it live | Package stage creates the `.jar` |
| **CD (Continuous Deployment)** | Goes one step further: every change that passes tests is **released automatically**, no click | Deploy stage copies the jar to `C:\deploy` |
| **CI/CD pipeline** | The automatic assembly line: Checkout → Build → Test → Package → Deploy | `cicd-pipeline` in Practical 3 |
| **Jenkins** | An automation server that runs pipelines | `http://localhost:8080` |
| **Maven** | A Java build tool. Reads `pom.xml`, downloads libraries, compiles, tests and packages | `mvn clean package` |
| **Virtual machine (VM)** | A whole computer (with its own OS) running inside another computer. Heavy and slow to start | VirtualBox |
| **Container** | A light, sealed box holding an app and everything it needs. Shares the host's OS, starts in seconds | `my-web-container` |
| **Docker** | The tool that builds and runs containers | `docker run` |
| **Image** | The template for a container, like a recipe or a class | `nginx`, `my-web-app:v1` |
| **Dockerfile** | The instructions to build an image | `FROM nginx:alpine` |
| **Docker Hub** | Online store of ready-made images | `docker pull nginx` |
| **Docker Compose** | Starts several containers together from one YAML file | Flask + MySQL in Practical 5 |
| **Volume** | Storage outside a container, so data survives when the container is deleted | `db_data` |
| **Configuration management** | Setting up many servers automatically and identically | Ansible |
| **Ansible** | Agentless tool that configures servers over SSH using YAML playbooks | `install-packages.yml` |
| **Infrastructure as Code (IaC)** | Describing servers and setup in files, instead of clicking by hand | Playbooks, Dockerfiles, `docker-compose.yml` |
| **Kubernetes (K8s)** | A system that runs and manages **many containers across many machines**: starts them, restarts failed ones, scales them, updates them with no downtime | `kubectl` |
| **Pod** | The smallest unit in Kubernetes: one or more containers running together | `my-pod` |
| **Deployment** | Tells Kubernetes "always keep N copies of this pod running" | `replicas: 3` |
| **Service** | Gives pods one stable address so users can reach them | NodePort `30080` |
| **Monitoring** | Watching numbers like CPU, memory and errors over time | Prometheus + Grafana |
| **Logging** | Collecting and searching what apps write to their logs | ELK: Elasticsearch, Logstash, Kibana |
| **YAML** | A simple text format for settings that uses **spaces for indentation** (never Tab) | Compose files, playbooks, K8s files |

### Container vs virtual machine

| | Container | Virtual machine |
| --- | --- | --- |
| Contains | App + its libraries | App + libraries + a whole operating system |
| Size | Megabytes | Gigabytes |
| Starts in | Seconds | Minutes |
| OS | Shares the host's kernel | Has its own full OS |
| Example | Docker container | VirtualBox, VMware |

**In one line:** a VM is like renting a whole house; a container is like renting a room in a shared house with its own lock.

### Docker vs Kubernetes

**Docker** builds and runs containers on **one machine**. **Kubernetes** manages **lots of containers across many machines**: if a container crashes it starts a new one, if traffic grows it adds copies, and it updates apps one copy at a time with no downtime. Docker is the worker; Kubernetes is the manager.

### Why companies use DevOps

- Faster releases: many times a day instead of once in months.
- Fewer bugs reach users, because every change is tested automatically.
- Same setup everywhere ("it works on my machine" problem solved by containers).
- Easy to scale and recover from failures.
- Teams share responsibility instead of blaming each other.

---

## Practical 1: Version control with Git

**In simple words:** we made a project folder, told Git to track it, saved versions of our files (commits), built a login form on a separate branch, joined it back into main (merge), then deliberately made two branches change the same line to see a merge conflict, and fixed it by hand.

**Analogy:** Git is like Google Docs version history, but for a whole folder. A **branch** is like making a photocopy of your notes to try an idea without spoiling the original. **Merging** is copying the good idea back into the original. A **conflict** is when you and a friend both changed the same sentence, so someone has to choose which one stays.

**What you did, step by step:** `git init` → set name/email → create 3 files → first commit → branch `feature-login` + login form → back to `main`, change `app.js` → merge (no conflict) → two branches change `style.css` line → conflict → fix → `git log --graph`.

### Viva questions

| Question | Answer |
| --- | --- |
| What is version control? | A system that records changes to files over time, so you can see history, go back to old versions and work in a team |
| What is Git? | A free, distributed version control system created by Linus Torvalds in 2005 |
| Why is Git called "distributed"? | Every developer has the full history on their own PC, not just on one central server |
| Git vs GitHub? | Git is the tool on your PC. GitHub is a website that hosts Git repos online for sharing and teamwork |
| What is a repository? | A project folder that Git tracks. Its history is stored in the hidden `.git` folder |
| What does `git init` do? | Creates a new, empty Git repository in the current folder |
| Why set `user.name` and `user.email`? | Every commit records who made it. These are labels, not a login |
| What is the staging area? | A waiting area where you put the changes you want in the next commit, using `git add` |
| What are the three states of a file? | Modified (changed), Staged (`git add`), Committed (`git commit`) |
| What is a commit? | A saved snapshot of the project with a message, author, time and a unique ID (hash) |
| What does `git status` show? | The current branch and which files are modified, staged or untracked |
| What is a branch? | An independent line of work. You can develop a feature without touching `main` |
| What does `git checkout -b feature-login` do? | Creates the branch `feature-login` and switches to it |
| What is `main`? | The default, main branch of the repo (older repos call it `master`) |
| What is a merge? | Combining the changes from one branch into another |
| Why did the first merge have no conflict? | The branches changed different files (`index.html` and `app.js`) |
| What is a merge conflict? | Two branches changed the same line of the same file, so Git can't decide which to keep |
| What do `<<<<<<<`, `=======`, `>>>>>>>` mean? | Start of the current branch's version, the divider, and the end of the incoming branch's version |
| How do you resolve a conflict? | Edit the file to the final version, delete the markers, then `git add` and `git commit` |
| What does `git commit -am` do? | Stages all changed tracked files and commits them in one step |
| What does `git log --graph --oneline --all` show? | All commits, one per line, drawn as a tree showing where branches split and join |
| What is `HEAD`? | A pointer to the commit/branch you're currently on |
| `git merge` vs `git rebase`? | Merge joins histories with a merge commit. Rebase replays your commits on top of the other branch for a straight-line history |
| `git clone` vs `git pull` vs `git push`? | Clone copies a remote repo. Pull downloads new commits. Push uploads your commits |
| What is `.gitignore`? | A file listing files Git should ignore, such as build output |

---

## Practical 2: Jenkins Freestyle job with Maven

**In simple words:** we installed Jenkins, connected it to a small Java project on GitHub, and told it to check GitHub every 2 minutes. Whenever there's a new commit, Jenkins downloads the code and runs Maven, which compiles it, runs the test and makes a `.jar` file. That automatic "build and test on every change" is **Continuous Integration**.

**Analogy:** Jenkins is a robot assistant that keeps watching the project's notebook (GitHub). The moment someone writes something new, it checks the work (compile + test) and reports pass (green) or fail (red), so mistakes are caught immediately.

**What you did:** install Java + Jenkins → unlock, plugins, admin user → add `Maven3` in Tools → Freestyle job `java-build-job` → Git URL + `*/main` → Poll SCM `H/2 * * * *` → Maven goals `clean package` → Build Now → BUILD SUCCESS → commit on GitHub → build #2 starts by itself.

### Viva questions

| Question | Answer |
| --- | --- |
| What is Jenkins? | An open-source automation server, written in Java, used for CI/CD: it builds, tests and deploys code automatically |
| What is Continuous Integration? | Developers push code often, and every push is automatically built and tested, so bugs are found early |
| Why does Jenkins need Java? | Jenkins itself is a Java program |
| Which port does Jenkins use? | 8080 by default (`http://localhost:8080`) |
| Where is the first admin password? | In `initialAdminPassword` under `C:\ProgramData\Jenkins\.jenkins\secrets` |
| What are plugins? | Add-ons that give Jenkins features, such as Git, Pipeline and Maven support |
| What is a Freestyle job? | The simplest Jenkins job, set up by filling in forms (source code, triggers, build steps) |
| What is Maven? | A build tool for Java that manages libraries and compiles, tests and packages the project |
| What is `pom.xml`? | Project Object Model: Maven's file with the project name, version, Java version and dependencies |
| What is a dependency? | A library the project needs. Here, JUnit for testing |
| What does `mvn clean package` do? | `clean` deletes old output. `package` compiles, runs tests and creates the `.jar` in `target/` |
| Name Maven's lifecycle phases | validate, compile, test, package, verify, install, deploy |
| What is JUnit? | A Java testing framework. `AppTest` checks that `add(2, 3)` returns 5 |
| Why add `Maven3` in Manage Jenkins → Tools? | So Jenkins downloads and uses Maven itself. Leaving it on (Default) gave the error `'mvn' is not recognized` |
| What is SCM? | Source Code Management, such as Git |
| What is Poll SCM? | Jenkins checks the repo on a schedule and builds only if there's a new commit |
| What does `H/2 * * * *` mean? | Every 2 minutes. The five fields are minute, hour, day, month, weekday. `H` spreads jobs out |
| Poll SCM vs webhook? | Polling: Jenkins asks GitHub regularly. Webhook: GitHub tells Jenkins instantly. Webhooks can't reach `localhost`, so we used polling |
| What is the Console Output? | The full log of a build. It ends with `Finished: SUCCESS` or `FAILURE` |
| What is a workspace? | The folder where Jenkins downloads and builds the code for a job |
| What happens if a test fails? | Maven stops with BUILD FAILURE and the Jenkins build turns red |
| What does "Started by an SCM change" mean? | The build was triggered automatically by a new commit, not by clicking Build Now |

---

## Practical 3: CI/CD pipeline with Jenkins and Maven

**In simple words:** instead of filling in forms like in Practical 2, we wrote the whole build process as code in a file called **Jenkinsfile**, kept in GitHub with the project. Jenkins reads it and runs five stages one after another: **Checkout → Build → Test → Package → Deploy**. Each stage shows as a green or red box in Stage View. Deploy copies the finished `.jar` to `C:\deploy`. A commit on GitHub starts the whole pipeline automatically.

**Analogy:** a factory assembly line. Raw material (code) goes in at one end, passes through stations (stages), each station checks or adds something, and a finished product (the deployed app) comes out the other end. If one station fails, the line stops there.

**What you did:** Jenkinsfile in the repo → Pipeline job `cicd-pipeline` → Pipeline script from SCM, `*/main`, Script Path `Jenkinsfile` → Build Now → 5 green stages → jar in `C:\deploy` → commit → build #2 starts by itself.

### Viva questions

| Question | Answer |
| --- | --- |
| What is CI/CD? | CI: every change is built and tested automatically. CD: the tested app is automatically delivered or deployed |
| Continuous Delivery vs Continuous Deployment? | Delivery: always ready to release, but a person clicks to release. Deployment: every passing change goes live automatically |
| What is a CI/CD pipeline? | An automated series of stages that takes code from commit to deployment |
| What is a Jenkins Pipeline job? | A job whose steps are defined as code in a Jenkinsfile, split into stages |
| What is a Jenkinsfile? | A text file in the Git repo that defines the pipeline. This is called *Pipeline as Code* |
| Why keep the pipeline in Git? | It gets version history, review and backup like the rest of the code, and is easy to reuse |
| Declarative vs scripted pipeline? | Declarative starts with `pipeline { }` and has a fixed, easy structure (we used this). Scripted starts with `node { }` and is plain Groovy code |
| What language is a Jenkinsfile? | Groovy |
| What does `agent any` mean? | Run on any available Jenkins machine |
| What does `tools { maven 'Maven3' }` do? | Uses the Maven installation named `Maven3` and puts `mvn` on the PATH |
| What does `checkout scm` do? | Downloads the code from the repo the job is configured with |
| What are `stages`, `stage` and `steps`? | `stages` holds all stages. Each `stage` is one phase. `steps` are the commands inside it |
| Name the five stages and what each does | Checkout (get code), Build (`mvn compile`), Test (`mvn test`), Package (create jar), Deploy (copy jar to `C:\deploy`) |
| Why `bat` and not `sh`? | Jenkins runs on Windows, so commands run through `cmd.exe`. On Linux it would be `sh` |
| Why `\\` in paths? | In a Jenkinsfile, a backslash must be written as `\\` |
| What is Stage View? | The job page view showing each stage as a box with its status and time |
| How was the pipeline triggered automatically? | `triggers { pollSCM('H/2 * * * *') }` checks GitHub every 2 minutes |
| Why must build #1 run manually first? | The trigger inside the Jenkinsfile is registered only after Jenkins reads the file once |
| Freestyle job vs Pipeline job? | Freestyle: forms, no stages, settings stored only in Jenkins. Pipeline: code in Git, stages, easy to version and reuse |
| What happens if the Test stage fails? | The pipeline stops; Package and Deploy don't run, and that stage turns red |
| What is an artifact? | The output of a build, such as `java-maven-demo-1.0.jar` |

---

## Practical 4: Docker basics

**In simple words:** we downloaded a ready-made web server (nginx) from Docker Hub and ran it in a container on port 8080. Then we made our own "Hello Docker" web page, wrote a Dockerfile to package it with nginx into our own image `my-web-app:v1`, ran it on port 8081, and practised the everyday commands: list, logs, stop and remove.

**Analogy:** an **image** is like a cake recipe plus all ingredients packed in a box. A **container** is an actual cake baked from it: you can bake many cakes from one recipe. The **Dockerfile** is the written recipe. **Docker Hub** is a recipe shop. **Port mapping** is like a door number: knock on door 8081 of your PC and you reach door 80 inside the container.

**What you did:** `docker pull nginx` → run on 8080 → create `index.html` → Dockerfile (`FROM nginx:alpine`, `COPY`, `EXPOSE 80`) → `docker build -t my-web-app:v1 .` → run on 8081 → `ps`, `images`, `logs`, `stop`, `rm`.

**Errors you fixed (good to mention):** `docker` not recognized (added Docker's folder to PATH), missing `.` in `docker build`, port 8081 already used by `schema-registry` (stopped it), container name already in use (`docker rm -f`).

### Viva questions

| Question | Answer |
| --- | --- |
| What is Docker? | An open-source platform to build, ship and run applications inside containers |
| What is a container? | A lightweight, isolated package that runs an app with everything it needs, sharing the host's OS kernel |
| What is an image? | A read-only template used to create containers. Built from a Dockerfile or pulled from Docker Hub |
| Image vs container? | An image is the blueprint (like a class). A container is a running instance of it (like an object) |
| Container vs virtual machine? | A container shares the host OS kernel, so it's small (MBs) and starts in seconds. A VM has a full OS, so it's big (GBs) and slow |
| What problem does Docker solve? | "It works on my machine": the app runs the same everywhere because its environment is packaged with it |
| What is Docker Hub? | Docker's public online registry of images, such as `nginx`, `mysql`, `python` |
| What is a Dockerfile? | A text file with step-by-step instructions to build an image |
| What does `FROM nginx:alpine` do? | Uses the official nginx image based on Alpine Linux as the starting point |
| Why Alpine? | It's a very small Linux (about 5 MB), so the image is small and fast |
| What does `COPY` do? | Copies files from your PC into the image |
| What does `EXPOSE 80` do? | Documents that the container listens on port 80 |
| What does `docker build -t my-web-app:v1 .` do? | Builds an image named `my-web-app` with tag `v1` from the Dockerfile in the current folder (`.`) |
| What is a tag? | A version label for an image, such as `v1` or `latest` |
| What does `docker run -d --name x -p 8081:80 image` do? | Runs a container in the background (`-d`), names it `x`, and maps PC port 8081 to container port 80 |
| What is port mapping? | Connecting a port on the host to a port inside the container so it can be reached from the browser |
| `docker pull` vs `docker run`? | `pull` only downloads an image. `run` creates and starts a container (pulling first if needed) |
| What does `docker ps` show? | Running containers with their IDs, images, ports and names |
| `docker ps` vs `docker ps -a`? | `ps` shows only running containers. `-a` also shows stopped ones |
| What does `docker images` show? | All images stored on the PC |
| What does `docker logs` show? | The output of a container. For nginx, one line per web request |
| `docker stop` vs `docker rm`? | `stop` stops a running container. `rm` deletes a stopped container. `rm -f` stops and deletes |
| `docker rm` vs `docker rmi`? | `rm` removes a container. `rmi` removes an image |
| Why did we stop Jenkins for Step 1? | Jenkins and nginx both wanted port 8080, and only one program can use a port |
| What is Docker Engine? | The background service (daemon) that actually builds and runs containers |

---

## Practical 5: Docker Compose multi-container app

**In simple words:** real apps usually need more than one part, such as a website and a database. We put a Python (Flask) website in one container and a MySQL database in another, described both in one file (`docker-compose.yml`), and started them together with one command. The website read the `students` table from MySQL and showed it in the browser at port 8000. A **volume** kept the database's data safe, and we practised scaling, logs and shutting everything down.

**Analogy:** if `docker run` is ordering one dish, **Docker Compose** is ordering a full thali: one order (`docker compose up`) and every item arrives together, already arranged to work with each other. The two containers sit on the same "table" (network), so the website just calls the database by its name, `db`.

**What you did:** create `app.py`, `requirements.txt`, `Dockerfile`, `init.sql` → `docker-compose.yml` with `web` (port 8000:80) and `db` (`mysql:8.0`, volume, healthcheck) → `docker compose up -d --build` → `docker compose ps` → table at localhost:8000 → scale, logs → `docker compose down -v`.

**Errors you fixed:** `docker` not recognized in a new window (PATH fix), `empty compose file` (files were 0 bytes), `mapping values are not allowed` (Python code had been pasted into the compose file).

### Viva questions

| Question | Answer |
| --- | --- |
| What is Docker Compose? | A tool to define and run multi-container applications using one YAML file and one command |
| Why not just use `docker run` twice? | Compose keeps all settings in one file, starts containers in order, sets up the network, and is easy to repeat |
| What is a service in Compose? | One container definition, such as `web` or `db` |
| What file does Compose use? | `docker-compose.yml` (or `compose.yaml`), written in YAML |
| What is YAML? | A simple text format for configuration that uses indentation with spaces (never Tab) |
| How does the web app find MySQL? | Compose puts both on the same network, and the service name `db` works as a hostname |
| What does `build: .` mean? | Build this service's image from the Dockerfile in the current folder |
| What does `image: mysql:8.0` mean? | Use the official MySQL 8.0 image from Docker Hub |
| What does `ports: "8000:80"` mean? | Port 8000 on the PC goes to port 80 inside the web container |
| What are `MYSQL_ROOT_PASSWORD` and `MYSQL_DATABASE`? | Environment variables: the root password and the database MySQL creates on first start |
| What is a volume? Why use it? | Storage outside the container. Without it, the database data is lost when the container is removed |
| What does `init.sql` do? | MySQL runs it automatically on first start to create the `students` table and rows |
| What does `depends_on` do? | Starts `db` before `web`. With `condition: service_healthy`, `web` waits until MySQL is ready |
| What is a healthcheck? | A command Docker runs regularly to check the container is really working (`mysqladmin ping`) |
| Why `$$` in the healthcheck? | In Compose, `$$` means a literal `$`, so the variable is read inside the container |
| What does `docker compose up -d --build` do? | Builds images if needed and starts all services in the background |
| What does `docker compose ps` show? | The status of this project's containers, e.g. `Up (healthy)` |
| What does `docker compose logs web` show? | The output of the web service |
| What is scaling? | Running more or fewer copies of a service: `--scale web=0` or `=1` |
| Why couldn't we scale `web` above 1? | Only one container can use host port 8000 |
| `docker compose down` vs `down -v`? | `down` removes containers and the network. `-v` also deletes the volumes (the data) |
| What is Flask? | A small Python web framework used to build the web app |

---

## Practical 6: Introduction to Ansible

**In simple words:** imagine you have to set up Apache on 100 servers. Doing it by hand on each one is slow and mistakes creep in. With **Ansible**, you write the setup once in a playbook (a YAML list of tasks), and Ansible logs in to every server over SSH and does it for you, the same way everywhere. We used 1 control node and 2 managed nodes (as containers), listed the servers in an inventory, tested them with `ping`, ran a playbook that installed Apache, curl, git and vim and copied our web page, and ran it again to show nothing changed the second time (idempotency).

**Analogy:** the control node is a teacher with a checklist (playbook). The managed nodes are students. The teacher reads the class list (inventory) and makes sure each student has done every item. If a student already did an item, the teacher just ticks it as "ok" and doesn't make them do it again: that's idempotency.

**What you did:** 3 containers (`control`, `node1`, `node2`) → install Ansible on `control` → `inventory.ini` with `[webservers]` → `ansible all -m ping` (pong) → `install-packages.yml` → first run `changed=4` → pages at localhost:8091/8092 → second run `changed=0`.

### Viva questions

| Question | Answer |
| --- | --- |
| What is Ansible? | An open-source automation tool for configuration management, software installation and deployment |
| What is configuration management? | Keeping many servers set up correctly and identically, automatically |
| Why is Ansible "agentless"? | Managed nodes need only SSH and Python. Nothing extra is installed on them |
| How does Ansible connect to servers? | Over SSH (on Windows servers, WinRM) |
| What is a control node? | The machine where Ansible is installed and from which playbooks are run |
| What is a managed node? | A server that Ansible configures |
| Can Windows be a control node? | Not directly. It needs Linux, e.g. WSL or a Linux container, which is why we used containers |
| What is an inventory? | A file listing managed nodes and their groups, such as `[webservers]` |
| What is a host group? | A named set of hosts in the inventory, so one playbook can target all of them |
| What does `ansible all -m ping` do? | Checks Ansible can log in to every host and run Python. It returns `pong`. It's not a network ping |
| What is an ad-hoc command? | A one-off command: `ansible <hosts> -m <module> -a <args>` |
| What is a playbook? | A YAML file containing plays; each play runs tasks on a group of hosts |
| What is a task? | One step in a playbook, which calls a module |
| What is a module? | A built-in unit of work, such as `apt`, `service`, `copy`, `ping`, `command` |
| What does the `apt` module do? | Manages packages on Ubuntu/Debian: `update_cache`, install with `state: present` |
| What does the `service` module do? | Manages services: `state: started` runs it now, `enabled: yes` starts it on boot |
| What does the `copy` module do? | Copies a file from the control node to the managed nodes |
| What does `become: yes` do? | Runs tasks with admin (sudo) rights, which installing software needs |
| What is idempotency? | Running the playbook again gives the same result without making changes if the server is already correct (`changed=0`) |
| What do `ok`, `changed`, `failed`, `unreachable` mean? | ok: already correct. changed: Ansible changed something. failed: a task error. unreachable: couldn't connect |
| What is `ansible.cfg`? | Ansible's settings file. We set the inventory file and turned off SSH host key checking |
| What is `sshpass` for? | Lets Ansible log in with a password instead of SSH keys |
| Ansible vs shell scripts? | Ansible is idempotent, works on many servers at once, and describes the final state instead of every command |
| Ansible vs Puppet/Chef? | Ansible is agentless and uses YAML. Puppet and Chef need an agent on every server |
| What is a role? | A reusable, organised bundle of tasks, files and templates |

---

## Rapid-fire general questions

| Question | Answer |
| --- | --- |
| What is DevOps? | A culture and set of practices that joins development and operations and automates building, testing, releasing and running software |
| Main goals of DevOps? | Faster releases, fewer failures, quick recovery, and teamwork between dev and ops |
| Name some DevOps tools | Git, GitHub, Jenkins, Maven, Docker, Kubernetes, Ansible, Terraform, Prometheus, Grafana, ELK |
| What is Kubernetes? | An open-source system that automatically deploys, scales and manages containers across many machines (originally from Google) |
| Why is it called K8s? | There are 8 letters between the K and the s |
| What is a Pod? | The smallest unit in Kubernetes: one or more containers that run together |
| What is a Node? | A machine (physical or virtual) in a Kubernetes cluster that runs pods |
| What is a Cluster? | A group of nodes managed together by Kubernetes |
| What is a Deployment? | A Kubernetes object that keeps a set number of pod copies (replicas) running and handles updates |
| What is a Service? | Gives a stable address to a group of pods. Types: ClusterIP, NodePort, LoadBalancer |
| What is `kubectl`? | The command-line tool to control Kubernetes |
| What is a rolling update? | Replacing old pods with new ones a few at a time, so the app never goes down |
| What is a rollback? | Going back to the previous version (`kubectl rollout undo`) |
| What is scaling? | Changing the number of copies to match traffic. Horizontal = more copies; vertical = bigger machine |
| What is orchestration? | Automatically managing many containers: placing, starting, restarting, scaling and connecting them |
| What is Infrastructure as Code? | Describing servers and setup in files so they can be versioned and recreated automatically |
| What is monitoring? | Tracking metrics like CPU, memory, requests and errors over time (Prometheus + Grafana) |
| What is the ELK stack? | Elasticsearch (stores and searches logs), Logstash (collects and processes logs), Kibana (shows logs and charts) |
| What is microservices architecture? | Building an app as many small independent services, each in its own container |
| What is a registry? | A store for images, e.g. Docker Hub |
| What is SSH? | Secure Shell: an encrypted way to log in to and run commands on a remote machine |
| What is a port? | A numbered "door" on a computer for network traffic, e.g. 80 (HTTP), 8080 (Jenkins), 3306 (MySQL), 22 (SSH) |
| What is localhost? | Your own computer (IP 127.0.0.1) |
| What is YAML used for here? | Docker Compose files, Ansible playbooks and Kubernetes files |

### Which tool does what (quick match)

| If they ask about... | Answer with |
| --- | --- |
| Tracking code changes | Git |
| Hosting code online | GitHub |
| Building Java code | Maven |
| Automating build/test/deploy | Jenkins |
| Packaging an app to run anywhere | Docker |
| Running several containers together | Docker Compose |
| Setting up many servers the same way | Ansible |
| Managing containers at large scale | Kubernetes |
| Watching performance | Prometheus + Grafana |
| Searching logs | ELK stack |

---

## Viva tips

- **Start with one line, then add detail.** Example: "Docker is a tool that runs apps in containers. A container packages the app with everything it needs, so it runs the same on any machine."
- **Use your own practical as the example.** "In my practical, Jenkins built my Maven project every time I committed to GitHub."
- **Know your errors.** Examiners like hearing what went wrong and how you fixed it: `'mvn' is not recognized` (chose Maven3), port already allocated (stopped the other container), `docker` not recognized (fixed PATH).
- **Know the port numbers you used:** Jenkins 8080, nginx 8080, my-web-app 8081, Compose web 8000, MySQL 3306, Ansible nodes 8091/8092.
- **If you don't know an answer,** say what you do know about the topic and link it to your practical rather than guessing.
- **Learn the five-line flow by heart:** Git stores code → Jenkins + Maven build and test it (CI) → the pipeline deploys it (CD) → Docker packages it → Ansible and Kubernetes run and manage it on servers.
