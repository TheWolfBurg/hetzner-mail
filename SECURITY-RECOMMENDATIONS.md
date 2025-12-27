# Sicherheitsempfehlungen für mail.clocklight.de

**Stand:** 27. Dezember 2025
**Status:** Analyse & Empfehlungen

---

## Aktueller Sicherheitsstatus

### ✅ Bereits implementiert (GUT)
- ✅ **Fail2ban** für SSH, Mailcow Auth, Mailcow Postfix
- ✅ **Automatische Updates** (unattended-upgrades)
- ✅ **SSL/TLS** via Let's Encrypt (ACME)
- ✅ **SSH-Key basierte Authentifizierung** für Backups
- ✅ **Monitoring & Alerting** (täglich + bei Problemen)
- ✅ **Docker Firewall-Regeln** aktiv
- ✅ **Beszel Monitoring** mit Authentifizierung

---

## Empfohlene Verbesserungen

### 🔴 KRITISCH (sofort umsetzen)

#### 1. SSH Root-Login einschränken
**Problem:** Root-Login per SSH ist erlaubt
**Risiko:** Angreifer können direkt als root einloggen

**Lösung:**
```bash
# /etc/ssh/sshd_config ändern:
PermitRootLogin prohibit-password  # Nur SSH-Key, kein Passwort
# ODER noch besser:
PermitRootLogin no                 # Root-Login komplett verbieten

# Vorher: Non-root User mit sudo-Rechten erstellen!
adduser admin
usermod -aG sudo admin
# SSH-Key für admin-User kopieren

# SSH neu starten:
systemctl restart sshd
```

**Wichtig:** Teste erst mit einer zweiten SSH-Session, bevor du die erste schließt!

#### 2. Security Headers für Webserver hinzufügen
**Problem:** Keine Security-Headers (HSTS, X-Frame-Options, etc.)
**Risiko:** XSS, Clickjacking, MITM-Angriffe

**Lösung - Caddy Snippet erstellen:**
```bash
# /srv/caddy/snippets/security_headers.caddy
header {
    # HSTS - Force HTTPS for 1 year
    Strict-Transport-Security "max-age=31536000; includeSubDomains; preload"

    # Prevent clickjacking
    X-Frame-Options "SAMEORIGIN"

    # Prevent MIME sniffing
    X-Content-Type-Options "nosniff"

    # XSS Protection
    X-XSS-Protection "1; mode=block"

    # Referrer Policy
    Referrer-Policy "strict-origin-when-cross-origin"

    # Permissions Policy
    Permissions-Policy "geolocation=(), microphone=(), camera=()"
}
```

**In Site-Configs einbinden:**
```
beszel.clocklight.de {
    import ../snippets/security_headers.caddy
    import ../snippets/compression.caddy
    reverse_proxy beszel:8090
}
```

#### 3. Offene Ports prüfen
**Problem:** Port 8080 und 8090 sind direkt exponiert
**Risiko:** Unnötige Angriffsfläche

**Aktion:**
```bash
# Was läuft auf Port 8080?
docker ps | grep 8080
lsof -i :8080

# Port 8090 (Beszel) sollte NUR über Caddy erreichbar sein
# In docker-compose.yml ändern:
ports:
  - "127.0.0.1:8090:8090"  # Nur localhost, nicht 0.0.0.0
```

---

### 🟡 WICHTIG (zeitnah umsetzen)

#### 4. UFW Firewall aktivieren (falls gewünscht)
**Status:** UFW ist inaktiv, iptables/Docker verwalten Firewall
**Empfehlung:** UFW kann aktiviert werden, ist aber NICHT zwingend nötig

**Wenn UFW aktiviert werden soll:**
```bash
# VORSICHT: Erst Regeln setzen, dann aktivieren!
ufw default deny incoming
ufw default allow outgoing
ufw allow 22/tcp comment 'SSH'
ufw allow 25/tcp comment 'SMTP'
ufw allow 80/tcp comment 'HTTP'
ufw allow 443/tcp comment 'HTTPS'
ufw allow 587/tcp comment 'SMTP Submission'
ufw allow 993/tcp comment 'IMAPS'
ufw allow 8443/tcp comment 'Mailcow Admin'

# Docker-Integration
ufw allow from 172.16.0.0/12 to any

# UFW aktivieren
ufw enable
```

**Alternative:** Bei Docker-Setup kann UFW Probleme machen.
**Empfehlung:** Aktuelles iptables/Docker-Setup ist OK, UFW ist optional.

#### 5. Passwort-Authentifizierung für SSH deaktivieren
**Status:** Unbekannt (nicht in Config gesetzt)
**Empfehlung:** Explizit deaktivieren

