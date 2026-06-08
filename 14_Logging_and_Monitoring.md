# Topic 14: Logging & Monitoring

Production mein jab aapke paas 100s of containers chal rahe hote hain, toh ye pata lagana ki kaunsa container crash hua aur kyun, sirf **Logging** aur **Monitoring** se hi possible hai.

---

### 1. Docker Logging Drivers
Docker by default containers ke `stdout` aur `stderr` ko capture karta hai. Inhe handle karne ke liye alag-alag drivers hote hain:

| Driver | Description | Use Case |
| :--- | :--- | :--- |
| **json-file** | Default. Logs ko JSON files mein store karta hai host par. | Local development, small setups. |
| **syslog** | Logs ko host ke syslog daemon ko bhejta hai. | Centralized Linux logging. |
| **journald** | Systemd journal mein logs store karta hai. | Modern Linux distributions. |
| **awslogs / fluentd** | External services ko logs bhejta hai. | Large scale production / ELK Stack. |

---

### 2. Log Rotation (Disk Space Bachane ke liye)
Agar aap default `json-file` driver use karte hain, toh logs ka size badhta jayega aur server ki disk full ho sakti hai. Isse bachne ke liye `daemon.json` mein rotation set karein:

```json
{
  "log-driver": "json-file",
  "log-opts": {
    "max-size": "10m",
    "max-file": "3"
  }
}
```
*Iska matlab: Har log file max 10MB ki hogi, aur purani sirf 3 files hi rakhi jayengi.*

---

### 3. Monitoring Containers
Monitoring ke do tarike hote hain:

#### A. Basic Monitoring (CLI)
- **`docker stats`**: Live CPU, Memory, Network usage dekhne ke liye.
- **`docker top <container>`**: Container ke andar ke processes dekhne ke liye.
- **`docker events`**: Docker engine ki real-time activity track karne ke liye.

#### B. Production Monitoring (Advanced)
Production mein hum ek **Monitoring Stack** use karte hain:
- **cAdvisor (Container Advisor):**
  - Ye Google ka tool hai jo har container ki resource usage (CPU, Memory, Network, Disk) ko analyze karta hai.
  - Ye automatically host par chal rahe saare containers ka data collect karta hai aur ek `/metrics` endpoint par expose karta hai.
  - Prometheus isi endpoint se data read karta hai.
- **Prometheus:** Un metrics ko store karta hai (Time-series database).
- **Grafana:** Metrics ko sunder Dashboards mein dikhata hai.

---

### 4. Centralized Logging: ELK Stack
Jab 100 containers ho, toh har ek par ja kar `docker logs` dekhna impossible hai. Isliye hum **ELK Stack** use karte hain:

1.  **Elasticsearch:** Ek search engine jo saare logs ko store karta hai aur unhe fast search karne ki power deta hai.
2.  **Logstash / Fluentd:** Ye "Log Shipper" hote hain. Ye containers se logs collect karte hain, unhe format (e.g., parsing JSON) karte hain, aur Elasticsearch ko bhejte hain.
3.  **Kibana:** Ek dashboard jahan aap logs ko search, filter aur visualize kar sakte hain.

*Pro Tip: Docker logs ke liye **Filebeat** ka use karna bahut popular hai kyunki ye lightweight hai.*

---

### 5. Application Health Checks
Sirf container ka "Up" hona kafi nahi hai. Dockerfile mein `HEALTHCHECK` use karein:
```dockerfile
HEALTHCHECK --interval=30s --timeout=3s \
  CMD curl -f http://localhost/ || exit 1
```
*Isse Docker ko pata chalta hai ki container ke andar ki app actually kaam kar rahi hai ya "Unhealthy" hai.*

---

### 💡 Perspectives

#### 🟢 Beginner's Corner
"Logging aur Monitoring ka matlab hai 'pata hona ki kya ho raha hai'. Shuruat mein `docker stats` aur `docker logs -f` se dosti karo. Ye aapke best friends hain jab code phat-ta hai. Log rotation hamesha set karo, warna ek din server ki disk full ho jayegi aur aapko samajh nahi aayega ki kya hua!"

