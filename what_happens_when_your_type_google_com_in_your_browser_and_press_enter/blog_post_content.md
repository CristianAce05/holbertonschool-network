# What Happens When You Type https://www.google.com and Press Enter?

You press Enter. Less than 200 milliseconds later, Google's homepage appears. But what actually happened between your keypress and that rendered page?

This is one of the most common software engineering interview questions — because the answer touches every layer of the web stack. Here is the full journey.

---

## The Full Picture

![Request flow diagram](https://mermaid.ink/img/pako:eNqFVc1uGzcQfhViTzYgy5YsR7IOBfQbCVUkQavUSLNFTe1SK8IrckNyo6iOgR6CnFIkaIsE7aWn9Bn6PH6B9hE6w11ZP5uiK0hL2t_MfDPzDXnr-DJgTt3xxDySK39BlSHTticIPE_dzuS55_zzx8fPf__1njSVXGmmPE8crWVCzDpmZGFMrOunp6vVqhhKGUas6Mvlsed854nUiU5moaLxgrSHLrHefv3JridMyygxXAqLJtkD__q-1Wj1OoDNIpJTMnJJi_oLloNOOu5o8I3lOWF-ojR_yVLXL1OqfXcM9rWi_Rzn7UejKdpKaciQLhlxmULLQ9x00AYY_O6iyNE2211w4-m0B-jHtiCkkZiFVNxQg9yGbg7eH1v2lnNA-mOkzYphkZQq5WL54qxYOytWHu0Wda9Q5OTkq9eeY-tDllxrz3m9V5svVwzNHkqQL0rmFRMkNm134xZ2udpk6K0EdvBYjXx9MosGUcyXKiD3b3_G1DMbWFkLJoKckHrT6XgjpQ-kI3y1jg0UriWFYD4KihylmFNg7u63Z9rCYsMv6VER6AW9YVhu99nQMoD3SaP1tV3D-8B24FoNuPu2PlOGz7lPDSsQn8cLEIZOuGH71p1ha_JsPO2gjrakkSjo9UXCtEFnsYQBrFTOD7uNjLFhGB3fD97-u079YXfSSOv0_hMOcKbHvpgrqo1KfJOo_YnqXqVl_Uy6XLEVjSLkNOeRYUqTJY24z2WiiVF0DhnvJzhogvH97x8x1EDSgDRpRIWfjmHAISKfJYZpQn0ltSbazpA-zPSBP5w-KFNwesVm0MtGHH9pOvG56jQt8XcfCGIzFIQdhly8AtvH3c5xzqoxHlvGvyFj8B5hD1E-W_uQCaYoku5NnwxAqzqWQrO8r3bz-REy-PQGnbWpoTOqrTyaPIRNBNpwYwoKVfc__gn2x3tJZ0lkQxGsBV1yH8KluoChAK458hkaMGpNAghpp6d5SCyDKQbtFphCEhn9Pz4VCIopq88nA8QCuS12R22paqwkBzbSFrmDgjNnPBq6HdukX97YJo1pyMgmDhSKCwLXiiKz7JrZqgLvoYwXXhyRlDdJvDkp7BGY4tKDI0PC4kbIlUAcDE-KeJgay7R7lf51W_iDudy0G31sUvDE9fW1U3BCxQOnDlPECs6SqSXFrXOLHj3HLNgSrOqwDKi68eB6vQMb6P-3Ui43Zkom4WKzSWLoIGtzCuIHxJxGGiG2Pi2ZCOPUSxcV68Op3zqvYHsO10L1vHQJ33L58lG1VnDWTv3isliqVSvVSvnsvFSrlO8Kzg82KNwj1Yu7fwF_OXHE?type=png)

---

## 1. DNS Request

Your browser knows the name `www.google.com` but needs an IP address to open a connection. It queries the **Domain Name System (DNS)** — the internet's phonebook.

The lookup cascades through a chain: browser cache → OS cache → recursive resolver → root nameserver → `.com` TLD nameserver → Google's authoritative nameserver. Each step narrows it down until the authoritative server returns the IP (e.g. `142.250.80.36`). The result is cached with a TTL so future lookups skip most of the chain.

---

## 2. TCP/IP

With an IP in hand, the browser opens a **TCP connection** to port **443** (the HTTPS port) using a three-way handshake:

- **SYN** — browser sends a synchronize packet with a sequence number `x`
- **SYN-ACK** — server acknowledges `x+1` and sends its own sequence number `y`
- **ACK** — browser acknowledges `y+1`; connection established

**IP** handles routing each packet across the network independently. **TCP** sits on top and guarantees reliable, ordered delivery — retransmitting lost packets and reassembling them in the right order.

---

## 3. Firewall

Before reaching any server, traffic passes through **firewalls**. On your end, your router filters outbound traffic. On Google's end, perimeter firewalls inspect every incoming packet.

Firewalls work at multiple levels: stateless rules (IP/port), stateful inspection (tracking active sessions), and Web Application Firewalls (WAFs) that scan HTTP traffic for attack patterns like SQL injection or XSS. A clean GET request on port 443 passes through unchallenged.

---

## 4. HTTPS / SSL

Because the URL uses `https://`, the browser performs a **TLS handshake** before sending any data:

1. Browser advertises supported TLS versions and cipher suites
2. Server responds with its **SSL certificate**
3. Browser verifies the certificate against a trusted **Certificate Authority** — confirming this is the real Google
4. Both sides derive a shared **session key** using asymmetric cryptography
5. All subsequent communication is symmetrically encrypted

This prevents eavesdropping and man-in-the-middle attacks. The padlock in your address bar is the result of this process.

---

## 5. Load Balancer

Google handles billions of requests per day. No single server could manage that. The encrypted request first hits a **load balancer**, which distributes traffic across a pool of servers — routing to the nearest or least-busy one, and automatically rerouting around failed instances.

Google uses both Layer 4 (TCP/IP-level) and Layer 7 (HTTP-level) load balancing, combined with **Anycast routing** to steer users to the closest data center geographically.

---

## 6. Web Server

The load balancer forwards the request to a **web server** (Nginx, Apache, or Google's own). The web server parses the HTTP request, serves any static assets directly (CSS, images, JS), and forwards dynamic requests — like a search page — further down the chain to the application server.

---

## 7. Application Server

The **application server** runs the business logic: authentication, session management, search ranking, personalization, ad selection. It coordinates multiple internal microservices, each contributing a piece of the final page, and assembles the full HTML response.

---

## 8. Database

To generate the response, the application server queries **databases**:

- Distributed stores like **Bigtable** and **Spanner** hold the search index across billions of web pages
- User stores hold preferences and history (if signed in)
- **Redis** or **Memcached** caching layers serve hot data without hitting primary storage

Results are returned to the application server, which composes the final HTML and sends it back up the chain.

---

## The Return

The response travels back through the same layers — encrypted, load-balanced, through the firewall — and arrives at your browser. The browser parses the HTML, fetches linked assets (each a mini version of this same journey), builds the DOM and CSSOM, runs JavaScript, and paints the page.

All of this — DNS, TCP handshake, TLS, firewall, load balancer, web server, app server, database — in under 200 milliseconds.

---

*Written as part of the Holberton School curriculum on networking fundamentals.*
