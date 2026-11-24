# Scaling Breakout (Scale Up) for Price Bailey events

Two-screen browser game. The big screen shows the game and a QR code. The player scans it and steers from their phone. The game can also be played on the big screen itself. Both routes show a details form first, and details go straight to HubSpot.

## Structure

```
index.html              big screen (game, QR code, big-screen form)
controller/index.html   phone controller (form, slide pad, end screen)
lib/link.js             connection between the two screens (MQTT + BroadcastChannel)
lib/hubspot.js          HubSpot settings and submission, shared by both screens
lib/mqtt.min.js         bundled messaging library (mqtt.js 5.16.1)
lib/qrcode.js           bundled QR code library (qrcode-generator 1.5.2)
assets/                 logo
```

## Settings to check before an event

- `lib/hubspot.js`: `EVENT_NAME` (must be set, it is recorded against every lead), `GAME_NAME`, `PRIVACY_URL`, consent wording.
- `index.html` settings block (top of the script): `FORM_IDLE_CLOSE_MS`, `END_AUTO_RESET_MS`, `CONTROLLER_TIMEOUT_MS`, `CONTROLLER_LOST_RESET_MS`, `SHOW_INGAME_CTA`.
- `controller/index.html` settings block: pad inset, send rate, button speed, resend timings.
- Brand colours are CSS variables at the top of both pages.

## How the connection works

The big screen creates a four-character room code and shows a QR code to `controller/?room=CODE`. Both pages relay messages over `wss://broker.emqx.io:8084/mqtt` and `wss://broker.hivemq.com:8884/mqtt` together, de-duplicated by message id. Topics are `pricebailey/scaling-breakout/ROOM/screen` and `.../controller`. BroadcastChannel is also used, so two tabs in one browser work as a desk test.

- Controller to screen: `hb`, `mv` (x 0 to 1), `start`, `newgame`.
- Screen to controller: `hb`, `st` (state, score, level, phase name, message), `bye`.
- Heartbeat every 2 seconds, other side treated as gone after 6 seconds.
- Only the first controller to connect is accepted. Others see "Game in use".
- Add `?broker=ws://host:port` to either address to test against a local broker.

Both devices need internet. The public brokers have no uptime guarantee. For important events, move to a private broker or Firebase.

## Testing

1. Open `index.html` in one tab, then the controller address shown under the QR code in a second tab (desk test).
2. Add `&debug` to the controller address to show HubSpot's replies on the phone.
3. Play one full game and check the test contact in HubSpot.

## Data protection

Personal details are held in memory only. They are never written to the repository or browser storage, never shown on the big screen, and are cleared at the start of each game and on New game.

## Blocking before a stand

- Event name set.
- Compliance approve the consent wording and privacy notice link, and `PRIVACY_URL` is filled in.
- A test contact appears in HubSpot with every field filled, including `game_form_consent` when ticked.
- `price-baileymarketing-star.github.io` stays registered as a trusted domain in HubSpot. Any other host must be added too.
