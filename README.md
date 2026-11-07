# Free Proxy Scanner

> **Really tested** free proxy aggregator. Every proxy below was fetched from public sources, tunneled through with Xray/sing-box, and verified to actually change your egress IP. No CSV-validation theater, no blind relisting.

![working](badges/working.json) ![alive](badges/alive.json) ![speed](badges/speed.json) ![max-speed](badges/max-speed.json) ![countries](badges/countries.json) ![sources](badges/sources.json) ![updated](badges/updated.json)

## What this is

A GitHub Actions pipeline (runs once a day) that fetches free proxy configs from **173 URL subscriptions** and **81 Telegram channels**, parses 9 protocols (vless / vmess / trojan / ss / hysteria2 / hysteria / tuic / socks / http), deduplicates them, and **actually connects through every single one** to separate working proxies from dead ones.

Latest run: **2026-11-07 02:54:36 UTC** — 177717 unique proxies collected from 173/173 URL sources and 81/81 Telegram channels; **4503** endpoints really tested, **1040** reachable, **994** verified working (egress IP confirmed changed) across **53 exit countries**.

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

All 994 working proxies are published in [`subs/all.txt`](subs/all.txt); this table shows the top 25.

| # | proxy | proto | server | exit | latency | speed | engine |
|---|-------|-------|--------|------|---------|-------|--------|
| 1 | #1 🇨🇦 CA → 🇨🇦 CA | `ss` | `149.22.95.183:443` | Canada | 214 ms | 82.2 Mbps | xray |
| 2 | #2 ❓ ?? → 🇺🇸 US | `trojan` | `resources.findlocaljobnetworkandbuildcommunityconnections.com:443` | United States | 260 ms | 61.6 Mbps | xray |
| 3 | #3 🇺🇸 US → 🇺🇸 US | `vless` | `15.204.97.197:23576` | United States | 249 ms | 58.7 Mbps | xray |
| 4 | #4 🇺🇸 US → 🇺🇸 US | `vless` | `15.204.97.214:23576` | United States | 130 ms | 55.5 Mbps | xray |
| 5 | #5 ❓ ?? → 🇺🇸 US | `vless` | `ww13.levikogjgfdd.ir:23576` | United States | 235 ms | 52.1 Mbps | xray |
| 6 | #6 🇺🇸 US → 🇺🇸 US | `vmess` | `84.17.41.2:18000` | United States | 49 ms | 51.2 Mbps | xray |
| 7 | #7 ❓ ?? → 🇺🇸 US | `trojan` | `216.106.186.46:443` | United States | 53 ms | 40.2 Mbps | xray |
| 8 | #8 🇺🇸 US → 🇺🇸 US | `vless` | `15.204.97.216:23576` | United States | 119 ms | 39.7 Mbps | xray |
| 9 | #9 ❓ ?? → 🇺🇸 US | `trojan` | `216.106.186.82:443` | United States | 63 ms | 36.4 Mbps | xray |
| 10 | #10 🇺🇸 US → 🇺🇸 US | `vless` | `15.204.97.219:23576` | United States | 78 ms | 35.5 Mbps | xray |
| 11 | #11 🇺🇸 US → 🇺🇸 US | `vless` | `15.204.97.206:23576` | United States | 83 ms | 32.5 Mbps | xray |
| 12 | #12 ❓ ?? → 🇺🇸 US | `trojan` | `blog.findlocaljobnetworkandbuildcommunityconnections.com:443` | United States | 106 ms | 31.7 Mbps | xray |
| 13 | #13 🇺🇸 US → 🇺🇸 US | `vless` | `15.204.97.209:23576` | United States | 60 ms | 31.6 Mbps | xray |
| 14 | #14 ❓ ?? → 🇺🇸 US | `trojan` | `216.106.186.33:443` | United States | 53 ms | 30.7 Mbps | xray |
| 15 | #15 ❓ ?? → 🇺🇸 US | `ss` | `169.197.142.68:19999` | United States | 182 ms | 29.7 Mbps | xray |
| 16 | #16 ❓ ?? → 🇺🇸 US | `trojan` | `216.106.186.32:443` | United States | 39 ms | 29.7 Mbps | xray |
| 17 | #17 ❓ ?? → 🇺🇸 US | `trojan` | `167.88.61.198:443` | United States | 172 ms | 28.7 Mbps | xray |
| 18 | #18 ❓ ?? → 🇺🇸 US | `trojan` | `167.88.61.20:443` | United States | 347 ms | 28.6 Mbps | xray |
| 19 | #19 ❓ ?? → 🇺🇸 US | `trojan` | `www.findlocaljobnetworkandbuildcommunityconnections.com:443` | United States | 99 ms | 28.4 Mbps | xray |
| 20 | #20 ❓ ?? → 🇺🇸 US | `vmess` | `169.197.142.68:1270` | United States | 4134 ms | 28.4 Mbps | xray |
| 21 | #21 🇺🇸 US → 🇺🇸 US | `vless` | `15.204.97.195:23576` | United States | 79 ms | 28.0 Mbps | xray |
| 22 | #22 🇺🇸 US → 🇺🇸 US | `vless` | `15.204.97.198:23576` | United States | 67 ms | 27.0 Mbps | xray |
| 23 | #23 ❓ ?? → 🇺🇸 US | `trojan` | `66.163.114.114:443` | United States | 327 ms | 26.7 Mbps | xray |
| 24 | #24 ❓ ?? → 🇺🇸 US | `vless` | `198.2.251.107:45197` | United States | 124 ms | 26.1 Mbps | xray |
| 25 | #25 🇺🇸 US → 🇺🇸 US | `ss` | `216.105.168.18:443` | United States | 129 ms | 25.4 Mbps | xray |

