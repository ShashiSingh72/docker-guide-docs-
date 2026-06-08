# Topic 3: Installation & Setup (The Professional Way)

Docker ko install karna sirf ek command chalana nahi hai. Ek DevOps engineer ko pata hona chahiye ki piche kya ho raha hai, security kaise maintain karni hai, aur production servers ko kaise optimize karna hai.

---

### 1. The Big Choice: Docker Desktop vs. Docker Engine 💻
Log aksar isme confuse hote hain. Inhe ek analogy se samajhte hain:

- **Analogy (The Car):**
    - **Docker Desktop:** Ye ek "Automatic Car" ki tarah hai. Isme AC, Music System, aur Power Steering sab pehle se laga hai (GUI, Kubernetes, Dashboard). Ye beginners aur local development ke liye best hai.
    - **Docker Engine:** Ye ek "F1 Race Car" ki tarah hai. Isme sirf wo cheezein hain jo speed aur performance ke liye zaroori hain. Ye production servers ke liye best hai.

| Feature | Docker Desktop | Docker Engine |
| :--- | :--- | :--- |
| **Target OS** | Windows, macOS, Linux (GUI) | Linux Servers (CLI only) |
| **GUI (Visuals)** | Haan, dashboard milta hai. | Nahi, sirf commands se kaam hota hai. |
| **Kubernetes** | In-built (Single click start). | Alag se install karna padta hai. |
| **Backend** | VM based (WSL2/Hyper-V/Apple Silicon). | Native (Directly on Linux Kernel). |

---

### 2. 🌟 Beginner's Corner (Explain Like I'm 5)
Docker install karna bilkul naya "Kitchen Set" laane jaisa hai. 
- **Docker Desktop** wo set hai jo ek box mein aata hai, jisme stove, bartan, aur masala sab saath milta hai. Bas rakho aur khana banana shuru karo.
- **Docker Engine** wo hai jisme aap stove alag se lete hain, gas connection alag se, aur bartan alag se. Thoda mehnat lagti hai setup mein, lekin ye professional chefs (DevOps Engineers) ke liye best hota hai kyunki wo har cheez apni marzi se set kar sakte hain.

---

### 3. Linux Installation Deep Dive (Ubuntu Standard)
Production mein 90% cases mein hum Linux use karte hain. Niche ke steps ko dhyan se follow karein:

**Step 1: Cleanup (Purana Kachra Saaf Karein)**
```bash
sudo apt-get remove docker docker-engine docker.io containerd runc
```

**Step 2: Repository Setup (Sahi Address Set Karein)**
```bash
sudo apt-get update
sudo apt-get install ca-certificates curl gnupg lsb-release
```

**Step 3: GPG Key (Security Check)**
Docker ke official servers se software lene ke liye hum unki "Digital Key" add karte hain.
```bash
sudo mkdir -p /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
```

**Step 4: Install Docker Engine**
```bash
sudo apt-get update
sudo apt-get install docker-ce docker-ce-cli containerd.io docker-compose-plugin
```

---

### 4. 🛠️ Technical Deep Dive: Post-Installation Magic
Sirf install karne ke baad jab aap `docker ps` chalate hain, toh "Permission Denied" aata hai. Iske piche ka logic kya hai?

**The Unix Socket Concept:**
Docker Daemon ek file ke through baat karta hai: `/var/run/docker.sock`. 
Default mein is file ka owner `root` hota hai. Isliye sirf `sudo` ke saath docker chalta hai.

**The Fix (Sudo-less Docker):**
Hum apne user ko `docker` group mein add karte hain.
```bash
sudo usermod -aG docker $USER
newgrp docker # Changes ko turant apply karne ke liye
```
*Technical Logic:* Jab aap group mein add hote hain, toh aapko us socket file ko read/write karne ki permission mil jati hai bina root bane.

---

### 5. ❓ Common Confusion: "Why do I need WSL2 on Windows?"
Linux par Docker "Native" chalta hai kyunki Docker ko Linux Kernel chahiye. 
Lekin Windows ka apna kernel alag hai. Isliye Windows par hum **WSL2 (Windows Subsystem for Linux)** install karte hain. Ye piche se ek choti si Linux machine chalta hai, aur Docker Desktop ussi machine ke kernel ko use karta hai.

---

### 6. Advanced Configuration: `daemon.json` (The Rulebook)
Production mein hum `/etc/docker/daemon.json` file banate hain Docker ki fitrat badalne ke liye.

