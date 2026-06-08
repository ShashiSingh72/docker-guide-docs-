# Topic 15: CICD Integration (Docker in Pipelines)

Modern software development mein Docker ka sabse bada use **CI/CD (Continuous Integration / Continuous Deployment)** pipelines mein hota hai. Isse hum "Build Once, Run Anywhere" ka sapna sach karte hain.

---

### 1. Docker in CI (Continuous Integration)
CI pipeline ka kaam hai code ko test karna aur uski image banana.
1.  **Code Commit:** Developer GitHub/GitLab par code push karta hai.
2.  **Build Stage:** Pipeline (Jenkins/GitHub Actions) `docker build` command chalati hai.
3.  **Test Stage:** Image ke andar unit tests chalaye jate hain.
4.  **Push Stage:** Agar tests pass ho jayein, toh image ko Registry (Docker Hub/ECR) par push kar diya jata hai.

---

### 2. Docker in CD (Continuous Deployment)
CD pipeline ka kaam hai nayi image ko production server par deploy karna.
1.  **Pull:** Production server par naya image tag pull kiya jata hai.
2.  **Deploy:** Purana container stop karke naya container start kiya jata hai (`docker compose up -d`).

---

### 3. Real-world Examples

#### A. GitHub Actions Workflow (`.github/workflows/main.yml`)
```yaml
jobs:
  build-and-push:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - name: Login to Docker Hub
        run: echo "${{ secrets.DOCKER_PASSWORD }}" | docker login -u "${{ secrets.DOCKER_USER }}" --password-stdin
      - name: Build and Push
        run: |
          docker build -t myuser/myapp:${{ github.sha }} .
          docker push myuser/myapp:${{ github.sha }}
```

