# Topic 11: Multi-Container Apps (Docker Compose)

Jab humein ek se zyada containers (jaise Web App, Database, Redis) ko ek saath chalana aur manage karna hota hai, toh har baar `docker run` command likhna mushkil ho jata hai. Iske liye hum **Docker Compose** ka use karte hain.

---

### 1. What is Docker Compose?
Docker Compose ek tool hai jise hum "YAML" file (`docker-compose.yml`) ke through multi-container applications ko define aur run karne ke liye use karte hain. Isse hum "Infrastructure as Code" ki tarah treat kar sakte hain.

---

### 2. The `docker-compose.yml` Structure
Ek basic Compose file aisi dikhti hai:

```yaml
version: '3.8'  # Compose file format version

services:
  web:
    build: .             # Dockerfile se image build karo
    ports:
      - "5000:5000"
    networks:
      - app-net
    depends_on:
      - db               # DB start hone ke baad hi web start ho

  db:
    image: postgres:15-alpine
    environment:
      POSTGRES_PASSWORD: example
    volumes:
      - db-data:/var/lib/postgresql/data
    networks:
      - app-net

networks:
  app-net:               # Custom network (Automatic DNS)

volumes:
  db-data:               # Persistent storage for DB
```

---

### 3. Advanced: Splitting YAML Files & `extends` 📄
Badi applications mein ek hi YAML file bahut lambi ho jati hai. Isse manage karne ke liye hum do methods use karte hain:

#### A. Multiple Compose Files
Aap dev aur prod ki settings alag rakh sakte hain:
- `docker-compose.yml` (Base settings)
- `docker-compose.override.yml` (Dev settings - auto picked)
- `docker-compose.prod.yml` (Prod specific settings)

*Command:* `docker compose -f docker-compose.yml -f docker-compose.prod.yml up -d`

#### B. The `extends` Keyword
Agar aapko ek service ki configuration dusri service mein reuse karni hai (like same environment variables or logging), toh `extends` use karein.

```yaml
# common-services.yml
services:
  base-app:
    image: my-app:latest
    logging:
      driver: "json-file"

# docker-compose.yml
services:
  web:
    extends:
      file: common-services.yml
      service: base-app
    ports:
      - "80:80"
```

---

### 4. Smart Dependencies: `depends_on` + `healthcheck` 🏥
Sirf `depends_on` likhne se ye guarantee nahi milti ki app chal jayegi. Agar DB start ho gaya par "Ready" nahi hua, toh Web app crash ho sakti hai.

**Solution:** Healthchecks use karein.

```yaml
services:
  web:
    image: my-web-app
    depends_on:
      db:
        condition: service_healthy  # Wait for healthcheck to pass

  db:
    image: postgres
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U postgres"]
      interval: 10s
      timeout: 5s
      retries: 5
```
*Ab `web` container tabhi start hoga jab `db` ka healthcheck pass ho jayega.*

---

### 5. Essential Compose Commands
*   **`docker compose up -d`**: Saare containers ko background mein start karne ke liye.
*   **`docker compose down`**: Saare containers, networks, aur images (optional) ko stop aur remove karne ke liye.
*   **`docker compose ps`**: Compose dwara chalaye gaye containers ki list dekhne ke liye.
*   **`docker compose logs -f`**: Saare services ke logs ek saath dekhne ke liye.
*   **`docker compose build`**: Agar aapne Dockerfile mein badlav kiya hai, toh images phir se build karne ke liye.

---

### 🌟 Beginner's Corner
- **Simple Word mein:** Docker Compose ek "Orchestrator" hai jo batata hai kaunsa container kab aur kaise chalega.
- **Tip:** `.env` file ka use karein secrets ke liye. Kabhi bhi passwords seedha YAML file mein na likhein.

---

### 👨‍💻 Senior's Perspective
"Large-scale projects mein, hum `extends` aur `multiple files` ka use karke 'DRY' (Don't Repeat Yourself) principle follow karte hain. Ek aur advanced feature hai **Profiles**. Profiles se aap optional services (jaise monitoring tools or debuggers) ko group kar sakte hain aur unhe tabhi chala sakte hain jab zaroorat ho (`docker compose --profile debug up`)."

---

### 15 Production-Ready Interview Questions (Q&A)

