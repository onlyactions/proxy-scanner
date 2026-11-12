# Free Proxy Scanner

> **Really tested** free proxy aggregator. Every proxy below was fetched from public sources, tunneled through with Xray/sing-box, and verified to actually change your egress IP. No CSV-validation theater, no blind relisting.

![working](badges/working.json) ![alive](badges/alive.json) ![speed](badges/speed.json) ![max-speed](badges/max-speed.json) ![countries](badges/countries.json) ![sources](badges/sources.json) ![updated](badges/updated.json)

## What this is

A GitHub Actions pipeline (runs once a day) that fetches free proxy configs from **173 URL subscriptions** and **81 Telegram channels**, parses 9 protocols (vless / vmess / trojan / ss / hysteria2 / hysteria / tuic / socks / http), deduplicates them, and **actually connects through every single one** to separate working proxies from dead ones.

Latest run: **2026-11-12 02:56:46 UTC** — 176641 unique proxies collected from 173/173 URL sources and 81/81 Telegram channels; **4512** endpoints really tested, **1209** reachable, **1166** verified working (egress IP confirmed changed) across **56 exit countries**.

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

All 1166 working proxies are published in [`subs/all.txt`](subs/all.txt); this table shows the top 25.

| # | proxy | proto | server | exit | latency | speed | engine |
|---|-------|-------|--------|------|---------|-------|--------|
| 1 | #1 ❓ ?? → 🇺🇸 US | `trojan` | `resources.findlocaljobnetworkandbuildcommunityconnections.com:443` | United States | 192 ms | 73.5 Mbps | xray |
| 2 | #2 🇺🇸 US → 🇦🇺 AU | `vmess` | `84.17.41.2:18000` | Australia | 86 ms | 65.1 Mbps | xray |
| 3 | #3 ❓ ?? → 🇺🇸 US | `trojan` | `www.findlocaljobnetworkandbuildcommunityconnections.com:443` | United States | 81 ms | 57.5 Mbps | xray |
| 4 | #4 ❓ ?? → 🇺🇸 US | `trojan` | `216.106.186.155:443` | United States | 61 ms | 56.3 Mbps | xray |
| 5 | #5 ❓ ?? → 🇺🇸 US | `ss` | `38.143.66.146:19999` | United States | 30 ms | 55.8 Mbps | xray |
| 6 | #6 ❓ ?? → 🇺🇸 US | `trojan` | `216.106.186.32:443` | United States | 97 ms | 50.7 Mbps | xray |
| 7 | #7 ❓ ?? → 🇺🇸 US | `trojan` | `blog.findlocaljobnetworkandbuildcommunityconnections.com:443` | United States | 101 ms | 48.5 Mbps | xray |
| 8 | #8 🇺🇸 US → 🇺🇸 US | `vless` | `15.204.97.206:23576` | United States | 113 ms | 48.4 Mbps | xray |
| 9 | #9 ❓ ?? → 🇺🇸 US | `trojan` | `216.106.186.82:443` | United States | 45 ms | 48.3 Mbps | xray |
| 10 | #10 🇺🇸 US → 🇺🇸 US | `vless` | `15.204.97.219:23576` | United States | 69 ms | 47.0 Mbps | xray |
| 11 | #11 🇺🇸 US → 🇺🇸 US | `vless` | `15.204.97.197:23576` | United States | 74 ms | 45.8 Mbps | xray |
| 12 | #12 🇺🇸 US → 🇺🇸 US | `vless` | `15.204.97.209:23576` | United States | 91 ms | 45.4 Mbps | xray |
| 13 | #13 🇺🇸 US → 🇺🇸 US | `vless` | `15.204.97.216:23576` | United States | 72 ms | 44.6 Mbps | xray |
| 14 | #14 🇺🇸 US → 🇺🇸 US | `vless` | `15.204.97.195:23576` | United States | 61 ms | 42.7 Mbps | xray |
| 15 | #15 🇺🇸 US → 🇺🇸 US | `vless` | `15.204.97.198:23576` | United States | 58 ms | 42.5 Mbps | xray |
| 16 | #16 🇺🇸 US → 🇺🇸 US | `vless` | `15.204.97.214:23576` | United States | 68 ms | 41.8 Mbps | xray |
| 17 | #17 ❓ ?? → 🇺🇸 US | `hysteria2` | `47.89.232.46:8443` | United States | 132 ms | 29.1 Mbps | singbox |
| 18 | #18 ❓ ?? → 🇺🇸 US | `vless` | `198.2.251.107:45197` | United States | 188 ms | 28.8 Mbps | xray |
| 19 | #19 ❓ ?? → 🇺🇸 US | `trojan` | `167.88.61.85:443` | United States | 128 ms | 27.8 Mbps | xray |
| 20 | #20 🇺🇸 US → 🇦🇺 AU | `vmess` | `169.197.142.22:18000` | Australia | 3565 ms | 27.5 Mbps | xray |
| 21 | #21 ❓ ?? → 🇺🇸 US | `trojan` | `66.163.114.166:443` | United States | 229 ms | 27.3 Mbps | xray |
| 22 | #22 ❓ ?? → 🇺🇸 US | `ss` | `137.184.89.151:19999` | United States | 101 ms | 26.9 Mbps | xray |
| 23 | #23 🇺🇸 US → 🇺🇸 US | `trojan` | `167.88.61.175:443` | United States | 120 ms | 26.8 Mbps | xray |
| 24 | #24 ❓ ?? → 🇺🇸 US | `trojan` | `66.163.114.106:443` | United States | 148 ms | 26.6 Mbps | xray |
| 25 | #25 ❓ ?? → 🇺🇸 US | `trojan` | `167.88.61.198:443` | United States | 129 ms | 26.2 Mbps | xray |

