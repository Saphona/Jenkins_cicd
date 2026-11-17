# Jenkins Pipeline for Java based application using Maven, SonarQube, Argo CD, Helm and Kubernetes

<img width="1774" height="887" alt="image" src="https://github.com/user-attachments/assets/8b1faa46-6a6b-4773-8229-abb4e7c8af9f" />


Here are the step-by-step details to set up an end-to-end Jenkins pipeline for a Java application using SonarQube, Argo CD, Helm, and Kubernetes:

Prerequisites:

   -  Java application code hosted on a Git repository
   -   Jenkins server
   -  Kubernetes cluster
   -  Argo CD



This end-to-end Jenkins pipeline will automate the entire CI/CD process for a Java application, from code checkout to production deployment, using popular tools like SonarQube, Argo CD, Helm, and Kubernetes.



# Problems I Ran Into (and How They Got Fixed)

## Project: Installing Jenkins on AWS EC2

### Environment
- AWS EC2
- Ubuntu 26.04
- Initial Java version: 17
- Disk space: 6.7 GB
- Installation method: APT

---

## 1. Jenkins Was Not Opening

**Problem:** Browser showed `ERR_CONNECTION_REFUSED` when accessing port 8080.

**What happened:** Jenkins was not installed, so there was no Jenkins service running.

**How I identified it:**

```bash
sudo systemctl status jenkins
dpkg -l | grep jenkins
```

**Lesson:** Before checking AWS networking, first verify that the application is installed and running.

---

## 2. Jenkins Package Was Not Available

**Problem:** APT showed:

```text
Installed: (none)
Candidate: (none)
```

**What happened:** The Jenkins repository was configured, but APT could not access its trusted package information.

**Lesson:** Adding a repository does not mean its packages are ready to install. Always run `sudo apt update` and verify package availability.

---

## 3. Jenkins GPG Key Error

**Problem:**

```text
NO_PUBKEY 7198F4B714ABFC68
```

**What happened:** Ubuntu could not verify the Jenkins repository because the required public signing key was missing.

**How I fixed it:** Downloaded the updated official Jenkins signing key and configured the Jenkins LTS repository.

**Lesson:** Package repositories use signing keys to prove that software comes from a trusted source. Always use the current official repository instructions.

---

## 4. Java Version Compatibility

**Problem:** My server already had Java 17 installed, but the Jenkins version I planned to install required a newer supported Java version.

**What happened:** I installed Java without checking Jenkins' runtime requirements first.

**Important lesson:**

Different Jenkins versions support different Java versions. The required Java version also depends on the Jenkins release.

For example:
- An older Jenkins release might support Java 17.
- A newer Jenkins release might require Java 21 or later.

Always check the Java compatibility requirements for the exact Jenkins version before installing.

Useful command:

```bash
java -version
```

**Avoid:** Assuming that having Java installed means you have the correct Java version.

---

## 5. Disk Space Management

**Problem:** My EC2 root volume was only 6.7 GB.

**What I learned:** Jenkins uses storage for plugins, workspaces, build logs, artifacts and downloaded dependencies.

Useful commands:

```bash
df -h
sudo du -sh /var/lib/jenkins
sudo apt clean
sudo apt autoremove --dry-run
```

**Avoid:** Deleting random system files to free up space. Check what is consuming storage first.

---

# What I Should Have Done From the Beginning

Before installing any application on a Linux server:

1. **Check the OS:** Is my Ubuntu version supported?
2. **Check dependencies:** Which Java or other runtime version does this application require?
3. **Check compatibility:** Does the application version support my OS and runtime?
4. **Follow official documentation:** Use the current installation instructions, not outdated tutorials.
5. **Configure repositories:** Add the correct repository and signing key.
6. **Update APT:** Run `sudo apt update` and check for errors.
7. **Verify availability:** Run `apt-cache policy jenkins`.
8. **Install:** Install only after the prerequisites are satisfied.
9. **Verify:** Check the package, service and listening port.
10. **Configure networking:** Open the required AWS security group port.

---

# Key Takeaways

- Always check application version compatibility before installing.
- Different application releases can have different runtime requirements.
- Never blindly copy installation commands from old tutorials.
- Understand what APT repositories and GPG keys do.
- Verify installation before troubleshooting the application.
- Check disk space before deploying applications.
- Diagnose the actual problem before changing configurations.

**Biggest lesson:** A successful installation starts with checking compatibility and prerequisites, not immediately running installation commands.


# Problems I Ran Into (and How They Got Fixed) pt2

## 1. Permission Error While Downloading SonarQube

- **Problem:** I couldn't download the SonarQube ZIP file after switching to the `sonarqube` user.
- **Error:**
  ```text
  sudo: I'm sorry sonarqube. I'm afraid I can't do that
  ```
- **Cause:** I was logged in as the `sonarqube` user, which did not have sudo privileges.
- **Fix:** Switched back to root and downloaded the file from `/opt`.

  ```bash
  exit
  cd /opt

  wget https://binaries.sonarsource.com/Distribution/sonarqube/sonarqube-26.11.0.134175.zip
  ```

