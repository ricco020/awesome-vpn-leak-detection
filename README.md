# Awesome VPN Leak Detection [![Awesome](https://awesome.re/badge.svg)](https://awesome.re)

> A curated list of open-source tools, methodologies, browser extensions, papers and resources for detecting VPN leaks — DNS, WebRTC, IPv6, TCP timestamps, MTU fingerprinting and kill-switch failures.

A VPN that leaks identifying data is worse than no VPN — it gives a false sense of privacy while still exposing your real IP, ISP, DNS resolver or geo-location. This list collects everything needed to **measure, reproduce and remediate** these leaks.

Maintained by the team behind [AnonymFlow](https://www.anonymflow.com) — independent VPN reviews focused on actual leak resistance, not marketing claims.

## Contents

- [Leak categories](#leak-categories)
- [Browser-based testers](#browser-based-testers)
- [Open-source detection tools](#open-source-detection-tools)
- [Command-line / scripting tools](#command-line--scripting-tools)
- [Methodologies & test protocols](#methodologies--test-protocols)
- [Academic research & papers](#academic-research--papers)
- [Browser extensions for hardening](#browser-extensions-for-hardening)
- [Mobile-specific testers](#mobile-specific-testers)
- [Kill-switch validation](#kill-switch-validation)
- [DNS leak deep-dive](#dns-leak-deep-dive)
- [WebRTC leak deep-dive](#webrtc-leak-deep-dive)
- [IPv6 leak deep-dive](#ipv6-leak-deep-dive)
- [Datasets & benchmarks](#datasets--benchmarks)
- [Communities & forums](#communities--forums)
- [Related awesome lists](#related-awesome-lists)
- [Contributing](#contributing)

## Leak categories

A VPN connection can leak data through at least eight channels :

1. **DNS leaks** — your real ISP resolver answers queries while traffic is in the VPN tunnel.
2. **WebRTC leaks** — `RTCPeerConnection` exposes local + public IP via STUN even with VPN active.
3. **IPv6 leaks** — VPN tunnels IPv4 only, IPv6 traffic bypasses entirely.
4. **TCP timestamp leaks** — uptime clock reveals identity across reconnections.
5. **MTU / fingerprinting leaks** — packet sizes and TLS Client Hello reveal VPN provider.
6. **Kill-switch failures** — brief unprotected window during disconnect/reconnect.
7. **DNS-over-HTTPS bypass** — applications resolving via Cloudflare/Google directly.
8. **Application-level leaks** — Discord, Telegram, Steam exposing real IP via voice/P2P.

## Browser-based testers

| Tool | Detects | Source code | Notes |
|------|---------|-------------|-------|
| [AnonymFlow DNS leak test](https://www.anonymflow.com/en/tools/dns-leak-test) | DNS standard + extended | [GitHub](https://github.com/ricco020/anonymflow) | Cross-checks 6 resolvers, ASN match scoring |
| [AnonymFlow WebRTC leak test](https://www.anonymflow.com/en/tools/webrtc-leak-test) | WebRTC public + local + mDNS | [GitHub](https://github.com/ricco020/anonymflow) | STUN against multiple servers |
| [AnonymFlow IPv6 leak test](https://www.anonymflow.com/en/tools/ipv6-leak-test) | IPv6 traffic outside tunnel | [GitHub](https://github.com/ricco020/anonymflow) | Dual-stack reachability check |
| [BrowserLeaks](https://browserleaks.com/) | Multi-channel | Closed | Comprehensive fingerprinting suite |
| [IPLeak.net](https://ipleak.net/) | DNS + WebRTC + Geo | Closed | Long-standing reference |
| [DNSLeakTest.com](https://www.dnsleaktest.com/) | DNS only | Closed | Industry baseline, single-channel |

## Open-source detection tools

| Project | Language | Scope | License |
|---------|----------|-------|---------|
| [webrtc-leak-detector](https://www.npmjs.com/package/webrtc-leak-detector) ([source](https://github.com/ricco020/webrtc-leak-detector)) | JS | WebRTC public + local + mDNS | MIT |
| [dnsleaktest-cli](https://github.com/macvk/dnsleaktest) | Shell | DNS extended (Linux/Mac) | GPL-3.0 |
| [ipleak-cli](https://github.com/jonludlam/ipleak) | Go | Multi-channel | MIT |
| [vpncheck](https://github.com/mullvad/dnscheck) | Go | DNS + connection state | GPL-3.0 |
| [vpn-leak-tester](https://github.com/wiktor-k/vpn-leak-tester) | Python | Composite suite | Apache-2.0 |

## Command-line / scripting tools

For automated CI testing of VPN clients :

- **`dig +short @resolver1.opendns.com myip.opendns.com`** — Echo back the IP your DNS resolver sees you from. If it doesn't match your VPN exit, you have a DNS leak.
- **`curl -6 ifconfig.co`** — If this returns an IP when your VPN is IPv4-only, you have an IPv6 leak.
- **`tcpdump -i any 'port 53 and not host <vpn_dns_ip>'`** — Detect DNS queries leaving outside the VPN DNS server.
- **`nmcli connection show <vpn>`** — Verify the routes installed by the VPN client (look for `0.0.0.0/0` and `::/0`).
- **`ss -tunlp | grep -v <vpn_interface>`** — Connections bound to non-VPN interfaces.

## Methodologies & test protocols

- [AnonymFlow VPN leak audit protocol](https://www.anonymflow.com/en/blog/vpn-leak-audit-protocol) — 12-step reproducible audit (the methodology used in [our VPN tests](https://www.anonymflow.com/en/comparisons)).
- [That One Privacy Site — VPN comparison methodology](https://thatoneprivacysite.net/vpn-comparison-chart/) — Now archived but still the gold standard for audit transparency.
- [Mullvad No-Logs Audit](https://mullvad.net/en/blog/2025/cure53-security-assessment-mullvad) — Reference for how a no-logs claim should be verified.
- [Privacy Tools — VPN section](https://www.privacytools.io/providers/vpn/) — Curated list with leak-resistance as a criterion.

## Academic research & papers

- [A Glance through the VPN Looking Glass: IPv6 Leakage and DNS Hijacking in Commercial VPN clients](https://www.eecs.qmul.ac.uk/~hamed/papers/PETS2015VPN.pdf) — Khan, Mittal, McCoy, Kanich (2015). 14 of 14 commercial VPN providers tested leaked IPv6.
- [On the Hardness of WebRTC Leak Prevention](https://arxiv.org/abs/2006.01267) — Demonstrates WebRTC leaks persist even with VPN browser extensions claiming protection.
- [Identifying VPN Servers through Graph-Represented Behaviors](https://arxiv.org/abs/2007.04263) — How traffic fingerprinting identifies VPN exits.
- [Open to a fault: On the passive compromise of TLS keys via transient errors](https://eprint.iacr.org/2022/1014) — Side-channel leaks affecting VPN-secured TLS.

## Browser extensions for hardening

- **uBlock Origin** — `Privacy → Disable WebRTC` toggle (Firefox + Chromium).
- **NoScript** — Disable WebRTC by default per-site.
- **Disable WebRTC** ([Mozilla AMO](https://addons.mozilla.org/firefox/addon/happy-bonobo-disable-webrtc/)) — Single-purpose extension, audited source.
- **Privacy.com Settings** — built-in `media.peerconnection.enabled=false` in Firefox `about:config`.

## Mobile-specific testers

VPN apps on iOS and Android have additional leak vectors :

- **Always-On VPN bypass** — Per-app exclusions can re-introduce leaks.
- **VoLTE / IMS bypass** — Cellular signalling unaffected by VPN.
- **Captive portal detection** — `connectivitycheck.gstatic.com` resolves outside VPN by default on Android.

Tools :
- [VPN Tester iOS](https://github.com/iosre/VPNTester) (open-source)
- Android Debug Bridge : `adb shell ip route` after VPN connect — verify all traffic routes via `tun0`.

## Kill-switch validation

The kill-switch is the single most critical VPN feature. Validation steps :

1. Connect VPN.
2. Start a `ping -i 0.2 <target_outside_vpn>` to measure packet flow.
3. Disable the VPN network interface : `sudo ip link set <vpn_iface> down`.
4. Pings should immediately fail. Any successful packet = kill-switch failure.
5. Re-enable the interface. Pings should resume only after VPN tunnel re-established.

Reference test : [AnonymFlow VPN kill-switch test methodology](https://www.anonymflow.com/en/blog/vpn-killswitch-methodology).

## DNS leak deep-dive

Standard DNS leak tests query a single domain controlled by the testing site and report which resolver answered. **Extended** tests query 20+ randomized subdomains across multiple TLDs to detect :

- Round-robin DNS leaking partial queries.
- Smart DNS that forwards specific queries to ISP.
- DNS-over-HTTPS bypassing the VPN's DoH.

Reference implementation : [AnonymFlow extended DNS test source](https://github.com/ricco020/anonymflow/blob/main/lib/dns-leak-extended.ts).

## WebRTC leak deep-dive

Three independent leak vectors :

1. **Public IP via STUN** — `RTCPeerConnection.createOffer()` triggers STUN binding requests bypassing VPN routing in some configurations.
2. **Local IP** — Reveals LAN topology even with VPN.
3. **mDNS IP** — `.local` hostnames mitigate the local IP leak but introduce a new fingerprinting vector.

Reference : [webrtc-leak-detector npm](https://www.npmjs.com/package/webrtc-leak-detector).

## IPv6 leak deep-dive

VPN clients that tunnel IPv4 only leave IPv6 traffic exposed if :

- The OS has IPv6 enabled by default (all modern Linux, macOS, Windows).
- The VPN client doesn't install a `::/0 dev tun0` route.
- The VPN client doesn't disable IPv6 on physical interfaces.

Detection : reach any IPv6-only domain (e.g. `ipv6.google.com`). If the connection succeeds, IPv6 is leaking. The 2015 PETS paper (above) showed 14 of 14 commercial VPNs leaking at the time. Subsequent industry response was uneven — verify per-provider as of 2026.

## Datasets & benchmarks

- [AnonymFlow leak test results dataset](https://www.anonymflow.com/data/leak-test-results.json) — Public anonymous aggregate results.
- [Zenodo : VPN Leak Detection Dataset 2026](https://zenodo.org/) — DOI-registered academic dataset.

## Communities & forums

- [/r/VPN](https://www.reddit.com/r/VPN/) — Largest community, heavy moderation but heavy astroturfing too — read with critical eye.
- [/r/PrivacyToolsIO](https://www.reddit.com/r/PrivacyToolsIO/)
- [Hacker News : "Show HN: VPN ..."](https://hn.algolia.com/?q=VPN+leak)
- [Mullvad Forum](https://mullvad.net/help/) — Provider-specific but technically substantial.

## Related awesome lists

- [awesome-privacy](https://github.com/Lissy93/awesome-privacy) — Broad privacy resources.
- [awesome-security](https://github.com/sbilly/awesome-security) — General security.
- [awesome-osint](https://github.com/jivoi/awesome-osint) — The other side of the coin : what investigators see when leaks happen.

## Contributing

Contributions are welcome. Please read the [contribution guidelines](CONTRIBUTING.md) first.

The criteria for inclusion :
- Tool, paper or resource directly related to VPN leak detection or remediation.
- Open-source preferred (closed-source flagged explicitly).
- Active maintenance (last release < 2 years) OR seminal academic work.
- No affiliate links in this list — providers are listed only when they have substantive open-source tooling or methodology.

## License

[![CC0](https://licensebuttons.net/p/zero/1.0/88x31.png)](https://creativecommons.org/publicdomain/zero/1.0/)

To the extent possible under law, the maintainers have waived all copyright and related or neighboring rights to this work.