## Exit locations — which countries work

Every working proxy, grouped by the country of its **verified exit IP** (phase 2), with a dedicated subscription file per country. Currently 56 countries have at least one working exit.

| country | working | avg speed | best latency | share | list |
|---------|--------:|----------:|-------------:|------:|------|
| 🇺🇸 United States | 314 | 13.6 Mbps | 30 ms | 27% | [`US.txt`](subs/by-country/US.txt) |
| 🇳🇱 Netherlands | 166 | 5.1 Mbps | 429 ms | 14% | [`NL.txt`](subs/by-country/NL.txt) |
| 🇩🇪 Germany | 109 | 4.7 Mbps | 439 ms | 9% | [`DE.txt`](subs/by-country/DE.txt) |
| 🇯🇵 Japan | 78 | 2.0 Mbps | 347 ms | 7% | [`JP.txt`](subs/by-country/JP.txt) |
| 🇨🇦 Canada | 39 | 6.9 Mbps | 221 ms | 3% | [`CA.txt`](subs/by-country/CA.txt) |
| 🇫🇷 France | 38 | 5.0 Mbps | 442 ms | 3% | [`FR.txt`](subs/by-country/FR.txt) |
| 🇸🇬 Singapore | 34 | 3.8 Mbps | 490 ms | 3% | [`SG.txt`](subs/by-country/SG.txt) |
| 🇬🇧 United Kingdom | 32 | 5.3 Mbps | 394 ms | 3% | [`GB.txt`](subs/by-country/GB.txt) |
| 🇵🇱 Poland | 28 | 4.3 Mbps | 645 ms | 2% | [`PL.txt`](subs/by-country/PL.txt) |
| 🇪🇸 Spain | 26 | 4.2 Mbps | 518 ms | 2% | [`ES.txt`](subs/by-country/ES.txt) |
| 🇫🇮 Finland | 23 | 4.2 Mbps | 500 ms | 2% | [`FI.txt`](subs/by-country/FI.txt) |
| 🇭🇰 Hong Kong | 21 | 3.6 Mbps | 147 ms | 2% | [`HK.txt`](subs/by-country/HK.txt) |
| 🇮🇹 Italy | 19 | 5.2 Mbps | 486 ms | 2% | [`IT.txt`](subs/by-country/IT.txt) |
| 🇦🇺 Australia | 17 | 13.1 Mbps | 86 ms | 1% | [`AU.txt`](subs/by-country/AU.txt) |
| 🇸🇪 Sweden | 17 | 4.0 Mbps | 507 ms | 1% | [`SE.txt`](subs/by-country/SE.txt) |
| 🇰🇷 South Korea | 16 | 5.0 Mbps | 509 ms | 1% | [`KR.txt`](subs/by-country/KR.txt) |
| 🇧🇾 BY | 15 | 4.3 Mbps | 625 ms | 1% | [`BY.txt`](subs/by-country/BY.txt) |
| 🇨🇭 Switzerland | 14 | 5.1 Mbps | 493 ms | 1% | [`CH.txt`](subs/by-country/CH.txt) |
| 🇮🇳 India | 14 | 2.7 Mbps | 865 ms | 1% | [`IN.txt`](subs/by-country/IN.txt) |
| 🇧🇬 Bulgaria | 13 | 3.9 Mbps | 746 ms | 1% | [`BG.txt`](subs/by-country/BG.txt) |
| 🇹🇼 Taiwan | 9 | 3.5 Mbps | 454 ms | 1% | [`TW.txt`](subs/by-country/TW.txt) |
| 🇲🇩 Moldova | 9 | 3.3 Mbps | 1040 ms | 1% | [`MD.txt`](subs/by-country/MD.txt) |
| 🇦🇱 AL | 8 | 3.8 Mbps | 634 ms | 1% | [`AL.txt`](subs/by-country/AL.txt) |
| 🇷🇸 Serbia | 8 | 3.6 Mbps | 918 ms | 1% | [`RS.txt`](subs/by-country/RS.txt) |
| 🇱🇻 Latvia | 7 | 4.8 Mbps | 730 ms | 1% | [`LV.txt`](subs/by-country/LV.txt) |
| 🇷🇺 Russia | 7 | 4.2 Mbps | 663 ms | 1% | [`RU.txt`](subs/by-country/RU.txt) |
| 🇰🇿 Kazakhstan | 7 | 2.7 Mbps | 1229 ms | 1% | [`KZ.txt`](subs/by-country/KZ.txt) |
| 🇦🇹 Austria | 6 | 4.9 Mbps | 530 ms | 1% | [`AT.txt`](subs/by-country/AT.txt) |
| 🇮🇷 Iran | 6 | 3.7 Mbps | 778 ms | 1% | [`IR.txt`](subs/by-country/IR.txt) |
| 🇹🇷 Turkey | 6 | 3.9 Mbps | 893 ms | 1% | [`TR.txt`](subs/by-country/TR.txt) |
| 🇪🇪 Estonia | 5 | 3.7 Mbps | 517 ms | 0% | [`EE.txt`](subs/by-country/EE.txt) |
| 🇷🇴 Romania | 5 | 4.4 Mbps | 831 ms | 0% | [`RO.txt`](subs/by-country/RO.txt) |
| 🇨🇿 Czechia | 4 | 4.7 Mbps | 655 ms | 0% | [`CZ.txt`](subs/by-country/CZ.txt) |
| 🇬🇷 Greece | 4 | 3.4 Mbps | 1079 ms | 0% | [`GR.txt`](subs/by-country/GR.txt) |
| 🇿🇦 South Africa | 4 | 2.6 Mbps | 986 ms | 0% | [`ZA.txt`](subs/by-country/ZA.txt) |
| 🇦🇪 UAE | 4 | 1.9 Mbps | 1000 ms | 0% | [`AE.txt`](subs/by-country/AE.txt) |
| 🇲🇽 Mexico | 3 | 10.1 Mbps | 276 ms | 0% | [`MX.txt`](subs/by-country/MX.txt) |
| 🇧🇷 Brazil | 3 | 4.7 Mbps | 775 ms | 0% | [`BR.txt`](subs/by-country/BR.txt) |
| 🇹🇭 Thailand | 3 | 3.6 Mbps | 655 ms | 0% | [`TH.txt`](subs/by-country/TH.txt) |
| 🇺🇦 Ukraine | 3 | 4.4 Mbps | 897 ms | 0% | [`UA.txt`](subs/by-country/UA.txt) |
| 🇧🇪 Belgium | 2 | 5.5 Mbps | 586 ms | 0% | [`BE.txt`](subs/by-country/BE.txt) |
| 🇮🇩 Indonesia | 2 | 4.6 Mbps | 767 ms | 0% | [`ID.txt`](subs/by-country/ID.txt) |
| 🇨🇱 Chile | 2 | 4.4 Mbps | 638 ms | 0% | [`CL.txt`](subs/by-country/CL.txt) |
| 🇱🇰 LK | 2 | 3.9 Mbps | 728 ms | 0% | [`LK.txt`](subs/by-country/LK.txt) |
| 🇻🇳 Vietnam | 2 | 2.2 Mbps | 900 ms | 0% | [`VN.txt`](subs/by-country/VN.txt) |
| 🇲🇾 Malaysia | 2 | - | 1907 ms | 0% | [`MY.txt`](subs/by-country/MY.txt) |
| 🇮🇪 Ireland | 1 | 5.7 Mbps | 379 ms | 0% | [`IE.txt`](subs/by-country/IE.txt) |
| 🇳🇴 Norway | 1 | 5.1 Mbps | 509 ms | 0% | [`NO.txt`](subs/by-country/NO.txt) |
| 🇩🇰 Denmark | 1 | 4.5 Mbps | 884 ms | 0% | [`DK.txt`](subs/by-country/DK.txt) |
| 🇸🇦 Saudi Arabia | 1 | 4.4 Mbps | 1055 ms | 0% | [`SA.txt`](subs/by-country/SA.txt) |
| 🇱🇹 Lithuania | 1 | 2.2 Mbps | 975 ms | 0% | [`LT.txt`](subs/by-country/LT.txt) |
| 🇵🇰 PK | 1 | 1.4 Mbps | 2243 ms | 0% | [`PK.txt`](subs/by-country/PK.txt) |
| 🇭🇺 Hungary | 1 | 1.0 Mbps | 6715 ms | 0% | [`HU.txt`](subs/by-country/HU.txt) |
| 🇵🇪 PE | 1 | 0.9 Mbps | 1253 ms | 0% | [`PE.txt`](subs/by-country/PE.txt) |
| 🇨🇳 China | 1 | - | 1127 ms | 0% | [`CN.txt`](subs/by-country/CN.txt) |
| 🇳🇬 NG | 1 | - | 1377 ms | 0% | [`NG.txt`](subs/by-country/NG.txt) |

