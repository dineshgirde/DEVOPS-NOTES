# Jenkins Workspace — Practical Guide (Hinglish)

Is guide mein sirf **Jenkins Workspace** samjhayenge — kya hota hai, kaha hota hai, aur practical mein kaise use/dekh/clean karte hain. Step by step, easy language mein.

---

## 1. Workspace kya hota hai (Simple mein)

Jab bhi tum Jenkins mein koi **job/build** chalate ho, Jenkins **khud automatically ek folder bana deta hai** — usi folder ko **"Workspace"** kehte hain.

Ye woh jagah hai jaha:
- Tumhara code GitHub se **download (checkout)** hota hai
- Build commands chalte hain (compile, test, package waghera)
- Temporary files, logs, output — sab kuch yahi banta/save hota hai

**Socho aise:** Workspace = tumhara apna kamra jaha tum akele baithke kaam karte ho. Har job ka apna alag kamra hota hai.

---

## 2. Workspace kaha hota hai (Default Path)

```
/var/lib/jenkins/workspace/JOB_NAME/
```

Example: agar tumhari job ka naam `my-app` hai, to workspace hoga:

```
/var/lib/jenkins/workspace/my-app/
```

Agar Jenkins ke **multiple agents** hain, to har agent pe **alag-alag** workspace banta hai (usi agent ke disk pe) — kyunki har agent apna kaam khud handle karta hai.

---

## 3. Practical — Workspace ko khud dekhna (Step by Step)

### Step 1: Jenkins mein ek simple job banao

1. Jenkins UI kholo: `http://your-jenkins-ip:8080`
2. **New Item** pe click karo
3. Naam do: `workspace-test`
4. **Freestyle project** select karo → **OK**

### Step 2: Build step mein workspace dekhne wala command daalo

1. Scroll karo **Build Steps** tak
2. **Add build step** → **Execute shell**
3. Ye likho:
   ```bash
   echo "Mera workspace path ye hai:"
   pwd
   echo "Is folder ke andar ye files hain:"
   ls -la
   ```
4. **Save** karo

### Step 3: Build chalao

1. **Build Now** pe click karo
2. Build history mein us build pe click karo
3. **Console Output** kholo

Tumhe dikhega kuch aisa:
```
Mera workspace path ye hai:
/var/lib/jenkins/workspace/workspace-test
Is folder ke andar ye files hain:
total 0
```

**Ye hi hai tumhara workspace** — abhi khali hai kyunki humne kuch banaya nahi.

### Step 4: Workspace mein file banao (proof ke liye)

Job ke build step mein ye add karo:

```bash
echo "Hello from workspace" > myfile.txt
cat myfile.txt
ls -la
```

**Build Now** dabao, **Console Output** check karo — ab `myfile.txt` dikhega workspace ke andar.

### Step 5: Server pe directly jaakar bhi dekh sakte ho

Jenkins server machine mein SSH karo:

```bash
ssh -i your-key.pem ubuntu@jenkins-server-ip
cd /var/lib/jenkins/workspace/workspace-test
ls -la
cat myfile.txt
```

Yahan tumhe wahi file dikhegi jo Jenkins ne build ke time banayi thi — kyunki workspace ek **real folder hai server ke disk pe**, koi magic nahi hai.

---

## 4. Workspace Clean karna (Purani files hatana)

Kabhi-kabhi purani files naye build se conflict karti hain, isliye workspace ko clean karna padta hai.

### Pipeline (Jenkinsfile) mein clean karna

```groovy
pipeline {
    agent any
    stages {
        stage('Clean Workspace') {
            steps {
                cleanWs()   // purana sab kuch delete kar deta hai
            }
        }
        stage('Checkout Code') {
            steps {
                git 'https://github.com/your-repo.git'
            }
        }
        stage('Build') {
            steps {
                sh 'echo "Building fresh..."'
            }
        }
    }
}
```

**Note:** `cleanWs()` use karne ke liye Jenkins mein **"Workspace Cleanup Plugin"** install hona chahiye (Manage Jenkins → Plugins mein check kar lena).

### Manually (Freestyle job mein) clean karna

Job Configuration mein:
1. **General** section mein jao
2. **"Delete workspace before build starts"** checkbox tick karo (agar option dikhe, ya plugin install karo)

---

## 5. Workspace ka Path Jenkinsfile mein use karna

Agar tumhe apne script mein workspace ka path chahiye, Jenkins ek built-in variable deta hai: `WORKSPACE`

```groovy
pipeline {
    agent any
    stages {
        stage('Show Workspace') {
            steps {
                sh 'echo "Workspace path hai: $WORKSPACE"'
                sh 'echo "data" > $WORKSPACE/output.txt'
            }
        }
    }
}
```

Isse tumhe hardcode nahi karna padta path — Jenkins khud sahi path de deta hai, chahe kisi bhi agent pe chale.

---

## 6. Workspace vs Shared Directory — Yaad rakhne wali baat

| Cheez | Workspace | Shared Directory (NFS) |
|---|---|---|
| Kaun banata hai | Jenkins khud, automatically | Tum manually banate ho |
| Kitne hote hain | Har agent ka apna alag | Sab agents ek hi share karte hain |
| Kab tak rehta hai | Build khatam hone pe clean ho sakta hai | Permanent, jab tak khud delete na karo |
| Default Path | `/var/lib/jenkins/workspace/job-name/` | Jo bhi tum banao, jaise `/mnt/jenkins-shared/` |
| Use kab karna | Normal build/compile/test ke liye | Do agents ke beech data pass karne ke liye |

---

## 7. Quick Command Reference

| Kaam | Command / Syntax |
|---|---|
| Current workspace path dekhna (shell mein) | `pwd` |
| Workspace ke andar files dekhna | `ls -la` |
| Jenkinsfile mein workspace path use karna | `$WORKSPACE` |
| Workspace clean karna (Pipeline) | `cleanWs()` |
| Server pe directly workspace dekhna | `cd /var/lib/jenkins/workspace/JOB_NAME` |

---

## 8. Zaroori Notes

- Workspace **automatically** banta hai — tumhe manually kuch setup nahi karna padta (Shared Directory ke ulat, jo tumhe khud NFS se banana padta hai).
- Agar same job **alag-alag agents** pe chale, to har agent pe **alag workspace** banega — data automatically share nahi hota. Agar data share karna hai to Shared Directory (NFS) use karo.
- `cleanWs()` use karna best practice hai taaki purani files se conflict na ho naye build mein.
- Workspace ka path hamesha `$WORKSPACE` variable se lena chahiye Jenkinsfile mein — hardcode mat karo, kyunki agent badalne pe path bhi badal sakta hai.
