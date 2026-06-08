# Topic 18: Troubleshooting 101

Production mein bugs aana lazmi hai. Ek Senior DevOps engineer wo hai jo problem ko dekh kar turant samajh jaye ki kahan check karna hai. Is topic mein hum Docker ke common errors aur unka solution seekhenge.

---

### 1. The Debugging Framework (Step-by-Step)
Jab container kaam na kare, toh ye order follow karein:

1.  **Check Status:** `docker ps -a` (Kya container chal raha hai ya exit ho gaya?)
2.  **Check Logs:** `docker logs --tail 100 <container>` (App ke andar kya error aaya?)
3.  **Inspect Metadata:** `docker inspect <container>` (IP, Ports, Environment variables sahi hain?)
4.  **Live Stats:** `docker stats` (Kya RAM/CPU ki wajah se crash ho raha hai?)
5.  **Inside View:** `docker exec -it <container> sh` (Container ke andar se network/files check karo).

---

### 2. Common Errors & Solutions

| Error | Possible Cause | Fix |
| :--- | :--- | :--- |
| **Exited (137)** | Out of Memory (OOM) | RAM limit badhayein ya app ka memory leak check karein. |
| **Exited (1)** | Application Error | Logs dekhein, code mein koi bug ya missing dependency hai. |
| **Exec format error** | Architecture mismatch | Agar Mac M1/M2 par build karke Linux Intel par chala rahe hain. Use `--platform linux/amd64`. |
| **Connection Refused** | Port mapping issue | Check karein ki app sahi port par listen kar rahi hai aur mapping sahi hai. |
| **No space left on device** | Disk Full | `docker system prune -a` chalayein aur logs/volumes saaf karein. |

---

### 3. Networking Troubleshooting
Agar do containers baat nahi kar paa rahe:
1.  Check karein ki dono same **Network** mein hain ya nahi (`docker network inspect`).
2.  Container ke andar se `ping <other_container_name>` karke dekhein.
3.  Check karein ki firewall (iptables) host machine par traffic block toh nahi kar rahi.

---

### 4. Advanced: Debugging 'veth' interfaces on the host
Jab aap container banate hain, Docker host par ek virtual interface banata hai (veth - virtual ethernet).
- **Find veth for a container:**
  ```bash
  # Container ke andar ka index pata karein
  docker exec <container_id> cat /sys/class/net/eth0/iflink
  # Host par wo index wala interface dhundhein
  ip ad | grep <index>
  ```
- **Why?** Agar networking slow hai ya packet drop ho rahe hain, toh aap host par `tcpdump -i vethxxxx` chala kar traffic monitor kar sakte hain.

---

### 5. Docker Events (Real-time tracking)
Agar koi container achanak band ho raha hai, toh naya tab khol kar ye chalayein:
```bash
docker system events
```
Ye aapko batayega ki exactly kis waqt par `die`, `kill`, ya `oom` event trigger hua. Isme aap `--filter` bhi laga sakte hain: `docker events --filter 'container=myapp'`.

---

### 🚨 Production Crisis Checklist (Quick Fix)
Jab production phata ho (Outage), toh ye 5 cheezein turant check karein:
1.  **Disk Space:** `df -h` (Kya Docker root partition full hai?)
2.  **RAM/Swap:** `free -m` (Kya system swapping kar raha hai?)
3.  **Container State:** `docker ps | grep -v Up` (Kaunse containers baar-baar restart ho rahe hain?)
4.  **Zombie Processes:** `ps aux | grep 'defunct'` (Kya container ke andar processes clean-up nahi ho rahe?)
5.  **External Dependencies:** Kya DB ya API down hai jise container connect kar raha hai?

---

### 💡 Beginner's Corner
> "Beta, jab bhi error aaye, ghbrana mat. Sabse pehle `docker logs` dekho. 90% errors wahin mil jate hain. Agar wahan kuch na dikhe, tabhi aage inspect ya exec par jao. Simple rakho, pehle basics check karo!"

