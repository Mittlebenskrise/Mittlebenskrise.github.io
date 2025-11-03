# 1  Vorbereitung: FritzNAS aktivieren[](https://www.andwil.de/weblog/linux-fritznas-mounten-cifs#1-vorbereitung-fritznas-a)

Falls noch nicht geschehen, muss zunächst FritzNAS aktiviert werden:

0. Schließe einen USB-Stick oder eine USB-Festplatte an einen freien USB-Port deiner FritzBox an – dieses Gerät wird als Datenspeicher verwendet werden.
1. Logge dich auf der Admin-Oberfläche der Fritzbox ein. Normalerweise ist diese unter `http://fritz.box` erreichbar, ansonsten direkt über ihre IP (standardmäßig `192.168.178.1`).
2. Unter „Heimnetz“ → „Speicher (NAS)“ setze den Haken bei „**Speicher (NAS) aktiv**“. An dieser Stelle kannst du auch angeben, welche USB-Datenträger freigegeben werden sollen.
3. Weiter unten auf der selben Seite ist der Haken bei „**Zugriff über ein Netzlaufwerk (SMB) aktiv**“ zu aktivieren. Hier kannst du auch einen Namen definieren – standardmäßig lautet dieser `FRITZ.NAS`. Merke dir ihn gut! Achte auch darauf, dass der Domänenname zu deiner Samba-Konfiguration passt (bzw. lasse den Default-Wert `Workgroup` unangetastet).
4. Lege einen **FritzNAS-Benutzer** an (e.g. `fritznas`): Das geht unter „System“ → „FRITZ!Box Benutzer“. Dieser muss „Zugang zu NAS-Inhalten“ haben (Haken setzen!), außerdem müssen die Verzeichnisse angegeben werden, auf welche der Nutzer zugreifen kann – in unserem Beispiel hat der Nutzer Zugriff auf alle verfügbaren Speicher, siehe Bildschirmfotos. (box: Der einfachste Ansatz wäre wohl, einen User einzurichten, der Zugriff auf alle FritzBox-Funktionen sowie Lese- und Schreibrechte auf alle NAS-Speicher hat – man ahnt aber schon, dass dies nicht besonders clever wäre.  
    Gerade in Zeiten von Verschlüsselungstrojanern empfiehlt es sich natürlich, genau abzuwägen, ob jeder NAS-User wirklich Schreibrechte braucht – beispielsweise wenn dieser nur abgespeicherte Medien abspielen soll.)
5. Notiere dir die **IP-Adresse deiner FritzBox**. Du findest sie ebenfalls im Admin-Panel unter „Heimnetz“ → „Heimnetzübersicht“ → „Netzwerkverbindungen“. Standardmäßig lautet sie `192.168.178.1`.

