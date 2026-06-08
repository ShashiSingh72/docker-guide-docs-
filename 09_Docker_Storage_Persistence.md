# Topic 9: Docker Storage (Persistence)

By default, Docker containers "stateless" hote hain. Matlab agar container delete hua, toh uske andar ka saara data khatam ho jayega. Is data ko bachane ke liye hum **Storage Persistence** ka use karte hain.

---

### 1. Types of Storage in Docker

| Type | Storage Location | Best For |
| :--- | :--- | :--- |
| **Volumes** | Docker managed area (`/var/lib/docker/volumes/`) | Production data, Databases. |
| **Bind Mounts** | Anywhere on the Host System | Source code sharing (Dev environment). |
| **tmpfs Mount** | Host System Memory (RAM) | Secrets/Passwords (No disk storage). |

---

### 2. Deep Dive: How Overlay2 Works?

Docker images layers mein banti hain, aur ye layers **Overlay2** storage driver handle karta hai. Iske main 3 parts hote hain:

1.  **LowerDir:** Ye image ki layers hoti hain. Ye "Read-Only" hoti hain.
2.  **UpperDir:** Ye container ki "Writable Layer" hoti hai. Jo bhi aap naya data likhte hain, wo yahan save hota hai.
3.  **MergedDir:** Ye wo view hai jo aapko container ke andar dikhta hai. Ye LowerDir aur UpperDir ka mixture hota hai.

**Copy-on-Write (CoW):** Agar aap kisi purani file (LowerDir) ko edit karte hain, toh Docker pehle usse UpperDir mein copy karta hai aur phir edit karne deta hai. Isse original image layer kabhi change nahi hoti.

---

### 3. Database Performance: Volumes vs Bind Mounts

Production mein Databases ke liye kya use karein?

- **Volumes (Recommended):**
    - **Performance:** Docker volumes Linux kernel ke native features use karti hain, isliye high I/O performance milti hai.
    - **Isolation:** Docker manage karta hai, isliye host ke permissions issues nahi aate.
    - **Backups:** `docker volume inspect` se asani se manage aur backup ho sakti hain.
- **Bind Mounts:**
    - **Performance:** Thoda slow ho sakti hain, especially macOS aur Windows par, kyunki file system translation ka overhead hota hai.
    - **Dependency:** Host machine ke folder structure aur permissions (User ID/Group ID) par depend karti hai, jo production mein errors ka kaaran banta hai.

**Verdict:** Database ke liye hamesha **Named Volumes** use karein!

---

### 4. Docker Volumes
*   **Named Volume:** `docker run -v my_data:/app/data nginx`
*   **Anonymous Volume:** Docker khud ek random ID de deta hai.

---

### 5. Bind Mounts (The Development Way)
- `docker run -v /home/user/project:/app nginx`
- Jab aap host par code badalte hain, wo container mein turant dikhta hai (Hot Reloading).

---

### 💡 Beginner's Corner
- **Simple Rule:** Agar production mein data save karna hai, toh Volume use karo. Agar development mein code share karna hai, toh Bind Mount use karo.
- **Don't Forget:** `docker rm -v` command use karein agar aap chahte hain ki container ke saath uski anonymous volume bhi delete ho jaye.
- **Path:** Docker volumes hamesha Linux par `/var/lib/docker/volumes/` mein milengi.

---

### 👨‍💻 Senior's Perspective
"Database performance ke liye 'Storage Drivers' ka role bahut bada hota hai. Hamesha check karein ki aapka system `overlay2` use kar raha hai ya nahi (`docker info` dekhein). Production mein hum 'Volume Plugins' use karte hain taaki data AWS EBS ya Azure Disk par store ho sake, jisse agar poora server bhi ud jaye, toh hamara data safe rahe."

---

### 15 Production-Ready Interview Questions (Q&A)

**Q1: Bind Mount aur Volume mein main difference kya hai?**
*Ans:* Volumes Docker manage karta hai (isolated), jabki Bind Mounts host ke kisi bhi folder ko map karte hain.

**Q2: Agar hum ek non-empty host folder ko mount karein, toh kya hoga?**
*Ans:* Container ke andar ka original data "hide" ho jayega aur sirf host folder ka data dikhega.

**Q3: `docker volume prune` kya karta hai?**
*Ans:* Unused (dangling) volumes ko delete karta hai.

**Q4: "Dangling Volume" kya hoti hai?**
*Ans:* Wo volume jo kisi bhi container se attached nahi hai.

**Q5: Kya hum ek hi volume ko multiple containers mein mount kar sakte hain?**
*Ans:* Haan, ye data sharing ke liye best tarika hai.

**Q6: `-v` aur `--mount` flag mein kya farak hai?**
*Ans:* `--mount` zyada verbose hai aur production mein recommend kiya jata hai kyunki ye error handling behtar karta hai.

**Q7: Dockerfile mein `VOLUME` instruction ka kya use hai?**
*Ans:* Ye batata hai ki is path par data persistent rehna chahiye. Agar user koi volume nahi deta, toh Docker "Anonymous Volume" bana deta hai.

**Q8: Databases ke liye kaunsa mount best hai?**
*Ans:* Named Volumes.

**Q9: `tmpfs` mount kab use karna chahiye?**
*Ans:* Jab sensitive data RAM mein rakhna ho aur disk par save na karna ho.

**Q10: Host machine par Docker volumes kahan store hoti hain?**
*Ans:* `/var/lib/docker/volumes/` mein.

**Q11: Overlay2 mein 'Copy-on-Write' mechanism kya hota hai?**
*Ans:* Jab container kisi image file ko modify karna chahta hai, toh Docker us file ko image layer (Read-only) se uthakar container layer (Writable) mein copy karta hai.

**Q12: Volume drivers kya hote hain?**
*Ans:* Ye Docker ko allow karte hain ki wo external storage (NFS, AWS EBS, Azure Files) ko as a volume use kare.

**Q13: Why is database performance better on Volumes than Bind Mounts?**
*Ans:* Volumes Docker ke optimized area mein hoti hain aur host OS ke file system features ko directly use karti hain, bina kisi permission mapping overhead ke.

**Q14: How to backup a Docker volume?**
*Ans:* Ek temporary container chala kar volume ko mount karein aur use `tar` command se compress karke host folder (via bind mount) mein save kar lein.

**Q15: Can we change the location where Docker stores volumes?**
*Ans:* Haan, `/etc/docker/daemon.json` file mein `data-root` ki value change karke hum Docker ka pura storage location badal sakte hain.

---
**Next Topic:** Phase 5, Topic 10: Networking Fundamentals.
