# Setting up Havelo storage

Havelo storage keeps your favorites, settings, cards and maps on a server you
run, and syncs them between your phones, tablets, TVs and computers. It runs as
one container with Docker Compose on an always-on Linux machine (amd64 or
ARM64). On amd64 it used about 110 MiB of memory with ten devices and 100 MiB
of data; ARM64 hardware hasn't been measured yet.

**Running Home Assistant OS or Supervised?** Install the Havelo Storage app
instead (see the README): Home Assistant installs, starts and backs it up, and
Havelo finds it on your Home Assistant and gets its setup code from Home
Assistant's notifications. Skip this guide.

## 1. Choose how devices reach it

Devices connect the way Havelo reaches Home Assistant. Pick one and use its
folder in [`examples/`](examples):

| Devices connect over                           | Example             | Address devices use                  |
| ---------------------------------------------- | ------------------- | ------------------------------------ |
| Your local network, plain HTTP                 | `local-http`        | `192.168.1.20` (this machine's IP)   |
| Your local network, HTTPS with a domain        | `local-https-caddy` | `https://havelo.example.com`         |
| Your tailnet, plain HTTP (Tailscale-encrypted) | `tailnet-http`      | `havelo.your-tailnet.ts.net`         |
| Your tailnet, HTTPS                            | `tailnet-https`     | `https://havelo.your-tailnet.ts.net` |

Plain HTTP on a local network isn't encrypted; use it on a network you trust,
and reserve the machine's IP on your router so it doesn't change. Tailnet
traffic is always encrypted, and devices need the Tailscale app. HTTPS needs a
certificate devices already trust: the Caddy and Tailscale examples obtain one
automatically. Self-signed certificates don't work.

## 2. Start it

Copy the example's folder to the machine, then from inside it:

```sh
cd examples/local-http
docker compose up -d
```

The HTTPS and tailnet examples need one setting first, shown at the top of
their `compose.yaml`: `HAVELO_DOMAIN` for Caddy, or a `TS_AUTHKEY` from the
Tailscale admin console.

## 3. Check it

```sh
docker compose exec data-service node src/admin.js doctor 192.168.1.20
```

Give the address devices will use. `doctor` checks that the service is running,
the database, the backup volume (with a rehearsed restore of the newest backup)
and that the address answers as this service. Fix anything marked `✗`; `!` is
advice. With `tailnet-https`, the tailnet name may not resolve inside the
container; if only that check fails, confirm by connecting a device.

## 4. Add your first device

```sh
docker compose exec data-service node src/admin.js bootstrap 192.168.1.20
```

This prints a QR code and setup link, valid for 10 minutes and once. In Havelo,
open Settings → Connections → Storage (or the Storage step of setup) and scan
it, or paste the link.

## 5. Add more devices and people

On a new device, open Settings → Storage and enter the server's address. It
shows an approval QR code. On a device that's already connected, choose
**Approve a device** and scan it. Owners can approve it for **Another
person**, who gets their own private data.

## 6. Keep backups somewhere else

The service makes a verified backup every day into the `havelo-backups` volume,
keeping 7 daily and 4 weekly copies. Owners see the last backup's age in
Settings → Storage, with a warning when backups stop. Those copies are on the
same disk, so copy them to another machine regularly, for example:

```sh
docker run --rm -v havelo-storage_havelo-backups:/backups:ro -v "$PWD":/out \
  busybox tar -czf /out/havelo-backups.tar.gz -C /backups .
```

Encrypt the copy if it leaves your home; it contains your data.

## 7. Restore a backup

```sh
docker compose stop data-service
docker compose run --rm --no-deps data-service node src/admin.js restore /backups/havelo-2026-11-18.sqlite
docker compose start data-service
```

The restore refuses while the service runs, checks the backup and keeps the
previous database beside the restored one until you remove it. Each device then
shows **Server restored** in Settings → Storage and asks how to rejoin: review
its newer changes, or use the restored data and keep a copy.

## 8. Upgrade

Change the image version in `compose.yaml` to the new release, then:

```sh
docker compose stop data-service
docker compose pull data-service
docker compose run --rm --no-deps data-service node src/admin.js migrate /data/pre-upgrade-1.sqlite
docker compose up -d
```

`migrate` first writes a verified backup to the file you name (use a new name
each time) and changes nothing if any step fails. Keep that backup with the
previous image version until the new one has run well.
