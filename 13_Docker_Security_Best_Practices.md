# Topic 13: Docker Security Best Practices

Docker containers ko secure banana sirf option nahi, balki production mein ek requirement hai. Agar security sahi nahi hai, toh ek compromised container poore host machine ka access attacker ko de sakta hai.

---

### 1. Run as Non-Root User (Most Important)
By default, Docker containers root user ke taur par chalte hain. Agar koi attacker container mein ghus gaya, toh uske paas root privileges hongi.

**Solution:** Dockerfile mein hamesha ek non-root user banayein aur `USER` instruction use karein.
```dockerfile
RUN groupadd -r myuser && useradd -r -g myuser myuser
USER myuser
```

---

### 2. Image Scanning & Trusted Sources
- **Official Images Only:** Hamesha official ya verified images hi use karein.
- **Image Scanning:** Deployment se pehle images ko scan karein (Trivy, Docker Scout, ya Snyk use karke).
  - `docker scout quickview <image_name>`
- **Docker Content Trust (DCT):** `export DOCKER_CONTENT_TRUST=1` set karne se Docker sirf signed images hi pull karega.

---

### 3. Minimize the Attack Surface
- **Use Distroless/Alpine:** Jitni kam files image mein hongi, utne kam attack vectors honge.
- **Multi-stage Builds:** Build-time tools (git, compilers) ko final production image se hatayein.
- **No-New-Privileges:** Container ke andar processes ko apni privileges badhane se rokein.
  - `docker run --security-opt=no-new-privileges ...`

---

### 4. Storage & Network Security
- **Read-Only Root FS:** Container ke root file system ko read-only banayein taaki attacker koi malicious script download na kar sake.
  - `docker run --read-only ...`
- **Secrets Management:** Passwords aur API keys ko `ENV` variables mein na rakhein. Docker Secrets (Swarm) ya HashiCorp Vault use karein.
- **Disable Inter-Container Communication (ICC):** Default bridge par containers aapas mein baat kar sakte hain. Isse disable karna secure hai.
  - `--icc=false` in `daemon.json`.

---

### 5. Resource Constraints
Resource limits set karna bhi security ka part hai taaki **DoS (Denial of Service)** attack se bacha ja sake (Topic 12 mein cover kiya gaya hai).

---

### 6. Deep Dive: Kernel Security (AppArmor & Seccomp)
Sirf file system restrict karna kafi nahi hai, humein kernel-level protection bhi chahiye.

#### A. AppArmor Profiles
AppArmor ek Linux Security Module hai jo individual programs ki capabilities ko restrict karta hai. Docker ek default profile ke saath aata hai (`docker-default`), jo sensible defaults set karta hai (jaise mounting block devices ko rokna).
- **Custom Profile:** Aap apni security requirements ke hisaab se custom profiles bana sakte hain.
- **Usage:** `docker run --security-opt apparmor=your-profile-name ...`

#### B. Seccomp (Secure Computing Mode)
Seccomp system calls ko filter karta hai. Ek container ko hazaron syscalls ki zaroorat nahi hoti. Docker ka default seccomp profile ~300 me se ~40 syscalls ko block kar deta hai (jaise `reboot()`).
- **Custom Policy:** Agar aapki app sirf read-write karti hai, toh aap baaki sab block kar sakte hain.
- **Usage:** `docker run --security-opt seccomp=path/to/profile.json ...`

---

### 7. Linux Capabilities (CAP_ADD & CAP_DROP)
By default, Docker containers "full root" nahi hote, unke paas limited **Capabilities** hoti hain. Lekin "Least Privilege" principle ke liye humein inhein aur control karna chahiye.

- **`--cap-drop=ALL`**: Sabse pehle saari privileges cheen lo. Ye best practice hai.
- **`--cap-add`**: Sirf wahi capability wapas do jo zaroori hai.

**Common Capabilities:**
- `NET_ADMIN`: Network configuration change karne ke liye.
- `SYS_TIME`: System clock set karne ke liye.
- `CHOWN`: File ownership change karne ke liye.

**Example (Secure way):**
```bash
# Sab drop karo, sirf chown allow karo
docker run --cap-drop=ALL --cap-add=CHOWN my-secure-app
```

---

### 💡 Perspectives

#### 🟢 Beginner's Corner
"Dosto, shuruat mein bas itna yaad rakho: **Hamesha non-root user use karo** aur **--privileged flag se bachon**. Security ek samundar hai, par ye do steps aapko 80% khatron se bacha lenge. Docker Hub se random images uthana band karo, hamesha 'Official' tag dekho."

