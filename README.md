# scrcpy-hotspot

Mirror the phone (CMF Phone, Android 16, not rooted) to the laptop over the
phone's own hotspot, without a USB cable.

## A) After both devices have rebooted

1. **Phone:** make sure it's connected to a Wi-Fi network (for example
   school Wi-Fi). Wireless debugging only turns on while the phone is on
   Wi-Fi.
2. **Phone:** turn on the **Hotspot**.
3. **Phone:** tap the **Wireless debugging** Quick Settings tile so it's on.
4. **Laptop:** run `scrcpy-hotspot`.
   - It connects the laptop to the hotspot.
   - Port 5555 is off after a reboot, so it finds the Wireless debugging port
     and turns 5555 back on.
   - Then it starts mirroring.
5. **Phone (optional):** turn the Wireless debugging tile off again. Port 5555
   stays on until the phone reboots.

## B) Reconnecting after a previous connection (no phone reboot)

1. **Phone:** turn on the **Hotspot**.
2. **Laptop:** run `scrcpy-hotspot`.

If the laptop has rebooted but the phone hasn't, use B.

## How it works

- `scrcpy-hotspot` connects the laptop to the hotspot (the `hotspot=` line at
  the top of the script). Once connected, the phone is the laptop's default
  gateway, so the script gets the phone's IP from `ip route`.
  (`scrcpy --tcpip` on its own looks for the IP on `wlan0`, which is the wrong
  interface for the hotspot, so it fails.)
- It connects to adb on port 5555 and starts scrcpy.
- adb on 5555 turns off at every phone reboot. Keeping it on permanently
  (`persist.adb.tcp.port`) needs root. If 5555 is off, the script scans the
  phone with `nmap` for the Wireless debugging port (it changes each time),
  connects to it, and runs `adb tcpip 5555`.
- No pairing is needed: the laptop's adb key was authorized over USB, and
  Wireless debugging accepts it.

## One-time setup

**Laptop:** `~/nixos-config/configuration.nix` installs `scrcpy`,
`android-tools` (`adb`) and `nmap`, and puts `~/.local/bin` on PATH
(`environment.localBinInPath = true`). Link the script there:

```bash
ln -sf ~/src/scrcpy-hotspot/scrcpy-hotspot ~/.local/bin/scrcpy-hotspot
```

**Phone:**

- Hotspot settings: turn off **Turn off hotspot automatically**. Otherwise
  the hotspot switches itself off when nothing is connected, and the laptop
  falls back to another Wi-Fi.
- Developer options:
  - Turn on **USB debugging**. Plug in USB once and accept the laptop with
    "Always allow".
  - Turn on **Disable adb authorization timeout**, so the authorization never
    expires.
  - Under **Quick settings developer tiles**, turn on **Wireless
    debugging**, then add the tile to Quick Settings.
- The first time you tap the Wireless debugging tile on a Wi-Fi network,
  allow it for that network.

## Troubleshooting

| Message | Fix |
|---|---|
| `Hotspot 👾👾👾👾 not found` | The phone's hotspot is off. Turn it on. |
| `Wireless debugging not found` | The phone rebooted, so do guide A. The tile must be on and the phone must be on Wi-Fi. |
| `Could not connect to Wireless debugging` | The laptop isn't trusted. Plug in USB once and accept, or pair: Wireless debugging → **Pair device with pairing code**, then `adb pair <phone-ip>:<port> <code>`. |
| Anything else | Fallback: plug in USB and run `adb -d tcpip 5555`. |
