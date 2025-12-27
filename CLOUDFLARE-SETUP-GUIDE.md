# Cloudflare Hybrid-Setup - Schritt-für-Schritt-Anleitung

**Erstellt:** 27. Dezember 2025
**Server:** mail.clocklight.de (46.224.122.105)
**Ziel:** Hybrid-Setup mit Cloudflare für statische Websites

---

## Aktuelle DNS-Konfiguration

### Server-IP:
- **IPv4:** 46.224.122.105
- **IPv6:** 2a01:4f8:c2c:ae8e::1

### Domains & aktuelle IPs:
```
clocklight.de         → 46.224.122.105 (dieser Server) ✅
mail.clocklight.de    → 46.224.122.105 (dieser Server) ✅
beszel.clocklight.de  → 46.224.122.105 (dieser Server) ✅
wolfgang-burger.de    → 92.204.239.183 (anderer Server) ⚠️
```

**⚠️ Wichtig:** wolfgang-burger.de zeigt aktuell auf eine andere IP (92.204.239.183).
- Dateien existieren auf diesem Server (/var/www/wolfgang-burger.de)
- DNS zeigt aber auf anderen Server
- **Aktion:** Prüfen ob wolfgang-burger.de migriert werden soll

---

## Übersicht: Was wird geändert?

### DURCH CLOUDFLARE (Orange Cloud ☁️):
- ✅ **clocklight.de** - Statische Website
- ⚠️ **wolfgang-burger.de** - Nur wenn DNS geändert wird

### DIREKT / DNS ONLY (Grey Cloud):
- ✅ **mail.clocklight.de** - Mailserver (MUSS direkt bleiben)
- ✅ **beszel.clocklight.de** - Monitoring (MUSS direkt bleiben)

---

## Phase 1: Cloudflare-Account erstellen

### Schritt 1: Registrierung
1. Gehe zu: https://dash.cloudflare.com/sign-up
2. Email-Adresse eingeben
3. Passwort erstellen (mind. 12 Zeichen, komplex)
4. Email verifizieren

**⏱️ Dauer:** 5 Minuten

---

## Phase 2: Domain zu Cloudflare hinzufügen

### Schritt 2: clocklight.de hinzufügen

1. **Dashboard öffnen:** https://dash.cloudflare.com
2. **"Add Site" klicken**
3. **Domain eingeben:** `clocklight.de`
4. **Plan auswählen:** Free Plan (0€)
5. **"Continue" klicken**

### Schritt 3: DNS-Records überprüfen

Cloudflare scannt automatisch deine DNS-Records. Prüfe folgende Einträge:

**Sollte erkannt werden:**
```
Type: A
Name: @
Content: 46.224.122.105
Proxy: ☁️ Proxied (Orange Cloud)
```

**Falls NICHT automatisch erkannt, manuell hinzufügen:**
```
Type: A
Name: @
Content: 46.224.122.105
Proxy: ☁️ Proxied
TTL: Auto
```

**www-Subdomain hinzufügen:**
```
Type: CNAME
Name: www
Content: clocklight.de
Proxy: ☁️ Proxied
TTL: Auto
```

**WICHTIG: Andere Subdomains hinzufügen (OHNE Proxy!):**

**mail.clocklight.de (KRITISCH - DNS Only!):**
```
Type: A
Name: mail
Content: 46.224.122.105
Proxy: ⚠️ DNS Only (Grey Cloud) ← WICHTIG!
TTL: Auto
```

**beszel.clocklight.de (DNS Only!):**
```
Type: A
Name: beszel
Content: 46.224.122.105
Proxy: ⚠️ DNS Only (Grey Cloud)
TTL: Auto
```

**⚠️ KRITISCH:** mail.clocklight.de MUSS auf "DNS Only" sein, sonst funktioniert Email NICHT!

### Schritt 4: MX-Records prüfen (wichtig für Mail!)

**Falls MX-Records vorhanden:**
```
Type: MX
Name: @
Mail server: mail.clocklight.de
Priority: 10
Proxy: N/A (MX-Records können nicht geproxied werden)
```

**Weitere Mail-Records (falls vorhanden):**
- SPF (TXT Record)
- DKIM (TXT Record)
- DMARC (TXT Record)

**Diese werden automatisch importiert - bitte NICHT löschen!**

### Schritt 5: Nameserver ändern

Cloudflare zeigt dir zwei Nameserver:
```
Beispiel:
- cameron.ns.cloudflare.com
- dina.ns.cloudflare.com
```

