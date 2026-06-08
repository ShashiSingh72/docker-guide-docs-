# Topic 12: Resource Management (Limits & Quotas)

Production environment mein ye ensure karna bahut zaroori hai ki ek container poore server ke resources (CPU/RAM) na kha jaye. Isse hum "Noisy Neighbor" problem kehte hain. Docker humein in resources ko limit karne ke tools deta hai.

---

### 1. Memory Limits (RAM) & Swapping 🧠
Docker mein do tarah ki memory limits hoti hain:

*   **Hard Limit (`--memory` or `-m`):** Container isse zyada RAM use nahi kar sakta. Agar karega, toh Kernel usse kill kar dega (OOM Kill).
*   **Soft Limit (`--memory-reservation`):** Ye ek "guaranteed" memory hai. Jab system par load kam ho, container zyada RAM le sakta hai, lekin load badhne par Docker usse is limit tak wapas le aayega.

**Memory Swap ka Impact:**
By default, agar container RAM limit hit karta hai, toh wo `swap` (disk memory) use karne lagta hai. 
- **Performance Hit:** Swap RAM se 100x slow hota hai. Isliye app slow ho jayegi.
- **Control:** Aap `--memory-swap` flag use karke limit set kar sakte hain. Agar aap ise memory ke equal rakhte hain, toh swapping disable ho jayegi.

---

### 2. CPU Limits: Quota vs Period ⚙️
Docker CPU manage karne ke liye Linux ke CFS (Completely Fair Scheduler) ka use karta hai.

- **`--cpu-period`:** Ye microseconds (µs) mein hota hai. Default value 100,000 (100ms) hoti hai.
- **`--cpu-quota`:** Ye wo time hai jo container ko us 'period' mein milta hai.

**Example:**
Agar aap chahte hain ki container ko maximum 50% of 1 CPU mile:
- Period = 100,000 (100ms)
- Quota = 50,000 (50ms)
- Command: `docker run --cpu-period=100000 --cpu-quota=50000 nginx`

*Note: Inhe manually set karne se behtar hai `--cpus="0.5"` use karna, jo ye calculation background mein khud kar deta hai.*

---

### 3. Cgroups v1 vs v2 🛡️
Cgroups (Control Groups) wo Linux Kernel feature hai jisse Docker resources control karta hai.

- **Cgroups v1:** Purana system. Isme har resource (CPU, Memory) ki alag hierarchy hoti thi. Ye thoda complex aur kam efficient tha.
- **Cgroups v2:** Naya aur modern system (Ubuntu 22.04+ mein default). Isme unified hierarchy hai, jo behtar resource management aur pressure stall information (PSI) provide karti hai. Docker Desktop aur naye distros ab v2 hi use karte hain.

---

### 4. Resource Limits in Docker Compose
Compose file mein hum `deploy` section ka use karte hain (V3+):

```yaml
services:
  web:
    image: nginx:alpine
    deploy:
      resources:
        limits:
          cpus: '0.50'     # Hard limit (Max 50% of 1 core)
          memory: 512M    # Hard limit
        reservations:
          cpus: '0.25'     # Soft limit/Guarantee
          memory: 128M    # Soft limit
```

---

### 5. OOM Killer (Out of Memory)
Jab server ki RAM khatam ho jati hai, toh Linux Kernel **OOM Killer** ko activate karta hai. 
- OOM Killer un processes (containers) ko kill karta hai jinka "OOM Score" sabse zyada hota hai.
- Aap `--oom-kill-disable` use kar sakte hain, lekin ye dangerous hai kyunki isse poora Host machine crash ho sakta hai.

---

### 🌟 Beginner's Corner
- **Simple Word mein:** Resource management matlab containers ki "diet" (khana) fix karna taaki wo bahut zyada "mote" (resource hungry) na ho jayein.
- **Tip:** Hamesha `docker stats` run karke dekhein ki aapka container kitni RAM use kar raha hai, phir uske hisaab se limits set karein.

---

### 👨‍💻 Senior's Perspective
"Production mein, CPU limits set karte waqt dhyaan rakhein ki Java jaisi applications (JVM) ko initial startup ke liye zyada CPU chahiye hota hai. Agar aapne bahut tight limit set kar di, toh startup time bahut badh jayega. Memory ke case mein, hamesha **Memory Reservation** (Soft limit) ko use karein taaki idle time mein resources waste na hon, lekin Hard Limit hamesha host ki capacity se 10-20% kam hi rakhein taaki OS crash na ho."

