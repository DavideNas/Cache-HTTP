## Software per il caching

Finora abbiamo parlato della **cache HTTP** (cioè quella controllata dagli header e gestita da browser/CDN), ma adesso entriamo nel mondo dei **software di caching lato server/backend**, come **Redis, Memcached, Varnish** e simili.

Questi non si occupano di header HTTP direttamente, ma di **memorizzare i dati o le risposte** per velocizzare l’elaborazione lato server o applicazione.

---

## 🧠 CONCETTO BASE

La **cache applicativa o server-side** è un livello intermedio tra:

- **la sorgente dei dati (DB, API, filesystem)**
- **e il livello di servizio (app, API, web server)**

Il suo scopo è **evitare di rifare calcoli o query costose** memorizzando temporaneamente i risultati.

---

## ⚙️ 1️⃣ **Redis** (la più diffusa)

### 📍 Cos’è

Redis è un **data store in-memory** estremamente veloce (memoria RAM, non disco), usato per:

- cache
- sessioni utente
- queue (code di messaggi)
- rate limiting

Può anche persistere su disco, ma il suo punto forte è la **bassa latenza (<1ms)**.

### 🧩 Tipi di caching che può gestire

| Tipo                  | Descrizione                                          | Esempio                            |
| --------------------- | ---------------------------------------------------- | ---------------------------------- |
| **Application cache** | Salvi il risultato di una query o un calcolo         | `"user:42" → { name: "Luca" }`     |
| **Session cache**     | Memorizzi lo stato utente                            | `"session:abcd" → logged_in=true`  |
| **Page cache**        | Salvi intere risposte HTTP (usato con reverse proxy) | `"GET:/home"` → `<html>...</html>` |
| **Distributed cache** | Più server condividono la stessa cache               | Redis come “central store”         |

### 💡 Esempio d’uso (Node.js + Redis)

```js
const redis = require("redis");
const client = redis.createClient();

app.get("/users/:id", async (req, res) => {
  const id = req.params.id;
  const cached = await client.get(`user:${id}`);
  if (cached) return res.json(JSON.parse(cached));

  const user = await db.findUser(id);
  await client.setEx(`user:${id}`, 3600, JSON.stringify(user)); // TTL 1h
  res.json(user);
});
```

📌 _Effetto:_ la prima richiesta va su DB, le successive vengono servite dalla cache Redis.

---

## ⚙️ 2️⃣ **Memcached**

### 📍 Cos’è

Un altro **sistema di cache in-memory**, molto simile a Redis ma:

- più **semplice** (solo chiave → valore, nessuna struttura dati avanzata)
- nessuna **persistenza su disco**
- ottimo per caching puro (velocissimo)

### 🧩 Quando scegliere Memcached

- Quando ti serve solo **cache volatile** (nessuna persistenza)
- Quando hai **altissimo throughput** (API o siti con milioni di accessi)
- Quando non ti servono strutture complesse (liste, set, hash)

### 💡 Esempio (Python)

```python
import memcache

client = memcache.Client(['127.0.0.1:11211'])
client.set('key', 'value', time=60)
print(client.get('key'))
```

---

## ⚙️ 3️⃣ **Varnish Cache**

### 📍 Cos’è

Un **reverse proxy cache HTTP** (sta _davanti_ al tuo web server).
Perfetto per siti web e API ad alto traffico.

Riceve le richieste HTTP e:

- se trova la risposta in cache → la serve subito
- altrimenti → chiede al backend, memorizza la risposta e la serve

### 💡 Esempio di logica in VCL

```vcl
sub vcl_backend_response {
  if (beresp.status == 200 && beresp.ttl > 0s) {
    set beresp.ttl = 1h;
  }
}
```

📌 _Effetto:_ cache di tutte le risposte 200 per 1 ora, direttamente nel reverse proxy.

### 💪 Vantaggi

- Estremamente veloce per contenuti HTTP
- Gestione avanzata TTL, invalidazioni, header personalizzati
- Scalabile per grandi volumi di traffico (usato da siti come Wikipedia, GitHub, ecc.)

---

## ⚙️ 4️⃣ **Altri software e sistemi correlati**

| Nome                    | Tipo                             | Note                                    |
| ----------------------- | -------------------------------- | --------------------------------------- |
| **NGINX FastCGI Cache** | Reverse proxy cache              | Simile a Varnish, ma integrato in Nginx |
| **Squid**               | Proxy cache storico              | Ancora usato in ambienti enterprise     |
| **Hazelcast / Ehcache** | Java cache distribuita           | Usata in Spring, microservizi           |
| **KeyDB**               | Fork di Redis con multithreading | Compatibile con client Redis            |
| **CDN cache**           | Edge cache (Cloudflare, Akamai)  | Distribuisce contenuti globalmente      |

---

## 🔍 **Confronto sintetico**

| Sistema                      | Livello                | Persistenza      | Multi-server | Tipico uso                    |
| ---------------------------- | ---------------------- | ---------------- | ------------ | ----------------------------- |
| **Redis**                    | Applicativo / DB cache | ✅ opzionale     | ✅           | Cache, sessioni, queue        |
| **Memcached**                | Applicativo            | ❌ no            | ✅           | Cache volatile veloce         |
| **Varnish**                  | HTTP / Proxy           | ❌ (RAM)         | ✅           | Cache pagine web/API          |
| **Nginx Cache**              | HTTP / Proxy           | ❌               | ✅           | Cache di risposte FastCGI     |
| **Ehcache / Hazelcast**      | Applicativo (Java)     | ✅               | ✅           | Cache distribuita Java        |
| **CDN (Cloudflare, Akamai)** | Edge globale           | ✅ (controllata) | ✅           | Static assets e HTML pubblici |

---

## 🧠 Strategia tipica di combinazione

Molti sistemi moderni usano **più livelli di cache**:

```
Browser cache → CDN → Reverse proxy (Varnish) → App cache (Redis) → Database
```

Ogni livello riduce la probabilità di arrivare al database.
Più ti avvicini all’utente, più la cache serve dati velocemente.

---
