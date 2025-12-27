# Cloudflare vs. Caddy Rate Limiting - Empfehlung

**Stand:** 27. Dezember 2025
**Server:** mail.clocklight.de

---

## Aktuelle Situation

### Deine Domains & Services:
1. **mail.clocklight.de** - Mailcow (SMTP, IMAP, Webmail)
2. **beszel.clocklight.de** - Monitoring
3. **clocklight.de** - Statische Website
4. **wolfgang-burger.de** - Statische Website

---

## Empfohlener Hybrid-Ansatz

### ✅ DURCH CLOUDFLARE (empfohlen):
- **clocklight.de** - Statische Website
- **wolfgang-burger.de** - Statische Website

**Warum?**
- Kostenloser DDoS-Schutz
- CDN macht Seiten schneller
- Reduziert Server-Last
- Kein Risiko, da nur statische Inhalte

**Vorteile:**
- 🚀 Schnellere Ladezeiten (CDN)
- 🛡️ DDoS-Schutz ohne Server-Belastung
- 📊 Traffic-Analytics
- 🔒 Zusätzliche Firewall-Rules
- 💰 Komplett kostenlos (Free Tier)

### ❌ NICHT DURCH CLOUDFLARE:
- **mail.clocklight.de** - Mailserver
- **beszel.clocklight.de** - Monitoring

**Warum nicht?**

#### mail.clocklight.de:
- ❌ SMTP (Port 25, 587) funktioniert NICHT durch CF Proxy
- ❌ IMAP (Port 993) funktioniert NICHT durch CF Proxy
- ❌ POP3 (Port 995) funktioniert NICHT durch CF Proxy
- ⚠️ Mailcow-Admin (8443) könnte durch CF, aber:
  - Authentifizierung reicht normalerweise
  - Keine DDoS-Gefahr zu erwarten
  - Kein CDN-Vorteil (Admin-Interface)

**Alternative für Mailserver:**
- ✅ Fail2ban (bereits aktiv)
- ✅ Postfix Rate Limiting (in Mailcow integriert)
- ✅ Rspamd Rate Limiting (bereits aktiv)
- ✅ Optional: Caddy Rate Limiting für Port 8443

#### beszel.clocklight.de:
- Monitoring-Daten sollten direkt sein
- Authentifizierung ist vorhanden
- Keine öffentliche Website, daher kein DDoS-Ziel
- CF würde nur Latenz hinzufügen

---

## Detaillierter Vergleich

### Caddy Rate Limiting Plugin

#### Vorteile:
✅ Volle Kontrolle über Rules
✅ Datenschutz (keine Daten an Dritte)
✅ Kostenlos
✅ Niedrige Latenz
✅ Einfache Integration in bestehende Caddy-Config
✅ Funktioniert für ALLE Protokolle (nicht nur HTTP)

#### Nachteile:
❌ Verbraucht Server-Ressourcen
❌ Rate Limiting erst am Server (Traffic ist schon da)
❌ Bei echten DDoS-Attacken hilft es nicht
❌ Keine WAF-Features
❌ Kein Bot-Protection
❌ Kein Caching/CDN

#### Wann verwenden:
- Für Services, die NICHT durch CF können (Mailserver)
- Für interne/geschützte Services (Beszel)
- Wenn Datenschutz kritisch ist
- Bei geringem DDoS-Risiko

#### Konfiguration:
```caddy
# /srv/caddy/snippets/rate_limiting.caddy
rate_limit {
    zone dynamic {
        key {remote_host}
        events 100      # Max 100 Requests
        window 1m       # pro Minute
    }

    # Härtere Limits für Login-Endpoints
    match {
        path /admin/* /api/auth/*
        events 10
        window 1m
    }
}
```

---

### Cloudflare

#### Vorteile:
✅ DDoS-Schutz auf Netzwerk-Ebene (Traffic wird vorher gefiltert)
✅ CDN/Caching (schnellere Ladezeiten weltweit)
✅ Web Application Firewall (WAF)
✅ Bot-Protection (Challenge-Pages)
✅ Analytics & Insights
✅ Kostenloser Tier verfügbar
✅ Reduziert Server-Last dramatisch (bis zu 90%)
✅ Auto-Minify CSS/JS
✅ SSL/TLS Management
✅ Firewall Rules (IP-Blocking, Geo-Blocking, etc.)

