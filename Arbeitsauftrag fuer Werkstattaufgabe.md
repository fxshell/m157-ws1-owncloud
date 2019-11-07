# Arbeitsauftrag fuer Werkstattaufgabe

Markdown-Dokument für Arbeitauftrag fuer Werkstattauftrag.

***

## Inhaltsverzeichnis

* [Autoren, Versionierung des Dokumentes](/Arbeitsauftrag%20fuer%20Werkstattaufgabe.md#autoren-versionierung-des-dokumentes)
* [Einfuerung](/Arbeitsauftrag%20fuer%20Werkstattaufgabe.md#einfuerung)
* [Materialliste](/Arbeitsauftrag%20fuer%20Werkstattaufgabe.md#materialliste)
* [Installationsanleitung](/Arbeitsauftrag%20fuer%20Werkstattaufgabe.md#installationsanleitung)
* [Qualitaetskontrolle](/)
* [Error-Handling](/)

***

# Autoren, Versionierung des Dokumentes
## Autoren
Diese Markdown Dokumentation wurde von Luca Widmer zusammen mit Till Wigger für das Modul 157 an der TBZ entworfen.

## Versionierung des Dokumentes
| Datum         | Beschreibung  | Versionierung  |
| ------------- |:-------------:| -----:|
| 03. Oktober 2019      | Markdown Dokument initialisiert | 0.1 |
| 10. Oktober 2019      | Inhaltsverzeichnis und Titel erstellt      |   0.2 |
| 31. Oktober 2019      | Kapitel Autoren, Versionierung des Dokumentes erstellt und fertiggestellt     |   0.3 |
| 31. Oktober 2019      | Kapitel Einfuerung erstellt und fertiggestellt     |   0.4 |
| 31. Oktober 2019      | Kapitel Materialliste erstellt und fertiggestellt     |   0.5 |
| 01. November 2019 | Kapitel Installationsanleitung erstellt und fertiggestellt | 0.6 |
| 01. November 2019 | Kapitel Installationsanleitung angepasst | 0.6.1 |
| 01. November 2019 | Inhaltsverzeichnis angepasst | 0.6.2 |
| 07. Novemebr 2019 | Zusatzaufgabe erstellt | 0.7 |


***

# Einfuerung
Zum jetzigen Stand sind bereits Unterlagen für Schüler zum installieren und konfigurieren von einer ownCloud Umgebung vorhanden. Die Unterlagen sind aber mittlerweile nicht mehr ganz aktuell und haben in gewissen Punkten verbesserungsbedarf. OwnCloud ist eine freie Software für das Speichern von Daten (Filehosting) auf einem eigenen Server. Bei Einsatz eines entsprechenden Clients wird dieser automatisch mit einem lokalen Verzeichnis synchronisiert. Dadurch kann von mehreren Rechnern auf einen konsistenten Datenbestand zugegriffen werden. Der Auftrag ist, die Cloud Software ownCloud auf einem von der Schule zur Verfügung gestellten Raspberry Pi’s zu installieren & konfigurieren. Zusätzlich muss noch Fail2Ban installiert und die nicht benötigten Ports bei der UFW Firewall deaktiviert werden.

***

# Materialliste
## Hardware
* Raspberry Pi 3b+
* Notebook
* LAN Kabel
* MicroSD Karte
* Monitor
* Tastatur & Maus

## Software
* SSH (Putty)

***

# Installationsanleitung
## Service User erstellen
1. Mit dem foldenden Befel einen neuen User erstellen.<br>
```sudo adduser owncloud```
2. Den zuvor erstellen User zu de sudoers hinzufügen.<br>
```usermod -aG sudo owncloud```
3. Mit dem owncloud User einloggen.<br>
4. Ins Home Vezeichnis des owncloud Users wechseln.<br>

## PHP Extensions, Tools, Apache2 & MySQL installieren
1. Den folgenden Befehl ausführen und die Installation der PHP Extensions bestätigen.<br>
```sudo apt-get install -y vim wget mysql-server apache2 php-intl php-mbstring php-gd php-curl php-mysql php-zip php-dom php-xmlwriter php-simplexml```

## ownCloud vorbereiten
1. Mit dem wget Befehl ownCloud herunterladen.<br>
```download.owncloud.org/community/owncloud-10.3.0.tar.bz2```
2. Das heruntergeladene Archiv entpacken.<br>
```tar -xjf owncloud-10.3.0.tar.bz2```
3. Das entpackte Verzeichnis ins Webroot kopieren.<br>
```sudo cp -r /home/owncloud/owncloud /var/www/html/```

## Apache konfigurieren
1. Eine neue Konfigurationsdatei erstellen.<br>
```vim /etc/apache2/sites-available/owncloud.conf```
2. Die Konfiguration wie folg anpassen.<br>
    ```Alias /owncloud "/var/www/owncloud/"
    <Directory /var/www/owncloud/>
        Options +FollowSymlinks
        AllowOverride All
	    <IfModule mod_dav.c>
            Dav off
        </IfModule>
	    SetEnv HOME /var/www/owncloud
        SetEnv HTTP_HOME /var/www/owncloud
	</Directory>
3. Symlink erstellen.<br>
```ln -s /etc/apache2/sites-available/owncloud.cof /etc/apache2/sites-enabled/owncloud.comf```
4. Den owner des owncloud Verzeichnisses ändern.<br>

## MySQL konfigurieren
1. Mit dem MySQL Server verbinden.<br>
2. Datenbank erstellen.<br>
```CREATE DATABASE owncloud;```
3. Datenbank User erstellen.<br>
```CREATE USER 'ownlcoud'@'localhost' IDENTIFIED BY 'jedeliebtTBZ';```
4. Dem User die Rechte für die owncloud Datenbank zuteilen.<br>
```GRANT ALL PRIVILEGES ON owncloud.* TO 'owncloud'@'localhost;```
```FLUSH PRIVILEGES;```
5. MySQL Verbindung trennen.<br>

## SSL-Zertifikat erstellen und signieren
1. In ein Verteichnis wechseln, welches nur vom Root erreichbar ist.
```sudo cd /root```
2. Key generieren.<br>
```sudo openssl genrsa -out server.key 4096```
3. Zertifikat erstellen.
```openssl req -new -key server.key -out server.csr```
4. Zertifikat signieren.<br>
```sudo openssl x509 -req -days 365 -in server.csr -signkey server.key -out server.crt```
5. Berechtigungen verteilen.<br>
```sudo chmod 400 server.key```
6. Virtual Hosts Datei anpassen.<br>
```sudo nano /etc/apache2/sites-available/owncloud.conf```
```<VirtualHost *:443>
DocumentRoot /var/www
ServerName NAME EURES SERVERS
SSLEngine on
SSLCertificateFile /root/server.crt
SSLCertificateKeyFile /root/server.key
</VirtualHost>
```
7. Virtal Hosts Datei anpassen.<br>
```sudo nano /etc/apache2/sites-available/default```
```<VirtualHost *:443>
DocumentRoot /var/www
ServerName NAME EURES SERVERS
SSLEngine on
SSLCertificateFile /root/server.crt
SSLCertificateKeyFile /root/server.key
</VirtualHost>
```
8. Virtual Hosts Datei anpassen.<br>
```sudo nano /etc/apache2/sites-available/default-ssl```
```<IfModule mod_ssl.c>
<VirtualHost *:443>
ServerAdmin webmaster@localhost

DocumentRoot /var/www
<IfModule mod_ssl.c>
SSLEngine on
SSLCertificateKeyFile /root/server.key
SSLCertificateFile /root/server.crt
SetEnvIf User-Agent ".MSIE." \
nokeepalive ssl-unclean-shutdown \
downgrade-1.0 force-response-1.0
</IfModule>
```
9. Apache2 Ports config anpassen.<br>
```sudo nano /etc/apache2/port.conf```
```VirtualHost *:443
Listen 443
```
10. SSL Konfiguration aktivieren.<br>
```sudo a2enmod ssl```
11. SSL Konfiguration testen.<br>
```apache2ctl configtest```
12. Apache2 neustarten.<br>
```apache2ctl restart```

## ownCloud konfigurieren

1. ownCloud Installation im Webbrowser unter https://hostname_or_ip/owncloud aufrufen.
2. Die SSL Meldung, welche erscheint, weil ein Selbstsigniertes SSL Zertifikat verwendet wurde, bestätigen und fortfahren.
3. Danach muss man den Admin User für die ownCloud installation definieren.
4. Um die Datenbank zu konfigurieren muss man auf Fortgeschritten klicken.
5. Den Eintrag "Datenverzeichnis" kann man so stehen lassen.
6. MySQL auswählen un den zuvor erstellte Datenbankbenutzer "owncloud" mit dem dazugehörigen Passwort angeben.
7. Die zuvor erstellt Datenbank "owncloud" und als Hosts "localhost" angeben.
8. Die konfiguration mit einem klick auf "Installation abschliessen" bestätigen.

## Fail2Ban installieren
1. Um Fail2Ban richtig zu installieren und zu konfigurieren benötigt man im Normalfall meherere Jahre erfahrung. Allerdings sollte auch ein Lernender im 3. Lehrjahr dieses Skill beherrschen. Desswegen muss Fail2Ban ohne Hilfe von dieser Installationsanleitung selber installiert und konfiguriert werden.<br>

## UFW Firewall installieren & konfigurieren
1. Zuerst muss die UFW Firewall auf dem Raspberry Pi insalliert werden.<br>
2. Es ist gut möglich, dass die Firewall nach der Installation icht automatisch startet. Desswegen muss, bevor irgenwelche konfiguration vorgenommen werden können, der Status der Firewall überprüft und falls es notwenig ist auch manuell gestartet werden.<br>
3. Alle Ports nach innen deaktivieren. Achtung! SSH funktioniert danach nicht mehr. Am besten wird der Befehl auf der Console ausgeführt und mit Schritt 4 SSH wieder erlaubt.<br>
```sudo ufw deafult deny incomming```
4. SSH incoming erlauben.<br>
```sudo ufw allow 22```
5. HTTPS incoming erlauben.<br>

## Zusatzsaufgabe
### Requirenments
* CHF 15.-

### Aufgabe
1. Eine Domain registrieren.
2. In den Domain DNS Einstellungen einen A Record auf die öffentliche IP Adresse der TBZ erstellen.
3. Lets Encrypt auf dem Raspberry installieren.
4. Gültiges SSL Zertifikat mit Lets Encrypt generieren.
5. Testen, ob die ownCloud Webseite Zertifiziert ist.

***

#Qualitaetskontrolle

***

#Error-Handling

***