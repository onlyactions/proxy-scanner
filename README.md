# Free Proxy Scanner

> **Really tested** free proxy aggregator. Every proxy below was fetched from public sources, tunneled through with Xray/sing-box, and verified to actually change your egress IP. No CSV-validation theater, no blind relisting.

![working](badges/working.json) ![alive](badges/alive.json) ![speed](badges/speed.json) ![max-speed](badges/max-speed.json) ![countries](badges/countries.json) ![sources](badges/sources.json) ![updated](badges/updated.json)

## What this is

A GitHub Actions pipeline (runs once a day) that fetches free proxy configs from **173 URL subscriptions** and **81 Telegram channels**, parses 9 protocols (vless / vmess / trojan / ss / hysteria2 / hysteria / tuic / socks / http), deduplicates them, and **actually connects through every single one** to separate working proxies from dead ones.

Latest run: **2026-11-09 02:55:26 UTC** — 177472 unique proxies collected from 173/173 URL sources and 81/81 Telegram channels; **4509** endpoints really tested, **1081** reachable, **1046** verified working (egress IP confirmed changed) across **51 exit countries**.

## How the testing works (the *really* part)

```mermaid
flowchart LR
  S1[URL subs<br/>+ Telegram] --> F[fetch &<br/>parse]
  F --> C[clean &<br/>dedupe]
  C --> T1[phase 1<br/>connectivity<br/>gstatic 204]
  T1 -->|reachable| T2[phase 2<br/>exit IP check<br/>ip-api]
  T1 -->|dead| X[drop]
  T2 -->|egress changed| T3[1 MB speed<br/>test]
  T2 -->|same IP| X2[mark leak]
  T3 --> R[README + subs<br/>+ badges]
```

1. **Phase 1 — connectivity.** Each proxy is loaded as an outbound into a real Xray or sing-box instance (or used directly via `curl -x` for plain http/socks). An HTTPS request to `gstatic.com/generate_204` goes through the tunnel with **DNS resolved on the proxy side**. Only an HTTP 204 counts as reachable.
2. **Phase 2 — egress verification.** An ip-api request through the same tunnel must return an exit IP **different from the runner's own IP**, plus its country and AS. Proxies that pass phase 1 but keep your IP are marked as leaks and excluded from the working list. Server country vs exit country mismatches are flagged in the tables.
3. **Throughput.** Working proxies download 1 MB from `speed.cloudflare.com` through the tunnel; the measured rate is the speed you see in the tables.
4. **Engine fallback.** If a proxy fails on Xray, it is retried on sing-box (and vice versa) before being declared dead — some configs only work on one core.

## Top working proxies

All 1046 working proxies are published in [`subs/all.txt`](subs/all.txt); this table shows the top 25.

