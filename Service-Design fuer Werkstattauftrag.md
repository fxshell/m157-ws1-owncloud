# Service-Design fuer Werkstattauftrag

Markdown-Dokument für Service-Design fuer Werkstattauftrag.

***

## Inhaltsverzeichnis

* [Autoren, Versionierung des Dokumentes](/)
* [Funktion des Services](/)
* [Benoetigte Hard- und Software](/)
* [Installationsanleitung](/)
* [Testing](/)
* [Uebergabe an den Betrieb](/)
* [Quellen](/)

***

# Autoren, Versionierung des Dokumentes
## Autoren
Diese Markdown Dokumentation wurde von den Pionieren deiner Mudda graziös entworfen.

## Versionierung des Dokumentes
| Datum         | Beschreibung  | Versionierung  |
| ------------- |:-------------:| -----:|
| 03. Oktober 2019      | Markdown Dokument initialisiert | 0.1 |
| 10. Oktober 2019      | Inhaltsverzeichnis und Titel erstellt      |   0.2 |
| 31. Oktober 2019 | Kapitel Autoren, Versionierung des Dokumentes erstellt und fertiggestellt | 0.3 |
| 31. Oktober 2019 | Kapitel Funktion des Services erstellt und fertiggestellt | 0.4 |
| 31. Oktober 2019 | Kapitel Benoetigte Hard- und Software erstellt und fertiggestellt | 0.5 |
| 31. Oktober 2019 | Kapitel Installationsanleitung erstellt und fertiggestellt | 0.6 |
| 01. November 2019 | Kapitel Installationsanleitung angepasst | 0.6.1 |


***

# Funktion des Services
ownCloud ist eine Software für das Speichern von Daten (Filehosting) auf einem eigenen Server. Bei Einsatz eines entsprechenden Clients wird dieser automatisch mit einem lokalen Verzeichnis synchronisiert. Dadurch kann von mehreren Rechnern auf einen konsistenten Datenbestand zugegriffen werden. 

***

# Benoetigte Hard- und Software
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
```su owncloud```
4. Ins Home Vezeichnis des owncloud Users wechseln.<br>
```cd /home/owncloud/```

## PHP Extensions, Tools, Apache2 & MySQL installieren
1. Den folgenden Befehl ausführen und die Installation der PHP Extensions bestätigen.<br>
```sudo apt-get install -y vim wget mysql-server apache2 php-intl php-mbstring php-gd php-curl php-mysql php-zip php-dom php-xmlwriter php-simplexml```

## ownCloud vorbereiten
1. Mit dem folgenden Befehl ownCloud herunterladen.<br>
```wget https://download.owncloud.org/community/owncloud-10.3.0.tar.bz2```
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
```sudo chown -R www-data /var/www/html/owncloud/```

## MySQL konfigurieren
1. Mit dem MySQL Server verbinden.<br>
```sudo mysql -u root -p```
2. Datenbank erstellen.<br>
```CREATE DATABASE owncloud;```
3. Datenbank User erstellen.<br>
```CREATE USER 'ownlcoud'@'localhost' IDENTIFIED BY 'jedeliebtTBZ';```
4. Dem User die Rechte für die owncloud Datenbank zuteilen.<br>
```GRANT ALL PRIVILEGES ON owncloud.* TO 'owncloud'@'localhost;```
```FLUSH PRIVILEGES;```
5. MySQL Verbindung trennen.<br>
```exit;```

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
```sudo nano 7etc/apache2/sites-available/default```
```<VirtualHost *:443>
DocumentRoot /var/www
ServerName NAME EURES SERVERS
SSLEngine on
SSLCertificateFile /root/server.crt
SSLCertificateKeyFile /root/server.key
</VirtualHost>
```
8. Virtual Hosts Datei anpassen.<br>
```sudo nano 7etc/apache2/sites-available/default-ssl```
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
1. Fail2Ban mit folgendem Befehl installieren.<br>
```sudo apt install fail2ban```

## UFW Firewall konfigurieren
1. Zuerst muss die UFW Firewall auf dem Raspberry Pi insalliert werden.<br>
```sudo apt install ufw```
2. Falls die Firewall nicht automatisch gestartet wird, kann man das mit folgenden Befehl erledigen.<br>
```sudo ufw enable```
3. Alle Ports nach innen deaktivieren. Achtung! SSH funktioniert danach nicht mehr. Am besten wird der Befehl auf der Console ausgeführt und mit Schritt 4 SSH wieder erlaubt.<br>
```sudo ufw deafult deny incomming```
4. SSH nach innen erlauben.<br>
```sudo ufw allow 22```
5. HTTPS incoming erlauben.<br>
```sudo ufw allow 443```

***

#Testing

***

#Uebergabe an den Betrieb

***

#Quellen

***