---

### 15 Production-Ready Interview Questions (Q&A)

**Q1: "Noisy Neighbor" problem kya hai aur Docker isse kaise solve karta hai?**
*Ans:* Jab ek container saare host resources (RAM/CPU) use kar leta hai aur dusre containers ke liye kuch nahi bachta, toh usse Noisy Neighbor kehte hain. Docker `Cgroups` ka use karke har container ke liye resource limits set karta hai.

**Q2: Hard limit (`--memory`) aur Soft limit (`--memory-reservation`) mein kya difference hai?**
*Ans:* Hard limit se upar jate hi container kill ho jayega. Soft limit container ko tab tak zyada RAM use karne deta hai jab tak host par resource ki kami na ho.

**Q3: OOM Score kya hota hai?**
*Ans:* Ye ek value hai jo Linux Kernel har process ko deta hai. Jis process ki memory usage uski limit ke paas hoti hai, uska OOM Score badh jata hai aur OOM Killer usse pehle kill karta hai.

**Q4: `--cpus="0.5"` ka kya matlab hai?**
*Ans:* Iska matlab hai ki container maximum aadha CPU core (50% of 1 CPU) use kar sakta hai.

**Q5: Agar container memory limit touch kar le, toh kya wo hamesha crash hota hai?**
*Ans:* Hamesha nahi. Agar `swap` enabled hai, toh Docker swap memory use karne lagega (jo slow hoti hai). Agar swap bhi khatam ho jaye, tab OOM Kill hoga.

**Q6: Production mein resource limits set karna kyun mandatory hai?**
*Ans:* Taaki koi malicious code ya application bug (memory leak) poore physical server ko down na kar de. Isse application ki availability aur stability bani rehti hai.

**Q7: `docker stats` mein "Limit" column kya dikhata hai?**
*Ans:* Agar aapne koi limit set ki hai, toh wo wahan dikhegi. Agar nahi ki, toh wo host machine ki total RAM dikhayega.

**Q8: `--cpu-shares` kab kaam aate hain?**
*Ans:* Jab CPU par heavy load ho aur containers aapas mein resource ke liye "lad" rahe hon. Ye priority decide karne mein madad karta hai. Default value 1024 hoti hai.

**Q9: Kya hum chalte hue container ki limits change kar sakte hain bina restart kiye?**
*Ans:* Haan, `docker update` command se. Example: `docker update --cpus 2 <container_id>`.

**Q10: Docker memory limits ke liye kernel ka kaunsa feature use karta hai?**
*Ans:* Docker Linux Kernel ke **Control Groups (Cgroups)** feature ka use karta hai.

**Q11: Cgroups v2 mein v1 ke mukable main improvement kya hai?**
*Ans:* Cgroups v2 unified architecture use karta hai, jisse controller management asaan ho jati hai. Isme rootless containers support behtar hai aur I/O control zyada granular hai.

**Q12: `--cpu-period` aur `--cpu-quota` ka relation explain karein.**
*Ans:* `period` wo time window hai (e.g. 100ms) aur `quota` wo maximum time hai jo container ko us window mein milega. Agar quota 20ms hai aur period 100ms, toh container ko 20% CPU milega.

**Q13: Swap memory use karne ke nuksaan kya hain?**
*Ans:* Disk I/O involvement ki wajah se performance drastically drop ho jati hai. Latency badh jati hai, jo real-time applications ke liye fatal ho sakti hai.

**Q14: Pressure Stall Information (PSI) kya hai?**
*Ans:* Ye Cgroups v2 ka feature hai jo batata hai ki kitna time CPU, Memory ya I/O shortage ki wajah se processes wait kar rahe hain. Isse humein 'Resource Contention' ka sahi pata chalta hai.

**Q15: `oom_score_adj` kya hota hai?**
*Ans:* Ye ek setting hai jisse hum Linux kernel ko batate hain ki kisi container ko OOM Killer se kitna 'bachana' hai ya kitna 'pehle' kill karna hai. Database containers ke liye hum ise low rakhte hain taaki wo last mein kill hon.

---
**Next Topic:** Phase 7, Topic 13: Docker Security Best Practices.
