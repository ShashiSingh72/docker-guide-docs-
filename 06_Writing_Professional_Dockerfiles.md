# Topic 6: Writing Professional Dockerfiles

Dockerfile ek simple text file hoti hai jisme instructions likhe hote hain ki humari image kaise banegi. Isse "Blue-print" samajh lijiye.

---

### 1. Common Dockerfile Instructions

| Instruction | Meaning | Production Tip |
| :--- | :--- | :--- |
| **FROM** | Base image set karta hai (e.g., `FROM node:18`) | Hamesha specific version use karein, `latest` nahi. |
| **WORKDIR** | Container ke andar ka working directory set karta hai. | Hamesha use karein taaki files scattered na hon. |
| **COPY** | Host se files container mein copy karta hai. | Sirf zaroori files hi copy karein. |
| **ADD** | COPY jaisa hi hai, par ye URL se files download kar sakta hai aur tar files ko extract kar sakta hai. | Zyada tar **COPY** hi use karein. |
| **RUN** | Image build karte waqt commands chalata hai (e.g., `apt install`). | Multiple RUN commands ko `&&` se jodein taaki layers kam banein. |
| **ENV** | Environment variables set karta hai. | Secrets (passwords) ke liye use na karein. |
| **ARG** | Build-time variables (Image banne ke baad khatam ho jate hain). | Version numbers ke liye best hai. |
| **EXPOSE** | Documentation ke liye ki container kaunsa port use karega. | Ye actually port open nahi karta, sirf batata hai. |
| **CMD** | Container start hone par kaunsi command chalegi. | Isse `docker run` se override kiya ja sakta hai. |
| **ENTRYPOINT** | Container ka main executable set karta hai. | Isse override karna mushkil hota hai. |

---

### 2. CMD vs ENTRYPOINT (The Interview King Question)

Bahut log isme confuse hote hain. Simple shabdon mein:

*   **CMD:** Ye default arguments deta hai. Agar user `docker run image command` likhta hai, toh CMD override ho jata hai.
*   **ENTRYPOINT:** Ye "fixed" hota hai. Aap jo bhi `docker run` mein likhenge, wo ENTRYPOINT ke peeche as an argument lag jayega.

**Best Practice:** Dono ko saath use karein!
```dockerfile
ENTRYPOINT ["python"]
CMD ["app.py"]
```
*Ab agar aap sirf `docker run` chalayenge toh `python app.py` chalega. Lekin agar aap `docker run image test.py` chalayenge, toh `python test.py` chalega.*

---

### 3. Practical Example: Complex Dockerfile (Line-by-Line Breakdown)

Doston, real world mein Dockerfile aisi dikhti hai. Isse dhyan se samjho:

```dockerfile
# Stage 1: Build Stage
FROM node:18-alpine AS build_stage
WORKDIR /app
COPY package*.json ./
RUN npm install
COPY . .
RUN npm run build

# Stage 2: Final Production Stage
FROM nginx:stable-alpine
COPY --from=build_stage /app/dist /usr/share/nginx/html
EXPOSE 80
CMD ["nginx", "-g", "daemon off;"]
```

**Line-by-Line Breakdown:**
1.  **`FROM node:18-alpine AS build_stage`**: Humne Node.js image li aur use ek naam de diya `build_stage`. Alpine version liya taaki size chota rahe.
2.  **`WORKDIR /app`**: Container ke andar `/app` folder banaya aur wahan shift ho gaye.
3.  **`COPY package*.json ./`**: Pehle sirf package files copy ki. Kyun? Taaki agar code badle par dependencies na badle, toh Docker `npm install` ko cache se utha le. **(Caching Strategy)**.
4.  **`RUN npm install`**: Dependencies install ki.
5.  **`COPY . .`**: Ab poora source code copy kiya.
6.  **`RUN npm run build`**: App ko build kiya (e.g., React/Angular build).
7.  **`FROM nginx:stable-alpine`**: **Yahan magic hai!** Humne ek nayi, fresh image li Nginx ki. Purani saari layers (node_modules etc.) ab final image mein nahi aayengi.
8.  **`COPY --from=build_stage /app/dist /usr/share/nginx/html`**: Humne `build_stage` se sirf build folder (dist) uthaya aur Nginx ke folder mein daal diya.
9.  **`EXPOSE 80`**: Bataya ki port 80 use hoga.
10. **`CMD ["nginx", "-g", "daemon off;"]`**: Nginx ko foreground mein chalaya.

---

### 4. What is 'Build Context'? (docker build .)

Jab aap chalate ho `docker build -t myapp .`, toh wo aakhiri wala **dot (.)** bahut important hai.
*   Iska matlab hai: "Current directory ko as a **Context** Docker Daemon ko bhej do."
*   Docker pehle aapki saari files ko ek tar ball banata hai aur Docker Engine ko bhejta hai.
*   **Danger:** Agar aapki directory mein 2GB ki unwanted files hain, toh `docker build` bahut slow ho jayega, bhale hi aap unhe COPY na kar rahe hon.
*   **Solution:** `.dockerignore` file banayein aur usme `node_modules`, `.git` jaise folders daal dein.

---

### 💡 Beginner's Corner
Naye log aksar `ADD` use karte hain har jagah. Meri advice: **Bhul jao ki ADD exist karta hai.** 99% cases mein `COPY` hi sahi hai. `ADD` tabhi use karo jab URL se file lani ho ya tar file extract karni ho. Keep it simple!

