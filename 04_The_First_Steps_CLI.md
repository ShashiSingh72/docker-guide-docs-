# Topic 4: The First Steps (Essential CLI Commands)

Ab hum actual hands-on shuru karenge. Docker use karne ke liye aapko kuch basic lekin bahut important commands ka pata hona chahiye.

---

### 1. Docker Images Commands
Container chalane se pehle aapko "Image" chahiye hoti hai.

*   **`docker pull <image_name>`**: Docker Hub se image download karne ke liye.
    - Example: `docker pull nginx`
*   **`docker images`**: Aapke system mein jitni images download hain unki list dekhne ke liye.
*   **`docker rmi <image_id/name>`**: Kisi image ko delete karne ke liye.

---

### 2. Running a Container (The Core Command)
Sabse zyada use hone wali command:
`docker run [OPTIONS] IMAGE [COMMAND] [ARG...]`

#### Deep Dive into Flags (Flags ka asli matlab):

*   **`-i` (Interactive):** Iska matlab hai ki container ka STDIN (Standard Input) open rahega. Matlab aap container ko commands bhej sakte ho.
*   **`-t` (TTY):** Ye ek pseudo-terminal allocate karta hai. Bina iske, aapka terminal "dumb" lagega (na sahi se colors dikhenge, na formatting).
*   **`-it` (Commonly used together):** Inhe humesha saath isliye use karte hain taaki humein ek "Interactive Terminal" mile. Bina `-i` ke aap type nahi kar payenge, aur bina `-t` ke aapka screen terminal jaisa behave nahi karega.
*   **`-d` (Detached):** Iska matlab "Background" mein chalao. Jaise hi aap enter maarenge, Docker container ID dekar aapko wapas host machine ke prompt par le aayega. Web servers aur databases ke liye ye mandatory hai.
*   **`-p` (Port Mapping):** `HOST_PORT:CONTAINER_PORT`. Maan lo container ke andar app port 80 par chal rahi hai, par aap bahar se use 8080 par dekhna chahte ho: `-p 8080:80`.
*   **`-v` (Volumes):** `HOST_PATH:CONTAINER_PATH`. Persistent storage ke liye use hota hai. Container delete hone par bhi data host par safe rahega.
*   **`--name`:** Aapke container ko ek human-readable naam dene ke liye. Agar ye nahi doge, toh Docker "boring_darwin" ya "hungry_leakey" jaise random naam de dega.
*   **`--rm`:** Container stop hote hi apne aap delete ho jaye. Debugging ke liye best hai taaki "stopped containers" ka kachra jama na ho.
*   **`-e` (Environment Variables):** Container ke andar config bhejane ke liye. Jaise: `-e DB_PASSWORD=mysecret`.

---

### 3. Container Management Commands
Jab container chal raha ho, toh usse manage kaise karein:

*   **`docker ps`**: Sirf **running** containers dekhne ke liye.
*   **`docker ps -a`**: Saare containers (Running + Stopped) dekhne ke liye.
*   **`docker stop <container_id/name>`**: Chalte hue container ko rokne ke liye.
*   **`docker start <container_id/name>`**: Ruke hue container ko phir se chalane ke liye.
*   **`docker restart <container_id/name>`**: Container ko restart karne ke liye.
*   **`docker rm <container_id/name>`**: Container ko delete karne ke liye (Delete karne se pehle stop karna zaroori hai).
    - Tip: Sabhi stopped containers ek saath delete karne ke liye: `docker container prune`.

---

### 4. Container ke andar kya ho raha hai? (Debugging)
Production mein ye commands roz kaam aati hain:

*   **`docker logs <container_id/name>`**: Container ke output logs dekhne ke liye.
*   **`docker logs -f <container_id>`**: Real-time logs dekhne ke liye (Follow mode).
*   **`docker exec -it <container_id> bash`**: Ek chalte hue container ke andar ghusne ke liye (Best for debugging).
*   **`docker inspect <container_id>`**: Container ki saari detail (IP address, Ports, Mounts) dekhne ke liye.
*   **`docker stats`**: Ye dekhne ke liye ki container kitni RAM aur CPU use kar raha hai.

---

### 🚨 Common Mistakes (Savadhan!)

1.  **Run vs Start ka Confusion:** Naye log har baar `docker run` chalate hain container chalane ke liye. Yaad rakhein, `run` hamesha ek **naya** container banata hai. Agar aapne pehle ek container banaya tha aur wo stop ho gaya hai, toh `docker start` use karein, `run` nahi. Warna aapke paas 50 stopped containers ho jayenge same image ke!
2.  **Mapping wrong ports:** `-p 80:3000` mein log confuse hote hain. Yaad rakho: `Bahar:Andar` (Host:Container).
3.  **Forgetting `-d` for Servers:** Agar aap `nginx` run kar rahe ho bina `-d` ke, toh aapka terminal lock ho jayega. Ctrl+C dabate hi container bhi stop ho jayega!
4.  **Deleting Image before Container:** Aap image tab tak delete nahi kar sakte jab tak koi container (bhale hi stopped ho) usse use kar raha ho. Pehle `docker rm` container, phir `docker rmi` image.

---