#### 🔴 Senior's Perspective
"Production mein security 'afterthought' nahi hoti. Hum **Policy as Code** (jaise OPA/Kyverno) use karte hain taaki koi bhi 'insecure' container deploy hi na ho sake. Runtime security tools jaise **Falco** ka use karo jo container ke andar suspicious activity (jaise unexpected shell execution) ko detect karke alert bhej sakein. Container security is about layers (Defense in Depth)."

---

### 15 Production-Ready Interview Questions (Q&A)

**Q1: Docker container ko root user se chalana kyun khatarnak hai?**
*Ans:* Agar container root se chal raha hai aur attacker usse break karke host machine tak pahunch jata hai, toh uske paas host ka bhi root access ho sakta hai (Container Escape).

**Q2: "Privileged Container" kya hota hai aur ye kyun avoid karna chahiye?**
*Ans:* `--privileged` flag container ko host machine ke saare hardware aur kernel features ka access de deta hai. Ye security ke liye sabse bada khatra hai.

**Q3: `.dockerignore` file security mein kaise madad karti hai?**
*Ans:* Ye sensitive files jaise SSH keys, `.env` files, aur backup files ko image mein jane se rokti hai, jo galti se image ke saath leak ho sakti hain.

**Q4: Docker Content Trust (DCT) kya hai?**
*Ans:* DCT ye ensure karta hai ki aap jo image pull kar rahe hain wo digitaly signed hai aur original publisher ne hi bheji hai, kisi ne beech mein change nahi ki.

**Q5: Image vulnerability scanning tool (jaise Trivy) kya dhundta hai?**
*Ans:* Ye image ki libraries (OS packages, npm, pip) ko CVE (Common Vulnerabilities and Exposures) database se match karta hai aur purani/vulnerable versions ki report deta hai.

**Q6: `--read-only` flag ka kya fayda hai?**
*Ans:* Ye container ke poore file system ko read-only bana deta hai. Agar koi attacker container mein ghus kar koi virus ya script likhna chahega, toh wo nahi kar payega.

**Q7: Docker socket (`/var/run/docker.sock`) ko container mein mount karna kyun mana hai?**
*Ans:* Docker socket ka access milne ka matlab hai poore Docker engine ka control. Container ke andar se koi bhi naya container bana kar host ka root access le sakta hai.

**Q8: "Distroless" images security ke liye best kyun hain?**
*Ans:* Kyunki inme koi shell (`sh`, `bash`) ya package manager (`apt`, `apk`) nahi hota. Attacker agar ghus bhi jaye, toh uske paas koi tool nahi hoga execute karne ke liye.

**Q9: `no-new-privileges` security option kya karta hai?**
*Ans:* Ye container ke processes ko `setuid` ya `setgid` binaries use karke apni privileges badhane (privilege escalation) se rokta hai.

**Q10: Docker secrets `ENV` variables se zyada secure kyun hain?**
*Ans:* `ENV` variables `docker inspect` ya logs mein asani se dikh jate hain. Secrets memory mein store hote hain aur sirf authorized containers ko hi unka access milta hai.

**Q11: Seccomp aur AppArmor mein kya difference hai?**
*Ans:* Seccomp system calls (syscalls) par focus karta hai (process kernel se kya maang raha hai), jabki AppArmor program ke behavior aur resources (files, network) par focus karta hai.

**Q12: Agar mujhe container mein networking tools chalane hain, toh kya mujhe --privileged use karna chahiye?**
*Ans:* Nahi! Aapko sirf specific capability deni chahiye: `--cap-add=NET_ADMIN`. Isse sirf networking access milega, baaki kernel features secure rahenge.

**Q13: "Container Escape" kya hota hai?**
*Ans:* Ye ek aisi vulnerability hai jahan ek process container ki boundary tod kar host OS ke resources access kar leta hai. Isse bachne ke liye kernel security profiles aur user namespaces zaroori hain.

**Q14: User Namespaces ka Docker mein kya kaam hai?**
*Ans:* Isse container ke andar ka root user, host machine ke ek non-privileged user se map ho jata hai. Yani container ke andar aap 'root' hain, par host ke liye aap 'normal user' hain.

**Q15: Docker Image Signing (Cosign/Notary) kyun zaroori hai?**
*Ans:* Taaki pipeline se production tak ye ensure kiya ja sake ki image ke saath kisi ne tempering nahi ki hai aur wahi image deploy ho rahi hai jo scan/test hui thi.

---
**Next Topic:** Phase 7, Topic 14: Logging & Monitoring.
