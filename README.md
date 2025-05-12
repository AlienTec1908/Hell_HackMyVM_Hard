# Hell (HackMyVM) - Penetration Test Bericht

![Hell.png](Hell.png)

**Datum des Berichts:** 24. April 2023  
**VM:** Hell  
**Plattform:** HackMyVM ([Link zur VM](https://hackmyvm.eu/machines/machine.php?vm=Hell))  
**Autor der VM:** DarkSpirit  
**Original Writeup:** [https://alientec1908.github.io/Hell_HackMyVM_Hard/](https://alientec1908.github.io/Hell_HackMyVM_Hard/)

---

## Disclaimer

**Wichtiger Hinweis:** Dieser Bericht und die darin enthaltenen Informationen dienen ausschließlich zu Bildungs- und Forschungszwecken im Bereich der Cybersicherheit. Die hier beschriebenen Techniken und Werkzeuge dürfen nur in legalen und autorisierten Umgebungen (z.B. auf eigenen Systemen oder mit ausdrücklicher Genehmigung des Eigentümers) angewendet werden. Jegliche illegale Nutzung der hier bereitgestellten Informationen ist strengstens untersagt. Der Autor übernimmt keine Haftung für Schäden, die durch Missbrauch dieser Informationen entstehen. Handeln Sie stets verantwortungsbewusst und ethisch.

---

## Inhaltsverzeichnis

1.  [Zusammenfassung](#zusammenfassung)
2.  [Verwendete Tools](#verwendete-tools)
3.  [Phase 1: Reconnaissance](#phase-1-reconnaissance)
4.  [Phase 2: Initial Access (Anonymous FTP & Credential Discovery)](#phase-2-initial-access-anonymous-ftp--credential-discovery)
5.  [Phase 3: Privilege Escalation (Nicht dokumentiert)](#phase-3-privilege-escalation-nicht-dokumentiert)
6.  [Proof of Concept (Anonymous FTP)](#proof-of-concept-anonymous-ftp)
7.  [Flags](#flags)
8.  [Empfohlene Maßnahmen (Mitigation)](#empfohlene-maßnahmen-mitigation)

---

## Zusammenfassung

Dieser Bericht dokumentiert die Analyse der virtuellen Maschine "Hell" von HackMyVM (Schwierigkeitsgrad: Hard). Die initiale Erkundung identifizierte offene FTP- (Port 21) und SSH-Dienste (Port 22). Der FTP-Dienst erlaubte **anonymen Zugriff**, was die sofortige Kompromittierung einer ersten Flag (`flag.txt`) ermöglichte. Eine weitere Untersuchung des FTP-Servers enthüllte eine versteckte Datei `.passwd`, die das Klartext-Passwort `webserver2023!` enthielt.

Der vorliegende Writeup impliziert, dass dieses Passwort zusammen mit dem Benutzernamen `beilul` (dessen Herkunft im Log nicht dokumentiert ist) für einen erfolgreichen SSH-Login verwendet wurde. Die anschließenden Schritte zur Privilegieneskalation von `beilul` zu `root` sind im bereitgestellten Log **nicht dokumentiert**, werden aber durch das Auffinden der finalen User- und Root-Flags am Ende des Berichts impliziert. Die kritischste Schwachstelle war der offene, anonyme FTP-Zugang mit sensiblen Dateien.

---

## Verwendete Tools

*   `arp-scan`
*   `nmap`
*   `ftp`
*   `cat`
*   `lftp` (versucht, erfolglos/inkonklusiv)
*   `ssh-keyscan`
*   `ping6`
*   `ssh` (versucht, Login-Schritte fehlen)
*   `hydra` (für Passwort-/Benutzer-Verifizierung)
*   `ls` (via FTP)
*   `get` (via FTP)

---

## Phase 1: Reconnaissance

1.  **Netzwerk-Scan:**
    *   `arp-scan -l` identifizierte den Host `192.168.2.135` (VirtualBox VM) als Ziel.

2.  **Port-Scan (Nmap):**
    *   Ein umfassender `nmap`-Scan (`nmap -sS -sC -T5 -A 192.168.2.135 -p-`) offenbarte:
        *   **Port 21 (FTP):** vsftpd 3.0.5. **Kritisch:** Anonymer Login erlaubt (`ftp-anon: Anonymous FTP login allowed`). Das Nmap-Skript listete auch direkt eine lesbare `flag.txt` im FTP-Root.
        *   **Port 22 (SSH):** OpenSSH 8.9p1 Ubuntu 3ubuntu0.1
    *   IPv6-Scans (`nmap -6 [...]`) bestätigten die Erreichbarkeit von FTP und SSH auch über IPv6 (`fe80::a00:27ff:feee:412c`).

---

## Phase 2: Initial Access (Anonymous FTP & Credential Discovery)

1.  **Anonymer FTP-Zugriff:**
    *   Verbindung mit `ftp 192.168.2.135`.
    *   Login als `Anonymous` mit beliebigem Passwort war erfolgreich.

2.  **Download der ersten Flag:**
    *   `ls` bestätigte die sichtbare `flag.txt`.
    *   `get flag.txt` lud die Datei erfolgreich herunter.
    *   `cat flag.txt` zeigte die erste Flag: `HELL{4N0NYM0U5_15_7H3_B357_U53R}`.

3.  **Entdeckung der Passwortdatei:**
    *   `ls -la` innerhalb der FTP-Sitzung enthüllte die versteckte Datei `.passwd`.
    *   `get .passwd` lud diese Datei erfolgreich herunter.

4.  **Extraktion des Passworts:**
    *   `cat .passwd` zeigte den Inhalt: `The password is: webserver2023!`

5.  **Implizierter nächster Schritt (Nicht im Log dokumentiert):**
    *   Obwohl der Ursprung des Benutzernamens `beilul` im Log nicht erklärt wird, deutet der weitere Verlauf darauf hin, dass ein SSH-Login als `beilul` mit dem Passwort `webserver2023!` erfolgte. Die konkreten Befehle hierfür fehlen.

---

## Phase 3: Privilege Escalation (Nicht dokumentiert)

**Wichtiger Hinweis:** Die spezifischen Schritte, die zur Eskalation von Benutzerrechten (`beilul`) zu Root-Rechten führten, sind im zugrundeliegenden Writeup-Log **nicht enthalten**.

Es wird **angenommen**, dass nach dem erfolgreichen SSH-Login als `beilul:webserver2023!` Methoden wie die Überprüfung von `sudo -l`, die Suche nach SUID/GUID-Binaries, die Analyse von Cronjobs oder die Ausnutzung von Kernel- oder Dienst-Schwachstellen angewendet wurden. Der Erfolg dieser Phase wird lediglich durch das Vorhandensein der finalen Root-Flag am Ende des Berichts impliziert.

---

## Proof of Concept (Anonymous FTP)

**Kurzbeschreibung:** Der FTP-Server auf Port 21 ist unsicher konfiguriert und erlaubt anonymen Lesezugriff auf das Wurzelverzeichnis. Dies ermöglichte das Herunterladen von sensiblen Dateien, einschließlich einer Flag und einer Datei, die ein Klartext-Passwort enthielt.

**Schritte:**
1.  Verbinde dich mit dem FTP-Server:
    ```bash
    ftp 192.168.2.135
    ```
2.  Logge dich als `Anonymous` mit einem beliebigen Passwort ein.
3.  Liste versteckte Dateien auf:
    ```ftp
    ftp> ls -la 
    ```
    *(Zeigt `.passwd` und `flag.txt`)*
4.  Lade die Dateien herunter:
    ```ftp
    ftp> get flag.txt
    ftp> get .passwd
    ftp> quit
    ```
5.  Zeige den Inhalt der Passwortdatei an:
    ```bash
    cat .passwd 
    ```
**Ergebnis:** Zugriff auf die erste Flag und das Passwort `webserver2023!`.

---

## Flags

*   **Flag 1 (via Anonymous FTP):**
    ```
    HELL{4N0NYM0U5_15_7H3_B357_U53R} 
    ```
    *(Gefunden in `flag.txt` auf dem FTP-Server)*

*   **User Flag (Impliziert):**
    ```
    HELL{0V3RF10W_F0R_B3G1NN3R5}
    ```
    *(Vermutlich in `/home/beilul/user.txt`, Schritte zum Erhalt fehlen)*

*   **Root Flag (Impliziert):**
    ```
    HELL{R54C7F7001_OR_M4NU41?}
    ```
    *(Vermutlich in `/root/root.txt`, Schritte zur Privilegieneskalation fehlen)*

---

## Empfohlene Maßnahmen (Mitigation)

*   **FTP-Sicherheit:**
    *   **DRINGEND:** Deaktivieren Sie den anonymen FTP-Zugriff (`anonymous_enable=NO` in `vsftpd.conf`). Anonymer Zugriff stellt fast immer ein hohes Sicherheitsrisiko dar.
    *   Entfernen Sie sensible Dateien (Flags, Passwortdateien wie `.passwd`) vom FTP-Server-Verzeichnis.
    *   Überprüfen Sie die Dateiberechtigungen im FTP-Verzeichnis sorgfältig.
*   **Passwort-Management:**
    *   **Speichern Sie niemals Passwörter im Klartext** in Dateien, insbesondere nicht auf öffentlich zugänglichen Diensten.
    *   Ändern Sie das kompromittierte Passwort `webserver2023!` sofort für alle Benutzerkonten, die es verwenden (insbesondere `beilul` und potenziell `root`, falls es wiederverwendet wurde).
    *   Erzwingen Sie die Verwendung starker, einzigartiger Passwörter für alle Benutzer und Dienste.
*   **SSH-Härtung:**
    *   Implementieren Sie Ratenbegrenzung oder Tools wie `fail2ban`, um Brute-Force-Angriffe auf SSH zu erschweren.
    *   Bevorzugen Sie Schlüssel-Authentifizierung gegenüber Passwort-Authentifizierung.
*   **Privilegieneskalation:**
    *   **Da die Eskalationsmethode unbekannt ist:** Führen Sie eine gründliche Sicherheitsüberprüfung des Systems durch. Überprüfen Sie insbesondere:
        *   `sudo`-Regeln (`sudo -l` für alle relevanten Benutzer).
        *   Dateien mit SUID/GUID-Bits (`find / -perm -6000 -type f 2>/dev/null`).
        *   Dateien mit Capabilities (`getcap -r / 2>/dev/null`).
        *   Systemweite und benutzerdefinierte Cronjobs.
        *   Laufende Dienste und deren Konfigurationen.
        *   Kernel-Version auf bekannte Schwachstellen (`uname -a`).
    *   Stellen Sie sicher, dass das Prinzip der geringsten Rechte angewendet wird.

---

**Ben C. - Cyber Security Reports**