#### 🔴 Senior's Perspective
"Real-world production mein, hum **Sidecar Pattern** use karte hain logging ke liye. Logs ko kabhi bhi container ke andar file mein mat likho, hamesha `stdout` par bhejo. Monitoring sirf alerts ke liye nahi, balki **Capacity Planning** ke liye bhi hai. Agar aapki app 90% RAM use kar rahi hai, toh aapko auto-scaling ki zaroorat hai. Use **Loki** if you want a lightweight alternative to ELK."

---

### 15 Production-Ready Interview Questions (Q&A)

**Q1: Docker logs ki wajah se server disk full hone se kaise bacha jaye?**
*Ans:* Log rotation configure karke. `max-size` aur `max-file` options set karke hum purane logs ko automatically delete kar sakte hain.

**Q2: `docker logs` command sirf `stdout` aur `stderr` kyun dikhati hai?**
*Ans:* Kyunki Docker container ke main process (PID 1) ke standard output streams ko listen karta hai. Agar apki app kisi custom file (jaise `/var/log/app.log`) mein likh rahi hai, toh `docker logs` wo nahi dikhayega.

**Q3: ELK Stack (Elasticsearch, Logstash, Kibana) ka Docker mein kya kaam hai?**
*Ans:* Ye centralized logging ke liye use hota hai. Saare containers ke logs ek jagah collect karke search aur analyze karne mein madad karta hai.

**Q4: "json-file" driver ka sabse bada drawback kya hai?**
*Ans:* Ye host machine ki disk space consume karta hai aur logs ko analyze karna mushkil hota hai agar containers ka number bahut zyada ho.

**Q5: `docker stats` mein "PIDs" column kya represent karta hai?**
*Ans:* Ye batata hai ki us container ke andar total kitne active processes ya threads chal rahe hain.

**Q6: Healthcheck mein "Retries" option ka kya matlab hai?**
*Ans:* Agar healthcheck fail hota hai, toh Docker usse kitni baar phir se try karega "Unhealthy" mark karne se pehle.

**Q7: Container "Healthy" se "Unhealthy" hone par Docker kya karta hai?**
*Ans:* Docker engine khud kuch nahi karta, lekin Docker Compose (V3.4+) ya Kubernetes is info ka use karke container ko restart ya replace kar sakte hain.

**Q8: Fluentd ya Logspout jaise tools kyun use kiye jate hain?**
*Ans:* Ye "Log Collectors" hain. Ye Docker socket se logs read karte hain aur unhe external services (jaise CloudWatch ya Splunk) ko bhejte hain bina image change kiye.

**Q9: Kya hum container ke andar jaye bina uske logs monitor kar sakte hain?**
*Ans:* Haan, `docker logs -f <container_name>` se hum host machine se hi real-time logs dekh sakte hain.

**Q10: `docker events` command se kis tarah ke events track kiye ja sakte hain?**
*Ans:* Container attach, commit, copy, create, destroy, die, exec_create, pause, restart, start, stop, etc.

**Q11: cAdvisor metrics ko scrape karne ke liye Prometheus hi kyun use hota hai?**
*Ans:* Kyunki cAdvisor metrics ko Prometheus format mein expose karta hai, aur Prometheus ek powerful time-series database hai jo aggregation aur alerting (Alertmanager) handle kar sakta hai.

**Q12: ELK aur EFK stack mein kya difference hai?**
*Ans:* ELK mein 'L' stands for Logstash. EFK mein 'F' stands for Fluentd. Fluentd zyada lightweight hai aur containerized environments mein aksar prefer kiya jata hai.

**Q13: "Pull-based" vs "Push-based" monitoring kya hai?**
*Ans:* Prometheus "Pull-based" hai (wo cAdvisor se data khichta hai). ELK "Push-based" hai (Logstash data ko Elasticsearch mein push karta hai).

**Q14: Docker logs ko `syslog` par bhejna kyun secure mana jata hai?**
*Ans:* Kyunki agar host compromise hota hai ya disk crash hoti hai, toh logs remote syslog server par safe rahte hain. Host par logs ki copy nahi bachti.

**Q15: Grafana dashboards mein "Alerting" set karna kyun zaroori hai?**
*Ans:* Kyunki koi bhi 24/7 dashboards nahi dekhta. Alerts (e.g., Slack ya Email) aapko tabhi notify karte hain jab koi metric (jaise High CPU) threshold cross karti hai.

---
**Next Topic:** Phase 8, Topic 15: CICD Integration (Docker in Pipelines).