**Q1: `docker-compose up` aur `docker-compose start` mein kya difference hai?**
*Ans:* `up` hamesha naye containers banata hai (agar image/config change hui ho) aur unhe start karta hai. `start` sirf pehle se bane hue stopped containers ko chalu karta hai, wo koi naya configuration apply nahi karta.

**Q2: `depends_on` ka limitation kya hai?**
*Ans:* `depends_on` sirf container ke start hone ka intezar karta hai, lekin ye ensure nahi karta ki container ke andar ki service (jaise Database) puri tarah ready hai. Iske liye humein custom scripts ya healthchecks use karne padte hain.

**Q3: `.env` file aur `environment:` section mein kya relation hai?**
*Ans:* `.env` file values ko store karti hai, aur YAML file unhe `${VAR_NAME}` syntax se access karti hai. Isse hum secrets ko source code se alag rakh sakte hain.

**Q4: `docker-compose down -v` command kya karti hai?**
*Ans:* Ye containers aur networks ke saath-saath **Volumes** ko bhi delete kar deti hai. Isse hamesha saavdhani se use karein kyunki data loss ho sakta hai.

**Q5: Compose file mein `build: .` ka kya matlab hai?**
*Ans:* Iska matlab hai ki Docker ko current directory mein `Dockerfile` dhundni hai aur usse image build karke container chalana hai.

**Q6: Hum `docker-compose` ki jagah `docker compose` (bina hyphen) kyun use karte hain?**
*Ans:* `docker-compose` (with hyphen) ek purana Python-based tool (V1) tha. `docker compose` (V2) ab Docker CLI ka hi ek part hai jo Go language mein likha gaya hai aur zyada fast hai.

**Q7: "Docker Compose Project Name" kya hota hai?**
*Ans:* By default, Docker Compose aapke folder ka naam as a project name use karta hai. Isliye har container ke naam ke aage folder ka naam lag jata hai (e.g., `myapp_web_1`).

**Q8: Compose mein networking kaise kaam karti hai?**
*Ans:* Compose apne aap ek "default" bridge network banata hai aur saare services ko usme dal deta hai. Isse saari services ek dusre ko unke **Service Name** (e.g., `web`, `db`) se access kar sakti hain.

**Q9: `docker-compose.override.yml` ka kya use hai?**
*Ans:* Ye file developer ki local settings (jaise extra port mapping ya volumes) ke liye hoti hai. Docker Compose automatically `docker-compose.yml` aur `override.yml` ko merge kar deta hai.

**Q10: Kya hum Docker Compose ko production mein use kar sakte hain?**
*Ans:* Single-server production ke liye ye kaafi accha hai. Lekin agar aapko multiple servers (Cluster) par scale karna hai, toh Docker Swarm ya Kubernetes zyada behtar options hain.

**Q11: `extends` keyword ka main benefit kya hai?**
*Ans:* Code reusability. Agar aapke paas multiple services hain jo same image, logging, ya environment variables share karti hain, toh aap ek 'base' service define karke baaki sabko extend kar sakte hain.

**Q12: `condition: service_healthy` ka kya role hai?**
*Ans:* Ye `depends_on` ko advanced banata hai. Iska matlab hai ki dependent container tabhi start hoga jab primary container ka healthcheck status 'healthy' ho jayega, sirf 'running' hona kaafi nahi hai.

**Q13: Multiple YAML files use karte waqt merge order kya hota hai?**
*Ans:* Pehle basic file (`-f file1.yml`) load hoti hai, phir naya file uske upar settings overwrite karta hai. Agar same key hai, toh baad wali file ki value final hogi.

**Q14: Compose mein `secrets` aur `configs` ka kya role hai?**
*Ans:* Ye sensitive data (like SSL certs) ko securely mount karne ke liye use hote hain. Production mein, ye images mein hardcode karne se behtar option hai.

**Q15: `docker compose up --scale web=3` command kya karti hai?**
*Ans:* Ye `web` service ke 3 instances chala degi. Dhyaan rahe ki agar aapne host port (like 80:80) map kiya hai, toh ye fail ho jayega kyunki ek port par 3 instances nahi chal sakte (use range like `8080-8082:80`).

---
**Next Topic:** Phase 7, Topic 12: Resource Management (Limits & Quotas).