**Analogy:** Ye ghar ke "Rules" ki list jaisa hai. 
- "Raat ko 10 baje ke baad shor nahi" -> `log-driver`
- "Sirf jaani-pehchani dukaano se saman lo" -> `insecure-registries`

**Example (Production Ready):**
```json
{
  "log-driver": "json-file",
  "log-opts": {
    "max-size": "10m",
    "max-file": "3"
  },
  "storage-driver": "overlay2"
}
```
*Isse aapka server kabhi 'Disk Full' ki wajah se crash nahi hoga!*

---

### 7. Senior's Perspective: Automation is Key
Senior engineers kabhi manually install nahi karte. Wo **Ansible, Terraform, ya Shell Scripts** use karte hain taaki 100 servers par ek saath sahi setup ho sake. Hamesha installation ke baad `docker info` check karein taaki aapko storage driver aur logging ki details mil sakein.

---

### 15 Production-Ready Interview Questions

#### Original 10 Questions:
1.  **Q: Hum Docker group mein user ko kyun add karte hain?**
    *Ans:* Taaki humein har docker command ke liye `sudo` na likhna pade. Docker socket ka access `root` ya `docker` group ke paas hota hai.
2.  **Q: Docker Desktop backend mein Windows par kya use karta hai?**
    *Ans:* Ye WSL2 (Windows Subsystem for Linux) ya Hyper-V VM use karta hai ek Linux kernel provide karne ke liye.
3.  **Q: `/etc/docker/daemon.json` file ka main purpose kya hai?**
    *Ans:* Is file se hum Docker Daemon ki settings customize karte hain, jaise log rotation, storage driver, aur DNS settings.
4.  **Q: Sudo-less Docker setup ke security risks kya hain?**
    *Ans:* Agar koi user `docker` group mein hai, toh wo asani se host machine ka root access le sakta hai (mounting / to container). Isliye trusted users ko hi add karein.
5.  **Q: Docker logs ki wajah se disk full hone se kaise bacha jaye?**
    *Ans:* `daemon.json` mein `log-driver` aur `log-opts` (max-size, max-file) set karke.
6.  **Q: Docker ko boot time par automatic start kaise karte hain?**
    *Ans:* `sudo systemctl enable docker` command use karke.
7.  **Q: Docker CE aur Docker EE mein kya farak hai?**
    *Ans:* CE (Community Edition) free hai. EE (Enterprise Edition) paid hai jisme extra security aur support milta hai.
8.  **Q: GPG keys verify kyun karte hain?**
    *Ans:* Taaki hum confirm kar sakein ki software official source se hi aaya hai aur secure hai.
9.  **Q: Kya hum Docker Engine upgrade kar sakte hain bina containers delete kiye?**
    *Ans:* Haan, `apt-get install` se upgrade karne par data safe rehta hai, bas daemon restart ke waqt thoda downtime hota hai.
10. **Q: Insecure registry config kab kaam aati hai?**
    *Ans:* Jab company ki private registry bina SSL (HTTP) ke chal rahi ho.

#### 5 New Tougher Questions:
11. **Q: Agar `docker info` mein storage driver 'vfs' dikh raha hai toh ye problem kyun hai?**
    *Ans:* 'vfs' bahut slow aur space-heavy hota hai. Production mein 'overlay2' hona chahiye. Ye aksar tab hota hai jab host kernel purana ho ya incompatibility ho.
12. **Q: `newgrp docker` aur `sudo usermod` ke baad logout karne mein kya farak hai?**
    *Ans:* `usermod` group change karta hai par wo next login par apply hota hai. `newgrp` usi terminal session mein turant group changes apply kar deta hai bina logout kiye.
13. **Q: Docker Desktop ki memory limits kahan se change hoti hain?**
    *Ans:* Docker Desktop ki GUI settings mein 'Resources' tab se. Linux Engine par ye limits host system ki memory par depend karti hain.
14. **Q: Docker setup mein 'Context' ka kya matlab hai?**
    *Ans:* Docker Context se aap ek hi client se multiple Docker Daemons (jaise local, remote server, ya cloud) ko switch karke control kar sakte hain.
15. **Q: Production mein 'Rootless' Docker install karne ka sabse bada challenge kya hai?**
    *Ans:* Rootless mode mein privileged ports (1-1024) bind karna mushkil hota hai aur kuch storage/network features restrict ho jate hain.

---
**Next Topic:** Phase 2, Topic 4: The First Steps (Essential CLI Commands).