### 👨‍💻 Senior's Perspective
> "Production debugging ek art hai. Hamesha `docker stats` aur `docker events` ka terminal side mein khula rakho. Agar container random exit ho raha hai bina kisi log ke, toh samajh jao ki Kernel's OOM Killer ne usse mara hai. Iske liye `dmesg | grep -i oom` check karna sikh lo."

---

### 15 Production-Ready Interview Questions (Q&A)

**Q1: Container "Exited (0)" aur "Exited (1)" mein kya farak hai?**
*Ans:* "Exited (0)" ka matlab hai container ne apna kaam successfully khatam kiya aur band ho gaya. "Exited (1)" (ya koi bhi non-zero code) ka matlab hai ki koi error aaya hai.

**Q2: App container ke andar chal rahi hai, par bahar se access nahi ho rahi. Kya check karenge?**
*Ans:* 1. Port mapping (`docker ps`), 2. App ka listening IP (0.0.0.0 hona chahiye, 127.0.0.1 nahi), 3. Host machine ka firewall/security group.

**Q3: "Docker system prune" command kya-kya delete karti hai?**
*Ans:* Stopped containers, unused networks, dangling images, aur build cache. (Note: Volumes delete nahi hote jab tak `--volumes` flag na lagaya jaye).

**Q4: Container ke andar `curl` ya `ping` nahi mil raha. Kaise debug karenge?**
*Ans:* Bahut si production images (Alpine/Slim) mein ye tools nahi hote. Aapko temporarily install karne padenge (`apk add curl` ya `apt update && apt install curl`) ya multi-stage build mein debug tools add karne chahiye.

**Q5: Host machine restart hone ke baad containers nahi chale. Kyun?**
*Ans:* Kyunki un par `restart policy` set nahi thi. `always` ya `unless-stopped` use karna chahiye.

**Q6: "Image pull backoff" (K8s concept in Docker context) kyun hota hai?**
*Ans:* 1. Wrong image name/tag, 2. Private registry login issue, 3. Network issue, 4. Rate limit (Docker Hub).

**Q7: Ek container bahut zyada Disk I/O use kar raha hai. Kaise pata lagayenge?**
*Ans:* `docker stats` command se "BLOCK I/O" column check karke.

**Q8: `/var/lib/docker` folder poora bhar gaya hai. Kya karein?**
*Ans:* 1. Purani images aur containers delete karein, 2. Log rotation enable karein, 3. Unused volumes delete karein (`docker volume prune`).

**Q9: Container ke andar environment variables sahi se apply hue ya nahi, kaise dekhein?**
*Ans:* `docker exec <container> env` command se.

**Q10: Docker daemon response nahi de raha (`Cannot connect to the Docker daemon`). Pehla step kya hoga?**
*Ans:* Check karein ki docker service chal rahi hai ya nahi: `sudo systemctl status docker`. Agar nahi, toh restart karein: `sudo systemctl restart docker`.

**Q11: 'Zombie Process' container mein kyun bante hain aur kaise rokein?**
*Ans:* Jab PID 1 (main process) apne child processes ko 'reap' nahi karta. Solution: Docker run mein `--init` flag use karein ya `tini` ko entrypoint banayein.

**Q12: `docker inspect` se container ka IP address kaise nikalte hain (one liner)?**
*Ans:* `docker inspect -f '{{range .NetworkSettings.Networks}}{{.IPAddress}}{{end}}' <container_id>`

**Q13: Container ke andar 'Read-only file system' error kyun aata hai?**
*Ans:* Ya toh aapne container ko `--read-only` flag se chalaya hai, ya host machine ka file system corrupt ho gaya hai aur OS ne usse safety ke liye read-only mode mein daal diya hai.

**Q14: Docker logs bahut zyada disk space kha rahe hain. Permanent solution kya hai?**
*Ans:* `daemon.json` mein `log-driver` aur `log-opts` (max-size, max-file) configure karein taaki logs rotate hote rahein.

**Q15: 'docker cp' command ka debugging mein kya use hai?**
*Ans:* Agar aapka container crash ho gaya hai aur aapko uske andar ki koi config file ya log file dekhni hai, toh `docker cp` se aap usse host machine par copy karke examine kar sakte hain.

---
**Course Conclusion:** Aapka Docker Mastery course ab complete ho gaya hai!