- **Lesson:** Use root or an account with sudo privileges for installation tasks. Use the dedicated application user to run SonarQube.

## 2. Understanding Why SonarQube Was Installed in `/opt`

- **Problem:** I wasn't sure why the installation directory was `/opt`.
- **Observation:** The installation instructions used `/opt/sonarqube`.
- **Cause:** I wasn't familiar with the Linux filesystem structure.
- **Fix:** Learned that `/opt` is commonly used for manually installed third-party applications.

  ```bash
  cd /opt
  ```

- **Lesson:** Understand the purpose of Linux directories before installing applications. `/opt` is a standard location, not a compulsory requirement.

## 3. Understanding the `chown -R` Command

- **Problem:** I didn't understand why ownership needed to be changed recursively.
- **Command:**
  ```bash
  chown -R sonarqube:sonarqube /opt/sonarqube
  ```
- **Cause:** SonarQube was downloaded and extracted as root, but the application needed to run under its dedicated `sonarqube` user.
- **Fix:** Changed ownership of the installation directory and its contents to the SonarQube user and group.

- **Lesson:**
  - `chown` changes ownership.
  - `-R` means recursive.
  - `sonarqube:sonarqube` represents the owner and group.

## 4. Understanding the `chmod -R` Command

- **Problem:** I didn't understand why `-R` was required in the permissions command.
- **Command:**
  ```bash
  chmod -R 775 /opt/sonarqube
  ```
- **Cause:** Without `-R`, permissions would only change on the specified directory, not its contents.
- **Fix:** Learned that recursive permission changes apply to subdirectories and files as well.

- **Lesson:**
  - `chmod` changes permissions.
  - `-R` means recursive.
  - `775` means owner and group have read, write and execute permissions; others have read and execute permissions.

  Note: Applying `775` recursively is broad. A more restrictive, file-specific permission setup is preferable for a real server.

## 5. SonarQube ZIP Extraction

- **Problem:** I thought the extraction process was stuck.
- **Observation:** The terminal displayed multiple `creating:` and `inflating:` messages.
- **Cause:** The ZIP extraction was processing many files and directories.
- **Fix:** Waited until the terminal returned to the root prompt:

  ```text
  root@ip-172-31-36-241:/opt#
  ```

- **Lesson:** ZIP extraction can produce a lot of output. When the shell prompt returns without an error, the command has finished.

## 6. Choosing the SonarQube Token for Jenkins

- **Problem:** I wasn't sure which token type to select when integrating SonarQube with Jenkins.
- **Observation:** The available options were:
  - Global Analysis Token
  - User Token
- **Cause:** I wasn't familiar with the different token types.
- **Fix:** Selected **Global Analysis Token** for the Jenkins CI/CD integration.
- **Lesson:** Use an appropriate analysis token for CI/CD tools and store it securely in Jenkins Credentials rather than hardcoding it in a Jenkinsfile.

---

### What I Should Have Done From the Beginning

1. Launch an EC2 instance with sufficient RAM, CPU and disk space.
2. Configure the required security group rules, including SSH and SonarQube port 9000 restricted to my IP.
3. Update Ubuntu and install the required Java version and utilities.
4. Create a dedicated `sonarqube` user.
5. Switch to root before downloading and extracting the SonarQube binary.
6. Install SonarQube under `/opt/sonarqube`.
7. Change ownership to the dedicated SonarQube user.
8. Configure the required Linux kernel settings.
9. Switch to the `sonarqube` user and start the application.
10. Check the service status and logs before accessing it through the browser.

### Key Takeaways

- **Linux users and permissions matter:** Root performs administrative tasks, while dedicated users run applications.
- **Understand Linux directories:** `/opt` is commonly used for third-party applications.
- **Recursive commands:** `-R` applies changes to directories and their contents.
- **Ownership vs permissions:** `chown` changes who owns files; `chmod` changes what users can do with them.
- **Don't panic at terminal output:** Long extraction output does not necessarily indicate a problem.
- **Secure CI/CD credentials:** Store SonarQube tokens in Jenkins Credentials.
- **Always verify:** Check Java compatibility, kernel requirements, service status and application logs before troubleshooting further.

  # Problems I Ran Into (and How I Fixed Them) pt3

## 1. Kind Cluster Creation Failed Due to Port Conflict

- **Problem:** Kind could not create the Kubernetes cluster because port 8080 was already occupied.
- **Error:**
  ```text
  ports are not available:
  exposing port TCP 0.0.0.0:8080
  bind: Only one usage of each socket address is normally permitted.
  ```
- **Cause:** Another application was already listening on Windows port 8080.
- **Investigation:**
  ```powershell
  netstat -ano | findstr :8080
  tasklist /FI "PID eq 4728"
  ```
  Identified `httpd.exe` (Apache HTTP Server), PID 4728.
