## Come e quando (casi d'uso)

**Checklist operativa e sintetica** su _dove e come impostare la cache HTTP_, in base all’ambiente in cui lavori.

---

## 🧱 **1️⃣ Web server statico (Apache / Nginx)**

**📍 Dove:** nel file `.htaccess` (Apache) o `nginx.conf`

**🎯 Quando usarlo:**
Per file statici: immagini, CSS, JS, font, PDF, ecc.

**✅ Cosa fare:**

**👉 Apache**

```apache
<IfModule mod_headers.c>
  <FilesMatch "\.(js|css|jpg|png|gif|svg|woff2)$">
    Header set Cache-Control "public, max-age=31536000, immutable"
  </FilesMatch>
</IfModule>
```

**👉 Nginx**

```nginx
location ~* \.(js|css|jpg|png|gif|svg|woff2)$ {
    add_header Cache-Control "public, max-age=31536000, immutable";
}
```

📌 _Effetto:_ il browser e la CDN cacheano le risorse per 1 anno.

---

## ⚙️ **2️⃣ Applicazione backend (Node.js, Django, Laravel, Spring, ecc.)**

**📍 Dove:** nel codice del controller / endpoint

**🎯 Quando:**
Per API, HTML o contenuti generati dinamicamente.

**✅ Cosa fare:**

**👉 Node.js / Express**

```js
app.get("/api/data", (req, res) => {
  res.set("Cache-Control", "public, max-age=60");
  res.json({ message: "Hello" });
});
```

**👉 Django**

```python
from django.views.decorators.cache import cache_control

@cache_control(public=True, max_age=60)
def my_view(request):
    ...
```

**👉 Laravel**

```php
return response($data)
    ->header('Cache-Control', 'public, max-age=60');
```

📌 _Effetto:_ la risposta è cacheata per 60s da browser e CDN, ma resta “fresca” per le API.

---

## ☁️ **3️⃣ CDN / Reverse proxy (Cloudflare, Varnish, Fastly, ecc.)**

**📍 Dove:** nella console di configurazione o file di regole

**🎯 Quando:**
Per gestire la cache globale e scaricare carico dal server origin.

**✅ Cosa fare:**

- **Rispetta gli header del backend**, oppure
- **Sovrascrivi** (es. per servire più a lungo file statici)

**Esempio Cloudflare (regola pagina):**

```
Edge Cache TTL: 1 giorno
Browser Cache TTL: Rispetta l’header origin
```

**Esempio Varnish (VCL):**

```vcl
sub vcl_backend_response {
  set beresp.ttl = 1d;
}
```

📌 _Effetto:_ la CDN serve i file dalla propria cache per 24 ore, senza contattare l’origine.

---

## 🧠 **4️⃣ Client / Browser / Frontend**

**📍 Dove:** nel codice della richiesta o nella configurazione fetch/XHR

**🎯 Quando:**
Se vuoi forzare un comportamento particolare nella richiesta.

**✅ Cosa fare:**

**👉 Esempio fetch**

```js
fetch("/api/data", { cache: "no-store" });
```

**👉 Esempio cURL**

```bash
curl -H "Cache-Control: no-cache" https://miosito.com/api/data
```

📌 _Effetto:_ ignora qualsiasi cache locale o CDN e forza una richiesta “fresca”.

---

## 🧩 **5️⃣ Meta tag e Service Worker (solo web app PWA)**

**📍 Dove:** in `<head>` o negli script del service worker

**🎯 Quando:**
Solo se vuoi controllare la cache lato browser in modo personalizzato.

**✅ Esempi:**

```html
<meta http-equiv="Cache-Control" content="no-store" />
```

oppure in un **service worker**:

```js
self.addEventListener("fetch", (event) => {
  event.respondWith(caches.match(event.request) || fetch(event.request));
});
```

📌 _Effetto:_ controllo fine della cache solo in ambiente browser, non lato server.

---

## 📋 **Riepilogo pratico**

| Contesto                          | Dove si configura        | Esempio tipico                            | Obiettivo               |
| --------------------------------- | ------------------------ | ----------------------------------------- | ----------------------- |
| **Server statico (Apache/Nginx)** | Config file / .htaccess  | `Cache-Control: public, max-age=31536000` | Cache lunga per asset   |
| **Backend app**                   | Codice del controller    | `res.set('Cache-Control', 'max-age=60')`  | Cache breve per API     |
| **CDN / Proxy**                   | Console / file VCL       | TTL di edge cache                         | Offload e latenza bassa |
| **Client / Browser**              | fetch o header richiesta | `Cache-Control: no-store`                 | Forza reload            |
| **Service Worker / Meta tag**     | HTML / JS                | caching personalizzato                    | Controllo lato client   |

---