| # | proxy | proto | server | exit | latency | speed | engine |
|---|-------|-------|--------|------|---------|-------|--------|
| 1 | #1 ❓ ?? → 🇺🇸 US | `trojan` | `216.106.186.82:443` | United States | 251 ms | 96.5 Mbps | xray |
| 2 | #2 🇨🇦 CA → 🇨🇦 CA | `ss` | `149.22.95.183:443` | Canada | 284 ms | 73.8 Mbps | xray |
| 3 | #3 🇺🇸 US → 🇺🇸 US | `vless` | `15.204.97.209:23576` | United States | 228 ms | 62.2 Mbps | xray |
| 4 | #4 🇺🇸 US → 🇺🇸 US | `vless` | `15.204.97.216:23576` | United States | 364 ms | 60.0 Mbps | xray |
| 5 | #5 🇺🇸 US → 🇺🇸 US | `vless` | `15.204.97.197:23576` | United States | 246 ms | 59.3 Mbps | xray |
| 6 | #6 ❓ ?? → 🇺🇸 US | `trojan` | `resources.findlocaljobnetworkandbuildcommunityconnections.com:443` | United States | 360 ms | 59.1 Mbps | xray |
| 7 | #7 🇺🇸 US → 🇺🇸 US | `vless` | `15.204.97.206:23576` | United States | 234 ms | 59.0 Mbps | xray |
| 8 | #8 ❓ ?? → 🇺🇸 US | `trojan` | `www.findlocaljobnetworkandbuildcommunityconnections.com:443` | United States | 211 ms | 49.1 Mbps | xray |
| 9 | #9 🇺🇸 US → 🇺🇸 US | `vless` | `15.204.97.195:23576` | United States | 65 ms | 41.7 Mbps | xray |
| 10 | #10 🇺🇸 US → 🇦🇺 AU | `vmess` | `84.17.41.2:18000` | Australia | 37 ms | 41.3 Mbps | xray |
| 11 | #11 ❓ ?? → 🇺🇸 US | `trojan` | `blog.findlocaljobnetworkandbuildcommunityconnections.com:443` | United States | 92 ms | 41.1 Mbps | xray |
| 12 | #12 ❓ ?? → 🇺🇸 US | `ss` | `38.143.66.146:19999` | United States | 37 ms | 36.5 Mbps | xray |
| 13 | #13 ❓ ?? → 🇺🇸 US | `trojan` | `216.106.186.66:443` | United States | 1054 ms | 34.2 Mbps | xray |
| 14 | #14 ❓ ?? → 🇺🇸 US | `trojan` | `216.106.186.33:443` | United States | 58 ms | 33.2 Mbps | xray |
| 15 | #15 🇺🇸 US → 🇺🇸 US | `vless` | `15.204.97.219:23576` | United States | 69 ms | 33.2 Mbps | xray |
| 16 | #16 🇺🇸 US → 🇺🇸 US | `vless` | `15.204.97.198:23576` | United States | 62 ms | 32.8 Mbps | xray |
| 17 | #17 ❓ ?? → 🇺🇸 US | `trojan` | `216.106.186.32:443` | United States | 38 ms | 32.4 Mbps | xray |
| 18 | #18 ❓ ?? → 🇺🇸 US | `ss` | `49.51.39.148:443` | United States | 111 ms | 32.4 Mbps | singbox |
| 19 | #19 ❓ ?? → 🇺🇸 US | `trojan` | `216.106.186.46:443` | United States | 65 ms | 32.2 Mbps | xray |
| 20 | #20 ❓ ?? → 🇺🇸 US | `vless` | `91.193.19.58:443` | United States | 62 ms | 31.2 Mbps | xray |
| 21 | #21 🇺🇸 US → 🇺🇸 US | `vless` | `15.204.97.214:23576` | United States | 77 ms | 29.8 Mbps | xray |
| 22 | #22 ❓ ?? → 🇺🇸 US | `trojan` | `167.88.61.20:443` | United States | 311 ms | 27.6 Mbps | xray |
| 23 | #23 ❓ ?? → 🇺🇸 US | `vless` | `169.128.83.4:57259` | United States | 105 ms | 27.3 Mbps | singbox |
| 24 | #24 ❓ ?? → 🇺🇸 US | `vless` | `3h-unitedstates3.09vpn.com:8443` | United States | 173 ms | 27.1 Mbps | singbox |
| 25 | #25 ❓ ?? → 🇺🇸 US | `trojan` | `167.88.61.85:443` | United States | 367 ms | 27.0 Mbps | xray |

## Exit locations — which countries work

Every working proxy, grouped by the country of its **verified exit IP** (phase 2), with a dedicated subscription file per country. Currently 51 countries have at least one working exit.