[![Aktiviere die NAS-Funktion und definiere, welche USB-Geräte freigegeben werden sollen. Aktiviere außerdem den SMB-Zugriff.](https://www.andwil.de/images/d/9/3/2/c/d932cf764620a03cf16089544beca583f0d904a5-fritznas.png)](https://www.andwil.de/user/pages/01.weblog/2020-01-05-linux-fritznas-mounten-cifs/fritznas.png) [![Lege einen neuen Fritzbox-Benutzer an&nbsp;…](https://www.andwil.de/images/5/a/3/b/c/5a3bc215e5b3df2357d6afc6768d25db3573ca49-fritznas-user1.png)](https://www.andwil.de/user/pages/01.weblog/2020-01-05-linux-fritznas-mounten-cifs/fritznas-user1.png) [![… und definiere ein sicheres Passwort. Merke dir den Benutzernamen und aktivieren den Zugriff auf NAS-Inhalte. Vergiss nicht, auch die Zugriffsrechte auf den richtigen USB-Speicher zu gewähren.](https://www.andwil.de/images/8/b/1/b/9/8b1b99d3c8e6b0653c228a941eabd5d19095a6b6-fritznas-user2.png)](https://www.andwil.de/user/pages/01.weblog/2020-01-05-linux-fritznas-mounten-cifs/fritznas-user2.png) [![Falls nicht bekannt, ermittle die IP deiner Fritzbox und merke sie dir für die folgende Konfigration.](https://www.andwil.de/images/b/b/8/e/5/bb8e5892f0ff691e8e9d1689d99ead4e16ccb454-fritzbox-ip.png)](https://www.andwil.de/user/pages/01.weblog/2020-01-05-linux-fritznas-mounten-cifs/fritzbox-ip.png)

Glückwunsch, dein FritzNAS ist fertig konfigutiert – das ist die halbe Miete! Im nächsten Schritt wird dein Linux-Rechner so konfiguriert, dass er auf den FritzNAS zugreifen kann.

# 2  FritzNAS unter Linux einbinden[](https://www.andwil.de/weblog/linux-fritznas-mounten-cifs#2-fritznas-unter-linux-ei)

## 2.1  SAMBA und CIFS installieren[](https://www.andwil.de/weblog/linux-fritznas-mounten-cifs#2-1-samba-und-cifs-instal)

Weiter geht's mit dem Client, also deinem Linux-Computer:

6. Es müssen **SAMBA** und **CIFS** über die Paketverwaltung installiert werden. Ersteres ermöglicht überhaupt erst den Zugriff auf den FritzNAS, Zweiteres wird zum Einbinden aus dem Terminal heraus benötigt. Unter Debian und Ubuntu sieht die Installation beispielsweise so aus:
    
    ```shell
    $ sudo apt install samba cifs-utils
    ```
    

**Spätestens nach einem Neustart deines Rechners** sollte der FritzNAS bereits als Netzwerkgerät in deinem Dateimanager sichtbar sein – wenn dir das genügt, kannst du jetzt das Lesen dieser Anleitung einstellen.

![](https://www.andwil.de/user/pages/01.weblog/2020-01-05-linux-fritznas-mounten-cifs/fritznas-dolphin.png)

Mit KDE-Dateimanager Dolphin auf den FritzNAS zugreifen: fühlt sich fast wie das eigene Heimverzeichnis an.

## 2.2  FritzNAS vom Terminal aus einbinden[](https://www.andwil.de/weblog/linux-fritznas-mounten-cifs#2-2-fritznas-vom-terminal)

Möchtest du den FritzNAS nicht nur aus dem grafischen Desktop heraus nutzen, sondern **auch vom Terminal aus**, wird es hier noch mal interessant:

7. Lege im Home-Verzeichnis eine Datei mit dem Namen `.smbcredentials` an, z.B. mit dem Befehl `nano ~/.smbcredentials`. Sie enthält den FritzNAS-Benutzernamen und das zugehörige Passwort und ist folgendermaßen aufgebaut:
    
    ```
    username=FritzNASBenutzername
    password=UltrageheimesPasswort
    ```
    
    Das hat den Charme, dass man beim Einbinden sein Passwort nicht jedes Mal im Klartext in die Konsole tippen muss.
    
8. Lege ein Verzeichnis an, in dem der NAS-Speicher später auftauchen soll, z.B. mit dem Befehl `mkdir ~/fritzNAS` oder über deinen grafischen Datei-Manager.
    

### 2.2.1  Temporär einbinden[](https://www.andwil.de/weblog/linux-fritznas-mounten-cifs#2-2-1-temporaer-einbinden)

Soll der FritzNAS nur einmalig eingebunden werden oder die Konfiguration getestet werden, kann dieser Befehl genutzt werden:

```shell
$ sudo mount -t cifs -o credentials=$HOME/.smbcredentials,vers=3.0,noserverino //192.168.178.1/FRITZ.NAS/ /mnt/
```

Hier kommen deine Konfigurationen von oben zum Tragen! Verwende die in [Schritt 5](https://www.andwil.de/weblog/linux-fritznas-mounten-cifs#schritt-ip-adresse) ermittelte **IP-Adresse** sowie die **FritzNAS-Bezeichnung** aus [Schritt 3](https://www.andwil.de/weblog/linux-fritznas-mounten-cifs#schritt-nas-name). Wenn du die Standard-Werte beibehalten hast, kannst du obigen Befehl 1:1 ins Terminal kopieren.

Alte FritzBoxen (Fritz!OS-Versionen bis einschließlich 6.x) verwenden eine veraltete Samba-Protokollversion. Das NAS kann trotzdem noch eingebunden werden, schreibe hierfür statt `vers=3.0` einfach `vers=1.0`.

Die Dateien auf dem NAS können dann unter `/mnt/` eingesehen werden. Alternativ kann natürlich auch ein anderes Verzeichnis angegeben werden.

### 2.2.2  Permanent einbinden[](https://www.andwil.de/weblog/linux-fritznas-mounten-cifs#2-2-2-permanent-einbinden)

Soll der FritzNAS dagegen automatisch bei jedem Systemstart eingebunden werden, muss die Datei `/etc/fstab` mit Root-Rechten bearbeitet werden. Beachte, dass dein Rechner in diesem Fall **per Ethernet-Kabel am Netzwerk** hängen sollte, damit beim Boot-Vorgang bereits eine Netzwerkverbindung besteht.

Wenn du mal per WiFi online bist, auch nicht schlimm: Der Rechner fährt trotzdem normal hoch. Sobald du mit dem WLAN verbunden bist, starte den Befehl `sudo mount -a` manuell, um den FritzNAS einzubinden.

Lege zunächst ein **Backup der `fstab`** an:

```shell
$ sudo cp /etc/fstab{,~NAS.bak}
```

Jetzt führe den Befehl

```shell
$ sudoedit /etc/fstab
```

aus und ergänze folgende Zeile (beachte auch den darauffolgenden Kasten!):

```
# <file system> <mount point>   <type>  <options>       <dump>  <pass>
(…)
//192.168.178.1/FRITZ.NAS/ /home/{DeinBenutzername}/fritzNAS cifs credentials=/home/{DeinBenutzername}/.smbcredentials,vers=3.0,noserverino,uid=1000,gid=1000,x-systemd.automount,x-systemd.requires=network-online.target 0 0
```

Auch hier müssen wieder folgende Parameter bei Bedarf angepasst werden:

- die **IP-Adresse der FritzBox** (siehe [Schritt 5](https://www.andwil.de/weblog/linux-fritznas-mounten-cifs#schritt-ip-adresse), default: `192.168.178.1`)
- der **Name des NAS** ([Schritt 3](https://www.andwil.de/weblog/linux-fritznas-mounten-cifs#schritt-nas-name), default: `FRITZ.NAS`)
- der **_absolute_ Pfad zum Zielverzeichnis**, der in [Schritt 8](https://www.andwil.de/weblog/linux-fritznas-mounten-cifs#nas-pfad) angelegt wurde –Abkürzungen wie `~/` oder `$HOME` funktionieren hier nicht
- der **_absolute_ Pfad zur `.smbcredentials`**
- Wenn auf deinem Rechner mehrere Benutzerkonten angelegt sind, müssen die Werte für **uid** und **gid** ggf. angepasst werden, damit du Lese- und Schreibzugriff auf das Verzeichnis bekommst. Du erfährst diese Werte mit dem Befehl `id`, so in etwa sieht die Ausgabe dann aus:
    
    ```
    andwil@denkbrett:~$ id andwil [↵]
    uid=1000(andwil) gid=1000(andwil) Gruppen=1000(andwil),4(adm),…
    ```
    

**Teste die Konfiguration** mit dem Befehl `sudo mount -a`. Das ist bequemer, als bei jeder Anpassung der `fstab` den Rechner neuzustarten. Wenn es unlösbare Probleme geben sollte, spiele einfach das Backup wieder ein – so kannst du nichts kaputt spielen.

# 3  Troubleshooting/Stolpersteine[](https://www.andwil.de/weblog/linux-fritznas-mounten-cifs#3-troubleshooting-stolper)

- Wenn Du eine **alte FritzBox** hast (**Fritz!OS-Version niedriger als 7.x**), muss eine ältere Samba-Protokollversion verwendet werden. Diese Information wird dem `mount`-Befehl mit dem Zusatz `vers=1.0` mitgegeben:
    
    ```shell
    sudo mount -t cifs -o credentials=$HOME/.smbcredentials,vers=1.0 //192.168.178.1/FRITZ.NAS/ /mnt/
    ```
    
    Analog dazu, sollte der `fstab`-Eintrag folgendermaßen aussehen:
    
    ```
    //192.168.178.1/FRITZ.NAS/ /home/{DeinBenutzername}/fritzNAS cifs credentials=/home/{DeinBenutzername}/.smbcredentials,vers=1.0,uid=1000,gid=1000,x-systemd.automount,x-systemd.requires=network-online.target 0 0
    ```
    
- Wenn dein Rechner per WiFi mit der FritzBox verbunden ist, kann es zu Problemen beim automatischen Einbinden des FritzNAS kommen, weil die Netzwerkverbindung in den meisten Fällen erst nach dem Laden der grafischen Benutzeroberfläche hergestellt wird. Einen Blick wert ist ein [Workaround](https://znil.net/index.php/Raspberry_Pi_Windows_Freigabe_beim_Booten_automatisch_mounten#Netz_und_doppelter_Boden), der dafür sorgt, dass zwei Minuten nach dem Start des Rechners alle `fstab`-Einträge (neu) eingebunden werden:
    1. Bearbeite bzw. lege eine Crontab-Datei an. Führe dazu nacheinander diese Befehle aus:
        
        ```shell
        $ sudo -s
        # export EDITOR=nano
        # crontab -e
        ```
        
    2. Es öffnet sich der Edior „Nano“, vielleicht findest du schon ein paar Crontab-Einträge vor. Ergänze am Ende die folgende Zeile:
        
        ```
        @reboot sleep 120 && mount -a
        ```
        
    3. Beende Nano (Strg+x) und bestätige deine Änerung mit j (bzw. y bei englischer Systemsprache).
    4. verlasse den `sudo -s`-Modus mit
        
        ```shell
        # exit
        ```
        
        Der Cron-Eintrag ist ab dem nächsten Neustart wirksam.
        
- Die beliebte Umleitung [http://fritz.box](http://fritz.box/) anstelle der IP funktioniert mit den hier gezeigten Befehlen übrigens _nicht_!