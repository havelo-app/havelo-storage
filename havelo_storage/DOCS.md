# Havelo Storage

Keeps Havelo's favorites, views, cards and maps in sync across your devices,
and lets each person in your home keep their own. Your data stays on this Home
Assistant and is included in its backups.

## Connect your first device

1. Start the app.
2. In Havelo, open **Settings → Connections → Storage** and choose **Havelo
   Storage on your Home Assistant**. Havelo sets it up with the code the app
   posts to Home Assistant's notifications.

If Havelo can't read the notification, copy the code from it, choose **First
device? Use a setup invitation** and paste it. A code works once, for 10
minutes.

The app's **Log** also shows a setup QR code for each address in its
configuration (`homeassistant.local` by default). If your devices reach Home
Assistant another way, such as by IP address or a tailnet name, add it there
and restart the app for a new code.

After the first device, each new device shows an approval code that a
connected device approves.

## Connecting securely

Devices connect over HTTP on your local network. For HTTPS, put the port
behind `tailscale serve` or a reverse proxy with a trusted certificate, and use
that address.

## Backups

Home Assistant backups include this app's data. Restoring one brings the data
back as it was then; each device then rejoins and chooses how to combine any
newer changes it holds.

Uninstalling the app deletes its data. Take a backup first.
