## Parametri pubblici

Riepilogo dei principali **parametri pubblici** (cioè header HTTP) che riguardano la cache, con spiegazione e uso comune.

---

### 🔑 Principali header e direttive

| Header / direttiva | Dove si usa                          | Cosa fa                                                                                                                                        | Note                                                                                                                                |
| ------------------ | ------------------------------------ | ---------------------------------------------------------------------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------- |
| `Cache-Control`    | Risposta (e qualche volta richiesta) | Definisce le regole di caching moderne: dove, quanto a lungo, chi può cachare. ([Imperva][1])                                                  | Le direttive interne includono `max-age`, `no-cache`, `no-store`, `public`, `private`, `s-maxage`, `immutable` ([GeeksforGeeks][2]) |
| `Expires`          | Risposta                             | Data assoluta in cui la risorsa diventa “scaduta”. ([Almanacco Web 2024][3])                                                                   | Usato più con HTTP/1.0, `Cache-Control` ha priorità se entrambi presenti. ([Almanacco Web 2024][3])                                 |
| `ETag`             | Risposta                             | Tag univoco che rappresenta la versione della risorsa → permette validazione condizionale (If-None-Match)                                      | Usato per risorse che possono essere “ri-validate” piuttosto che ricaricate sempre                                                  |
| `Last-Modified`    | Risposta                             | Data dell’ultima modifica della risorsa → usato con `If-Modified-Since` dal client per validare                                                | Meno preciso dell’ETag in certe condizioni                                                                                          |
| `Vary`             | Risposta                             | Indica che la versione della risorsa può variare in base a certi header della richiesta (es. `Accept-Encoding`, `User-Agent`) ([Bomberbot][4]) | Importante soprattutto per cache condivise o varianti di risorse (mobile/desktop)                                                   |
| `Age`              | Risposta da cache intermedia         | Numero di secondi da quando la risorsa è stata “cacheata” nella cache intermedia/proxy ([Wikipedia][5])                                        | Utile per debug: capisci quanto la copia è “vecchia”                                                                                |
| `Cache-Status`     | Risposta                             | Header emergente standard per capire come è stata gestita la cache (HIT, MISS, etc) ([IETF][6])                                                | Ancora in bozza/uso crescente                                                                                                       |
| `Pragma: no-cache` | Richiesta o risposta                 | Modo legacy (HTTP/1.0) per indicare che la risorsa non deve essere usata senza validazione ([Wikipedia][5])                                    | Oggi si preferisce `Cache-Control: no-cache`                                                                                        |

---

### 📋 Esempio sintetico delle direttive più comuni in `Cache-Control`

- `public` → risorsa può essere cachata da **qualsiasi** cache (browser, proxy, CDN)
- `private` → solo browser della singola utenza può cachare, non proxy condivisi
- `no-cache` → può essere cachata, ma **deve** essere validata dal server prima di riusarla
- `no-store` → non deve essere memorizzata in alcuna cache
- `max-age=<seconds>` → la risorsa è “fresca” per quel numero di secondi dalla richiesta
- `s-maxage=<seconds>` → simile a `max-age` ma **solo per cache condivise** (proxy/CDN)
- `must-revalidate` → dopo la scadenza, la cache **deve** validare con l’origine prima di servire
- `immutable` → la risorsa **non cambierà**, quindi la cache può considerarla “fissa” per il tempo indicato ([Jono Alderson][7])

---

### 🎯 Qualche nota utile

- Se sia `Expires` che `Cache-Control` sono presenti, **Cache-Control prevale**. ([Almanacco Web 2024][3])
- L’header `Vary` è importante quando una risorsa ha **più varianti** (es. immagini responsive) — senza `Vary` rischi che una cache serva una variante sbagliata. ([Jono Alderson][7])
- Anche se una risorsa è cacheata, **non significa** che venga sempre servita \*senza contatto al server\*\*: se ci sono direttive di validazione (`no-cache`, `must-revalidate`), il server può essere sentito.
- Alcuni header sono utili **solo per debug** o livello intermedio di caching (es. `Age`, `Cache-Status`) e non controllano davvero il comportamento della cache da soli.
- Le cache devono distinguere tra header “end-to-end” (che viaggiano fino al client) e “hop-by-hop” (che non vanno memorizzati nelle cache). ([W3C][8])

---

Se vuoi, posso preparare una lista **completa** (20-30 parametri) con anche direttive meno comuni (es. `must-understand`, `proxy-revalidate`, `stale-while-revalidate`, `stale-if-error`, ecc) e spiegazione rapida per ciascuna. Vuoi che lo faccia?

[1]: https://www.imperva.com/learn/performance/cache-control/?utm_source=chatgpt.com "What is Cache-Control and How HTTP Cache Headers Work | CDN Guide | Imperva"
[2]: https://www.geeksforgeeks.org/computer-networks/http-headers-cache-control/?utm_source=chatgpt.com "HTTP headers | cache-control - GeeksforGeeks"
[3]: https://almanac.httparchive.org/en/2020/caching?utm_source=chatgpt.com "Caching | 2020 | The Web Almanac by HTTP Archive"
[4]: https://www.bomberbot.com/http/an-in-depth-guide-to-http-caching-leveraging-cache-control-and-vary-headers/?utm_source=chatgpt.com "An In-Depth Guide to HTTP Caching: Leveraging Cache-Control and Vary Headers - Bomberbot"
[5]: https://en.wikipedia.org/wiki/List_of_HTTP_header_fields?utm_source=chatgpt.com "List of HTTP header fields"
[6]: https://www.ietf.org/archive/id/draft-ietf-httpbis-cache-header-10.html?utm_source=chatgpt.com "The Cache-Status HTTP Response Header Field"
[7]: https://www.jonoalderson.com/performance/http-caching/?utm_source=chatgpt.com "A complete guide to HTTP caching - Jono Alderson"
[8]: https://www.w3.org/Protocols/rfc2616/rfc2616-sec13.html?utm_source=chatgpt.com "HTTP/1.1: Caching in HTTP"