**Bei deinem Domain-Registrar:**
1. Einloggen (z.B. Namecheap, GoDaddy, Hetzner, etc.)
2. Domain-Management öffnen
3. Nameserver ändern auf die von Cloudflare angegebenen
4. Speichern

**⏱️ Dauer:** 10-15 Minuten
**⏳ Propagation:** 1-24 Stunden (meist nach 10-30 Min aktiv)

---

## Phase 3: SSL/TLS-Konfiguration

### Schritt 6: SSL/TLS Mode einstellen

**Cloudflare Dashboard:**
1. Gehe zu: **SSL/TLS** > **Overview**
2. Wähle: **Full (strict)**

**Warum "Full (strict)"?**
- ✅ Cloudflare ↔ Server ist verschlüsselt
- ✅ Cloudflare verifiziert Caddy's SSL-Zertifikat
- ✅ Keine Browser-Warnungen
- ✅ Ende-zu-Ende-Verschlüsselung

**NICHT verwenden:**
- ❌ **Flexible:** Nur Browser ↔ CF verschlüsselt, CF ↔ Server unverschlüsselt
- ❌ **Full:** Verschlüsselt, aber CF verifiziert Zertifikat nicht

### Schritt 7: Automatisches HTTPS aktivieren

**SSL/TLS** > **Edge Certificates**:
- ✅ **Always Use HTTPS:** ON
- ✅ **Automatic HTTPS Rewrites:** ON
- ✅ **Minimum TLS Version:** 1.2
- ✅ **Opportunistic Encryption:** ON
- ✅ **TLS 1.3:** ON

---

## Phase 4: Performance-Optimierung

### Schritt 8: Caching konfigurieren

**Caching** > **Configuration**:
- ✅ **Caching Level:** Standard
- ✅ **Browser Cache TTL:** Respect Existing Headers

**Page Rules (Optional - 3 kostenlos):**

**Rule 1: Cache alles für clocklight.de:**
```
URL: clocklight.de/*
Settings:
- Cache Level: Cache Everything
- Browser Cache TTL: 1 month
- Edge Cache TTL: 1 month
```

### Schritt 9: Speed-Optimierungen

**Speed** > **Optimization**:
- ✅ **Auto Minify:** CSS ✓, JavaScript ✓, HTML ✓
- ✅ **Brotli Compression:** ON
- ✅ **Early Hints:** ON

---

## Phase 5: Security-Einstellungen

### Schritt 10: Security Level

**Security** > **Settings**:
- **Security Level:** Medium (Standard)
- **Challenge Passage:** 30 Minutes
- **Browser Integrity Check:** ON

### Schritt 11: Firewall-Rules (Optional)

**Beispiel: Deutschland-only für Admin-Bereiche:**
```
Security > WAF > Firewall Rules > Create Rule

Rule Name: Block non-German Admin Access
Expression:
  (http.request.uri.path contains "/admin") and
  (ip.geoip.country ne "DE")
Action: Block
```

**Beispiel: Rate Limiting (5 Rules kostenlos):**
```
Rule Name: Aggressive Requests
Expression:
  (http.request.uri.path contains "/wp-login") and
  (cf.threat_score gt 10)
Action: Challenge (Captcha)
```

---

## Phase 6: Verifizierung & Testing

### Schritt 12: DNS-Propagation prüfen

**Warte 10-30 Minuten, dann teste:**

```bash
# 1. DNS-Auflösung prüfen
dig +short clocklight.de

# Sollte CF-IPs zeigen (104.x.x.x oder 172.x.x.x)
# NICHT mehr 46.224.122.105
```

### Schritt 13: Website testen

```bash
# 2. HTTP-Headers prüfen
curl -I https://clocklight.de

# Sollte zeigen:
# server: cloudflare
# cf-ray: ...
```

**Im Browser öffnen:**
- https://clocklight.de
- https://www.clocklight.de

**Sollte funktionieren:** ✅

### Schritt 14: Mailserver testen (KRITISCH!)

```bash
# 3. Mail-DNS prüfen
dig +short mail.clocklight.de

# MUSS zeigen: 46.224.122.105 (NICHT CF-IP!)

# 4. SMTP testen
telnet mail.clocklight.de 25
# Sollte verbinden und SMTP-Banner zeigen

# 5. Webmail testen
curl -I https://mail.clocklight.de

# Sollte zeigen: via: 1.1 Caddy (NICHT cloudflare)
```

### Schritt 15: Beszel testen

```bash
# 6. Beszel prüfen
dig +short beszel.clocklight.de
# MUSS zeigen: 46.224.122.105

curl -I https://beszel.clocklight.de
# Sollte zeigen: via: 1.1 Caddy
```

