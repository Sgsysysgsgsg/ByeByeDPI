# Roblox connectivity investigation

## Scope

This fork investigates Roblox connection failures on Android. A successful HTTP/SOCKS strategy test is not proof that a Roblox experience can join: Roblox uses HTTPS for account/content traffic and dynamically assigned UDP traffic for live experiences.

## Findings from source review

- The Android VPN routes IPv4 through the local ByeDPI SOCKS endpoint and can optionally route IPv6.
- The HEV tunnel is configured with UDP forwarding enabled (`udp: udp`).
- Startup currently probes the local SOCKS endpoint over TCP, then starts the tunnel and waits a fixed 100 ms. This verifies that the local SOCKS listener answers, not that Roblox UDP traffic reaches an experience server.
- The fork originally pinned `hev-socks5-tunnel` to commit `4d6c334` from 2026-02-06. Upstream commit `fc3f900` from 2026-11-21 fixes UDP frame allocation to use the actual SOCKS address length. This is a relevant correctness fix for a game that depends on UDP, but it is not proof that it caused the current ISP-specific failure.
- The fork originally pinned `hufrea/byedpi` to `ba53229` from 2026-03-26. The pin is now advanced to upstream `e268477` (2026-11-04) to include intervening native-engine fixes.

## Changes made

- Updated the two native submodule pins to current upstream commits:
  - `hufrea/byedpi`: `e2684772fbda45f2ff4fc220922a010370534216`
  - `heiher/hev-socks5-tunnel`: `fc3f90040b8d4b537341c96538669a9b2e35d514`
- Added an Android release build workflow that checks out submodules recursively and uploads APK artifacts.

## What still needs device-side validation

1. Build the APK and install it on the target Android device.
2. Run Roblox with the diagnostic VPN enabled.
3. Record whether login/home loads, whether the Play action launches, and whether the experience joins.
4. Capture sanitized logcat output around the connection attempt. Do not include account credentials, cookies, authorization headers, or personal identifiers.
5. Compare TCP/HTTPS reachability with UDP experience connectivity. A green HTTPS strategy result alone must not be presented as a successful Roblox test.

## References

- Roblox general connection guidance: https://en.help.roblox.com/hc/en-us/articles/203312880-General-Connection-Problems
- Roblox firewall/router guidance: https://en.help.roblox.com/hc/en-us/articles/203312840-Firewall-and-Router-Issues
