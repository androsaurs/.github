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
household policies and (since0.7.0) bounded direct servo control, never owner
replacement or raw microphone access. Keys are
hashed in stand NVS; Android uses Keystore encryption. Hub credentials are private
server data and must not appear in API responses/logs. Hub enrollment requires
an authenticated phone, an available pinned stand, and an unclaimed 1:1 mapping.
Only local unicast IPv4 endpoints on the fixed port are accepted by the hub.
Revoking the hub key does not break the phone/stand pairing.

Standard firmware servo commands accept the owner or delegated hub key:
`POST /v1/servos/center`, `/v1/servos`, `/v1/servos/stop`. The uncalibrated envelope
is1450..1550us and at most5seconds per explicit command, followed by automatic PWM
off. Boot never starts manual PWM. The phone is not in this command path.
See [direct servo control](../../androsaur-stand/firmware/docs/servos.md).
OTA upload and confirmation remain owner-only.

The phone keeps local defaults, people and capabilities without the hub. Lost
stand connectivity falls back to phone hardware. Unknown/stale stand state is
displayed separately from phone reachability, never reported as online.
Capabilities are advertised only when implemented: the bare board offers status
and policy storage, not functioning motors/audio/sensors. Firmware 0.6.4 implements
owner-only pinned HTTPS OTA into an inactive app slot with image/SHA-256 validation,
explicit owner confirmation of the running ELF hash, and rollback after 120 seconds
without confirmation. The delegated hub key cannot upload or confirm firmware.
Protocol and builder workflow: [stand OTA](../../androsaur-stand/firmware/docs/ota.md).
Audio transport and alarm override of night silence remain separate work;
unsupported actions fail.

## Visual tracking (owner decision, 2026-11-12)

The phone detects and selects a face, then sends signed normalised error:
`x=2*cx-1` (positive right), `y=1-2*cy` (positive up), centre `(0,0)`.
No angles, PWM values, mechanical geometry or actual servo positions are needed on Android.
ESP owns incremental corrections, deadband, slew rate and calibrated pulse limits.
The only feedback is the next camera observation, not a position sensor.

Camera results have a boot-qualified frame ID and monotonic analysis-start time.
The app polls at its existing roughly 4 Hz rate and only sends new, fresh results;
it never repeats a frame to manufacture a higher observation rate. A single latest
observation replaces any waiting one. TLS is established before streaming;
age is checked again after connection establishment before writing a request.
Owner-only `POST /v1/tracking` uses the same pinned HTTPS connection pool.
Session/sequence checks reject duplicates and older observations. ESP ceases changing
the commanded pulse after 500 ms without fresh input or on an explicit lost-target
event, retaining holding PWM. No blind search sweep is implemented.

GPIO, axis directions, neutral and mechanical pulse limits are ESP configuration.
PWM defaults OFF and limits default to an uncalibrated zero-width range; enabling
the flag alone cannot cause movement. Confirm power, wiring and mechanical limits
before enabling it. An ESP brownout was observed; replacing the cable restored
network connectivity. Firmware `0.3.2-tracking-links` was flashed via native USB on
2026-11-13 with a PHY Wi-Fi TX ceiling of 10 dBm. Motion still requires confirmed
calibration. TLS allocations use PSRAM with the standard 16 KB incoming record limit;
five concurrent HTTPS sessions are allowed. Authenticated stream ACKs also confirm
phone-to-stand liveness, independently of periodic metadata polls. The indicator allows
10 seconds for recovery after the last ACK; observation freshness remains 500 ms.
Response wait is 2 seconds, with a conflated latest-only queue and a 500 ms reconnect
backoff. Concurrent TLS handshakes can still delay traffic, especially on weak Wi-Fi.

The optional hub receives bounded observation batches from the phone independently
of stand connectivity. One diagnostic graph overlays X (blue) and Y (green), with
zero reference and gaps for lost targets/stale input. This is camera error, not
measured servo position. No images, face embeddings or recognised identities are
included. Hub diagnostics never participate in the servo control loop.

Since firmware `0.8.0-tracking-settings`, the hub can tune the stand directly using
owner/hub-authenticated `POST /v1/tracking/settings` and stop/resume it with
`POST /v1/tracking/control`. Settings are stored in stand NVS; the stand keeps
tracking without the hub. The UI uses approximate degrees for per-axis half-ranges,
degrees/second for speed, a correction step per 20 ms, smoothing time, dead zone,
and axis reversal. Angles use a nominal servo mapping, not measured shaft positions.
The fixed 20 ms control loop filters fresh camera error and limits speed/acceleration
independently of camera FPS. Narrowed limits are approached gradually. Observation
authorization, session ordering and the 500 ms freshness limit remain unchanged.


### Phone battery observations and charger policy

The phone also sends owner-authenticated `POST /v1/battery` on the same pinned HTTPS
pool: `{v:1,session,seq,ageMs,percent,charging}`. Percent is 0…100; charging is the
Android CHARGING status (FULL is not charging). Unknown readings are not fabricated.
Reports are sent on change (system state checked each second) and every 15 seconds,
with independent latest-only queues for battery and camera, one serial sender, no
camera or hub dependency. ESP rejects invalid, duplicate/retired or >60-second-old
reports. The hub's delegated credential cannot write phone observations.

ESP owns hysteresis, initially 40% resume / 80% pause, configurable in firmware.
Between thresholds it retains the last decision, including a phone session change.
At boot and after >10 minutes without fresh battery data, power is allowed. No hub
is needed. Actual Android charging and ESP chargeAllowed are separate status values;
outputEnabled describes whether a confirmed physical switch is configured. GPIO6
is proposed only; physical output defaults OFF with GPIO -1 until wiring/polarity
are confirmed. No swelling-prevention or thermal-safety guarantee is implied.


The phone additionally includes `battery: {percent,charging,ageMs}` in its normal
hub sync, authenticated as that device. This works without an attached stand and
never gates local ESP delivery. The hub stores the observation time, keeps missing
reports unknown/aged and marks readings older than 90 seconds stale. An older
observation cannot replace a newer one. The stand's charging decision arrives via
its separately pinned `/v1/status.battery`, with outputEnabled shown explicitly.
The device card distinguishes phone charging from the stand's power decision;
owner-only no-store battery fragments refresh every 15 seconds in a visible tab.