| country | working | avg speed | best latency | share | list |
|---------|--------:|----------:|-------------:|------:|------|
| 🇺🇸 United States | 306 | 14.3 Mbps | 37 ms | 29% | [`US.txt`](subs/by-country/US.txt) |
| 🇳🇱 Netherlands | 152 | 5.1 Mbps | 423 ms | 15% | [`NL.txt`](subs/by-country/NL.txt) |
| 🇩🇪 Germany | 82 | 4.5 Mbps | 445 ms | 8% | [`DE.txt`](subs/by-country/DE.txt) |
| 🇯🇵 Japan | 69 | 3.4 Mbps | 316 ms | 7% | [`JP.txt`](subs/by-country/JP.txt) |
| 🇨🇦 Canada | 42 | 9.3 Mbps | 243 ms | 4% | [`CA.txt`](subs/by-country/CA.txt) |
| 🇬🇧 United Kingdom | 38 | 5.1 Mbps | 454 ms | 4% | [`GB.txt`](subs/by-country/GB.txt) |
| 🇫🇷 France | 31 | 4.9 Mbps | 487 ms | 3% | [`FR.txt`](subs/by-country/FR.txt) |
| 🇸🇬 Singapore | 30 | 4.1 Mbps | 492 ms | 3% | [`SG.txt`](subs/by-country/SG.txt) |
| 🇰🇷 South Korea | 28 | 3.9 Mbps | 473 ms | 3% | [`KR.txt`](subs/by-country/KR.txt) |
| 🇸🇪 Sweden | 22 | 4.5 Mbps | 520 ms | 2% | [`SE.txt`](subs/by-country/SE.txt) |
| 🇵🇱 Poland | 21 | 4.8 Mbps | 644 ms | 2% | [`PL.txt`](subs/by-country/PL.txt) |
| 🇪🇸 Spain | 18 | 4.9 Mbps | 520 ms | 2% | [`ES.txt`](subs/by-country/ES.txt) |
| 🇮🇹 Italy | 17 | 5.2 Mbps | 491 ms | 2% | [`IT.txt`](subs/by-country/IT.txt) |
| 🇨🇭 Switzerland | 17 | 4.2 Mbps | 500 ms | 2% | [`CH.txt`](subs/by-country/CH.txt) |
| 🇦🇺 Australia | 14 | 11.2 Mbps | 37 ms | 1% | [`AU.txt`](subs/by-country/AU.txt) |
| 🇧🇾 BY | 13 | 4.3 Mbps | 582 ms | 1% | [`BY.txt`](subs/by-country/BY.txt) |
| 🇫🇮 Finland | 13 | 3.3 Mbps | 735 ms | 1% | [`FI.txt`](subs/by-country/FI.txt) |
| 🇧🇬 Bulgaria | 10 | 3.5 Mbps | 797 ms | 1% | [`BG.txt`](subs/by-country/BG.txt) |
| 🇲🇩 Moldova | 9 | 2.9 Mbps | 1045 ms | 1% | [`MD.txt`](subs/by-country/MD.txt) |
| 🇷🇺 Russia | 8 | 3.3 Mbps | 673 ms | 1% | [`RU.txt`](subs/by-country/RU.txt) |
| 🇮🇳 India | 8 | 2.9 Mbps | 888 ms | 1% | [`IN.txt`](subs/by-country/IN.txt) |
| 🇷🇸 Serbia | 8 | 3.5 Mbps | 922 ms | 1% | [`RS.txt`](subs/by-country/RS.txt) |
| 🇭🇰 Hong Kong | 7 | 3.5 Mbps | 578 ms | 1% | [`HK.txt`](subs/by-country/HK.txt) |
| 🇪🇪 Estonia | 7 | 4.1 Mbps | 696 ms | 1% | [`EE.txt`](subs/by-country/EE.txt) |
| 🇮🇷 Iran | 7 | 3.0 Mbps | 1425 ms | 1% | [`IR.txt`](subs/by-country/IR.txt) |
| 🇨🇿 Czechia | 6 | 3.8 Mbps | 664 ms | 1% | [`CZ.txt`](subs/by-country/CZ.txt) |
| 🇺🇦 Ukraine | 5 | 3.9 Mbps | 868 ms | 0% | [`UA.txt`](subs/by-country/UA.txt) |
| 🇱🇻 Latvia | 5 | 4.8 Mbps | 759 ms | 0% | [`LV.txt`](subs/by-country/LV.txt) |
| 🇷🇴 Romania | 5 | 3.4 Mbps | 609 ms | 0% | [`RO.txt`](subs/by-country/RO.txt) |
| 🇦🇪 UAE | 5 | 3.2 Mbps | 1118 ms | 0% | [`AE.txt`](subs/by-country/AE.txt) |
| 🇰🇿 Kazakhstan | 5 | 3.0 Mbps | 1219 ms | 0% | [`KZ.txt`](subs/by-country/KZ.txt) |
| 🇦🇹 Austria | 4 | 5.2 Mbps | 546 ms | 0% | [`AT.txt`](subs/by-country/AT.txt) |
| 🇹🇷 Turkey | 4 | 3.9 Mbps | 812 ms | 0% | [`TR.txt`](subs/by-country/TR.txt) |
| 🇲🇽 Mexico | 3 | 10.3 Mbps | 313 ms | 0% | [`MX.txt`](subs/by-country/MX.txt) |
| 🇳🇴 Norway | 3 | 4.2 Mbps | 904 ms | 0% | [`NO.txt`](subs/by-country/NO.txt) |
| 🇹🇼 Taiwan | 3 | 3.2 Mbps | 553 ms | 0% | [`TW.txt`](subs/by-country/TW.txt) |
| 🇿🇦 South Africa | 3 | 3.2 Mbps | 1012 ms | 0% | [`ZA.txt`](subs/by-country/ZA.txt) |
| 🇻🇳 Vietnam | 2 | 4.4 Mbps | 820 ms | 0% | [`VN.txt`](subs/by-country/VN.txt) |
| 🇧🇷 Brazil | 2 | 4.8 Mbps | 787 ms | 0% | [`BR.txt`](subs/by-country/BR.txt) |
| 🇹🇭 Thailand | 2 | 4.4 Mbps | 724 ms | 0% | [`TH.txt`](subs/by-country/TH.txt) |
| 🇱🇰 LK | 2 | 3.8 Mbps | 1169 ms | 0% | [`LK.txt`](subs/by-country/LK.txt) |
| 🇮🇪 Ireland | 1 | 6.0 Mbps | 399 ms | 0% | [`IE.txt`](subs/by-country/IE.txt) |
| 🇨🇱 Chile | 1 | 5.3 Mbps | 649 ms | 0% | [`CL.txt`](subs/by-country/CL.txt) |
| 🇮🇸 Iceland | 1 | 4.9 Mbps | 871 ms | 0% | [`IS.txt`](subs/by-country/IS.txt) |
| 🇮🇱 Israel | 1 | 4.6 Mbps | 792 ms | 0% | [`IL.txt`](subs/by-country/IL.txt) |
| 🇱🇺 Luxembourg | 1 | 4.4 Mbps | 857 ms | 0% | [`LU.txt`](subs/by-country/LU.txt) |
| 🇬🇷 Greece | 1 | 4.2 Mbps | 843 ms | 0% | [`GR.txt`](subs/by-country/GR.txt) |
| 🇩🇰 Denmark | 1 | 4.0 Mbps | 1134 ms | 0% | [`DK.txt`](subs/by-country/DK.txt) |
| 🇸🇦 Saudi Arabia | 1 | 3.6 Mbps | 1219 ms | 0% | [`SA.txt`](subs/by-country/SA.txt) |
| 🇲🇾 Malaysia | 1 | - | 772 ms | 0% | [`MY.txt`](subs/by-country/MY.txt) |
| 🇦🇱 AL | 1 | - | 963 ms | 0% | [`AL.txt`](subs/by-country/AL.txt) |