```bash
# /etc/ssh/sshd_config:
PasswordAuthentication no
ChallengeResponseAuthentication no
PubkeyAuthentication yes

systemctl restart sshd
```

#### 6. Backup-Verschlüsselung
**Problem:** Backups werden unverschlüsselt übertragen und gespeichert
**Risiko:** Bei Kompromittierung des Backup-Servers sind Daten lesbar

**Lösung:**
```bash
# GPG-Verschlüsselung für Backups
# Schlüssel generieren:
gpg --full-generate-key

# In backup-data.sh:
gpg --encrypt --recipient backup@mail.clocklight.de file.tar.gz
```

---

### 🟢 OPTIONAL (Nice-to-have)

#### 7. SSH-Port ändern
**Nutzen:** Reduziert automatisierte Angriffe (security through obscurity)
**Aufwand:** Gering

```bash
# /etc/ssh/sshd_config:
Port 2222  # Statt 22

# Fail2ban anpassen
# Firewall anpassen
systemctl restart sshd
```

#### 8. 2FA für SSH
**Nutzen:** Zusätzliche Sicherheitsebene
**Aufwand:** Mittel

```bash
apt install libpam-google-authenticator
# Google Authenticator einrichten
```

#### 9. Intrusion Detection
**Nutzen:** Erkennt Dateiänderungen
**Aufwand:** Hoch

```bash
# AIDE (Advanced Intrusion Detection Environment)
apt install aide
aide --init
```

#### 10. Rate Limiting auf Webserver
**Nutzen:** DDoS-Schutz
**Aufwand:** Gering

```bash
# Caddy Rate Limiting Plugin
# oder Cloudflare vorschalten
```

#### 11. Docker Security Hardening
```bash
# AppArmor Profile für Container
# Seccomp Profile
# User Namespaces
# Read-only Root Filesystem
```

---

## Priorisierte Umsetzung

### Sofort (30 Minuten):
1. ✅ SSH Root-Login einschränken (prohibit-password)
2. ✅ Security Headers hinzufügen
3. ✅ Offene Ports prüfen & einschränken

### Diese Woche:
4. Password-Authentication für SSH deaktivieren
5. Backup-Verschlüsselung implementieren

### Optional (wenn Zeit/Bedarf):
6. SSH-Port ändern
7. 2FA für SSH
8. UFW aktivieren (nur wenn gewünscht)

---

## Risikoeinschätzung

### Aktuelles Risiko-Level: 🟡 MITTEL

**Begründung:**
- ✅ Grundlegende Sicherheit ist gut (Fail2ban, Updates, SSL)
- ⚠️ Einige Best Practices fehlen (Root-Login, Security Headers)
- ⚠️ Backups sind unverschlüsselt
- ✅ Monitoring & Alerting vorhanden

**Nach Umsetzung der kritischen Punkte:** 🟢 GUT

---

## Weitere Empfehlungen

### Regelmäßige Wartung:
- **Wöchentlich:** Fail2ban-Logs prüfen
- **Monatlich:** Security-Updates manuell prüfen
- **Quartalsweise:** Backup-Restore testen
- **Jährlich:** Komplettes Security-Audit

### Monitoring erweitern:
```bash
# Failed SSH logins tracken
grep "Failed password" /var/log/auth.log | tail -20

# Ungewöhnliche Netzwerk-Verbindungen
netstat -tulpn | grep ESTABLISHED

# Docker-Container auf Updates prüfen
docker images --format "{{.Repository}}:{{.Tag}}" | xargs -n1 docker pull
```

---

## Checkliste

### Kritisch:
- [ ] SSH: PermitRootLogin prohibit-password setzen
- [ ] Security Headers in Caddy hinzufügen
- [ ] Port 8090 nur auf localhost binden
- [ ] Port 8080 identifizieren & ggf. schließen

### Wichtig:
- [ ] PasswordAuthentication no setzen
- [ ] Backup-Verschlüsselung einrichten
- [ ] Non-root Admin-User erstellen

### Optional:
- [ ] SSH-Port ändern
- [ ] 2FA für SSH
- [ ] UFW aktivieren (wenn gewünscht)
- [ ] Docker Security Hardening

---

## Fazit

**Deine aktuelle Sicherheit ist solide für einen Mailserver.**

Die wichtigsten Grundlagen sind implementiert:
- Fail2ban schützt vor Brute-Force
- Automatische Updates halten System aktuell
- Monitoring erkennt Probleme frühzeitig

**Empfehlung:** Setze die 3 kritischen Punkte um (30 Min Aufwand), dann hast du ein sehr gutes Sicherheitsniveau.

Alles weitere ist "Nice-to-have" und hängt von deinem Sicherheitsbedürfnis ab.

---

**Letztes Update:** 27. Dezember 2025
**Nächste Review:** März 2026