#### Nachteile:
❌ Externe Abhängigkeit (Single Point of Failure)
❌ Datenschutz: CF sieht allen Traffic (DSGVO-relevant)
❌ TLS-Terminierung bei CF (Man-in-the-Middle-Position)
❌ Vendor Lock-in
❌ NUR HTTP/HTTPS (kein SMTP, IMAP, POP3, etc.)
❌ Bei Problemen: CF-Support notwendig
❌ Kann manchmal false-positives haben (echte User blocken)

#### Wann verwenden:
- Für öffentliche Websites
- Bei hohem Traffic
- Wenn DDoS-Risiko besteht
- Für statische Inhalte (optimal)
- Wenn CDN-Vorteile genutzt werden sollen

#### Setup:
1. Domain zu CF transferieren (oder NS auf CF zeigen)
2. DNS Records anlegen
3. Proxy-Status auf "Orange Cloud" setzen
4. SSL/TLS Mode: "Full (strict)"
5. Firewall Rules konfigurieren

---

## Konkrete Empfehlung für dein Setup

### Phase 1: Statische Websites durch Cloudflare (EMPFOHLEN)

**Domains:**
- clocklight.de
- wolfgang-burger.de

**Warum?**
- Komplett risikolos (nur statische Inhalte)
- Massive Performance-Verbesserung
- Kostenloser DDoS-Schutz
- Reduziert Server-Last

**Setup-Schritte:**
1. Cloudflare-Account erstellen (kostenlos)
2. Domains hinzufügen
3. Nameserver bei Domain-Registrar ändern
4. DNS-Records konfigurieren:
   ```
   clocklight.de -> A -> <server-ip> (Proxied ☁️)
   www.clocklight.de -> CNAME -> clocklight.de (Proxied ☁️)
   ```
5. SSL/TLS Mode: "Full (strict)"
6. Fertig!

**Caddy bleibt Origin-Server:**
- CF → Caddy → Websites
- Caddy macht weiterhin SSL-Terminierung
- CF cached statische Inhalte

### Phase 2: Mailserver OHNE Cloudflare (EMPFOHLEN)

**Domains:**
- mail.clocklight.de
- beszel.clocklight.de

**Warum?**
- SMTP/IMAP funktioniert nicht durch CF Proxy
- Direkter Zugriff notwendig
- Fail2ban & Mailcow-eigenes Rate Limiting reicht

**Schutz durch:**
1. ✅ Fail2ban (bereits aktiv)
2. ✅ Postfix Rate Limiting (Mailcow-integriert)
3. ✅ Rspamd Rate Limiting (bereits aktiv)
4. ✅ Security Headers (bereits implementiert)
5. Optional: Caddy Rate Limiting für Webmail

**Optional: Caddy Rate Limiting hinzufügen**
```caddy
# Für mail.clocklight.de (Webmail-Schutz)
mail.clocklight.de {
    import ../snippets/rate_limiting.caddy
    reverse_proxy https://46.224.122.105:8443 { ... }
}
```

### Phase 3: Monitoring ohne Cloudflare (EMPFOHLEN)

**beszel.clocklight.de:**
- Kein öffentlicher Service
- Authentifizierung vorhanden
- Kein DDoS-Risiko
- CF würde nur Latenz hinzufügen

---

## Kosten-Vergleich

| Lösung | Kosten | Vorteile |
|--------|--------|----------|
| **Nur Caddy Rate Limiting** | 0€ | Volle Kontrolle, Datenschutz |
| **CF Free Tier** | 0€ | DDoS-Schutz, CDN, WAF |
| **CF Pro** | ~20€/Monat | Bessere Analytics, Image-Optimization |
| **CF Business** | ~200€/Monat | 100% Uptime-SLA, Custom SSL |

**Empfehlung:** CF Free Tier für Websites = 0€

---