A `??` exit means the geo lookup could not resolve the exit IP. Base64 variants of every country file sit next to them as `<CC>.base64.txt`.

## Protocol distribution

| protocol | tested | alive | working | alive rate |
|----------|-------:|------:|--------:|-----------:|
| `vless` | 2692 | 514 | 490 | 19% |
| `ss` | 621 | 250 | 246 | 40% |
| `vmess` | 922 | 143 | 136 | 16% |
| `trojan` | 180 | 123 | 123 | 68% |
| `hysteria2` | 85 | 48 | 48 | 56% |
| `socks` | 4 | 3 | 3 | 75% |
| `http` | 5 | 0 | 0 | 0% |

Per-protocol working lists: [`subs/by-proto/`](subs/by-proto/) (one `vless.txt`, `vmess.txt`, ... per protocol, plus `.base64.txt` variants).

## Engine breakdown

Which core verified each working proxy. The engine fallback means a proxy that failed on its primary core got a second chance on the other one.

| engine | working | note |
|--------|--------:|------|
| Xray-core | 935 | [`allxray.txt`](subs/by-engine/allxray.txt) |
| sing-box | 108 | [`allsingbox.txt`](subs/by-engine/allsingbox.txt) |
| direct (curl) | 3 | plain http/socks, [`alldirect.txt`](subs/by-engine/alldirect.txt) |

## Speed & latency profile

| speed tier | proxies |
|-----------|--------:|
| >= 5 Mbps | 588 |
| 1 - 5 Mbps | 344 |
| 0.25 - 1 Mbps | 37 |
| < 0.25 Mbps | 0 |
| unmeasured | 77 |

