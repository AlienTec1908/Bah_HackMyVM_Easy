# Bah - HackMyVM Writeup

![Bah VM Icon](Bah.png)

Dieses Repository enthält das Writeup für die HackMyVM-Maschine "Bah" (Schwierigkeitsgrad: Easy), erstellt von DarkSpirit. Ziel war es, initialen Zugriff auf die virtuelle Maschine zu erlangen und die Berechtigungen bis zum Root-Benutzer zu eskalieren.

## VM-Informationen

*   **VM Name:** Bah
*   **Plattform:** HackMyVM
*   **Autor der VM:** DarkSpirit
*   **Schwierigkeitsgrad:** Easy
*   **Link zur VM:** [https://hackmyvm.eu/machines/machine.php?vm=Bah](https://hackmyvm.eu/machines/machine.php?vm=Bah)

## Writeup-Informationen

*   **Autor des Writeups:** Ben C.
*   **Datum des Berichts:** 10. Oktober 2022
*   **Link zum Original-Writeup (GitHub Pages):** [https://alientec1908.github.io/Bah_HackMyVM_Easy/](https://alientec1908.github.io/Bah_HackMyVM_Easy/)

## Kurzübersicht des Angriffspfads

Der Angriff auf die Bah-Maschine umfasste die folgenden Schritte:

1.  **Reconnaissance:**
    *   Identifizierung der Ziel-IP (`192.168.2.128`) mittels `arp-scan`.
    *   Ein `nmap`-Scan offenbarte zwei offene Ports: HTTP (80, Nginx 1.18.0, qdPM Login) und MySQL (3306, MariaDB). Der MySQL-Port war extern erreichbar.
2.  **Web Enumeration (qdPM) & Information Disclosure:**
    *   `gobuster` wurde auf Port 80 verwendet und fand die qdPM-Konfigurationsdatei `/core/config/databases.yml`.
    *   Diese Datei enthielt Klartext-Zugangsdaten für die MySQL-Datenbank: `qpmadmin:qpmpazzw`.
3.  **Database Enumeration:**
    *   Mit den gefundenen Zugangsdaten wurde eine Verbindung zur MySQL-Datenbank hergestellt.
    *   In der Datenbank `hidden` wurden in der Tabelle `users` weitere Klartext-Zugangsdaten gefunden: `jwick:Ihaveafuckingpencil` und `rocio:Ihaveaflower`.
    *   Die Tabelle `url` listete potenzielle Subdomains auf, darunter `party.bah.hmv`.
4.  **Subdomain Enumeration:**
    *   `ffuf` (oder manuelle Prüfung) bestätigte, dass `party.bah.hmv` aktiv ist und eine Login-Seite anzeigt.
5.  **Initial Access (qpmadmin? -> rocio):**
    *   Der Bericht deutet einen unklaren initialen Zugriff als `qpmadmin` an, der dann zu einer Reverse Shell führte.
    *   Alternativ und direkter wurde sich mit den aus der Datenbank stammenden Credentials (`rocio:Ihaveaflower`) via `su rocio` (nach Erhalt der `qpmadmin`-Shell) zum Benutzer `rocio` gewechselt.
    *   Die User-Flag (`/home/rocio/user.txt`) wurde gelesen.
6.  **Privilege Escalation (rocio zu root):**
    *   `rocio` hatte keine `sudo`-Rechte.
    *   `pspy64` entdeckte einen als Root laufenden Prozess: `/usr/bin/shellinaboxd -q`.
    *   Es wurde festgestellt (impliziert durch den Exploit), dass die Webseite unter `http://party.bah.hmv/devel/` eine Funktion bot, die es ermöglichte, ein Skript auszuführen.
    *   Ein Reverse-Shell-Skript (`/tmp/dev`) wurde auf das Zielsystem hochgeladen.
    *   Durch Interaktion mit der Webseite (`http://party.bah.hmv/devel/` und Klick auf "Connect") wurde das Skript `/tmp/dev` als Root ausgeführt, was zu einer Root-Shell auf dem Angreifer-System führte.
7.  **Flags:**
    *   Die User-Flag wurde als `rocio` gelesen.
    *   Die Root-Flag wurde aus `/root/root.txt` gelesen.

## Verwendete Tools

*   `arp-scan`
*   `nmap`
*   `gobuster`
*   `curl`
*   `searchsploit`
*   `cat`
*   `mysql`
*   `ffuf`
*   `vi`
*   `nc` (netcat)
*   `python3` (pty, http.server)
*   `export`
*   `passwd`
*   `mkdir`
*   `echo`
*   `su`
*   `ls`
*   `sudo`
*   `pspy64`
*   `shellinaboxd` (als Ziel)
*   `wget`
*   `chmod`
*   `bash`
*   `id`
*   `cd`

## Identifizierte Schwachstellen (Zusammenfassung)

*   **Extern erreichbarer MySQL-Dienst:** Port 3306 war ohne Firewall-Beschränkung zugänglich.
*   **Preisgabe von Datenbank-Zugangsdaten in Konfigurationsdatei:** `databases.yml` war über HTTP erreichbar und enthielt Klartext-Credentials.
*   **Speicherung von Klartextpasswörtern in Datenbank:** Die `hidden.users`-Tabelle enthielt unverschlüsselte Benutzerpasswörter.
*   **Unsichere `shellinaboxd`-Konfiguration/Implementierung:** Ein als Root laufender `shellinaboxd`-Dienst ermöglichte über eine Webseite (`party.bah.hmv/devel/`) die Ausführung eines vom Benutzer hochgeladenen Skripts (`/tmp/dev`) mit Root-Rechten.

## Flags

*   **User Flag (`/home/rocio/user.txt`):** `HdsaMoiuVdsaeqw`
*   **Root Flag (`/root/root.txt`):** `HMVssssshell323`

---

**Wichtiger Hinweis:** Dieses Dokument und die darin enthaltenen Informationen dienen ausschließlich zu Bildungs- und Forschungszwecken im Bereich der Cybersicherheit. Die beschriebenen Techniken und Werkzeuge sollten nur in legalen und autorisierten Umgebungen (z.B. eigenen Testlaboren, CTFs oder mit expliziter Genehmigung) angewendet werden. Das unbefugte Eindringen in fremde Computersysteme ist eine Straftat und kann rechtliche Konsequenzen nach sich ziehen.

---
*Bericht von Ben C. - Cyber Security Reports*
