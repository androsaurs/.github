# Agent, stand and hub

Owner decisions, 2026-11-12: Android is the minimum autonomous agent. Its optional
stand is a 1:1 hardware extension, with its own permanent ID, IP, firmware and
capabilities. The hub presents one agent card with nested phone/stand states.
A separate stand database record preserves hardware identity across phone changes.

Phone and stand communicate directly without a hub. The optional hub also reaches
the stand directly for household policy/status even when the phone is unavailable.
The stand protocol remains independent of Android; additional future sensors and
clients use explicitly granted capabilities. It never accepts a second owner.

## Trust and initial setup

Initial setup uses Bluetooth LE. Espressif Wi-Fi Provisioning Security 2 (patch 1)
provides SRP-6a, AES-GCM and a mandatory random proof of possession. Username is
`androsaur`; Security 0/1 and unpatched Security 2 are rejected. An unconfigured
stand advertises ANDRO_XXXXXX automatically, without a setup time limit. Its
128-bit PoP is generated once in NVS and supplied on an exterior QR label together
with the hardware ID, BLE name and TLS certificate fingerprint. It survives normal
reboots and ownership changes. Normal startup never prints it in logs.

The user flow is: power on → Add stand → scan exterior QR → choose a 2.4 GHz
network → enter its password. Android delegates QR scanning to the signature-
protected camera module; the main app never opens a camera. Network names are
scanned by the stand, so a phone on 5 GHz can still choose a compatible network.
The QR label is a physical credential, not a link to an external website.

BLE remains available after pairing: Wi-Fi changes and explicit owner-authorized
release work directly from the phone even without Wi-Fi or a hub. The encrypted
phone store retains the label secret for these operations. Another phone needs
the label and three deliberate power cycles (3–5 seconds each), followed by an
explicit replacement confirmation within two minutes. Power cycles alone do not
erase ownership. BOOT remains only a legacy emergency mechanism inside the board.

Manufacturing has a separate one-time UART `factory-label` export utility,
`androsaur-stand/firmware/tools/setup_label.py`, which generates a private printable
PNG in ignored `labels.local/`. It exports no owner/hub key or Wi-Fi password.

The secured `stand-owner` endpoint accepts `{phoneId,key,ssid,password}` (random 256-bit owner
key) and returns `{ok,id,fingerprint}` (SHA-256 of the device TLS certificate DER).
Wi-Fi configuration is applied as part of this protected exchange. Status reports
the persistent owner `phoneId`, which the hub verifies before accepting a mapping.
Same owner/key retries are idempotent; another owner is rejected. Android saves
the pending binding before provisioning Wi-Fi in the same secured BLE session.
Wi-Fi credentials are never sent to the hub. Discovery uses
`_androsaur-stand._tcp`, port 8443; mDNS names/addresses are untrusted until TLS
pin verification succeeds. No HTTP fallback is allowed on the stand.
Android retains the last HTTPS-verified address across restarts. If that address
fails, it tries mDNS, then reads the current station IPv4 through authenticated
BLE `info`, checking hardware identity, certificate fingerprint and owner ID.
Every candidate still requires pinned HTTPS and owner authentication; discovery
does not depend on the hub or repeated QR scans.
Protected endpoint operations also include `info`, `networks`, and owner-key-only
`release`. Recovery is explicitly requested and requires the physical recovery
window; possession of the QR alone cannot replace an existing owner.

The phone can delegate a different 256-bit hub key over pinned HTTPS. Phone rights
include hardware and hub credential management; hub rights include status and
household policies, never owner replacement or raw microphone access. Keys are
hashed in stand NVS; Android uses Keystore encryption. Hub credentials are private
server data and must not appear in API responses/logs. Hub enrollment requires
an authenticated phone, an available pinned stand, and an unclaimed 1:1 mapping.
Only local unicast IPv4 endpoints on the fixed port are accepted by the hub.
Revoking the hub key does not break the phone/stand pairing.

The phone keeps local defaults, people and capabilities without the hub. Lost
stand connectivity falls back to phone hardware. Unknown/stale stand state is
displayed separately from phone reachability, never reported as online.
Capabilities are advertised only when implemented: the bare board offers status
and policy storage, not functioning motors/audio/sensors. OTA/audio transport and
alarm override of night silence remain separate work; unsupported actions fail.