## Exit locations — which countries work

Every working proxy, grouped by the country of its **verified exit IP** (phase 2), with a dedicated subscription file per country. Currently 53 countries have at least one working exit.

| country | working | avg speed | best latency | share | list |
|---------|--------:|----------:|-------------:|------:|------|
| 🇺🇸 United States | 282 | 14.3 Mbps | 39 ms | 28% | [`US.txt`](subs/by-country/US.txt) |
| 🇳🇱 Netherlands | 145 | 5.2 Mbps | 436 ms | 15% | [`NL.txt`](subs/by-country/NL.txt) |
| 🇩🇪 Germany | 71 | 5.0 Mbps | 441 ms | 7% | [`DE.txt`](subs/by-country/DE.txt) |
| 🇯🇵 Japan | 68 | 4.5 Mbps | 304 ms | 7% | [`JP.txt`](subs/by-country/JP.txt) |
| 🇸🇬 Singapore | 55 | 4.1 Mbps | 488 ms | 6% | [`SG.txt`](subs/by-country/SG.txt) |
| 🇨🇦 Canada | 41 | 10.2 Mbps | 214 ms | 4% | [`CA.txt`](subs/by-country/CA.txt) |
| 🇬🇧 United Kingdom | 37 | 4.7 Mbps | 432 ms | 4% | [`GB.txt`](subs/by-country/GB.txt) |
| 🇫🇷 France | 29 | 4.7 Mbps | 461 ms | 3% | [`FR.txt`](subs/by-country/FR.txt) |
| 🇵🇱 Poland | 28 | 4.4 Mbps | 536 ms | 3% | [`PL.txt`](subs/by-country/PL.txt) |
| 🇫🇮 Finland | 20 | 3.8 Mbps | 513 ms | 2% | [`FI.txt`](subs/by-country/FI.txt) |
| 🇮🇹 Italy | 18 | 5.4 Mbps | 488 ms | 2% | [`IT.txt`](subs/by-country/IT.txt) |
| 🇸🇪 Sweden | 18 | 4.5 Mbps | 546 ms | 2% | [`SE.txt`](subs/by-country/SE.txt) |
| 🇪🇸 Spain | 16 | 5.3 Mbps | 456 ms | 2% | [`ES.txt`](subs/by-country/ES.txt) |
| 🇨🇭 Switzerland | 12 | 4.9 Mbps | 501 ms | 1% | [`CH.txt`](subs/by-country/CH.txt) |
| 🇰🇷 South Korea | 11 | 5.4 Mbps | 462 ms | 1% | [`KR.txt`](subs/by-country/KR.txt) |
| 🇧🇾 BY | 11 | 4.6 Mbps | 704 ms | 1% | [`BY.txt`](subs/by-country/BY.txt) |
| 🇹🇷 Turkey | 11 | 4.0 Mbps | 836 ms | 1% | [`TR.txt`](subs/by-country/TR.txt) |
| 🇭🇰 Hong Kong | 10 | 5.2 Mbps | 579 ms | 1% | [`HK.txt`](subs/by-country/HK.txt) |
| 🇷🇴 Romania | 9 | 3.7 Mbps | 609 ms | 1% | [`RO.txt`](subs/by-country/RO.txt) |
| 🇲🇩 Moldova | 9 | 3.0 Mbps | 1042 ms | 1% | [`MD.txt`](subs/by-country/MD.txt) |
| 🇷🇸 Serbia | 8 | 3.8 Mbps | 920 ms | 1% | [`RS.txt`](subs/by-country/RS.txt) |
| 🇧🇬 Bulgaria | 8 | 3.6 Mbps | 904 ms | 1% | [`BG.txt`](subs/by-country/BG.txt) |
| 🇮🇳 India | 8 | 2.4 Mbps | 877 ms | 1% | [`IN.txt`](subs/by-country/IN.txt) |
| 🇮🇷 Iran | 6 | 3.2 Mbps | 1455 ms | 1% | [`IR.txt`](subs/by-country/IR.txt) |
| 🇰🇿 Kazakhstan | 6 | 3.0 Mbps | 1194 ms | 1% | [`KZ.txt`](subs/by-country/KZ.txt) |
| 🇦🇹 Austria | 5 | 4.3 Mbps | 520 ms | 1% | [`AT.txt`](subs/by-country/AT.txt) |
| 🇺🇦 Ukraine | 5 | 3.8 Mbps | 865 ms | 1% | [`UA.txt`](subs/by-country/UA.txt) |
| 🇨🇿 Czechia | 4 | 4.8 Mbps | 686 ms | 0% | [`CZ.txt`](subs/by-country/CZ.txt) |
| 🇹🇼 Taiwan | 4 | 3.4 Mbps | 546 ms | 0% | [`TW.txt`](subs/by-country/TW.txt) |
| 🇦🇪 UAE | 4 | 3.9 Mbps | 1013 ms | 0% | [`AE.txt`](subs/by-country/AE.txt) |
| 🇲🇽 Mexico | 3 | 8.4 Mbps | 312 ms | 0% | [`MX.txt`](subs/by-country/MX.txt) |
| 🇪🇪 Estonia | 3 | 4.6 Mbps | 528 ms | 0% | [`EE.txt`](subs/by-country/EE.txt) |
| 🇱🇻 Latvia | 3 | 4.7 Mbps | 772 ms | 0% | [`LV.txt`](subs/by-country/LV.txt) |
| 🇷🇺 Russia | 3 | 3.7 Mbps | 889 ms | 0% | [`RU.txt`](subs/by-country/RU.txt) |
| 🇦🇺 Australia | 2 | 4.7 Mbps | 746 ms | 0% | [`AU.txt`](subs/by-country/AU.txt) |
| 🇧🇷 Brazil | 2 | 4.8 Mbps | 760 ms | 0% | [`BR.txt`](subs/by-country/BR.txt) |
| 🇿🇦 South Africa | 2 | 3.3 Mbps | 1019 ms | 0% | [`ZA.txt`](subs/by-country/ZA.txt) |
| 🇻🇳 Vietnam | 2 | 1.8 Mbps | 1017 ms | 0% | [`VN.txt`](subs/by-country/VN.txt) |
| 🇮🇪 Ireland | 1 | 6.9 Mbps | 411 ms | 0% | [`IE.txt`](subs/by-country/IE.txt) |
| 🇧🇪 Belgium | 1 | 5.9 Mbps | 483 ms | 0% | [`BE.txt`](subs/by-country/BE.txt) |
| 🇱🇰 LK | 1 | 5.7 Mbps | 1110 ms | 0% | [`LK.txt`](subs/by-country/LK.txt) |
| 🇨🇱 Chile | 1 | 5.3 Mbps | 677 ms | 0% | [`CL.txt`](subs/by-country/CL.txt) |
| 🇹🇭 Thailand | 1 | 4.7 Mbps | 5702 ms | 0% | [`TH.txt`](subs/by-country/TH.txt) |
| 🇬🇷 Greece | 1 | 4.4 Mbps | 810 ms | 0% | [`GR.txt`](subs/by-country/GR.txt) |
| 🇱🇹 Lithuania | 1 | 4.4 Mbps | 985 ms | 0% | [`LT.txt`](subs/by-country/LT.txt) |
| 🇩🇰 Denmark | 1 | 4.2 Mbps | 875 ms | 0% | [`DK.txt`](subs/by-country/DK.txt) |
| 🇵🇭 Philippines | 1 | 4.0 Mbps | 949 ms | 0% | [`PH.txt`](subs/by-country/PH.txt) |
| 🇸🇦 Saudi Arabia | 1 | 3.6 Mbps | 1438 ms | 0% | [`SA.txt`](subs/by-country/SA.txt) |
| 🇵🇪 PE | 1 | 2.3 Mbps | 1723 ms | 0% | [`PE.txt`](subs/by-country/PE.txt) |
| 🇦🇱 AL | 1 | 2.0 Mbps | 2490 ms | 0% | [`AL.txt`](subs/by-country/AL.txt) |
| 🇮🇱 Israel | 1 | 1.0 Mbps | 766 ms | 0% | [`IL.txt`](subs/by-country/IL.txt) |
| 🇲🇾 Malaysia | 1 | - | 529 ms | 0% | [`MY.txt`](subs/by-country/MY.txt) |
| 🇳🇴 Norway | 1 | - | 586 ms | 0% | [`NO.txt`](subs/by-country/NO.txt) |

