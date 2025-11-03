Changed for Ionos now:

```
Präfix: 72eb5ccf18c64bebaf6d1d152b5669c2
Verschlüsselung: 8x3xpbeJhiWbfR_EA62Zhs_cQerEMx9xtKl2NEKnt0ducRfSFiXkAaYRBxdUnznwteqrk9EAgIgbqgKeXZ7msg
```

Open https://developer.hosting.ionos.de/docs/dns

Click "Authorize"

Login with: `72eb5ccf18c64bebaf6d1d152b5669c2.8x3xpbeJhiWbfR_EA62Zhs_cQerEMx9xtKl2NEKnt0ducRfSFiXkAaYRBxdUnznwteqrk9EAgIgbqgKeXZ7msg`

DynamicDNS - POST

## Old Version

from https://github.com/qdm12/ddns-updater?tab=readme-ov-file

1. Create a directory of your choice, say _~/data_ with a file named **config.json** inside:
 
```shell
mkdir data
touch data/config.json
# Owned by user ID of Docker container (1000)
chown -R 1000 data
# all access (for creating json database file data/updates.json)
chmod 700 data
# read access only
chmod 400 data/config.json
```

2. Write a JSON configuration in _~/data/config.json_, for example (see [[#Dynv6 Configuration]] for Dynv6):

```json
{
	"settings": [
		{
			"provider": "namecheap",
			"domain": "example.com",
			"host": "@",
			"password": "e5322165c1d74692bfa6d807100c0310"
		}
	]
}
```

You can find more information in the [configuration section](https://github.com/qdm12/ddns-updater?tab=readme-ov-file#configuration) to customize it.

3. Run the container with

```shell
docker run -d -p 8001:8000/tcp -v "$(pwd)"/data:/updater/data qmcgaw/ddns-updater
```

  ⚠️ If you use IPv6, you might need to set `-e IPV6_PREFIX=/64` (`/64` is your prefix, depending on your ISP)
  
```shell
docker run -d -p 8001:8000/tcp -e IPV6_PREFIX=/64 -v "$(pwd)"/data:/updater/data qmcgaw/ddns-updater
```

4. (Optional) You can also set your JSON configuration as a single environment variable line (i.e. `{"settings": [{"provider": "namecheap", ...}]}`), which takes precedence over config.json. Note however that if you don't bind mount the `/updater/data` directory, there won't be a persistent database file `/updater/updates.json` but it will still work.

### Dynv6 Configuration

```json
{
  "settings": [
    {
      "provider": "dynv6",
      "domain": "mittlebenskrise.dynv6.net",
      "host": "@",
      "token": "eRy19QWeovFxdFi9pKrVRiZtDxpA1R",
      "ip_version": "ipv4",
      "provider_ip": true
    },
    {
      "provider": "dynv6",
      "domain": "mittlebenskrise.dynv6.net",
      "host": "@",
      "token": "eRy19QWeovFxdFi9pKrVRiZtDxpA1R",
      "ip_version": "ipv6",
      "provider_ip": true
    }
  ]
}
```

### Compulsory parameters

- `"domain"`
- `"host"` is your host and can be a subdomain or `"@"`
- `"token"` that you can obtain [here](https://dynv6.com/keys#token)

### Optional parameters

- `"ip_version"` can be `ipv4` (A records) or `ipv6` (AAAA records), defaults to `ipv4 or ipv6`
- `"provider_ip"` can be set to `true` to let your DNS provider determine your IPv4 address (and/or IPv6 address) automatically when you send an update request, without sending the new IP address detected by the program in the request.