A `??` exit means the geo lookup could not resolve the exit IP. Base64 variants of every country file sit next to them as `<CC>.base64.txt`.

## Protocol distribution

| protocol | tested | alive | working | alive rate |
|----------|-------:|------:|--------:|-----------:|
| `vless` | 2761 | 662 | 629 | 24% |
| `ss` | 641 | 235 | 234 | 37% |
| `vmess` | 842 | 154 | 147 | 18% |
| `trojan` | 186 | 113 | 111 | 61% |
| `hysteria2` | 73 | 44 | 44 | 60% |
| `socks` | 3 | 1 | 1 | 33% |
| `tuic` | 1 | 0 | 0 | 0% |
| `http` | 5 | 0 | 0 | 0% |

Per-protocol working lists: [`subs/by-proto/`](subs/by-proto/) (one `vless.txt`, `vmess.txt`, ... per protocol, plus `.base64.txt` variants).

## Engine breakdown

Which core verified each working proxy. The engine fallback means a proxy that failed on its primary core got a second chance on the other one.

| engine | working | note |
|--------|--------:|------|
| Xray-core | 1051 | [`allxray.txt`](subs/by-engine/allxray.txt) |
| sing-box | 114 | [`allsingbox.txt`](subs/by-engine/allsingbox.txt) |
| direct (curl) | 1 | plain http/socks, [`alldirect.txt`](subs/by-engine/alldirect.txt) |