A `??` exit means the geo lookup could not resolve the exit IP. Base64 variants of every country file sit next to them as `<CC>.base64.txt`.

## Protocol distribution

| protocol | tested | alive | working | alive rate |
|----------|-------:|------:|--------:|-----------:|
| `vless` | 2596 | 489 | 466 | 19% |
| `ss` | 595 | 253 | 239 | 43% |
| `vmess` | 1022 | 142 | 135 | 14% |
| `trojan` | 214 | 113 | 112 | 53% |
| `hysteria2` | 68 | 40 | 40 | 59% |
| `socks` | 3 | 3 | 2 | 100% |
| `http` | 5 | 0 | 0 | 0% |

Per-protocol working lists: [`subs/by-proto/`](subs/by-proto/) (one `vless.txt`, `vmess.txt`, ... per protocol, plus `.base64.txt` variants).

## Engine breakdown

Which core verified each working proxy. The engine fallback means a proxy that failed on its primary core got a second chance on the other one.

| engine | working | note |
|--------|--------:|------|
| Xray-core | 887 | [`allxray.txt`](subs/by-engine/allxray.txt) |
| sing-box | 105 | [`allsingbox.txt`](subs/by-engine/allsingbox.txt) |
| direct (curl) | 2 | plain http/socks, [`alldirect.txt`](subs/by-engine/alldirect.txt) |

