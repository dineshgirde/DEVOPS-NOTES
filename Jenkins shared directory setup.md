# Jenkins Shared Directory Setup (NFS on AWS)

This guide explains how to create a **shared directory** that multiple Jenkins machines (master + agents) can access, using **NFS** on AWS EC2.

## What is this for?

When you have more than one Jenkins machine (e.g. one server + one or more agents), each machine normally has its own separate workspace. A shared directory lets all machines read/write the **same folder**, so one job's output can be used by another job without copying files manually.

---

## Quick Step-by-Step (What to install where)

| # | Step | Run on |
|---|---|---|
| 1 | Launch 2 EC2 instances (`nfs-server`, `nfs-client`), same VPC/subnet | AWS Console |
| 2 | SSH into both machines | Your terminal |
| 3 | Install `nfs-kernel-server` (the "share" software) | **nfs-server only** |
| 4 | Create shared folder: `sudo mkdir -p /mnt/jenkins-shared` + `sudo chmod 777 /mnt/jenkins-shared` | **nfs-server only** |
| 5 | Edit `/etc/exports`, add the export line | **nfs-server only** |
| 6 | Run `sudo exportfs -arv` + restart `nfs-kernel-server` | **nfs-server only** |
| 7 | Open port `2049` (NFS) in Security Group | AWS Console |
| 8 | Install `nfs-common` (the "connect" software) | **nfs-client only** |
| 9 | Create mount point + run `mount` command | **nfs-client only** |
| 10 | Test: write a file on server, read it on client | **Both** |

**Key point:** `nfs-kernel-server` and `nfs-common` are two different packages — the server needs one, the client needs the other. You don't install both on both machines.

## Requirements

- 2 AWS EC2 instances (Ubuntu), in the **same VPC and subnet**
  - `nfs-server` → will host Jenkins + the shared folder
  - `nfs-client` → will act as a second machine/agent that accesses the shared folder
- SSH access to both

---

## Step 1: Launch 2 EC2 instances

- Go to AWS Console → EC2 → Launch Instance
- Launch two Ubuntu instances: `nfs-server` and `nfs-client`
- Make sure both are in the same VPC and subnet

## Step 2: Open the NFS port in Security Group

- Go to `nfs-server` instance → Security tab → Security Group → Inbound Rules → Add Rule
  - Type: `NFS`
  - Port: `2049`
  - Source: `nfs-client`'s Security Group (or its private IP)

## Step 3: SSH into `nfs-server` and install Jenkins (if not already installed)

```bash
ssh -i your-key.pem ubuntu@nfs-server-public-ip

sudo apt update
sudo apt install openjdk-17-jre -y
curl -fsSL https://pkg.jenkins.io/debian-stable/jenkins.io-2023.key | sudo tee /usr/share/keyrings/jenkins-keyring.asc > /dev/null
echo deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc] https://pkg.jenkins.io/debian-stable binary/ | sudo tee /etc/apt/sources.list.d/jenkins.list > /dev/null
sudo apt update
sudo apt install jenkins -y
sudo systemctl start jenkins
sudo systemctl enable jenkins
```

## Step 4: Install NFS server and create the shared folder

```bash
sudo apt install nfs-kernel-server -y
sudo mkdir -p /mnt/jenkins-shared
sudo chown jenkins:jenkins /mnt/jenkins-shared
sudo chmod 775 /mnt/jenkins-shared
```

## Step 5: Share the folder (exports file)

```bash
sudo nano /etc/exports
```

Add this line (replace `10.0.0.0/16` with your actual VPC CIDR):

```
/mnt/jenkins-shared 10.0.0.0/16(rw,sync,no_subtree_check,no_root_squash)
```

Save and exit (`Ctrl+O`, `Enter`, `Ctrl+X`), then apply changes:

```bash
sudo exportfs -arv
sudo systemctl restart nfs-kernel-server
```

## Step 6: SSH into `nfs-client` and mount the shared folder

```bash
ssh -i your-key.pem ubuntu@nfs-client-public-ip

sudo apt install nfs-common -y
sudo mkdir -p /mnt/jenkins-shared
sudo mount NFS_SERVER_PRIVATE_IP:/mnt/jenkins-shared /mnt/jenkins-shared
```

Replace `NFS_SERVER_PRIVATE_IP` with the **private IP** of `nfs-server` (e.g. `10.0.1.5`).

## Step 7: Test that it works

On `nfs-server`:

```bash
echo "hello from server" > /mnt/jenkins-shared/test.txt
```

On `nfs-client`:

```bash
cat /mnt/jenkins-shared/test.txt
```

If you see `hello from server`, the shared directory is working correctly.

## Step 8: Use it in Jenkins

### Option A: Freestyle Job (using Jenkins UI)

1. Open Jenkins in your browser: `http://nfs-server-public-ip:8080`
2. Click **New Item** → enter a name (e.g. `test-shared-dir`) → select **Freestyle project** → **OK**
3. Scroll to **Build Steps** → click **Add build step** → **Execute shell**
4. In the text box, add:
   ```bash
   echo "Build output from $BUILD_NUMBER" > /mnt/jenkins-shared/output.txt
   cat /mnt/jenkins-shared/output.txt
   ```
