# Awesome VPN Leak Detection [![Awesome](https://awesome.re/badge.svg)](https://awesome.re)

> A short, checked list of tools, papers and reproducible tests for detecting VPN leaks: DNS, WebRTC, IPv6 and kill-switch failures.

A VPN that leaks your real IP, ISP resolver or location gives a false sense of privacy. This list keeps only entries whose link was opened and whose description was checked against the source (last check: 19 November 2026). Entries we could not verify were removed.

Maintained by the team behind [AnonymFlow](https://www.anonymflow.com) and [VPNSmith](https://www.vpnsmith.com). Entries we publish ourselves are marked **(ours)**.

## Contents

- [Leak categories](#leak-categories)
- [Browser-based testers](#browser-based-testers)
- [Open-source detection tools](#open-source-detection-tools)
- [Command-line checks](#command-line-checks)
- [Browser extensions](#browser-extensions)
- [Known browser pitfalls](#known-browser-pitfalls)
- [Kill-switch validation](#kill-switch-validation)
- [Research](#research)
- [Related awesome lists](#related-awesome-lists)
- [Contributing](#contributing)

## Leak categories

1. **DNS leaks**: your ISP resolver answers queries while traffic goes through the tunnel.
2. **WebRTC leaks**: `RTCPeerConnection` exposes local and public addresses via ICE candidates.
3. **IPv6 leaks**: the tunnel carries IPv4 only and IPv6 traffic goes out directly.
4. **Kill-switch failures**: traffic leaves unprotected while the tunnel drops or reconnects.
5. **DNS-over-HTTPS bypass**: an application resolves through its own DoH provider instead of the VPN resolver.

## Browser-based testers

| Tool | Detects | Notes |
|------|---------|-------|
| [AnonymFlow DNS leak test](https://www.anonymflow.com/en/tools/dns-leak-test) **(ours)** | DNS | Lists the resolvers that answer your queries |
| [VPNSmith IP & WebRTC leak test](https://www.vpnsmith.com/en/tools/ip-leak-test) **(ours)** | Public IP, WebRTC | Compares your public IP with WebRTC candidates |
| [BrowserLeaks](https://browserleaks.com/) | Multi-channel | Closed source, broad fingerprinting suite |
| [IPLeak.net](https://ipleak.net/) | DNS, WebRTC, geo | Closed source |
| [DNSLeakTest.com](https://www.dnsleaktest.com/) | DNS | Closed source, standard and extended test |
| [Mullvad connection check](https://mullvad.net/en/check) | IP, DNS, WebRTC | Works with any VPN, not only Mullvad |

## Open-source detection tools

| Project | Language | Scope | License |
|---------|----------|-------|---------|
| [dnsleaktest](https://github.com/macvk/dnsleaktest) | Shell, Batch, Python | DNS (uses bash.ws) | MIT |
| [dns-leak-detector-cli](https://github.com/ricco020/dns-leak-detector-cli) **(ours)** | Python | DNS, optional ASN match against your VPN | MIT |
| [webrtc-leak-detector](https://github.com/ricco020/webrtc-leak-detector) **(ours)** | JavaScript | WebRTC local and public candidates | MIT |

## Command-line checks

- `dig +short @resolver1.opendns.com myip.opendns.com`: the address your DNS path is seen from. It should be your VPN exit.
- `curl -6 ifconfig.co`: if this answers while your VPN is IPv4-only, IPv6 is leaking.
- `tcpdump -i any 'port 53 and not host <vpn_dns_ip>'`: DNS queries that leave outside the VPN resolver.
- `ip route` and `ip -6 route`: check that the default routes point to the tunnel interface.

## Browser extensions

- **uBlock Origin**: Settings, Privacy, "Prevent WebRTC from leaking local IP addresses" ([wiki](https://github.com/gorhill/uBlock/wiki/Prevent-WebRTC-from-leaking-local-IP-address)).
- **Disable WebRTC** ([Firefox Add-ons](https://addons.mozilla.org/firefox/addon/happy-bonobo-disable-webrtc/)): single-purpose toggle for `media.peerconnection.enabled`.
- **VPN Leak Guard** ([Chrome Web Store](https://chromewebstore.google.com/detail/dlpikagegmmcphbaalmcedldgpnononn)) **(ours)**: one-click IP, WebRTC and DNS check, sets Chrome's `webRTCIPHandlingPolicy` to block non-proxied UDP. Free, no account.
- Firefox without an extension: `about:config`, `media.peerconnection.enabled = false`.

## Known browser pitfalls

- **Chrome does not apply `webRTCIPHandlingPolicy` to extension pages.** With the policy set to `disable_non_proxied_udp`, a normal tab returns 0 ICE candidates, while an extension page still returns host candidates with raw IPv4 and IPv6 addresses (no mDNS masking). A test run from an extension popup can therefore show a "leak" that websites do not see. Reproducible test with a one-permission probe extension: [chrome-webrtc-policy-extension-pages](https://gitlab.com/ler.eric/chrome-webrtc-policy-extension-pages) **(ours)**, measured on Chromium 156.0.8078.4. Always confirm with a test served from a normal web page.

## Kill-switch validation

1. Connect the VPN.
2. Run `ping -i 0.2 <host_outside_vpn>`.
3. Bring the tunnel interface down: `sudo ip link set <vpn_iface> down`.
4. Pings must fail at once. Any reply means the kill switch let traffic out.
5. Bring the interface back. Pings should resume only once the tunnel is up.

## Research

- [A Glance through the VPN Looking Glass: IPv6 Leakage and DNS Hijacking in Commercial VPN clients](https://petsymposium.org/2015/papers/02_Perta.pdf), Perta, Barbera, Tyson, Haddadi, Mei, PoPETs 2015. Analyses 14 popular commercial VPN services and finds that the majority suffer from IPv6 traffic leakage.
- [That One Privacy Site, VPN comparison chart](https://thatoneprivacysite.net/vpn-comparison-chart/): archived, still a reference for transparent comparison criteria.

## Related awesome lists

- [awesome-privacy](https://github.com/Lissy93/awesome-privacy)
- [awesome-security](https://github.com/sbilly/awesome-security)
- [awesome-osint](https://github.com/jivoi/awesome-osint)

## Contributing

Read the [contribution guidelines](CONTRIBUTING.md). Each entry must have a working link and a description that matches its source. No affiliate links.

## License

[![CC0](https://licensebuttons.net/p/zero/1.0/88x31.png)](https://creativecommons.org/publicdomain/zero/1.0/)

To the extent possible under law, the maintainers have waived all copyright and related or neighboring rights to this work.
