## 🧭 COS’È LA CACHE HTTP

La **cache HTTP** serve a **ridurre i tempi di caricamento** e **limitare le richieste al server** conservando copie locali (browser, proxy, CDN) delle risorse già scaricate — come immagini, CSS, JS o HTML.

Quando un client richiede una risorsa, il browser o un proxy può restituire una **versione salvata**, evitando di chiedere nuovamente al server (se le regole lo permettono).

---

## ⚙️ DOVE PUÒ ESSERE GESTITA LA CACHE

La cache può trovarsi in diversi livelli:

| Livello               | Descrizione                                                     |
| --------------------- | --------------------------------------------------------------- |
| **Browser Cache**     | Conserva risorse localmente sul dispositivo dell’utente.        |
| **Proxy Cache / CDN** | Cache intermedia tra client e server per ridurre la latenza.    |
| **Server Cache**      | Implementata lato backend per evitare calcoli o query ripetute. |

---

## 📜 PRINCIPALI HEADER HTTP DI CACHE

### 🔸 1. **Cache-Control**

È l’header più importante (introdotto in HTTP/1.1).
Controlla il comportamento della cache sia client che intermedi.

Esempio:

```
Cache-Control: public, max-age=3600
```

**Direttive principali:**

| Direttiva            | Significato                                                      |
| -------------------- | ---------------------------------------------------------------- |
| `public`             | Permette la cache anche su proxy/CDN.                            |
| `private`            | Solo il browser dell’utente può cachare.                         |
| `no-cache`           | Richiede la validazione al server prima di riusare la cache.     |
| `no-store`           | Non memorizzare **mai** nulla in cache.                          |
| `max-age=<seconds>`  | Durata massima della cache (in secondi).                         |
| `s-maxage=<seconds>` | Come `max-age`, ma solo per cache condivise (es. CDN).           |
| `must-revalidate`    | Se scaduta, la cache **deve** essere rivalidata col server.      |
| `immutable`          | Indica che la risorsa non cambierà (ottimo per file versionati). |

---

### 🔸 2. **Expires**

Header più vecchio (HTTP/1.0).
Definisce **una data di scadenza assoluta** nel futuro.

Esempio:

```
Expires: Wed, 25 Oct 2025 10:00:00 GMT
```

👉 È spesso **sovrascritto da `Cache-Control`**, se entrambi presenti.

---

### 🔸 3. **ETag (Entity Tag)**

Identifica **una versione specifica** di una risorsa.

Esempio:

```
ETag: "abc123"
```

Quando il browser fa una nuova richiesta, invia:

```
If-None-Match: "abc123"
```

→ Se il file non è cambiato, il server risponde con **304 Not Modified**, risparmiando banda.

---

### 🔸 4. **Last-Modified**

Indica **quando** la risorsa è stata modificata l’ultima volta.

Esempio:

```
Last-Modified: Wed, 25 Oct 2025 09:00:00 GMT
```

Il client può validare con:

```
If-Modified-Since: Wed, 25 Oct 2025 09:00:00 GMT
```

→ Se non è cambiata, riceve di nuovo un **304 Not Modified**.

---

### 🔸 5. **Vary**

Indica che la cache deve variare in base a certi header della richiesta.

Esempio:

```
Vary: Accept-Encoding
```

→ Significa che le versioni compresse (gzip/br) vengono cache separate.

---

## 💡 STRATEGIE COMUNI DI CACHE

| Scenario                                | Esempio Header                                       | Note                                           |
| --------------------------------------- | ---------------------------------------------------- | ---------------------------------------------- |
| **Static assets (CSS, JS, immagini)**   | `Cache-Control: public, max-age=31536000, immutable` | Versionamento nel nome file (es. `app.v2.js`). |
| **HTML dinamico**                       | `Cache-Control: no-cache`                            | Deve essere validato sempre.                   |
| **API REST**                            | `Cache-Control: public, max-age=60`                  | Aggiornamento rapido ma caching breve.         |
| **Risorse sensibili (es. dati utente)** | `Cache-Control: private, no-store`                   | Evita la cache su proxy.                       |

---

## 🧩 Interazione tra header

- Se presenti **Cache-Control** e **Expires**, vince **Cache-Control**.
- **ETag** e **Last-Modified** sono per la **validazione condizionale**, non per la durata.
- **no-store** disattiva qualsiasi tipo di cache (browser, proxy, CDN).

---

## 📈 Riassunto visuale

| Funzione         | Header principale          | Tipo                          |
| ---------------- | -------------------------- | ----------------------------- |
| Controllo durata | `Cache-Control`, `Expires` | Caching “forte”               |
| Validazione      | `ETag`, `Last-Modified`    | Caching “condizionale”        |
| Variazioni       | `Vary`                     | Specifica per header o lingua |

---