Latency (phase-1 HTTPS round trip): median **648 ms**, p90 **1893 ms**. 35 proxies passed connectivity but failed egress verification (leaked the runner IP or broke on the exit check) and are excluded.

## Source health

254 active, 0 auto-retired (10 fetch failures / 10 all-dead runs / 10 days without anything new). Best contributors this run:

| source | kind | tested | alive | new this run |
|--------|------|-------:|------:|-------------:|
| `NK` | url | 977 | 677 | 8731 |
| `EV` | url | 669 | 584 | 800 |
| `SK` | url | 882 | 582 | 0 |
| `HP` | url | 403 | 374 | 57 |
| `RD-VL` | url | 2262 | 308 | 3736 |
| `KA` | url | 2888 | 306 | 10756 |
| `HC` | url | 1281 | 297 | 188 |
| `Ni` | url | 339 | 266 | 90 |
| `RD-SS` | url | 512 | 244 | 338 |
| `ET` | url | 347 | 235 | 1822 |
| `F0` | url | 234 | 229 | 2 |
| `AG-PL` | url | 464 | 228 | 588 |
| `EV-SS` | url | 221 | 213 | 99 |
| `EV-VL` | url | 267 | 210 | 0 |
| `ST-VL` | url | 2072 | 192 | 364 |
| `SB` | url | 2156 | 159 | 1 |
| `LA` | url | 764 | 137 | 14895 |
| `RD-VM` | url | 641 | 132 | 3480 |
| `SB-VL` | url | 1927 | 127 | 3 |
| `EP-ALL` | url | 1325 | 124 | 0 |

## History

working per run (last 22): ▁▁▆▆▇█▇▆▇█▇▆▇▆▇▇█▇▇███

| date (UTC) | tested | alive | working | avg | max |
|------------|-------:|------:|--------:|----:|----:|
| 2026-11-09 03:34 | 4509 | 1081 | 1046 | 7.6M | 96.5M |
| 2026-11-08 03:39 | 4502 | 1098 | 1068 | 7.7M | 107.7M |
| 2026-11-07 03:34 | 4503 | 1040 | 994 | 7.6M | 82.2M |
| 2026-11-06 10:53 | 4846 | 900 | 837 | 8.5M | 49.1M |
| 2026-10-03 11:04 | 4500 | 899 | 860 | 8.7M | 82.3M |
| 2026-10-03 02:52 | 4500 | 1062 | 1030 | 8.0M | 75.5M |
| 2026-10-02 21:49 | 4500 | 890 | 861 | 8.9M | 68.7M |
| 2026-10-02 17:14 | 4500 | 847 | 807 | 8.6M | 88.1M |
| 2026-10-02 11:49 | 4500 | 861 | 798 | 7.9M | 85.3M |
| 2026-10-01 22:23 | 4500 | 904 | 878 | 7.5M | 90.2M |
| 2026-10-01 17:56 | 4500 | 861 | 794 | 7.3M | 58.6M |
| 2026-10-01 12:16 | 4500 | 931 | 883 | 7.3M | 34.4M |
| 2026-10-01 03:00 | 4500 | 1112 | 1075 | 8.1M | 45.4M |
| 2026-09-30 21:54 | 4500 | 899 | 866 | 9.3M | 69.4M |
| 2026-09-30 17:26 | 4500 | 838 | 795 | 7.3M | 57.9M |

## Subscription files

Everything under [`subs/`](subs/) is regenerated every run. Plain lists are one proxy URI per line; `.base64.txt` variants are the same list base64-encoded for clients that need the classic sub format.

