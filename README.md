# Two-Phone Fall Timer

Measure how long an object takes to fall between two points, using **two phones as camera "tripwires"**. No server, no app install, no accounts: it is one HTML file that runs in the phone's browser.

Each phone watches a thin band of its camera view. When the object crosses the band, the phone records the moment. The phones are linked directly to each other (WebRTC), they estimate the offset between their clocks, and the app shows the time between the two crossings.

The app only measures time. It does not use `g` or any physics formula to calculate the result.

## Features

- Pairing by **QR code** (or copy/paste text codes as a fallback), with no signalling server
- Works with **any background**: per-pixel background learning, brightness compensation, and compact-blob detection instead of simple motion thresholds
- **Clock synchronisation** between the two phones, with the uncertainty shown next to every result
- Switches the camera to its highest available frame rate after pairing
- Brightness control, optional exposure lock
- Adjustable detection band thickness, sensitivity, and a "sideways image" option for landscape use
- Run history for repeated drops

## Quick start

1. Host `index.html` over **HTTPS** (see below). Phone cameras are blocked on plain `http://` pages.
2. Open the page on both phones. Use the same Wi-Fi if you can.
3. On each phone, choose its role and tap **Start camera**:
   - **Top sensor**: the phone at the release end of the fall
   - **Bottom sensor**: the phone at the landing end
4. Pair:
   - Phone 1: **I create the code**, which shows a QR code
   - Phone 2: **I paste the code**, then **Scan QR**, and point the camera at phone 1's screen
   - Phone 2 now shows a reply QR; on phone 1 tap **Scan reply QR** and scan it
5. Both phones show **Other phone connected** and **Clock synced**.
6. Tap **Arm both phones**, keep the phones and scene still for 2 seconds, then drop the object.
7. The fall time appears on both phones.

If scanning does not work, open "Camera scan not working? Use text codes" and send the codes through any messaging app.

## Hosting on GitHub Pages

1. Put `index.html` in the root of the repository.
2. Go to **Settings → Pages**, select the branch and the root folder, and save.
3. Open the `https://<username>.github.io/<repo>/` address on both phones.

Any other HTTPS static host works too (Netlify Drop, Cloudflare Pages, and so on). The QR libraries are loaded from cdnjs, so the phones need internet access when they first load the page.

## Setting up the phones

- **Fall path:** the object should cross the orange band like a tripwire, meaning *perpendicular* to the band's long direction. With a horizontal band, that means a vertical fall.
- **Stability:** prop each phone on something solid. A moving camera makes the background look like motion.
- **Orientation:** landscape gives a wider view. Turn on the phone's auto-rotate. If the preview shows the fall moving left-to-right, tick **Image looks sideways**.
- **Distance:** keep the cameras close to the drop path so the object looks large and the band covers a short stretch of real distance.
- **Light:** bright, even light gives a short camera exposure and less motion blur on a fast object.
- **Object:** use a small, dense, contrasting object, such as a steel ball or bolt.
- **Same settings on both phones:** use the same band thickness and a similar distance from the path, so any early-trigger bias affects both phones about equally.

## Controls

| Control | What it does |
|---|---|
| Brightness | Camera exposure compensation if the phone supports it, otherwise a software gain |
| Lock exposure | Holds a fixed exposure time for steadier detection (needs good light) |
| Band thickness | Thicker catches fast objects but triggers when the object first enters the band |
| Image looks sideways | Uses a vertical band instead of a horizontal one |
| Smallest object width | How wide (as % of the frame) a change has to be before it counts as the object |
| Re-sync clocks | Repeats the clock measurement between the phones |

While idle, the **live detected width** reading should stay near 0. If it jumps around, raise the *smallest object width* slider or steady the phone.

## Accuracy

Two things limit the precision, and the app shows both next to the result:

- **Camera frame time:** each phone can only notice the object on a frame. At 30 fps that is about 33 ms per phone; 60 fps halves it. Browsers usually cap phone cameras at 30 or 60 fps.
- **Clock sync:** the phones measure their clock offset by exchanging messages. The reported value is half the best round-trip time, typically a few milliseconds over a direct local connection.

To verify your setup:

1. **Zero test:** make both phones see the same event at the same moment (for example, one object crossing both sight lines). The result should be close to 0 ms.
2. **Compare with a calculation outside the app.** For an object released a height `h` above the top sensor, with the sensors `d` apart, ideal free-fall (no air resistance) gives about `√(2(h+d)/g) − √(2h/g)`, with `g ≈ 9.81 m/s²`. Measure `h` and `d` between the cameras' sight lines.
3. Repeat the drop 10 or more times and average. A consistent bias in one direction points to a measurement or setup problem, not random noise.

If a result is about as long as the time to fall from the release point to the top sensor, the top phone probably triggered early, for example because your hand crossed its band.

## Troubleshooting

| Problem | What to try |
|---|---|
| "Camera unavailable" | Use the `https://` address, allow camera permission in the browser, and close other apps that use the camera |
| Pairing never connects | Put both phones on the same Wi-Fi, avoid guest or public networks that isolate devices, and create fresh codes on both phones (codes are single-use) |
| QR will not scan | Raise screen brightness, hold steady, try different distances, or use the text codes |
| Trigger never fires | Lower *smallest object width*, make the band thicker, add light, or use a larger and higher-contrast object |
| False triggers while armed | Steady the phone, raise *smallest object width*, and remove moving shadows, screens, or foliage from the band |
| Image too dark | Raise the brightness slider, or untick *Lock exposure* |
| Fall times are wrong by a constant amount | Run the zero test, then press **Re-sync clocks** before each run |

## How it works

- **Pairing:** a WebRTC data channel is set up by exchanging the connection description as a compressed QR code or text, so no signalling server is needed. A public STUN server is listed in case the phones are on different networks.
- **Detection:** each frame, a thin band of the video is reduced to a small grayscale grid. A per-pixel background and noise level are learned during the 2-second calibration. The app fires when changed pixels form a continuous block at least as wide as the chosen object width. Overall brightness shifts are compensated.
- **Timing:** each phone timestamps its own trigger with its local clock. The phones exchange ping/pong messages, keep the sample with the shortest round trip, and use it to convert the other phone's timestamp into their own clock. The fall time is the difference between the two.

## Browser support

Tested design target: Chrome on Android. Needs `getUserMedia`, `RTCPeerConnection`, and `CompressionStream`. QR scanning uses the browser's `BarcodeDetector` when available, with a JavaScript fallback. iOS Safari may work but was not a design target.
