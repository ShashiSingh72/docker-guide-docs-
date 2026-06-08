# Topic 5: Understanding Images & Layers

Docker Images ko sirf "executable" samajhna kafi nahi hai. Iske andar ki architecture (Layers) hi isse fast aur efficient banati hai.

---

### 1. What is a Docker Image?
Docker image ek lightweight, standalone, aur executable package hai jisme application ko chalane ke liye sab kuch hota hai:
- Code
- Runtime (jaise Node.js, Python)
- Libraries
- Environment variables
- Config files

**Note:** Image hamesha **Immutable (Read-Only)** hoti hai. Aap chalte hue image ko change nahi kar sakte.

---

### 2. The Layered Architecture (Union File System)
Docker images layers mein banti hain. Jab aap ek Dockerfile likhte hain, toh uski har line ek nayi layer banati hai.

#### 📝 UnionFS: The Magic Behind Layers
Union File System (UnionFS) ek aisi technology hai jo multiple directories (layers) ko ek ke upar ek "stack" karke humein aisa dikhati hai jaise wo ek hi directory ho. 

**Transparent Paper Analogy (Butter Paper Example):**
Sochiye aapke paas 4 butter paper (transparent paper) hain:
1.  Pehle paper par aapne ek ghar ka base banaya.
2.  Dusre paper par aapne khidkiyan banayi aur pehle ke upar rakh diya.
3.  Teesre paper par aapne chhat banayi aur uske upar rakh diya.
4.  Chauthe paper par aapne ek ped banaya aur sabse upar rakh diya.

Jab aap upar se dekhenge, toh aapko ek **poora ghar** dikhega. Lekin asliyat mein wo 4 alag-alag papers hain. Docker images bilkul aisi hi hoti hain!

**Benefits of Layers:**
- **Caching:** Agar aap sirf apna code (Layer 4) change karte hain, toh Docker pehli 3 layers ko cache se utha lega. Isse build time bahut fast ho jata hai.
- **Storage Savings:** Agar 10 images Ubuntu use kar rahi hain, toh Ubuntu ki layer disk par sirf ek hi baar store hogi.

---

### 3. Image Layer vs. Container Layer
Ye interview ka sabse favourite sawal hai.

*   **Image Layers (Read-Only):** Image ki saari layers read-only hoti hain. Inhe "Lower Layers" bhi kehte hain. Ek baar image ban gayi, toh ye pathar ki lakeer hai.
*   **Container Layer (Writeable):** Jab aap `docker run` karte hain, toh Docker image layers ke upar ek patli si **"Writeable Layer"** add kar deta hai.
    - Aap jo bhi files create ya delete karte hain, wo isi layer mein hoti hain.
    - Jab container delete hota hai, ye layer bhi delete ho jati hai. Isi liye containers "ephemeral" (temporary) hote hain.

---

### 4. Copy-on-Write (CoW) Strategy
Agar aapko image ki kisi file ko modify karna hai, toh Docker kaise karta hai?
1. Docker pehle wo file niche wali (Read-only) layers mein dhundta hai.
2. Us file ki ek copy "Container Layer" (Writeable layer) mein banata hai.
3. Phir aap us copy ko modify karte hain.
4. Original file (Image mein) kabhi change nahi hoti, wo niche chupi rehti hai.

#### ⚠️ The "Size" Trap: Files delete karne par size kam kyun nahi hota?
Maan lo Layer 2 mein ek 100MB ki file hai. Ab Layer 3 mein aapne command chalayi `rm file`. 
Aapko lagega image ka size 100MB kam ho jayega, par aisa **nahi** hota!
Kyunki Layer 2 Read-only hai, Docker use touch nahi kar sakta. `rm` command sirf Layer 3 mein ek "Whiteout" file create karti hai jo niche wali file ko **hide** kar deti hai. 
**Result:** Image mein wo 100MB ki file ab bhi hai, plus delete karne ki instruction ki layer alag! Isi liye humesha "Cleanup" same RUN command mein karte hain.

---

### 5. Managing Images CLI
*   **`docker history <image_name>`**: Ye dekhne ke liye ki image kaun-kaun si layers se bani hai.
*   **`docker inspect <image_name>`**: Image ki deep details (Metadata, Layers, Config).
*   **`docker tag <old_name> <new_name>`**: Image ko naya naam ya version (tag) dene ke liye.
    - Example: `docker tag my-app:v1 my-app:latest`

---

### 💡 Beginner's Corner
Naye log aksar puchte hain: "Kya main chalte hue container mein changes karke image update kar sakta hoon?" 
Technically `docker commit` se kar sakte ho, par ye **bahut buri practice** hai. Hamesha Dockerfile update karke nayi image build karo. Container ke andar manual changes kabhi track nahi hote!

