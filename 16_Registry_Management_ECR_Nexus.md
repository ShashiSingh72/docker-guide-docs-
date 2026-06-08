# Topic 16: Registry Management (ECR/Nexus)

Jab aap image build karte hain, toh usse kahin store karna padta hai taaki production servers usse pull kar sakein. Is storage ko **Registry** kehte hain.

---

### 1. Public vs Private Registries
- **Public Registry (e.g., Docker Hub):** Koi bhi aapki image pull kar sakta hai (agar public hai). Best for open source.
- **Private Registry (e.g., AWS ECR, Azure ACR, Nexus):** Sirf authorized log hi images dekh/pull kar sakte hain. Production ke liye yahi use hoti hai.

---

### 2. Popular Registry Options
1.  **Docker Hub:** Sabse popular. Isme official images hoti hain.
2.  **AWS ECR (Elastic Container Registry):** Agar aapka setup AWS par hai, toh ye best hai. IAM roles ke saath integration asaan hai.
3.  **Sonatype Nexus / JFrog Artifactory:** Agar aapko apni company ke network ke andar (on-premise) registry chahiye.
4.  **GitHub Container Registry (GHCR):** GitHub Actions ke saath seamless kaam karta hai.

---

### 3. Registry Workflow
1.  **Login:** `docker login <registry_url>`
2.  **Tag:** Image ko registry ke format mein tag karein.
    - `docker tag myapp:v1 123456789.dkr.ecr.us-east-1.amazonaws.com/myapp:v1`
3.  **Push:** `docker push <tagged_image>`
4.  **Pull:** Production server par: `docker pull <tagged_image>`

---

### 4. Image Lifecycle Policies (Production Tip)
Production registries mein hazaron images jama ho sakti hain, jo storage ki cost badhati hain.
- **AWS ECR Lifecycle Policy:** Ek rule banayein ki "Sirf last 30 images rakho aur baaki delete kar do" ya "Jo images 90 din se purani hain unhe delete kar do".

---

### 5. Security in Registries
- **Vulnerability Scanning:** Zyada tar modern registries (ECR, Docker Hub Pro) automatically image ko scan karti hain push hote hi.
- **IAM Roles:** Password share karne ke bajaye roles use karein (EC2 ko ECR pull permission dena).

---

### 6. Deep Dive: Garbage Collection in Private Registries
Private registries (jaise self-hosted Docker Registry ya Nexus) mein jab aap image delete karte hain, toh wo sirf "manifest" delete karti hain, actual data (layers) disk par hi rehta hai.
- **Problem:** Disk space kabhi khali nahi hoti even after deleting images.
- **Solution (Garbage Collection):** Aapko manual ya scheduled garbage collector chalana padta hai.
  - Command for self-hosted registry: `registry garbage-collect /etc/docker/registry/config.yml`
  - **Warning:** GC chalate waqt registry ko "read-only" mode mein daalna zaroori hai, warna image corruption ka khatra rehta hai.

---

### 7. Docker Credential Helpers: No more clear-text passwords!
Default mein `docker login` aapka password `~/.docker/config.json` mein base64 format mein save karta hai, jo ki secure nahi hai.
- **Credential Helpers:** Ye Docker ko batate hain ki credentials ko OS ke secure store (jaise macOS Keychain, Windows Credential Manager, ya AWS/GCP specific helpers) mein rakhein.
- **How to use:**
  - `docker-credential-ecr-login`: AWS ECR ke liye. Isse aapko `docker login` karne ki zaroori nahi padti, ye IAM roles se automatically token utha leta hai.
  - Config mein aisa dikhta hai:
    ```json
    { "credsStore": "ecr-login" }
    ```

---

### 💡 Beginner's Corner
> "Bhaiya, registry matlab Google Drive samajh lo containers ke liye. Bas yahan photos ki jagah code aur dependencies ki layers hoti hain. Shuruat mein Docker Hub use karo, fir company mein ECR ya Nexus par switch karna."

### 👨‍💻 Senior's Perspective
> "Production mein kabhi bhi 'Latest' tag par rely mat karo. Hamesha immutable tags aur registry lifecycle policies ka use karo. Garbage collection ka schedule set karna mat bhoolna, warna ek din storage full hone ki wajah se poora CI/CD pipeline ruk jayega!"