## Datenschutz-Überlegungen (DSGVO)

### Cloudflare & DSGVO:
- ✅ Cloudflare ist DSGVO-konform (DPA verfügbar)
- ✅ Server in Europa verfügbar
- ⚠️ CF ist US-Unternehmen (Schrems II beachten)
- ⚠️ CF sieht alle Requests (IP-Adressen, User-Agents, etc.)

**Für statische Websites:** Unkritisch
**Für Mailserver:** Nicht empfohlen (sensible Daten)

### Lösung:
- Statische Websites durch CF (unkritisch)
- Mailserver direkt (sensible Daten)
- Datenschutzerklärung anpassen (CF erwähnen)

---

## Performance-Vergleich

### Ohne Cloudflare:
```
Deutschland: ~20ms
USA: ~150ms
Asien: ~250ms
```

### Mit Cloudflare (CDN):
```
Deutschland: ~10ms (CF-Frankfurt)
USA: ~15ms (CF-New York)
Asien: ~20ms (CF-Singapur)
```

**Verbesserung:** 80-90% schneller weltweit

---

## Konkrete Implementierung

### Option A: Hybrid (EMPFOHLEN) ⭐

**Durch Cloudflare:**
- clocklight.de ☁️
- wolfgang-burger.de ☁️

**Direkt (ohne CF):**
- mail.clocklight.de 🔒
- beszel.clocklight.de 🔒

**Vorteile:**
- ✅ Beste Performance für Websites
- ✅ DDoS-Schutz für öffentliche Inhalte
- ✅ Mailserver funktioniert weiterhin
- ✅ Monitoring bleibt direkt
- ✅ 0€ Zusatzkosten

**Nachteile:**
- Etwas komplexere DNS-Konfiguration
- Zwei verschiedene Systeme

### Option B: Nur Caddy Rate Limiting

**Für:**
- mail.clocklight.de
- beszel.clocklight.de
- clocklight.de
- wolfgang-burger.de

**Vorteile:**
- ✅ Einfach
- ✅ Volle Kontrolle
- ✅ Datenschutz

**Nachteile:**
- ❌ Kein echter DDoS-Schutz
- ❌ Langsamere Ladezeiten international
- ❌ Höhere Server-Last

### Option C: Alles durch Cloudflare

**NICHT MÖGLICH** wegen Mailserver!

---

## Schritt-für-Schritt: Hybrid-Setup implementieren

### 1. Cloudflare-Account erstellen
```bash
# Auf cloudflare.com registrieren (kostenlos)
# Email verifizieren
```

### 2. Domains zu Cloudflare hinzufügen

**clocklight.de:**
```
1. "Add Site" klicken
2. Domain eingeben: clocklight.de
3. Free Plan auswählen
4. DNS-Records importieren (automatisch)
5. Nameserver bei Registrar ändern auf CF-Nameserver
```

**wolfgang-burger.de:**
```
Gleicher Prozess wie clocklight.de
```

### 3. DNS-Konfiguration in Cloudflare

**clocklight.de:**
```
Type: A
Name: @
Content: <server-ip>
Proxy: ☁️ Proxied (Orange Cloud)

Type: CNAME
Name: www
Content: clocklight.de
Proxy: ☁️ Proxied
```

**mail.clocklight.de:**
```
Type: A
Name: mail
Content: <server-ip>
Proxy: ⚠️ DNS Only (Grey Cloud) ← WICHTIG!
```

**beszel.clocklight.de:**
```
Type: A
Name: beszel
Content: <server-ip>
Proxy: ⚠️ DNS Only (Grey Cloud)
```

### 4. SSL/TLS-Konfiguration

**In Cloudflare Dashboard:**
```
SSL/TLS > Overview > Full (strict)
```

**Warum "Full (strict)"?**
- CF → Server Verschlüsselung
- CF verifiziert Caddy's Let's Encrypt Zertifikat
- Keine Warnungen

### 5. Firewall-Rules (Optional)

**Beispiel: Deutschland-only für Admin-Bereiche:**
```
Rule: Block non-German traffic to /admin/*
Expression: (http.request.uri.path matches "/admin/.*") and (ip.geoip.country ne "DE")
Action: Block
```