**Im Browser öffnen:**
- https://beszel.clocklight.de
- Login sollte funktionieren

---

## Phase 7: Monitoring & Analytics

### Schritt 16: Analytics aktivieren

**Analytics** > **Traffic**:
- Zeigt Requests/Stunde
- Bandwidth gespart
- Cache-Hit-Rate
- Top Countries
- Top Paths

**Wichtige Metriken:**
- **Cache-Hit-Rate:** Sollte >80% sein für statische Sites
- **Bandwidth gespart:** Zeigt wie viel Traffic CF cached
- **Threats:** Geblockte Bots/Attacken

---

## Troubleshooting

### Problem: "Too many redirects"

**Symptom:** Website lädt nicht, Browser zeigt "Redirect Loop"

**Lösung:**
```
SSL/TLS > Overview > Ändern zu "Full (strict)"
Cache leeren:
  Caching > Configuration > Purge Everything
```

### Problem: "Origin server not responding" (Error 521)

**Symptom:** CF kann Server nicht erreichen

**Lösung:**
```bash
# 1. Prüfe ob Caddy läuft
docker ps | grep caddy

# 2. Prüfe Caddy-Logs
docker logs caddy-webserver --tail 50

# 3. Prüfe ob Port 443 offen ist
ss -tlnp | grep :443
```

### Problem: "Invalid SSL Certificate" (Error 526)

**Symptom:** CF kann SSL nicht verifizieren

**Lösung:**
```
SSL/TLS > Overview > Ändern zu "Full" (temporär)
Oder: Caddy SSL-Zertifikat neu generieren lassen (Caddy macht das automatisch)
```

### Problem: Mail sendet/empfängt nicht

**Symptom:** Emails kommen nicht an

**Lösung:**
```
KRITISCH: mail.clocklight.de MUSS auf "DNS Only" sein!

Cloudflare Dashboard:
1. DNS > Finde mail.clocklight.de
2. Klicke auf Orange Cloud
3. Ändere zu Grey Cloud (DNS Only)
4. Speichern
```

### Problem: Website langsamer als vorher

**Symptom:** Schlechte Performance

**Lösung:**
```
1. Prüfe Cache-Hit-Rate in Analytics
   - Sollte >80% sein

2. Page Rules aktivieren:
   - Cache Everything für statische Pfade

3. Argo Smart Routing aktivieren (kostenpflichtig)
```

---

## Optional: wolfgang-burger.de migrieren

**Hinweis:** wolfgang-burger.de zeigt aktuell auf 92.204.239.183 (anderer Server)

### Soll wolfgang-burger.de zu diesem Server migriert werden?

**Falls JA:**

1. **Caddy-Config prüfen:**
```bash
cat /srv/caddy/sites/wolfgang-burger.de.caddy
# Sollte konfiguriert sein
```

2. **Website-Dateien prüfen:**
```bash
ls -la /var/www/wolfgang-burger.de/
# Sollten vorhanden sein
```

3. **In Cloudflare:**
```
Type: A
Name: @
Content: 46.224.122.105  ← Neue IP!
Proxy: ☁️ Proxied

Type: CNAME
Name: www
Content: wolfgang-burger.de
Proxy: ☁️ Proxied
```

4. **Warten bis DNS propagiert**

5. **Testen:**
```bash
dig +short wolfgang-burger.de
# Sollte CF-IP zeigen

curl -I https://wolfgang-burger.de
# Sollte funktionieren
```

**Falls NEIN:**
- wolfgang-burger.de bleibt auf altem Server
- Nur clocklight.de durch Cloudflare

---

## Checkliste: Cloudflare-Setup

### Vorbereitung:
- [ ] Cloudflare-Account erstellt
- [ ] Email verifiziert

### Domain clocklight.de:
- [ ] Domain zu CF hinzugefügt
- [ ] DNS-Records importiert
- [ ] A-Record @ → 46.224.122.105 (Proxied ☁️)
- [ ] CNAME www → clocklight.de (Proxied ☁️)
- [ ] A-Record mail → 46.224.122.105 (DNS Only ⚠️)
- [ ] A-Record beszel → 46.224.122.105 (DNS Only ⚠️)
- [ ] MX-Records vorhanden (falls Mail)
- [ ] Nameserver beim Registrar geändert
- [ ] DNS-Propagation abgewartet (10-30 Min)