## Speed & latency profile

| speed tier | proxies |
|-----------|--------:|
| >= 5 Mbps | 582 |
| 1 - 5 Mbps | 309 |
| 0.25 - 1 Mbps | 20 |
| < 0.25 Mbps | 0 |
| unmeasured | 83 |

Latency (phase-1 HTTPS round trip): median **621 ms**, p90 **1723 ms**. 46 proxies passed connectivity but failed egress verification (leaked the runner IP or broke on the exit check) and are excluded.

## Source health

254 active, 0 auto-retired (10 fetch failures / 10 all-dead runs / 10 days without anything new). Best contributors this run:

| source | kind | tested | alive | new this run |
|--------|------|-------:|------:|-------------:|
| `EV` | url | 690 | 624 | 208 |
| `SK` | url | 862 | 584 | 8687 |
| `NK` | url | 797 | 512 | 204 |
| `HP` | url | 486 | 437 | 83 |
| `RD-VL` | url | 2246 | 314 | 3675 |
| `HC` | url | 1272 | 300 | 134 |
| `AG-PL` | url | 433 | 295 | 1115 |
| `EV-VL` | url | 291 | 243 | 700 |
| `RD-SS` | url | 509 | 237 | 317 |
| `F0` | url | 229 | 218 | 1 |
| `ST-VL` | url | 2135 | 217 | 839 |
| `EV-SS` | url | 219 | 214 | 150 |
| `KA` | url | 2944 | 207 | 5745 |
| `Ni` | url | 310 | 206 | 107 |
| `SB` | url | 2273 | 160 | 0 |
| `LA` | url | 826 | 142 | 15794 |
| `RD-VM` | url | 581 | 131 | 3470 |
| `SB-VL` | url | 1940 | 126 | 1 |
| `EP-ALL` | url | 1325 | 124 | 0 |
| `ST-VM` | url | 584 | 120 | 63 |

