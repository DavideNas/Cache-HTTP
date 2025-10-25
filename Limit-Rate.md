## 🧭 Il Rate Limit

Il **rate limiting** e la **cache** sono due concetti spesso _vicini_ (entrambi servono a migliorare performance e stabilità), ma **non fanno la stessa cosa** e **interagiscono solo in certi punti**.

Vediamo tutto con chiarezza 👇

---

## 🧠 1️⃣ COS’È IL RATE LIMITING

Il **rate limiting** è un sistema che **controlla quante richieste un client può fare in un certo intervallo di tempo**, per evitare abusi o sovraccarichi.

📍Esempio:

> “Massimo 100 richieste per IP ogni 60 secondi”

Se un utente o un bot supera il limite → riceve una risposta tipo:

```
HTTP/1.1 429 Too Many Requests
Retry-After: 30
```

---

## ⚙️ 2️⃣ COSA FA (E COSA NON FA)

| Aspetto                           | Cache                        | Rate Limit                         |
| --------------------------------- | ---------------------------- | ---------------------------------- |
| **Scopo**                         | Evitare richieste ripetitive | Limitare numero di richieste       |
| **Beneficio**                     | Migliora prestazioni         | Protegge da overload / abuso       |
| **Memorizza dati?**               | Sì (risposte, oggetti)       | Sì, ma solo contatori              |
| **Riduce carico server?**         | ✅ Sì                        | ✅ Sì (indirettamente)             |
| **Risponde al posto del server?** | ✅ Sì (cache HIT)            | ❌ No (blocca o ritarda richieste) |

➡️ In pratica:

- la **cache** serve per _servire dati già pronti_
- il **rate limit** serve per _bloccare e regolare il traffico_

---

## 🧩 3️⃣ COME INTERAGISCONO CACHE E RATE LIMIT

Spesso **si usano insieme**, perché si **potenziano a vicenda**:

### 💡 Caso 1 — Cache riduce l’impatto sul rate limit

- Se un client fa 10 richieste identiche → la cache risponde dopo la prima
- Quindi **le altre 9 non contano** sul sistema di rate limit (o lo stressano meno)

### 💡 Caso 2 — Rate limit protegge la cache

- Se un utente spamma richieste diverse per _bypassare la cache_
  → il rate limit entra in gioco e **blocca l’abuso**

📈 In un’architettura ottimale:

```
Client → Rate Limiter → Cache → Backend
```

---

## 🧰 4️⃣ DOVE SI IMPLEMENTA IL RATE LIMIT

### 🔹 A. **Lato server applicativo**

Integrato nel codice dell’app o nel middleware.

**Esempio (Node.js + Express + Redis):**

```js
import rateLimit from "express-rate-limit";
import RedisStore from "rate-limit-redis";

const limiter = rateLimit({
  store: new RedisStore({
    sendCommand: (...args) => redisClient.sendCommand(args),
  }),
  windowMs: 60 * 1000, // 1 min
  max: 100, // 100 richieste/minuto
  message: "Too many requests",
});

app.use("/api/", limiter);
```

📌 Redis viene usato per **memorizzare i contatori per IP o token**.

---

### 🔹 B. **Lato reverse proxy / API gateway**

Molti gateway (Nginx, Kong, Traefik, Envoy, API Gateway AWS, Cloudflare) hanno moduli nativi per il rate limiting.

**Esempio Nginx:**

```nginx
limit_req_zone $binary_remote_addr zone=api_zone:10m rate=10r/s;

server {
  location /api/ {
    limit_req zone=api_zone burst=5 nodelay;
  }
}
```

📌 Qui il rate limit è gestito _prima_ che la richiesta raggiunga l’app.

---

### 🔹 C. **Lato CDN**

CDN come Cloudflare o Akamai possono limitare:

- per IP
- per URL
- per tipo di richiesta

📌 Ottimo per difesa DDoS o flood di richieste.

---

## 🧩 5️⃣ TIPI DI RATE LIMIT

| Tipo                   | Descrizione                                                               | Esempio                              |
| ---------------------- | ------------------------------------------------------------------------- | ------------------------------------ |
| **Fixed window**       | Conteggio per blocco di tempo fisso (es. ogni 60s)                        | 100 req/min                          |
| **Sliding window**     | Media mobile nel tempo → più fluido                                       | 100 req negli ultimi 60s             |
| **Token bucket**       | Ogni richiesta consuma un “token”; i token si rigenerano nel tempo        | 10 req/s con rigenerazione graduale  |
| **Leaky bucket**       | Come un imbuto: richieste entrano a velocità variabile ma escono costanti | Stabilizza il flusso                 |
| **Dynamic / Adaptive** | Adatta il limite in base al carico del server o priorità utente           | 50 req/min per free, 500 per premium |

---

## 💾 6️⃣ RUOLO DI REDIS NEL RATE LIMIT

Redis è **perfetto per gestire i contatori del rate limit**, perché:

- è **in-memory** (rapidissimo)
- supporta **atomicità** (operazioni incrementali sicure)
- può essere **condiviso tra più istanze** dell’app

**Esempio logico:**

```
INCR requests:IP:1
EXPIRE requests:IP:1 60
```

→ Incrementa il contatore e lo resetta ogni 60s.

---

## 🔐 7️⃣ RATE LIMITING E CACHING INSIEME: ARCHITETTURA TIPO

```
[Client]
   ↓
[CDN/Reverse Proxy] → caching statico (Cloudflare/Varnish)
   ↓
[Rate Limiter (Redis/Nginx)]
   ↓
[App Backend + Redis cache per dati]
   ↓
[DB o Elasticsearch]
```

🔸 **Cache**: evita di rifare lavoro inutile
🔸 **Rate limit**: evita che troppi client sovraccarichino i livelli sottostanti

---

## 🧭 8️⃣ In sintesi

| Aspetto               | Rate Limit                                              | Cache                  |
| --------------------- | ------------------------------------------------------- | ---------------------- |
| **Scopo**             | Limitare traffico                                       | Riutilizzare risultati |
| **Dove vive**         | Proxy, app, CDN                                         | Proxy, app, browser    |
| **Dati memorizzati**  | Contatori, timestamp                                    | Contenuti o risposte   |
| **Tecnologia tipica** | Redis, Nginx, API Gateway                               | Redis, Varnish, CDN    |
| **Interazione**       | Si integrano (la cache riduce il bisogno di rate limit) | Complementari          |

---