### SSL/TLS:
- [ ] SSL/TLS Mode: Full (strict)
- [ ] Always Use HTTPS: ON
- [ ] Automatic HTTPS Rewrites: ON
- [ ] TLS 1.3: ON

### Performance:
- [ ] Auto Minify aktiviert
- [ ] Brotli aktiviert
- [ ] Page Rules erstellt (optional)

### Security:
- [ ] Security Level: Medium
- [ ] Browser Integrity Check: ON
- [ ] Firewall Rules erstellt (optional)

### Testing:
- [ ] clocklight.de lädt im Browser
- [ ] www.clocklight.de funktioniert
- [ ] HTTPS funktioniert
- [ ] Headers zeigen "server: cloudflare"
- [ ] mail.clocklight.de zeigt direkte IP
- [ ] SMTP funktioniert (telnet mail.clocklight.de 25)
- [ ] Webmail funktioniert (https://mail.clocklight.de:8443)
- [ ] beszel.clocklight.de funktioniert
- [ ] Beszel-Login möglich

---

## Performance-Erwartung

### Vorher (ohne Cloudflare):
```
Deutschland: ~20-30ms
Europa: ~50-80ms
USA: ~120-180ms
Asien: ~200-300ms
```

### Nachher (mit Cloudflare):
```
Deutschland: ~10-15ms (CF Frankfurt)
Europa: ~15-25ms (CF Amsterdam, Paris, etc.)
USA: ~15-30ms (CF New York, LA, etc.)
Asien: ~20-40ms (CF Singapore, Tokyo, etc.)
```

**Verbesserung:** 70-90% schneller weltweit

### Bandwidth-Einsparung:
- **Cache-Hit-Rate:** 80-95% (statische Sites)
- **Bandwidth gespart:** 70-90%
- **Server-Last:** Reduziert um 80-90%

---

## Nach dem Setup

### Regelmäßige Checks:
- **Wöchentlich:** Analytics prüfen (Traffic, Threats)
- **Monatlich:** Cache-Hit-Rate optimieren
- **Quartalsweise:** Firewall-Rules anpassen

### Cloudflare Analytics nutzen:
- Traffic-Trends erkennen
- Angriffsversuche sehen
- Performance monitoren
- Cache-Effizienz prüfen

### Warnung bei Änderungen:
⚠️ **NIEMALS mail.clocklight.de auf Proxied setzen!**
⚠️ **MX-Records NICHT löschen!**
⚠️ **SPF/DKIM/DMARC-Records beibehalten!**

---

## Kosten

### Cloudflare Free Tier (0€):
- ✅ Unbegrenzter Traffic
- ✅ DDoS-Schutz
- ✅ SSL/TLS
- ✅ CDN
- ✅ Basic WAF
- ✅ 3 Page Rules
- ✅ 5 Firewall Rules
- ✅ Analytics

**Völlig ausreichend für dein Setup!**

### Upgrades (optional):
- **Pro:** ~20€/Monat - Bessere Analytics, Image-Optimization
- **Business:** ~200€/Monat - 100% Uptime-SLA, Custom SSL
- **Enterprise:** Custom Pricing - Dedicated Support

**Empfehlung:** Free Tier reicht völlig aus

---

## Support & Hilfe

### Cloudflare Community:
- https://community.cloudflare.com/

### Cloudflare Docs:
- https://developers.cloudflare.com/

### Bei Problemen:
1. Cloudflare Status: https://www.cloudflarestatus.com/
2. Community Forum durchsuchen
3. Ticket bei CF Support (nur Pro+)

---

## Nächste Schritte

### Sofort nach Setup:
1. ✅ Monitoring in Beszel checken (Server-Last sollte sinken)
2. ✅ Website-Performance testen (sollte schneller sein)
3. ✅ Mail testen (SMTP/IMAP/Webmail)

### Nach 24 Stunden:
1. Analytics checken (Traffic, Cache-Hit-Rate)
2. Performance-Verbesserung messen
3. Bandwidth-Einsparung prüfen

### Nach 1 Woche:
1. Firewall-Rules optimieren (falls nötig)
2. Page Rules erweitern (falls nötig)
3. Cache-Strategie anpassen

---

**Erstellt:** 27. Dezember 2025
**Letztes Update:** 27. Dezember 2025
**Version:** 1.0
**Status:** Ready to implement

---

## Bereit zum Start?

Folge den Schritten oben Schritt für Schritt.

**Geschätzte Gesamt-Dauer:** 45-60 Minuten
**Schwierigkeitsgrad:** Mittel
**Risiko:** Niedrig (bei korrekter Konfiguration)

Viel Erfolg! 🚀