### 👨‍💻 Senior's Perspective
Ek senior engineer hamesha image optimization par dhyan deta hai. "Multi-stage builds" use karein taaki final image mein sirf executable binary ho, source code ya build tools nahi. Hamesha `Alpine` ya `Distroless` images ka use karein security aur size ke liye.

---

### Interview Questions:
**Q: Why are Docker images called "Immutable"?**
*Answer:* Kyunki image banne ke baad uski layers ko change nahi kiya ja sakta. Agar kuch change karna hai, toh nayi image banani padegi (Build process).

**Q: What is a "Dangling Image"?**
*Answer:* Wo images jinka koi naam ya tag nahi hota (aksar `<none>:<none>` dikhti hain). Ye tab banti hain jab hum same naam se nayi image build karte hain. Inhe `docker image prune` se saaf kiya ja sakta hai.

---

### 6. Production-Ready Interview Questions (Q&A)

**Q1: Docker image "Immutable" kyun hoti hai?**
*Ans:* Taaki security aur consistency bani rahe. Agar image change nahi hogi, toh hum guarantee de sakte hain ki Testing mein jo chala, wahi Production mein chalega.

**Q2: Union File System (UnionFS) kaise kaam karta hai?**
*Ans:* Ye multiple directories (layers) ko ek ke upar ek rakh kar unhe ek single merged view banata hai. Isse layers share karna asaan ho jata hai.

**Q3: Copy-on-Write (CoW) ka performance par kya asar padta hai?**
*Ans:* Pehli baar file modify karne par thoda time lagta hai kyunki file copy hoti hai, lekin uske baad normal speed rehti hai. Isse disk space bahut bachti hai.

**Q4: Dangling Images (`<none>`) kyun banti hain?**
*Ans:* Jab hum ek hi naam aur tag se dobara image build karte hain, toh purani image se tag hat jata hai aur wo dangling ho jati hai.

**Q5: `docker history` command kahan kaam aati hai?**
*Ans:* Ye dikhati hai ki kis instruction ne kitna size liya. Image optimize karne ke liye ye sabse bada tool hai.

**Q6: Image ID aur Image Digest mein kya farak hai?**
*Ans:* Image ID ek random hash hai. Digest image ke content ka hash hai. Digest zyada secure hota hai kyunki agar content 1 bit bhi badla, digest badal jayega.

**Q7: Overlay2 storage driver ka role kya hai?**
*Ans:* Ye modern Linux systems ka default driver hai jo layers ko combine karta hai. Ye bahut fast aur efficient hai production ke liye.

**Q8: Kya hum ek image layer se dusri image layer mein files delete kar sakte hain?**
*Ans:* Aap command toh likh sakte hain, lekin wo file niche wali layer se delete nahi hogi, sirf upar wali layer mein "hidden" ho jayegi. Size kam nahi hoga.

**Q9: Image Pull Policy `always` ka kya matlab hai?**
*Ans:* Ye aksar Kubernetes mein use hota hai. Iska matlab hai ki har baar container start hone par naya image pull kiya jaye, bhale hi wo local mein ho.

**Q10: Sabse niche wali layer (Base Layer) ko change karne se kya hota hai?**
*Ans:* Uske upar ki saari layers cache invalid ho jayengi aur poori image phir se build karni padegi.

**Q11: "Flat" image kya hoti hai?**
*Ans:* Jab hum `docker export` aur `docker import` use karte hain, toh saari layers merge hokar ek single layer ban jati hai. Isse history khatam ho jati hai par size chota ho sakta hai.

**Q12: Docker image mein 'Manifest' ka kya kaam hai?**
*Ans:* Manifest ek JSON file hai jo batati hai ki image kaunsi layers se bani hai aur kis platform (amd64, arm64) ke liye hai.

**Q13: Squash feature kya hai image building mein?**
*Ans:* Building ke waqt `--squash` flag use karne se Docker saari nayi layers ko ek single layer mein merge kar deta hai. Ye image size kam karne mein help karta hai.

**Q14: Docker images disk par kahan store hoti hain?**
*Ans:* Linux par ye `/var/lib/docker/overlay2` directory mein store hoti hain. Har layer ka apna folder hota hai.

**Q15: Multi-architecture images kaise banti hain?**
*Ans:* `docker buildx` tool ka use karke. Isse aap ek hi tag ke andar multiple platforms (Windows, Linux, ARM) ke liye images push kar sakte ho.

---
**Next Topic:** Phase 3, Topic 6: Writing Professional Dockerfiles.