## Speed & latency profile

| speed tier | proxies |
|-----------|--------:|
| >= 5 Mbps | 580 |
| 1 - 5 Mbps | 411 |
| 0.25 - 1 Mbps | 49 |
| < 0.25 Mbps | 0 |
| unmeasured | 126 |

Latency (phase-1 HTTPS round trip): median **660 ms**, p90 **2043 ms**. 43 proxies passed connectivity but failed egress verification (leaked the runner IP or broke on the exit check) and are excluded.

## Source health

254 active, 0 auto-retired (10 fetch failures / 10 all-dead runs / 10 days without anything new). Best contributors this run:

| source | kind | tested | alive | new this run |
|--------|------|-------:|------:|-------------:|
| `NK` | url | 1082 | 741 | 7954 |
| `SK` | url | 1038 | 697 | 0 |
| `EV` | url | 777 | 679 | 454 |
| `HP` | url | 423 | 393 | 77 |
| `RD-VL` | url | 2276 | 385 | 3530 |
| `HC` | url | 1246 | 358 | 146 |
| `AG-PL` | url | 457 | 312 | 822 |
| `EV-VL` | url | 377 | 302 | 886 |
| `KA` | url | 2869 | 289 | 10754 |
| `Ni` | url | 362 | 269 | 93 |
| `ST-VL` | url | 2099 | 241 | 642 |
| `F0` | url | 238 | 235 | 4 |
| `RD-SS` | url | 530 | 232 | 359 |
| `EV-SS` | url | 228 | 215 | 0 |
| `ET` | url | 350 | 210 | 1506 |
| `SB` | url | 2176 | 144 | 1 |
| `RD-VM` | url | 597 | 140 | 3633 |
| `SB-VL` | url | 1908 | 136 | 2 |
| `EP-ALL` | url | 1325 | 124 | 0 |
| `LA` | url | 754 | 123 | 14301 |

