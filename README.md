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