### 6. Caching-Konfiguration

**Page Rules (3 kostenlos im Free Tier):**
```
1. clocklight.de/*
   Cache Level: Standard
   Browser Cache TTL: 1 month

2. wolfgang-burger.de/*
   Cache Level: Standard
   Browser Cache TTL: 1 month
```

### 7. Caddy-Konfiguration NICHT ändern!

**Wichtig:** Caddy läuft weiter wie bisher!
- CF routet Traffic zu deinem Server
- Caddy macht SSL-Terminierung
- Alles bleibt gleich

---

## Testing & Verification

### Nach Cloudflare-Setup testen:

```bash
# 1. DNS-Auflösung prüfen
dig clocklight.de
# Sollte CF-IPs zeigen (104.x.x.x oder 172.x.x.x)

dig mail.clocklight.de
# Sollte DEINE Server-IP zeigen (NICHT CF)

# 2. HTTP-Headers prüfen
curl -I https://clocklight.de
# Sollte zeigen: server: cloudflare

curl -I https://mail.clocklight.de
# Sollte zeigen: via: 1.1 Caddy (NICHT cloudflare)

# 3. Performance testen
curl -w "@curl-format.txt" -o /dev/null -s https://clocklight.de
# Sollte schneller sein mit CF

# 4. Mailserver testen
telnet mail.clocklight.de 25
# Sollte SMTP-Verbindung öffnen (funktioniert nur ohne CF!)
```

---

## Monitoring & Alerts

### Cloudflare Analytics
- Verfügbar unter: Dashboard > Analytics
- Zeigt:
  - Requests/Stunde
  - Bandwidth gespart
  - Geblockte Threats
  - Cache-Hit-Rate

### Wichtig:
- CF cached nur statische Inhalte (HTML, CSS, JS, Bilder)
- Dynamische Inhalte gehen durch zu Caddy
- Cache-Hit-Rate zeigt Effizienz

---

## Troubleshooting

### Problem: "Too many redirects"
**Lösung:**
```
Cloudflare > SSL/TLS > Overview
Ändern von "Flexible" zu "Full (strict)"
```

### Problem: "Origin server not responding"
**Lösung:**
```bash
# Prüfe ob Caddy läuft
docker ps | grep caddy

# Prüfe Caddy-Logs
docker logs caddy-webserver

# Prüfe Firewall
ufw status
```

### Problem: "Mail sendet/empfängt nicht"
**Lösung:**
```
mail.clocklight.de MUSS auf "DNS Only" (Grey Cloud) sein!
Niemals auf Proxied (Orange Cloud) setzen!
```

---

## Fazit & Empfehlung

### 🎯 Für dein Setup: Hybrid-Ansatz

**DURCH CLOUDFLARE (Orange Cloud ☁️):**
- ✅ clocklight.de
- ✅ wolfgang-burger.de

**DIREKT / DNS ONLY (Grey Cloud):**
- ✅ mail.clocklight.de
- ✅ beszel.clocklight.de

**Vorteile:**
- 🚀 Schnellere Websites (CDN)
- 🛡️ DDoS-Schutz für öffentliche Inhalte
- 📊 Traffic-Analytics
- 💰 0€ Kosten (Free Tier)
- 🔒 Mailserver funktioniert weiterhin perfekt
- 📈 Reduzierte Server-Last

**Aufwand:**
- ⏱️ Setup: 30-60 Minuten
- 🔧 Wartung: 0 (läuft automatisch)
- 🎓 Lernkurve: Gering

**Alternative:**
Wenn du 100% Kontrolle willst und KEIN DDoS-Risiko siehst:
- Caddy Rate Limiting für alle Domains
- Einfacher, aber ohne DDoS-Schutz

**Meine Empfehlung:**
**Setze Cloudflare für die statischen Websites um.**
Es ist kostenlos, risikolos und bringt deutliche Vorteile.

---

**Letztes Update:** 27. Dezember 2025
**Nächste Review:** Nach CF-Implementation (empfohlen in 1-2 Wochen)