### 👨‍💻 Senior's Perspective
Ek senior hamesha "Layer Ordering" par dhyan deta hai. Jo cheez sabse kam badalti hai (OS, Runtimes), wo upar. Jo sabse zyada badalti hai (Your Code), wo sabse niche. Isse developer ka build time 5 minute se 5 second par aa sakta hai!

---

### 5. Build Process CLI
Jab Dockerfile taiyar ho jaye, toh image kaise banayein?

```bash
# -t means Tag (naam dena)
# . (dot) means current directory mein Dockerfile hai
docker build -t my-custom-app:v1 .
```

---

### 6. Important Tips for Professional Dockerfiles:
1.  **Order Matters:** Sabse kam badalne wali cheezein (Base image, dependencies) upar rakhein, aur jo code roz badalta hai usse niche. Isse **Docker Caching** ka poora fayda milta hai.
2.  **Shell vs JSON Form:**
    - Shell form: `CMD npm start` (Behtar nahi hai, signals handle nahi karta).
    - JSON form: `CMD ["npm", "start"]` (**Recommended**, production ready).

---

### Interview Questions:
**Q: What is the purpose of `.dockerignore`?**
*Answer:* Jaise `.gitignore` hota hai, `.dockerignore` Docker ko batata hai ki kaunsi files image mein nahi bhejni (jaise `node_modules`, `.git`, logs). Isse image size chota rehta hai aur security badhti hai.

**Q: Can we have multiple CMD instructions in a Dockerfile?**
*Answer:* Likh toh sakte hain, lekin sirf **akhiri wali CMD** hi execute hogi. Baki sab ignore ho jayengi.

---

### 7. Production-Ready Interview Questions (Q&A)

**Q1: COPY aur ADD mein se kaunsa kab use karein?**
*Ans:* Hamesha **COPY** use karein local files ke liye. **ADD** sirf tab use karein jab aapko kisi URL se file download karni ho ya `.tar.gz` file ko auto-extract karna ho.

**Q2: WORKDIR use karna kyun zaroori hai?**
*Ans:* Kyunki agar aap `cd` use karte hain RUN command mein, toh wo sirf us layer tak rehta hai. WORKDIR aane wali saari commands ka base directory set kar deta hai.

**Q3: Shell form (`CMD node app.js`) aur Exec form (`CMD ["node", "app.js"]`) mein kya farak hai?**
*Ans:* Shell form `/bin/sh -c` chalta hai, jo signals (SIGTERM) ko app tak nahi pahunchne deta. **Exec form** direct binary chalta hai aur production ke liye best hai.

**Q4: ARG aur ENV mein kya difference hai?**
*Ans:* **ARG** sirf image build ke waqt available hota hai. **ENV** container chalte waqt bhi available hota hai.

**Q5: `.dockerignore` image security mein kaise madad karta hai?**
*Ans:* Ye `.env` files, SSH keys, aur private logs ko image mein jane se rokta hai, jisse sensitive data leak nahi hota.

**Q6: Multi-line RUN commands mein `&&` kyun use karte hain?**
*Ans:* Taaki saari commands ek hi image layer mein pack ho jayein. Agar alag-alag RUN likhenge, toh har command ek nayi layer banayegi aur image size badh jayega.

**Q7: Base image select karte waqt kya dhyan rakhna chahiye?**
*Ans:* 1. Size (Alpine use karein), 2. Security (Kam vulnerabilities wali image), 3. Compatibility (Dependencies sahi se chalein).

**Q8: LABEL instruction ka use kya hai?**
*Ans:* Ye image mein metadata add karta hai (jaise Maintainer, Version, Description). Isse images ko organize karna asaan hota hai.

**Q9: HEALTHCHECK instruction kyun zaroori hai?**
*Ans:* Ye Docker ko batata hai ki container sirf "Up" nahi hona chahiye, balki uske andar ki service sahi se kaam karni chahiye.

**Q10: Kya hum ek hi Dockerfile mein multiple FROM use kar sakte hain?**
*Ans:* Haan, isse **Multi-stage build** kehte hain. Ye build-time dependencies ko final image se hatane ke liye use hota hai.

**Q11: Dockerfile mein `USER` instruction ka kya importance hai?**
*Ans:* By default Docker containers `root` user se chalte hain jo ki security risk hai. `USER` instruction se hum ek non-root user set kar sakte hain production ke liye.

**Q12: `ONBUILD` instruction kab kaam aati hai?**
*Ans:* Ye tab kaam aati hai jab aap ek aisi image bana rahe ho jo dusre projects ke liye base image banegi. Ye commands tab chalengi jab koi aapki image ko `FROM` mein use karega.

**Q13: Build cache ko force-fully ignore kaise karein?**
*Ans:* `docker build --no-cache` command use karke. Ye tab zaroori hota hai jab aapko sure hona ho ki saari dependencies fresh download hon.

**Q14: Docker build mein `--build-arg` ka use case kya hai?**
*Ans:* Agar aapko build ke waqt koi dynamic value deni hai (jaise API_URL ya Version), toh aap Dockerfile mein `ARG` likh kar build command se value pass kar sakte hain.

**Q15: `STOPSIGNAL` instruction kya karti hai?**
*Ans:* Ye set karti hai ki `docker stop` karne par container ko kaunsa signal bheja jaye (default `SIGTERM` hota hai, par kuch apps `SIGQUIT` maangti hain).

---
**Next Topic:** Phase 3, Topic 7: Image Optimization (Production Grade).