5. Click **Save**, then click **Build Now**
6. Check **Console Output** — you should see the file content printed

### Option B: Pipeline Job (using a Jenkinsfile)

1. Open Jenkins → **New Item** → enter a name → select **Pipeline** → **OK**
2. Scroll to the **Pipeline** section, choose **Pipeline script**, and paste this:

```groovy
pipeline {
    agent any
    stages {
        stage('Write to Shared Directory') {
            steps {
                sh '''
                    echo "Build $BUILD_NUMBER output" > /mnt/jenkins-shared/output.txt
                    cat /mnt/jenkins-shared/output.txt
                '''
            }
        }
        stage('Read from Shared Directory') {
            steps {
                sh '''
                    echo "Reading shared file:"
                    cat /mnt/jenkins-shared/output.txt
                '''
            }
        }
    }
}
```

3. Click **Save**, then **Build Now**
4. Check **Console Output** — both stages should show the same file content

### Why this matters

- If this pipeline runs on **Agent-1**, and another job runs on **Agent-2** which also has `/mnt/jenkins-shared` mounted, Agent-2 can read the exact same `output.txt` file — no manual copying needed.
- This is useful for passing build artifacts (jars, configs, logs) between jobs that run on different machines.

### Option C: Real 2-Agent Pipeline (data moving between agents during build)

This shows how data actually flows between **two different agents** during a single pipeline run, using the shared directory.

**First, set up 2 Jenkins agents:**

1. In Jenkins UI → **Manage Jenkins** → **Nodes** → **New Node**
2. Create a node named `agent-1` (this can be the `nfs-server` machine itself, or another EC2 with the shared folder mounted)
3. Create a second node named `agent-2` (this must be the `nfs-client` machine, with `/mnt/jenkins-shared` mounted)
4. Connect both agents to Jenkins (via SSH or JNLP, using the agent connection instructions Jenkins gives you)

**Now use this Jenkinsfile:**

```groovy
pipeline {
    agent none
    stages {
        stage('Build on Agent 1') {
            agent { label 'agent-1' }
            steps {
                sh '''
                    echo "Building on agent-1..."
                    echo "This is build output" > /mnt/jenkins-shared/output.jar
                    echo "File written by agent-1:"
                    cat /mnt/jenkins-shared/output.jar
                '''
            }
        }
        stage('Use output on Agent 2') {
            agent { label 'agent-2' }
            steps {
                sh '''
                    echo "Now on agent-2, checking shared folder..."
                    cat /mnt/jenkins-shared/output.jar
                '''
            }
        }
    }
}
```

**What happens step by step during the build:**

| Step | What you'll see in Console Output |
|---|---|
| Stage 1 runs on `agent-1` | It writes `output.jar` into `/mnt/jenkins-shared/` (physically saved on the NFS server disk) |
| Jenkins moves to Stage 2 | No file is copied or transferred by Jenkins itself |
| Stage 2 runs on `agent-2` | It reads `/mnt/jenkins-shared/output.jar` — same file, same content, because the folder is mounted from the same NFS server |

If both agents did **not** share the same NFS folder, Stage 2 would fail with a "file not found" error — because `agent-2` would have no idea a file was written on `agent-1`.

**This is the actual proof it's working:** the file only exists once, on the server. Both agents just have a "window" into it.

### ⚠️ Important: Always use the full path in your script

Jenkins does **not** automatically know where your shared folder is. It only writes/reads files at the **exact path you give it** in the script. If you give the wrong path, the file gets created in Jenkins' local workspace instead — not in the shared NFS folder — and the other agent will never see it.

| Wrong (saves in local workspace, NOT shared) | Correct (saves in shared NFS folder) |
|---|---|
| `echo "data" > output.jar` | `echo "data" > /mnt/jenkins-shared/output.jar` |
| `./build/output.jar` | `/mnt/jenkins-shared/build/output.jar` |
| `cat output.jar` | `cat /mnt/jenkins-shared/output.jar` |

**Rule of thumb:** Always use the **full absolute path** (starting with `/mnt/jenkins-shared/...`) in every `sh` step that touches the shared folder. Never use a relative path (`./file` or just `file`) — otherwise the file stays local to that one agent and the other agent won't be able to find it.

### Verify from outside Jenkins too

You can always double check directly on the client machine (outside Jenkins) that the file Jenkins wrote is visible:

```bash
cat /mnt/jenkins-shared/output.txt
```

If it matches what Jenkins printed in Console Output, the shared directory setup is working end-to-end.

---

## Quick Command Reference (Shortcut)

| Task | Command |
|---|---|
| Install NFS server | `sudo apt install nfs-kernel-server -y` |
| Install NFS client | `sudo apt install nfs-common -y` |
| Edit exports file | `sudo nano /etc/exports` |
| Apply export changes | `sudo exportfs -arv` |
| Restart NFS service | `sudo systemctl restart nfs-kernel-server` |
| Mount shared folder | `sudo mount SERVER_IP:/path /mnt/local-folder` |
| Auto-mount on boot | Add entry in `/etc/fstab` |

---

## Notes

- Always use the **private IP**, not public IP, when mounting between EC2 instances.
- For production use, consider **AWS EFS** instead of manual NFS — it is managed, scalable, and easier to maintain.
