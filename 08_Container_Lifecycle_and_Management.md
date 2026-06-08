# Topic 8: Container Lifecycle & Advanced Management

Containers "ephemeral" (temporary) hote hain, lekin unka management production mein bahut critical hota hai. Is topic mein hum container ki life aur usse control karne ke advanced tarike seekhenge.

---

### 1. Container States (Lifecycle)
Ek container apni life mein in states se guzarta hai:

1.  **Created:** `docker create` se container ban jata hai par uska main process chalu nahi hota.
2.  **Running:** `docker run` ya `docker start` se process active ho jata hai.
3.  **Paused:** `docker pause` se process ko "freeze" kar diya jata hai.
4.  **Exited (Stopped):** Process khatam ho gaya (Ya toh kaam pura ho gaya ya error aaya).
5.  **Restarting:** Agar container crash hua aur `restart policy` set hai.
6.  **Dead:** Jab container delete ho raha ho ya koi major system error ho.

---

### 2. Signal Handling: SIGTERM vs SIGKILL

Jab aap `docker stop` ya `docker kill` chalate hain, toh Docker container ko "Signals" bhejta hai.

- **SIGTERM (Signal 15):** Ye "Gentle" tarika hai. Docker app ko bolta hai, "Bhai, band hone ka time aa gaya hai, apna kaam wrap up kar lo."
    - App logs save kar sakta hai, database connections close kar sakta hai.
    - `docker stop` pehle SIGTERM bhejta hai aur 10 seconds wait karta hai.
- **SIGKILL (Signal 9):** Ye "Brutal" tarika hai. Isme app ko kuch bhi karne ka mauka nahi milta, system turant process kill kar deta hai.
    - `docker kill` direct SIGKILL bhejta hai.
    - Agar app 10 sec mein stop nahi hui, toh `docker stop` bhi SIGKILL bhej deta hai.

---

### 3. Why PID 1 is Special? (The Init Problem)

Linux mein **PID 1** (init process) ki bahut izzat hoti hai. Wo baaki saare processes ka "baap" hota hai.

- **Normal Linux:** Init process signals (SIGTERM) ko handle karta hai aur unhe child processes ko pass karta hai.
- **Docker Container:** Aapki application PID 1 ban jati hai. Agar aapki app (jaise Python ya Java) signals handle karna nahi jaanti, toh `docker stop` kaam nahi karega, aur container 10 seconds baad force-kill hoga.
- **The Zombie Issue:** PID 1 ki zimmedari hoti hai "Zombie" processes ko clean karna. Agar aapki app ye nahi karti, toh container ki memory bhar jayegi.

**Solution:** `docker run --init` use karein. Ye `tini` ko PID 1 bana deta hai jo signals aur zombies ko sahi se handle karta hai.

---

### 4. Advanced Management Commands

#### A. Resource Monitoring (`docker stats`)
```bash
docker stats --no-stream
```

#### B. Deep Inspection (`docker inspect`)
```bash
docker inspect -f '{{range .NetworkSettings.Networks}}{{.IPAddress}}{{end}}' container_name
```

#### C. Executing Commands (`docker exec`)
```bash
docker exec -u root -it my_container bash
```

---

### 5. Container Restart Policies
- **no:** Default.
- **always:** Hamesha restart hoga.
- **unless-stopped:** Manual stop par restart nahi hoga.
- **on-failure:** Sirf error par restart hoga.

---

### 💡 Beginner's Corner
- **Tip:** Hamesha `docker stop` use karein `docker kill` ki jagah, taaki aapki app ko clean-up ka time mile.
- **Common Confusion:** `docker start` aur `docker run` alag hain. `run` naya container banata hai, `start` purane band pade container ko jagata hai.
- **Must Know:** Agar container turant exit ho raha hai, toh `docker logs` check karein.

---

### 👨‍💻 Senior's Perspective
"Production mein PID 1 issue bahut common hai. Hamesha entrypoint scripts mein `exec` command use karein (e.g., `exec python app.py`). Isse shell process hat jata hai aur aapki app direct PID 1 ban jati hai, jisse signal handling sahi chalti hai. Aur haan, health checks zaroor likhein, sirf process chalne ka matlab ye nahi ki app sahi kaam kar rahi hai."

---

### 15 Production-Ready Interview Questions (Q&A)

**Q1: `docker run` aur `docker create` mein kya difference hai?**
*Ans:* `docker run` create aur start dono karta hai. `docker create` sirf container config setup karta hai.

**Q2: Status 137 ka kya matlab hai?**
*Ans:* Container OOM (Out of Memory) kill hua hai ya manual `docker kill` kiya gaya hai.

**Q3: `docker pause` aur `docker stop` mein kya farak hai?**
*Ans:* `pause` CPU freeze karta hai (memory wahi rehti hai), `stop` process terminate kar deta hai.

**Q4: Container ke andar `top` command chalane par host ke processes kyun nahi dikhte?**
*Ans:* PID Namespace isolation ki wajah se.

**Q5: "Docker Events" command ka kya use hai?**
*Ans:* Real-time monitoring ke liye (container start, stop, die events dekhne ke liye).

**Q6: Kya chalte hue container ki RAM limit change ho sakti hai?**
*Ans:* Haan, `docker update --memory 1g container_id`.

**Q7: `docker attach` se bahar kaise niklein bina container stop kiye?**
*Ans:* `Ctrl+P + Ctrl+Q` (Detach shortcut).

**Q8: Health check aur Docker status mein kya difference hai?**
*Ans:* Status sirf process check karta hai, Health check app ki internal logic (e.g., DB connection) check karta hai.

**Q9: Zombie processes container mein kaise bante hain?**
*Ans:* Jab PID 1 child processes ke exit signals (SIGCHLD) ko reap (clean) nahi karta.

**Q10: `docker top` command kya dikhati hai?**
*Ans:* Container ke andar chal rahe actual processes ki list.

**Q11: SIGTERM milne par ek achhi application ko kya karna chahiye?**
*Ans:* New requests lena band karein, current requests puri karein, DB connection close karein, aur phir exit karein (Graceful Shutdown).

**Q12: Dockerfile mein `ENTRYPOINT ["python", "app.py"]` aur `ENTRYPOINT python app.py` mein kya farak hai?**
*Ans:* Pehla wala (Exec Form) app ko PID 1 banata hai. Dusra wala (Shell Form) `/bin/sh -c` chalata hai, jisse shell PID 1 ban jata hai aur app ko signals nahi milte.

**Q13: Why is `--init` flag recommended for heavy applications?**
*Ans:* Kyunki ye ek lightweight init system (tini) add karta hai jo signals ko correctly forward karta hai aur zombie processes ko automatically clean karta hai.

**Q14: Docker container 'Stop' hone par data kyun nahi udta?**
*Ans:* Kyunki container ka Writable Layer tab tak rehti hai jab tak container 'Delete' (`rm`) na kiya jaye. Stop sirf process ko band karta hai.

**Q15: How can you see why a container crashed?**
*Ans:* `docker inspect container_id` mein `State.Error` aur `State.ExitCode` check karein, aur `docker logs --tail 50` dekhein.

---
**Next Topic:** Phase 4, Topic 9: Docker Storage (Persistence).
