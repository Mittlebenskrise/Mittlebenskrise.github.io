## Ionos
Go to Ionos and create a subdomain for NGinx, e.g. `home.mittlebenskrise.de` by selecting the domain (`mittlebenskrise.de`) and adding the subdomain (`home`).

### New (see https://www.ionos.de/hilfe/domains/ip-adresse-konfigurieren/dynamisches-dns-ddns-einrichten-bei-company-name/):
* Activate Dynamic DNS support in Ionos.
* Create credentials on https://developer.hosting.ionos.de/keys
* Fill in the form and retrieve prefix and encrypted key.
  E.g.:

```
Präfix: 72eb5ccf18c64bebaf6d1d152b5669c2
Verschlüsselung: 8x3xpbeJhiWbfR_EA62Zhs_cQerEMx9xtKl2NEKnt0ducRfSFiXkAaYRBxdUnznwteqrk9EAgIgbqgKeXZ7msg
```

* Open https://developer.hosting.ionos.de/docs/dns
* Click "Authorize"
* Login with the key in the form prefix.key, e.g. `72eb5ccf18c64bebaf6d1d152b5669c2.8x3xpbeJhiWbfR_EA62Zhs_cQerEMx9xtKl2NEKnt0ducRfSFiXkAaYRBxdUnznwteqrk9EAgIgbqgKeXZ7msg`
* Select DynamicDNS – POST and click "Try it out"
* Modify the configuration by adding the subdomain and all sub-subdomains to the `domains` entry, e.g.
```
{
  "domains": [
    "home.mittlebenskrise.de",
    "*.home.mittlebenskrise.de",
    "auth.home.mittlebenskrise.de",
    "mac.home.mittlebenskrise.de",
    "backup.home.mittlebenskrise.de",
    "cloud-mobile.home.mittlebenskrise.de",
    "cloud.home.mittlebenskrise.de",
    "games-admin.home.mittlebenskrise.de",
    "games.home.mittlebenskrise.de",
    "portainer-mobile.home.mittlebenskrise.de",
    "containers.home.mittlebenskrise.de",
    "brother.home.mittlebenskrise.de",
    "diagram.home.mittlebenskrise.de",
    "mindmap.home.mittlebenskrise.de",
    "scan.home.mittlebenskrise.de",
    "update.home.mittlebenskrise.de",
    "fritz.home.mittlebenskrise.de",
    "webmin.home.mittlebenskrise.de",
    "adguard.home.mittlebenskrise.de",
    "zabbix.home.mittlebenskrise.de",
    "obsidian.home.mittlebenskrise.de",
    "webdav.home.mittlebenskrise.de",
    "homeassistant-mobile.home.mittlebenskrise.de",
    "homeassistant.home.mittlebenskrise.de",
    "kanban-mobile.home.mittlebenskrise.de",
    "kanban.home.mittlebenskrise.de",
    "www.home.mittlebenskrise.de",
    "paperless-mobile.home.mittlebenskrise.de",
    "paperless.home.mittlebenskrise.de",
    "photoprism-mobile.home.mittlebenskrise.de",
    "photos.home.mittlebenskrise.de",
    "emby-mobile.home.mittlebenskrise.de",
    "media.home.mittlebenskrise.de",
    "ssh.home.mittlebenskrise.de",
    "unifi.home.mittlebenskrise.de",
    "linkding.home.mittlebenskrise.de",
    "it-tools.home.mittlebenskrise.de",
    "pdf-tools.home.mittlebenskrise.de",
    "phpmyadmin.home.mittlebenskrise.de",
    "books-mobile.home.mittlebenskrise.de",
    "books.home.mittlebenskrise.de",
    "vaultwarden.home.mittlebenskrise.de",
    "fully.home.mittlebenskrise.de",
    "dozzle.home.mittlebenskrise.de",
    "arbeitszeit.home.mittlebenskrise.de",
    "upload.home.mittlebenskrise.de",
    "recipes-mobile.home.mittlebenskrise.de",
    "recipes.home.mittlebenskrise.de",
    "magazines.home.mittlebenskrise.de",
    "comics.home.mittlebenskrise.de",
    "pipeline.home.mittlebenskrise.de",
    "test.home.mittlebenskrise.de"
  ],
  "description": "home.mittlebenskrise.de DynamicDns"
}
```

* Click `Execute` to activate the configuration.
* From the result, collect the `updateUrl` and enter it in the Fritzbox (see below)