| file | contents | direct subscription URL |
|------|----------|--------------------------|
| [`all.txt`](subs/all.txt) | ALL working proxies, ranked by speed (1046) | `https://raw.githubusercontent.com/onlyactions/proxy-scanner/main/subs/all.txt` |
| [`all.base64.txt`](subs/all.base64.txt) | same, base64 | `https://raw.githubusercontent.com/onlyactions/proxy-scanner/main/subs/all.base64.txt` |
| [`top150.txt`](subs/top150.txt) | top 150 ranked (150) | `https://raw.githubusercontent.com/onlyactions/proxy-scanner/main/subs/top150.txt` |
| [`top150.base64.txt`](subs/top150.base64.txt) | same, base64 | `https://raw.githubusercontent.com/onlyactions/proxy-scanner/main/subs/top150.base64.txt` |
| [`by-proto/`](subs/by-proto/) | one list per protocol (6 protocols) | ...`/subs/by-proto/<proto>.txt` |
| [`by-engine/allxray.txt`](subs/by-engine/allxray.txt) | verified on Xray (935) | `https://raw.githubusercontent.com/onlyactions/proxy-scanner/main/subs/by-engine/allxray.txt` |
| [`by-engine/allsingbox.txt`](subs/by-engine/allsingbox.txt) | verified on sing-box (108) | `https://raw.githubusercontent.com/onlyactions/proxy-scanner/main/subs/by-engine/allsingbox.txt` |
| [`by-country/`](subs/by-country/) | one list per exit country (51 countries, e.g. [`US.txt`](subs/by-country/US.txt), [`DE.txt`](subs/by-country/DE.txt)) | ...`/subs/by-country/<CC>.txt` |
| [`sing-box-client.json`](subs/sing-box-client.json) | top 150 as a ready-to-run sing-box client config | `https://raw.githubusercontent.com/onlyactions/proxy-scanner/main/subs/sing-box-client.json` |
| [`xray-client.json`](subs/xray-client.json) | top 150 as a ready-to-run xray client config | `https://raw.githubusercontent.com/onlyactions/proxy-scanner/main/subs/xray-client.json` |
| [`archive/`](archive/) | gz snapshot of every collection (30 kept) | — |

## Use the proxies

- **v2rayN / v2rayNG / Nekobox / Hiddify / Shadowrocket**: add subscription `https://raw.githubusercontent.com/onlyactions/proxy-scanner/main/subs/all.base64.txt` (everything working) or `https://raw.githubusercontent.com/onlyactions/proxy-scanner/main/subs/top150.base64.txt` (curated top 150).
- **Want a specific country?** Grab `subs/by-country/<CC>.txt` (see the Exit locations table above) — e.g. `https://raw.githubusercontent.com/onlyactions/proxy-scanner/main/subs/by-country/US.txt`, `https://raw.githubusercontent.com/onlyactions/proxy-scanner/main/subs/by-country/DE.txt`.
- **Only one protocol?** `subs/by-proto/<proto>.txt` — e.g. `https://raw.githubusercontent.com/onlyactions/proxy-scanner/main/subs/by-proto/vless.txt`, `https://raw.githubusercontent.com/onlyactions/proxy-scanner/main/subs/by-proto/hysteria2.txt`.
- **sing-box**: download [`subs/sing-box-client.json`](subs/sing-box-client.json) and run `sing-box run -c sing-box-client.json` — it listens on `127.0.0.1:2080` (mixed) with a selector + auto urltest group.
- **Xray**: download [`subs/xray-client.json`](subs/xray-client.json), socks inbound on `127.0.0.1:2080`.
- Raw snapshots of every collection: [`archive/`](archive/) (30 kept).

## Repository layout

```
sources/            url_subs.yml + telegram.yml (scannable source lists)
scripts/            collect.py, test.py, report.py + lib/
state/              source health, alive-recall pool, geo cache (committed)
data/               stats, history, compact last-run results
subs/               THE PRODUCT: verified working proxies, split by
                    proto / engine / exit country + all + top150 lists
badges/             shields.io endpoint badges
archive/            gz snapshots of every collection
```

## Why most "free proxy lists" lie

The typical aggregator relists whatever it scraped, so 80-95% of entries are dead within hours — this run found **76% unreachable** and **77% unusable** overall. Only tunnels that completed a real HTTPS handshake, returned a verifiable foreign exit IP, and pushed a 1 MB download are counted as working here.

## Disclaimers

- Free proxies are **public and untrusted**. Anything you send through them can be observed or modified by their operators. Never use them for authentication, payments, or anything sensitive.
- All configs are collected from publicly posted sources (GitHub subscriptions and public Telegram channels); no credentials are guessed or brute-forced. Removal requests: open an issue.
- Availability is transient: the working list is re-verified daily, and past performance never guarantees future availability.