### 💡 Beginner's Corner
Doston, ghabrao mat! Shuruat mein bas 3 commands yaad rakho:
1. `docker run -d -p 80:80 --name myapp nginx` (Kuch chalane ke liye)
2. `docker ps` (Dekhne ke liye ki kya chal raha hai)
3. `docker logs -f myapp` (Check karne ke liye ki koi error toh nahi)

### 👨‍💻 Senior's Perspective
Senior devs hamesha `--rm` aur `--name` ka use karte hain debugging ke waqt. Aur haan, containers ko kabhi "VM" mat samajhna. Container tab tak hi zinda hai jab tak uska main process (PID 1) chal raha hai. Agar aapne `docker run ubuntu` chalaya aur koi command nahi di, toh wo turant exit ho jayega kyunki Ubuntu image ka default process (bash) finish ho gaya.

---

### 5. Container Lifecycle (Summary)
1.  **Created:** `docker create` (Container ban gaya par chala nahi).
2.  **Running:** `docker run` ya `docker start`.
3.  **Paused:** `docker pause` (Process temporarily ruk gaya).
4.  **Stopped:** `docker stop` (Process khatam ho gaya).
5.  **Deleted:** `docker rm`.

---

### Interview Tips:
**Q: What is the difference between `docker run` and `docker start`?**
*Answer:* `docker run` ek nayi image se naya container banata hai aur usse start karta hai. `docker start` ek purane stopped container ko phir se chalta hai.

**Q: Difference between `-d` and `-it`?**
*Answer:* `-d` (Detached) background mein chalta hai (jaise Database ya Web Server). `-it` (Interactive Terminal) humein container ke shell se connect kar deta hai.

---

### 6. Production-Ready Interview Questions (Q&A)

**Q1: `docker exec` aur `docker attach` mein kya farak hai?**
*Ans:* `exec` ek naya process chalata hai container ke andar (bash kholne ke liye best hai). `attach` aapko container ke main process (PID 1) se connect kar deta hai.

**Q2: `docker stop` aur `docker kill` mein kya difference hai?**
*Ans:* `stop` container ko `SIGTERM` bhejta hai (graceful shutdown). `kill` seedha `SIGKILL` bhejta hai (achanak band kar dena). Production mein hamesha `stop` use karein.

**Q3: Port mapping `-p 8080:80` ka kya matlab hai?**
*Ans:* Iska matlab hai ki Host machine ka port 8080 Container ke port 80 se map ho gaya hai. Bahar se log 8080 par request bhejenge.

**Q4: Ek crashed container ke logs kaise dekhenge?**
*Ans:* `docker logs <container_id>` se. Container band hone ke baad bhi uske logs Docker storage mein rehte hain jab tak container `rm` na ho jaye.

**Q5: Saare stopped containers ko ek saath delete kaise karein?**
*Ans:* `docker container prune` command se. Ye disk space khali karne ke liye best hai.

**Q6: `docker ps -a` aur `docker ps` mein kya difference hai?**
*Ans:* `ps` sirf wahi dikhata hai jo abhi chal rahe hain. `-a` (all) unhe bhi dikhata hai jo exit ho chuke hain ya stop hain.

**Q7: Restart policy `unless-stopped` ka kya matlab hai?**
*Ans:* Iska matlab hai ki agar server restart ho toh container apne aap start ho jaye, LEKIN agar aapne manually `docker stop` kiya hai toh wo restart na ho.

**Q8: `docker stats` command ka real-time use kya hai?**
*Ans:* Ye ek live dashboard dikhata hai (CPU, Memory, Network I/O). Agar koi container server slow kar raha hai, toh yahan se check kar sakte hain.

**Q9: Container ke andar ki files ko host par kaise copy karein?**
*Ans:* `docker cp <container_id>:/path/to/file ./host/path` command use karke.

**Q10: `docker inspect` se humein kya milta hai?**
*Ans:* Ye poora JSON metadata deta hai—IP address, Environment variables, Volumes, Network settings, aur Health status.

**Q11: `--entrypoint` flag ka kya use hai `docker run` mein?**
*Ans:* Ye Dockerfile mein set kiye gaye default entrypoint ko override karne ke liye use hota hai. Jaise agar image default mein web server chala rahi hai par aapko sirf version check karna hai.

**Q12: `docker run -m 512m` ka kya matlab hai?**
*Ans:* Ye container par memory limit lagata hai (is case mein 512MB). Agar container isse zyada RAM use karne ki koshish karega, toh Docker usse "Out of Memory" (OOM) kill kar sakta hai.

**Q13: `docker top <container_id>` command kya dikhati hai?**
*Ans:* Ye container ke andar chal rahe saare processes ki list dikhati hai (host machine ke view se).

**Q14: Hum ek chalte hue container ko "pause" kyun karte hain?**
*Ans:* Jab humein temporarily CPU resources khali karne hote hain bina container ko stop kiye (stop karne se network connections toot jate hain). Pause karne se process freeze ho jata hai.

**Q15: `--privileged` flag kab use hota hai aur ye dangerous kyun hai?**
*Ans:* Ye container ko host machine ke saare hardware (USB, Hard drives) ka access de deta hai. Ye security ke liye khatarnak hai kyunki container se host ko hack karna asaan ho jata hai.

---
**Next Topic:** Phase 3, Topic 5: Understanding Images & Layers.