### Old
Now for each service, create sub-subdomains in a similar way. However, the domain is still the same (`mittlebenskrise.de`), but the subdomain changes to `sub-subdomain-subdomain` (e.g. `emby.home`).

For each of the (sub-) subdomains, select to modify the DNS entries. First, remove all existing entries (e.g. AAAA, A, MX etc.). Then add a new record of type "NS" and enter `ns1.dynv6.com` as server. Repeat for `ns2.dynv6.com` and `ns3.dynv6.com`.

The subdomain and each sub-subdomain used with NGinx should now have three NS entries pointing to the dnyv6 servers.

## dynv6
In dynv6 create the subdomain under "My Domains" (e.g. `home.mittlebenskrise.de`). The domain type is "Single zone". 
Under zones the subdomain should now be listed. Select the zone and get to the "Records" tab. Now, for each of the sub-subdomains add an A record with the sub-subdomain entered in the "Name" field (e.g. `portainer` for `portainer.home.mittlebenskrise.de`).
Afterwards, the list under "Resource Records" should contain one AAAA entry for the subdomain and one A entry for each of the sub-subdomains.

Alternatively (and much easier), you can just add one wildcard A record, e.g. `*.home.mittlebenskrise.de`. In that case, you will need to route all unused sub-subdomains in Nginx to an error page by adding the following lines to the end of `default.conf`:

```
server {
    listen 80;
    listen [::]:80;
    listen 443 ssl;
    listen [::]:443 ssl;
    ssl_certificate     /var/www/html/ssl/home.mittlebenskrise.de/server.crt;
    ssl_certificate_key /var/www/html/ssl/home.mittlebenskrise.de/server.key;
    server_name *.home.mittlebenskrise.de;

    return 404;
}
```

## Port Forwarding with Fritzbox

In the Fritzbox under "Heimnetz – Netzwerk – Netzwerkverbindungen" find the server. Under "IPV6 Interface-ID" you can find the server's host part of the IPV6 address (e.g. `::a:b:c:d`).
Under "Internet – Freigaben – DynDNS" change the default DynDNS update URL as follows (replacing `<ip6addr>` by the host part of the server):

### New
Get the `updateUrl` from Ionos as described above and add the following in the Fritzbox:

```
updateUrl&ipv4=<ipaddr>&ipv6=<ip6lanprefix>:a:b:c:d

e.g.

updateUrl&ipv4=<ipaddr>&ipv6=<ip6lanprefix>:6a1d:efff:fe38:4b5b
```

Afterwards, in the domain records of Ionos you should see an A record with the IPV4 address and an AAAA record with the IPV6 prefix (can be found under "Internet – Online-Monitor" in the Fritzbox) followed by the host part for the subdomain and each sub-subdomain.

### Old
```
http://dynv6.com/api/update?hostname=<domain>&token=<username>&ipv4=<ipaddr> http://dynv6.com/api/update?hostname=<domain>&token=<username>&ipv6=::a:b:c:d&ipv6prefix=<ip6lanprefix>
```

Afterwards, in https://dynv6.com/zones on the "records" tab you should see an AAAA record with the IPV6 prefix (can be found under "Internet – Online-Monitor" in the Fritzbox) followed by the host part.

In order for the own domain to be forwarded to the server properly, you will also have to adjust the DNS rebind protection. In the Fritzbox go to "Heimnetz – Netzwerk – Netzwerkeinstellungen" and enter your domain under "DNS-Rebind-Schutz". Also, click on "IPv6-Einstellungen" below and modify the settings as follows (if not set already):
- Check "Router Advertisement im LAN aktiv" and select the first option ("Unique Local Addresses (ULA) zuweisen, solange keine IPv6-Internetverbindung besteht (empfohlen)").
- Uncheck "Auch IPv6-Präfixe zulassen, die andere IPv6-Router im Heimnetz bekanntgeben".
- Check "Diese FRITZ!Box stellt den Standard-Internetzugang zur Verfügung".
- Check "DNSv6-Server auch über Router Advertisement bekanntgeben (RFC 5006)".
- If you use Adguard Home, enter the Adguard's server address under "Lokaler DNSv6-Server" (you will also have to enter the server's IPV4 address in the "IPv4-Einstellungen" under "Lokaler DNS Server").
- Under "DHCPv6-Server im Heimnetz" select the first option ("DHCPv6-Server in der FRITZ!Box für das Heimnetz aktivieren") and choose "DNS-Server und IPv6-Präfix (IA_PD) zuweisen".

Once all this is done, you can add your port forwarding under "Internet – Freigaben – Portfreigaben"