#### B. Jenkins Pipeline (Jenkinsfile)
```groovy
pipeline {
    agent any
    stages {
        stage('Build Image') {
            steps {
                sh 'docker build -t myapp:${BUILD_NUMBER} .'
            }
        }
        stage('Push to ECR') {
            steps {
                sh 'docker push my-repo/myapp:${BUILD_NUMBER}'
            }
        }
    }
}

---

### 4. Docker-in-Docker (DinD) vs Docker-out-of-Docker (DooD)
Pipeline ke andar docker commands kaise chalayein?
- **DinD:** Container ke andar ek poora alag Docker engine chalta hai. (Complex aur slow).
- **DooD:** Host machine ke `/var/run/docker.sock` ko pipeline container mein mount kar diya jata hai. (Fast aur easy, production mein zyada use hota hai).

---

### 5. Self-hosted vs Cloud Runners
Pipeline chalane ke liye humein compute power chahiye hoti hai, jise hum "Runners" kahte hain.

| Feature | Cloud Runners (GitHub/GitLab Hosted) | Self-hosted Runners (Your Own Server) |
| :--- | :--- | :--- |
| **Maintenance** | Zero. Provider manage karta hai. | High. Aapko OS aur Docker update karna hoga. |
| **Security** | Safe, isolated environments. | Risk agar correctly isolated nahi hai. |
| **Performance** | Shared resources, thoda slow ho sakta hai. | Fast, kyunki resources dedicated hain. |
| **Cost** | Fixed free minutes, phir pay-as-you-go. | Free (sirf server ka rent dena hai). |
| **Customization** | Limited. | Full control (GPU, specialized hardware). |

---

### 6. Pipeline Strategy: Rollback using Docker Tags
Production mein agar naya code phat jaye, toh turant purane version par wapas jana (Rollback) zaroori hai.

**Strategy:**
1.  **Immutable Tags:** Kabhi bhi `latest` tag prod mein use na karein. Hamesha Git Commit SHA ya Semantic Versioning (`v1.2.3`) use karein.
2.  **Tag History:** Registry mein pichle 5-10 tags humesha save rakhein.
3.  **One-Click Rollback:** Pipeline mein ek parameter rakhein jisme aap purana tag pass karein aur wo turant deploy ho jaye.

**Example Command for Rollback:**
```bash
# Naya image phat gaya, toh purana tag deploy karo
docker compose up -d --detach --no-deps --scale app=1 myapp:old-git-sha
```

---

### 💡 Perspectives

#### 🟢 Beginner's Corner
"Dosto, CI/CD sunne mein bada lagta hai par hai bahut simple: Ye bas wahi commands hain jo aap manual chalate ho (`docker build`, `docker push`), par ab ek script (YAML file) unhe automate kar rahi hai. Shuruat GitHub Actions se karo, ye sabse easy aur beginner-friendly hai."

#### 🔴 Senior's Perspective
"Large scale par, hum **GitOps** model (jaise ArgoCD ya Flux) use karte hain. Isme pipeline image build karke Registry mein push karti hai, aur ek agent production state ko Git repository ke saath sync rakhta hai. Security ke liye **Vulnerability Scanning** aur **Image Signing** pipeline ka mandatory part hona chahiye. Rollback strategy hamesha tested honi chahiye, crash ke waqt dimaag kaam nahi karta!"

---

### 15 Production-Ready Interview Questions (Q&A)

**Q1: CI/CD pipeline mein `latest` tag use karna kyun mana hai?**
*Ans:* Kyunki `latest` tag se ye pata nahi chalta ki kaunsa version deploy hua hai. Agar rollback karna ho, toh purani image milna mushkil hota hai. Hamesha Git Commit SHA ya Build Number use karein.

**Q2: Docker-in-Docker (DinD) kya hota hai?**
*Ans:* Jab hum ek Docker container ke andar hi Docker engine install karke images build karte hain. Ye aksar Jenkins ya GitLab runners mein use hota hai.

**Q3: `/var/run/docker.sock` mount karne ka kya risk hai pipeline mein?**
*Ans:* Ye ek security risk hai. Agar pipeline compromised ho jaye, toh attacker host machine par koi bhi docker command chala sakta hai aur poora control le sakta hai.

**Q4: Pipeline mein image build fast karne ke liye kya kar sakte hain?**
*Ans:* 1. Build cache ka use karein (`--cache-from`), 2. Multi-stage builds use karein, 3. Choti base images (Alpine) use karein.

**Q5: Artifacts aur Docker Images mein kya farak hai CI mein?**
*Ans:* Artifacts sirf files hoti hain (jaise `.jar` ya `.zip`). Docker image ek poora environment hoti hai jisme app aur uski dependencies pack hoti hain.

**Q6: "Canary Deployment" Docker mein kaise karte hain?**
*Ans:* Nayi image ko sirf ek server ya ek chote percentage users ke liye deploy karna (e.g., Load balancer par 10% traffic naye container ko bhejna).

**Q7: Blue-Green Deployment kya hai?**
*Ans:* Do same environments (Blue aur Green) maintain karna. Ek par purana code chalta hai, dusre par naya deploy karke traffic switch kar diya jata hai. Zero downtime ke liye best hai.

**Q8: Pipeline mein secrets (jaise Docker Hub password) kaise handle karein?**
*Ans:* Kabhi bhi code mein mat likhein. Jenkins Credentials, GitHub Secrets, ya AWS Secrets Manager use karein.

**Q9: `docker build` fail hone par pipeline ka kya hota hai?**
*Ans:* Pipeline wahi stop ho jati hai aur developer ko error report milti hai. Isse broken code production mein jane se bach jata hai.

**Q10: Image ko scan karne ka sahi waqt kab hai pipeline mein?**
*Ans:* Image build hone ke turant baad aur Push karne se pehle. Agar vulnerability mile, toh push cancel kar dena chahiye.

**Q11: GitOps aur traditional CI/CD mein kya farak hai Docker ke context mein?**
*Ans:* Traditional CI/CD mein pipeline server par push karti hai (`Push model`). GitOps mein ek agent (like ArgoCD) Git repo ko watch karta hai aur changes ko cluster par pull karta hai (`Pull model`).

**Q12: Agar Registry down ho jaye, toh kya deployment ho sakti hai?**
*Ans:* Agar image pehle se host par cached hai, toh `docker run` chal jayega. Lekin naye nodes ya scale-up fail ho jayega. Isliye Registry ka high-availability (HA) hona zaroori hai.

**Q13: Build Matrix kya hota hai CI pipelines mein?**
*Ans:* Ek hi code ko multiple environments (e.g., different Node.js versions ya different OS architectures like arm64, amd64) par parallelly test aur build karna.

**Q14: Docker Buildx pipeline mein kyun use hota hai?**
*Ans:* Multi-architecture images (jo Windows, Mac, aur Linux sab par chalein) banane ke liye Buildx ka use kiya jata hai.

**Q15: Rollback ke waqt database changes ko kaise handle karein?**
*Ans:* Ye sabse mushkil part hai. Iske liye "Forward-only migrations" aur "Expand/Contract pattern" use kiya jata hai taaki naya code purane DB schema ke saath bhi kaam kar sake.

---
**Next Topic:** Phase 8, Topic 16: Registry Management (ECR/Nexus).