---

### 15 Production-Ready Interview Questions (Q&A)

**Q1: Docker Hub aur AWS ECR mein main difference kya hai?**
*Ans:* Docker Hub ek general-purpose public/private registry hai. AWS ECR ek fully managed AWS service hai jo AWS ke baaki services (ECS, EKS, IAM) ke saath tightly integrated hai.

**Q2: Image ko registry par push karne se pehle "Tag" karna kyun zaroori hai?**
*Ans:* Kyunki Tag mein Registry ka URL hota hai. Bina URL ke Docker ko nahi pata chalega ki image kis server par bhejni hai.

**Q3: Private registry se image pull karne ke liye "Authentication" kaise manage karte hain?**
*Ans:* `docker login` command se ya `config.json` file mein credentials store karke. Kubernetes mein hum `ImagePullSecrets` use karte hain.

**Q4: Registry Lifecycle Policy ka production mein kya fayda hai?**
*Ans:* Ye storage cost bachaati hai. Purani aur unused images ko automatic delete karke ye registry ko saaf rakhti hai.

**Q5: "Insecure Registry" kya hoti hai?**
*Ans:* Wo registry jo HTTPS ke bajaye HTTP par chal rahi ho. Docker default mein isse block karta hai jab tak aap `daemon.json` mein usse allow na karein.

**Q6: Kya hum ek hi image ko multiple registries par push kar sakte hain?**
*Ans:* Haan, bas aapko har registry ke liye alag tag banana padega aur har ek par separately `docker push` chalana padega.

**Q7: Registry mein "Immutable Tags" feature ka kya matlab hai?**
*Ans:* Agar ye enabled hai, toh aap kisi existing tag (jaise `v1`) ko overwrite nahi kar sakte. Ye production mein accidental changes se bachata hai.

**Q8: AWS ECR mein "Login Token" kitni der tak valid rehta hai?**
*Ans:* Default mein 12 ghante tak. Isliye CI/CD pipelines mein hum har baar naya token generate karte hain.

**Q9: Mirror Registry kya hoti hai?**
*Ans:* Ye Docker Hub ki ek copy hoti hai jo aapke local network mein hoti hai. Isse images pull karna fast hota hai aur internet bandwidth bachti hai.

**Q10: "Docker Content Trust" registry level par kaise kaam karta hai?**
*Ans:* Ye ye ensure karta hai ki registry par sirf signed aur verified images hi upload/download hon.

**Q11: Garbage Collection chalate waqt Registry ko Read-Only kyun karna chahiye?**
*Ans:* Kyunki agar GC chal raha hai aur ussi waqt koi nayi image push ho gayi, toh GC un layers ko delete kar sakta hai jo nayi image ko chahiye, jisse image corrupt ho jayegi.

**Q12: `credsStore` aur `credHelpers` mein kya difference hai?**
*Ans:* `credsStore` poori registry ke liye ek hi store (jaise Keychain) use karta hai, jabki `credHelpers` alag-alag registries ke liye alag-alag helpers (jaise ECR ke liye alag, GCR ke liye alag) specify karne deta hai.

**Q13: Agar aapka worker node ECR se image pull nahi kar paa raha, toh sabse pehle kya check karenge?**
*Ans:* 1. Node ka IAM Role (kya usse `AmazonEC2ContainerRegistryReadOnly` permission hai?), 2. Security Group/Network (kya ECR endpoint accessible hai?), 3. Credentials expiry.

**Q14: Nexus mein 'Hosted', 'Proxy', aur 'Group' repositories kya hoti hain?**
*Ans:* 'Hosted' aapki apni images ke liye hai, 'Proxy' Docker Hub jaisi external registries ko cache karne ke liye, aur 'Group' in sabko ek single URL ke peeche combine karne ke liye.

**Q15: Image signing (Notary) ka production mein kya role hai?**
*Ans:* Ye guarantee deta hai ki jo image build hui thi, wahi deploy ho rahi hai aur beech mein kisi ne usse modify nahi kiya hai.

---
**Next Topic:** Phase 8, Topic 17: The Road to Kubernetes (Swarm vs K8s).