## History

working per run (last 20): ▁▁▆▆▇█▇▆▇█▇▆▇▆▇▇█▇▇█

| date (UTC) | tested | alive | working | avg | max |
|------------|-------:|------:|--------:|----:|----:|
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
| 2026-09-30 11:48 | 4500 | 860 | 810 | 7.9M | 64.4M |
| 2026-09-30 02:56 | 4500 | 1030 | 973 | 10.3M | 92.2M |

## Subscription files

Everything under [`subs/`](subs/) is regenerated every run. Plain lists are one proxy URI per line; `.base64.txt` variants are the same list base64-encoded for clients that need the classic sub format.

| file | contents | direct subscription URL |
|------|----------|--------------------------|
| [`all.txt`](subs/all.txt) | ALL working proxies, ranked by speed (994) | `https://raw.githubusercontent.com/onlyactions/proxy-scanner/main/subs/all.txt` |
| [`all.base64.txt`](subs/all.base64.txt) | same, base64 | `https://raw.githubusercontent.com/onlyactions/proxy-scanner/main/subs/all.base64.txt` |
| [`top150.txt`](subs/top150.txt) | top 150 ranked (150) | `https://raw.githubusercontent.com/onlyactions/proxy-scanner/main/subs/top150.txt` |
| [`top150.base64.txt`](subs/top150.base64.txt) | same, base64 | `https://raw.githubusercontent.com/onlyactions/proxy-scanner/main/subs/top150.base64.txt` |
| [`by-proto/`](subs/by-proto/) | one list per protocol (6 protocols) | ...`/subs/by-proto/<proto>.txt` |
| [`by-engine/allxray.txt`](subs/by-engine/allxray.txt) | verified on Xray (887) | `https://raw.githubusercontent.com/onlyactions/proxy-scanner/main/subs/by-engine/allxray.txt` |
| [`by-engine/allsingbox.txt`](subs/by-engine/allsingbox.txt) | verified on sing-box (105) | `https://raw.githubusercontent.com/onlyactions/proxy-scanner/main/subs/by-engine/allsingbox.txt` |
| [`by-country/`](subs/by-country/) | one list per exit country (53 countries, e.g. [`US.txt`](subs/by-country/US.txt), [`DE.txt`](subs/by-country/DE.txt)) | ...`/subs/by-country/<CC>.txt` |
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

The typical aggregator relists whatever it scraped, so 80-95% of entries are dead within hours — this run found **77% unreachable** and **78% unusable** overall. Only tunnels that completed a real HTTPS handshake, returned a verifiable foreign exit IP, and pushed a 1 MB download are counted as working here.

## Disclaimers

- Free proxies are **public and untrusted**. Anything you send through them can be observed or modified by their operators. Never use them for authentication, payments, or anything sensitive.
- All configs are collected from publicly posted sources (GitHub subscriptions and public Telegram channels); no credentials are guessed or brute-forced. Removal requests: open an issue.
- Availability is transient: the working list is re-verified daily, and past performance never guarantees future availability.