- **Fix:** Proposed changing Kind's host port mappings to 8090 and 8445 instead of 8080 and 8443.
- **Lesson:** Always check host port availability before configuring Docker port mappings. Different applications cannot normally share the same host IP and port.

## 2. Kubernetes API Server Became Unreachable

- **Problem:** OLM installation failed because Kubernetes resources could not be validated.
- **Error:**
  ```text
  Get "https://127.0.0.1:64969/openapi/v2":
  dial tcp 127.0.0.1:64969:
  connectex: No connection could be made
  ```
- **Cause:** The Kind cluster was unavailable after cluster deletion and unsuccessful recreation attempts.
- **Investigation:**
  ```bash
  kind get clusters
  kubectl config current-context
  kubectl cluster-info
  kubectl get nodes
  ```
- **Fix:** Recreate the Kind cluster successfully and verify API server connectivity before installing Kubernetes components.
- **Lesson:** Kubernetes installations depend on a healthy API server. Verify cluster health before deploying operators or applications.

## 3. Argo CD Operator CSV Was Not Created

- **Problem:** The Argo CD Operator subscription was created, but no CSV appeared in the operators namespace.
- **Observation:**
  ```text
  subscription.operators.coreos.com/my-argocd-operator created

  OLM is already installed in olm namespace.

  kubectl get csv -n operators
  No resources found in operators namespace.
  ```
- **Cause:** Creating a Subscription does not mean the Operator installation is complete. OLM still needs to resolve the subscription, create an InstallPlan and install the Operator.
- **Investigation:** OLM pods were mostly running, but the OperatorHub catalog pod showed `0/1 Running`.
- **Next troubleshooting commands:**
  ```bash
  kubectl get subscription -n operators
  kubectl describe subscription my-argocd-operator -n operators
  kubectl get installplan -n operators
  kubectl get catalogsources -n olm
  ```
- **Lesson:** Understand the OLM installation sequence: CatalogSource → Subscription → InstallPlan → CSV → Operator. Check each stage instead of assuming a successful Subscription creation means installation is complete.

## 4. Understanding Docker Socket Mounting in Jenkins

- **Problem:** Needed to understand how a Jenkins Docker agent can execute Docker commands using the host's Docker daemon.
- **Configuration:**
  ```groovy
  agent {
    docker {
      image 'abhishekf5/maven-abhishek-docker-agent:v1'
      args '--user root -v /var/run/docker.sock:/var/run/docker.sock'
    }
  }
  ```
- **Cause:** A Docker agent container does not automatically have access to the host Docker daemon.
- **Explanation:** The socket mount exposes the host's Docker communication socket inside the Jenkins agent.
  ```bash
  -v /var/run/docker.sock:/var/run/docker.sock
  ```
  Docker CLI commands inside the agent can now communicate with the host Docker daemon.
- **Lesson:** Docker CLI is the client, while Docker daemon performs the actual operations. Socket mounting is powerful and introduces security risks because it grants extensive control over the host.

## 5. VmmemWSL Consuming Windows Memory

- **Problem:** VmmemWSL was consuming significant RAM while running Docker Desktop and Kind Kubernetes.
- **Observation:** VmmemWSL appeared in Windows Task Manager.
- **Cause:** WSL2 manages the Linux environment used by Docker Desktop. Running Kubernetes nodes, containers and other services increases its memory usage.
- **Suggested Fix:** Configure WSL2 resource limits:
  ```powershell
  notepad $env:USERPROFILE\.wslconfig
  ```
  Example:
  ```ini
  [wsl2]
  memory=4GB
  processors=2
  swap=2GB
  ```
  Shut down WSL2 when finished:
  ```powershell
  wsl --shutdown
  ```
- **Lesson:** Manage WSL2 resources when running local Kubernetes environments. Stop Docker and WSL2 when the development environment is no longer needed.

---

### What I Should Have Done From the Beginning

1. Check Docker Desktop and WSL2 resource availability.
2. Check whether required host ports are available.
3. Prepare the Kind configuration with control-plane and worker nodes and appropriate port mappings.
4. Create the cluster and verify:
   ```bash
   kind get clusters
   kubectl get nodes -o wide
   kubectl cluster-info
   ```
5. Confirm the correct Kubernetes context.
6. Install OLM and verify all its components are Ready.
7. Create the Argo CD Operator Subscription.
8. Verify CatalogSource, InstallPlan and CSV status before proceeding with Argo CD deployment.

### Key Takeaways

- **Kind:** Runs Kubernetes nodes as Docker containers.
- **Port mapping:** Connects Windows host ports to ports inside Kind nodes.
- **Kubernetes API:** A healthy API server is required for Kubernetes operations.
- **OLM:** Subscription creation and Operator installation are separate stages.
- **Docker socket:** Allows Jenkins agents to communicate with the host Docker daemon.
- **WSL2:** Docker Desktop and local Kubernetes workloads consume WSL2 memory.
- **Troubleshooting:** Identify the actual root cause before changing configurations or reinstalling components.