## History

working per run (last 25): ▁▁▆▆▆▇▆▆▆█▇▆▇▆▆▆█▆▆▇█████

| date (UTC) | tested | alive | working | avg | max |
|------------|-------:|------:|--------:|----:|----:|
| 2026-11-12 03:35 | 4512 | 1209 | 1166 | 7.0M | 73.5M |
| 2026-11-11 03:35 | 4521 | 1141 | 1102 | 10.1M | 108.1M |
| 2026-11-10 03:31 | 4511 | 1098 | 1052 | 8.2M | 76.8M |
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

## Subscription files

Everything under [`subs/`](subs/) is regenerated every run. Plain lists are one proxy URI per line; `.base64.txt` variants are the same list base64-encoded for clients that need the classic sub format.

| file | contents | direct subscription URL |
|------|----------|--------------------------|
| [`all.txt`](subs/all.txt) | ALL working proxies, ranked by speed (1166) | `https://raw.githubusercontent.com/onlyactions/proxy-scanner/main/subs/all.txt` |
| [`all.base64.txt`](subs/all.base64.txt) | same, base64 | `https://raw.githubusercontent.com/onlyactions/proxy-scanner/main/subs/all.base64.txt` |
| [`top150.txt`](subs/top150.txt) | top 150 ranked (150) | `https://raw.githubusercontent.com/onlyactions/proxy-scanner/main/subs/top150.txt` |
| [`top150.base64.txt`](subs/top150.base64.txt) | same, base64 | `https://raw.githubusercontent.com/onlyactions/proxy-scanner/main/subs/top150.base64.txt` |
| [`by-proto/`](subs/by-proto/) | one list per protocol (6 protocols) | ...`/subs/by-proto/<proto>.txt` |
| [`by-engine/allxray.txt`](subs/by-engine/allxray.txt) | verified on Xray (1051) | `https://raw.githubusercontent.com/onlyactions/proxy-scanner/main/subs/by-engine/allxray.txt` |
| [`by-engine/allsingbox.txt`](subs/by-engine/allsingbox.txt) | verified on sing-box (114) | `https://raw.githubusercontent.com/onlyactions/proxy-scanner/main/subs/by-engine/allsingbox.txt` |
| [`by-country/`](subs/by-country/) | one list per exit country (56 countries, e.g. [`US.txt`](subs/by-country/US.txt), [`DE.txt`](subs/by-country/DE.txt)) | ...`/subs/by-country/<CC>.txt` |
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

The typical aggregator relists whatever it scraped, so 80-95% of entries are dead within hours — this run found **73% unreachable** and **74% unusable** overall. Only tunnels that completed a real HTTPS handshake, returned a verifiable foreign exit IP, and pushed a 1 MB download are counted as working here.

## Disclaimers

- Free proxies are **public and untrusted**. Anything you send through them can be observed or modified by their operators. Never use them for authentication, payments, or anything sensitive.
- All configs are collected from publicly posted sources (GitHub subscriptions and public Telegram channels); no credentials are guessed or brute-forced. Removal requests: open an issue.
- Availability is transient: the working list is re-verified daily, and past performance never guarantees future availability.

