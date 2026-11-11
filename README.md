# Free Proxy Scanner

> **Really tested** free proxy aggregator. Every proxy below was fetched from public sources, tunneled through with Xray/sing-box, and verified to actually change your egress IP. No CSV-validation theater, no blind relisting.

![working](badges/working.json) ![alive](badges/alive.json) ![speed](badges/speed.json) ![max-speed](badges/max-speed.json) ![countries](badges/countries.json) ![sources](badges/sources.json) ![updated](badges/updated.json)

## What this is

A GitHub Actions pipeline (runs once a day) that fetches free proxy configs from **173 URL subscriptions** and **81 Telegram channels**, parses 9 protocols (vless / vmess / trojan / ss / hysteria2 / hysteria / tuic / socks / http), deduplicates them, and **actually connects through every single one** to separate working proxies from dead ones.

Latest run: **2026-11-11 02:56:41 UTC** — 176751 unique proxies collected from 173/173 URL sources and 81/81 Telegram channels; **4521** endpoints really tested, **1141** reachable, **1102** verified working (egress IP confirmed changed) across **52 exit countries**.

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

All 1102 working proxies are published in [`subs/all.txt`](subs/all.txt); this table shows the top 25.

| # | proxy | proto | server | exit | latency | speed | engine |
|---|-------|-------|--------|------|---------|-------|--------|
| 1 | #1 ❓ ?? → 🇺🇸 US | `trojan` | `216.152.155.54:443` | United States | 192 ms | 108.1 Mbps | xray |
| 2 | #2 ❓ ?? → 🇺🇸 US | `trojan` | `216.152.155.59:443` | United States | 23 ms | 89.2 Mbps | singbox |
| 3 | #3 ❓ ?? → 🇺🇸 US | `ss` | `216.106.178.169:19999` | United States | 118 ms | 82.2 Mbps | xray |
| 4 | #4 ❓ ?? → 🇺🇸 US | `ss` | `38.64.138.135:19999` | United States | 199 ms | 78.6 Mbps | xray |
| 5 | #5 ❓ ?? → 🇺🇸 US | `trojan` | `216.152.155.158:443` | United States | 90 ms | 76.6 Mbps | xray |
| 6 | #6 ❓ ?? → 🇺🇸 US | `vmess` | `38.64.138.187:1270` | United States | 94 ms | 74.7 Mbps | xray |
| 7 | #7 ❓ ?? → 🇺🇸 US | `vmess` | `104.21.7.133:80` | United States | 2010 ms | 69.1 Mbps | xray |
| 8 | #8 ❓ ?? → 🇺🇸 US | `vmess` | `162.251.60.97:1270` | United States | 94 ms | 60.8 Mbps | xray |
| 9 | #9 🇨🇦 CA → 🇺🇸 US | `vmess` | `188.114.97.6:80` | United States | 2765 ms | 57.3 Mbps | xray |
| 10 | #10 ❓ ?? → 🇺🇸 US | `trojan` | `170.101.101.19:443` | United States | 275 ms | 55.2 Mbps | xray |
| 11 | #11 ❓ ?? → 🇺🇸 US | `vmess` | `104.17.147.222:80` | United States | 3653 ms | 54.4 Mbps | xray |
| 12 | #12 ❓ ?? → 🇺🇸 US | `ss` | `38.128.66.32:19999` | United States | 45 ms | 53.4 Mbps | xray |
| 13 | #13 ❓ ?? → 🇨🇦 CA | `trojan` | `216.152.147.152:443` | Canada | 234 ms | 51.9 Mbps | xray |
| 14 | #14 ❓ ?? → 🇺🇸 US | `trojan` | `216.152.155.23:443` | United States | 29 ms | 51.5 Mbps | xray |
| 15 | #15 ❓ ?? → 🇺🇸 US | `vmess` | `38.128.66.212:1270` | United States | 31 ms | 51.2 Mbps | xray |
| 16 | #16 ❓ ?? → 🇺🇸 US | `trojan` | `170.101.101.53:443` | United States | 110 ms | 51.0 Mbps | xray |
| 17 | #17 ❓ ?? → 🇺🇸 US | `vless` | `216.152.155.189:443` | United States | 47 ms | 46.8 Mbps | xray |
| 18 | #18 ❓ ?? → 🇺🇸 US | `vless` | `108.61.119.63:18580` | United States | 33 ms | 46.1 Mbps | xray |
| 19 | #19 ❓ ?? → 🇺🇸 US | `vmess` | `162.251.61.46:1270` | United States | 49 ms | 44.9 Mbps | xray |
| 20 | #20 ❓ ?? → 🇨🇦 CA | `vmess` | `134.195.196.154:1270` | Canada | 87 ms | 44.6 Mbps | xray |
| 21 | #21 ❓ ?? → 🇺🇸 US | `vmess` | `38.64.138.135:1270` | United States | 57 ms | 42.7 Mbps | xray |
| 22 | #22 ❓ ?? → 🇺🇸 US | `vmess` | `38.128.66.249:1270` | United States | 38 ms | 42.7 Mbps | xray |
| 23 | #23 ❓ ?? → 🇺🇸 US | `ss` | `185.230.230.18:19999` | United States | 138 ms | 39.9 Mbps | xray |
| 24 | #24 🇺🇸 US → 🇺🇸 US | `vless` | `67.220.73.204:18662` | United States | 191 ms | 39.7 Mbps | xray |
| 25 | #25 ❓ ?? → 🇺🇸 US | `vmess` | `38.128.66.241:1270` | United States | 77 ms | 39.5 Mbps | xray |

## Exit locations — which countries work

Every working proxy, grouped by the country of its **verified exit IP** (phase 2), with a dedicated subscription file per country. Currently 52 countries have at least one working exit.

| country | working | avg speed | best latency | share | list |
|---------|--------:|----------:|-------------:|------:|------|
| 🇺🇸 United States | 305 | 20.1 Mbps | 23 ms | 28% | [`US.txt`](subs/by-country/US.txt) |
| 🇳🇱 Netherlands | 171 | 7.2 Mbps | 316 ms | 16% | [`NL.txt`](subs/by-country/NL.txt) |
| 🇩🇪 Germany | 96 | 6.1 Mbps | 318 ms | 9% | [`DE.txt`](subs/by-country/DE.txt) |
| 🇯🇵 Japan | 70 | 1.4 Mbps | 477 ms | 6% | [`JP.txt`](subs/by-country/JP.txt) |
| 🇨🇦 Canada | 38 | 14.8 Mbps | 82 ms | 3% | [`CA.txt`](subs/by-country/CA.txt) |
| 🇸🇬 Singapore | 38 | 3.6 Mbps | 621 ms | 3% | [`SG.txt`](subs/by-country/SG.txt) |
| 🇫🇷 France | 37 | 6.7 Mbps | 326 ms | 3% | [`FR.txt`](subs/by-country/FR.txt) |
| 🇬🇧 United Kingdom | 30 | 7.7 Mbps | 293 ms | 3% | [`GB.txt`](subs/by-country/GB.txt) |
| 🇵🇱 Poland | 27 | 4.6 Mbps | 482 ms | 2% | [`PL.txt`](subs/by-country/PL.txt) |
| 🇪🇸 Spain | 24 | 6.3 Mbps | 393 ms | 2% | [`ES.txt`](subs/by-country/ES.txt) |
| 🇨🇭 Switzerland | 21 | 6.0 Mbps | 370 ms | 2% | [`CH.txt`](subs/by-country/CH.txt) |
| 🇸🇪 Sweden | 19 | 5.3 Mbps | 382 ms | 2% | [`SE.txt`](subs/by-country/SE.txt) |
| 🇮🇹 Italy | 17 | 7.4 Mbps | 375 ms | 2% | [`IT.txt`](subs/by-country/IT.txt) |
| 🇦🇺 Australia | 15 | 12.5 Mbps | 72 ms | 1% | [`AU.txt`](subs/by-country/AU.txt) |
| 🇭🇰 Hong Kong | 15 | 4.1 Mbps | 602 ms | 1% | [`HK.txt`](subs/by-country/HK.txt) |
| 🇫🇮 Finland | 15 | 4.0 Mbps | 426 ms | 1% | [`FI.txt`](subs/by-country/FI.txt) |
| 🇰🇷 South Korea | 12 | 3.3 Mbps | 588 ms | 1% | [`KR.txt`](subs/by-country/KR.txt) |
| 🇧🇬 Bulgaria | 11 | 5.3 Mbps | 617 ms | 1% | [`BG.txt`](subs/by-country/BG.txt) |
| 🇧🇾 BY | 9 | 5.7 Mbps | 487 ms | 1% | [`BY.txt`](subs/by-country/BY.txt) |
| 🇲🇩 Moldova | 9 | 4.2 Mbps | 889 ms | 1% | [`MD.txt`](subs/by-country/MD.txt) |
| 🇮🇳 India | 9 | 3.1 Mbps | 802 ms | 1% | [`IN.txt`](subs/by-country/IN.txt) |
| 🇱🇻 Latvia | 8 | 6.0 Mbps | 515 ms | 1% | [`LV.txt`](subs/by-country/LV.txt) |
| 🇷🇸 Serbia | 8 | 4.9 Mbps | 705 ms | 1% | [`RS.txt`](subs/by-country/RS.txt) |
| 🇪🇪 Estonia | 7 | 5.3 Mbps | 411 ms | 1% | [`EE.txt`](subs/by-country/EE.txt) |
| 🇰🇿 Kazakhstan | 7 | 2.5 Mbps | 1078 ms | 1% | [`KZ.txt`](subs/by-country/KZ.txt) |
| 🇦🇹 Austria | 6 | 6.7 Mbps | 416 ms | 1% | [`AT.txt`](subs/by-country/AT.txt) |
| 🇷🇺 Russia | 6 | 5.7 Mbps | 509 ms | 1% | [`RU.txt`](subs/by-country/RU.txt) |
| 🇮🇷 Iran | 6 | 4.9 Mbps | 815 ms | 1% | [`IR.txt`](subs/by-country/IR.txt) |
| 🇹🇷 Turkey | 6 | 4.4 Mbps | 695 ms | 1% | [`TR.txt`](subs/by-country/TR.txt) |
| 🇷🇴 Romania | 5 | 5.8 Mbps | 647 ms | 0% | [`RO.txt`](subs/by-country/RO.txt) |
| 🇹🇼 Taiwan | 5 | 4.2 Mbps | 580 ms | 0% | [`TW.txt`](subs/by-country/TW.txt) |
| 🇮🇪 Ireland | 4 | 5.8 Mbps | 248 ms | 0% | [`IE.txt`](subs/by-country/IE.txt) |
| 🇦🇱 AL | 4 | 6.5 Mbps | 484 ms | 0% | [`AL.txt`](subs/by-country/AL.txt) |
| 🇨🇿 Czechia | 4 | 4.9 Mbps | 508 ms | 0% | [`CZ.txt`](subs/by-country/CZ.txt) |
| 🇦🇪 UAE | 4 | 3.7 Mbps | 838 ms | 0% | [`AE.txt`](subs/by-country/AE.txt) |
| 🇹🇭 Thailand | 4 | 2.2 Mbps | 816 ms | 0% | [`TH.txt`](subs/by-country/TH.txt) |
| 🇲🇽 Mexico | 3 | 11.9 Mbps | 239 ms | 0% | [`MX.txt`](subs/by-country/MX.txt) |
| 🇺🇦 Ukraine | 3 | 6.4 Mbps | 700 ms | 0% | [`UA.txt`](subs/by-country/UA.txt) |
| 🇿🇦 South Africa | 3 | 3.4 Mbps | 900 ms | 0% | [`ZA.txt`](subs/by-country/ZA.txt) |
| 🇻🇳 Vietnam | 3 | 2.9 Mbps | 1035 ms | 0% | [`VN.txt`](subs/by-country/VN.txt) |
| 🇬🇷 Greece | 3 | 2.7 Mbps | 653 ms | 0% | [`GR.txt`](subs/by-country/GR.txt) |
| 🇨🇱 Chile | 2 | 6.6 Mbps | 502 ms | 0% | [`CL.txt`](subs/by-country/CL.txt) |
| 🇧🇷 Brazil | 2 | 6.2 Mbps | 618 ms | 0% | [`BR.txt`](subs/by-country/BR.txt) |
| 🇮🇩 Indonesia | 2 | 3.8 Mbps | 946 ms | 0% | [`ID.txt`](subs/by-country/ID.txt) |
| 🇱🇰 LK | 2 | 1.7 Mbps | 1400 ms | 0% | [`LK.txt`](subs/by-country/LK.txt) |
| 🇧🇪 Belgium | 1 | 7.0 Mbps | 587 ms | 0% | [`BE.txt`](subs/by-country/BE.txt) |
| 🇮🇱 Israel | 1 | 6.2 Mbps | 601 ms | 0% | [`IL.txt`](subs/by-country/IL.txt) |
| 🇱🇹 Lithuania | 1 | 5.3 Mbps | 715 ms | 0% | [`LT.txt`](subs/by-country/LT.txt) |
| 🇭🇺 Hungary | 1 | 4.5 Mbps | 842 ms | 0% | [`HU.txt`](subs/by-country/HU.txt) |
| 🇸🇦 Saudi Arabia | 1 | 4.4 Mbps | 981 ms | 0% | [`SA.txt`](subs/by-country/SA.txt) |
| 🇳🇴 Norway | 1 | 3.2 Mbps | 409 ms | 0% | [`NO.txt`](subs/by-country/NO.txt) |
| 🇩🇰 Denmark | 1 | 2.9 Mbps | 663 ms | 0% | [`DK.txt`](subs/by-country/DK.txt) |

A `??` exit means the geo lookup could not resolve the exit IP. Base64 variants of every country file sit next to them as `<CC>.base64.txt`.

## Protocol distribution

| protocol | tested | alive | working | alive rate |
|----------|-------:|------:|--------:|-----------:|
| `vless` | 2780 | 596 | 573 | 21% |
| `ss` | 602 | 238 | 231 | 40% |
| `vmess` | 855 | 145 | 137 | 17% |
| `trojan` | 200 | 114 | 114 | 57% |
| `hysteria2` | 75 | 44 | 44 | 59% |
| `socks` | 3 | 3 | 2 | 100% |
| `tuic` | 1 | 1 | 1 | 100% |
| `http` | 5 | 0 | 0 | 0% |

Per-protocol working lists: [`subs/by-proto/`](subs/by-proto/) (one `vless.txt`, `vmess.txt`, ... per protocol, plus `.base64.txt` variants).

## Engine breakdown

Which core verified each working proxy. The engine fallback means a proxy that failed on its primary core got a second chance on the other one.

| engine | working | note |
|--------|--------:|------|
| Xray-core | 981 | [`allxray.txt`](subs/by-engine/allxray.txt) |
| sing-box | 119 | [`allsingbox.txt`](subs/by-engine/allsingbox.txt) |
| direct (curl) | 2 | plain http/socks, [`alldirect.txt`](subs/by-engine/alldirect.txt) |

## Speed & latency profile

| speed tier | proxies |
|-----------|--------:|
| >= 5 Mbps | 679 |
| 1 - 5 Mbps | 273 |
| 0.25 - 1 Mbps | 36 |
| < 0.25 Mbps | 0 |
| unmeasured | 114 |

Latency (phase-1 HTTPS round trip): median **512 ms**, p90 **2208 ms**. 39 proxies passed connectivity but failed egress verification (leaked the runner IP or broke on the exit check) and are excluded.

## Source health

254 active, 0 auto-retired (10 fetch failures / 10 all-dead runs / 10 days without anything new). Best contributors this run:

| source | kind | tested | alive | new this run |
|--------|------|-------:|------:|-------------:|
| `NK` | url | 927 | 611 | 8007 |
| `EV` | url | 673 | 583 | 1246 |
| `SK` | url | 882 | 569 | 0 |
| `HP` | url | 498 | 468 | 117 |
| `RD-VL` | url | 2278 | 340 | 3493 |
| `AG-PL` | url | 450 | 277 | 912 |
| `KA` | url | 2970 | 271 | 8896 |
| `HC` | url | 1165 | 261 | 231 |
| `ET` | url | 383 | 260 | 1594 |
| `Ni` | url | 340 | 257 | 140 |
| `EV-VL` | url | 295 | 236 | 0 |
| `ST-VL` | url | 2136 | 232 | 685 |
| `RD-SS` | url | 523 | 228 | 351 |
| `F0` | url | 231 | 219 | 1 |
| `EV-SS` | url | 225 | 208 | 0 |
| `SB` | url | 2331 | 197 | 4 |
| `RD-VM` | url | 594 | 129 | 3579 |
| `EP-ALL` | url | 1325 | 124 | 0 |
| `LA` | url | 754 | 122 | 14308 |
| `ST-VM` | url | 591 | 114 | 27 |

## History

working per run (last 24): ▁▁▆▆▇█▆▆▇█▇▆▇▆▆▇█▇▇█████

| date (UTC) | tested | alive | working | avg | max |
|------------|-------:|------:|--------:|----:|----:|
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
| 2026-10-01 03:00 | 4500 | 1112 | 1075 | 8.1M | 45.4M |

## Subscription files

Everything under [`subs/`](subs/) is regenerated every run. Plain lists are one proxy URI per line; `.base64.txt` variants are the same list base64-encoded for clients that need the classic sub format.

| file | contents | direct subscription URL |
|------|----------|--------------------------|
| [`all.txt`](subs/all.txt) | ALL working proxies, ranked by speed (1102) | `https://raw.githubusercontent.com/onlyactions/proxy-scanner/main/subs/all.txt` |
| [`all.base64.txt`](subs/all.base64.txt) | same, base64 | `https://raw.githubusercontent.com/onlyactions/proxy-scanner/main/subs/all.base64.txt` |
| [`top150.txt`](subs/top150.txt) | top 150 ranked (150) | `https://raw.githubusercontent.com/onlyactions/proxy-scanner/main/subs/top150.txt` |
| [`top150.base64.txt`](subs/top150.base64.txt) | same, base64 | `https://raw.githubusercontent.com/onlyactions/proxy-scanner/main/subs/top150.base64.txt` |
| [`by-proto/`](subs/by-proto/) | one list per protocol (7 protocols) | ...`/subs/by-proto/<proto>.txt` |
| [`by-engine/allxray.txt`](subs/by-engine/allxray.txt) | verified on Xray (981) | `https://raw.githubusercontent.com/onlyactions/proxy-scanner/main/subs/by-engine/allxray.txt` |
| [`by-engine/allsingbox.txt`](subs/by-engine/allsingbox.txt) | verified on sing-box (119) | `https://raw.githubusercontent.com/onlyactions/proxy-scanner/main/subs/by-engine/allsingbox.txt` |
| [`by-country/`](subs/by-country/) | one list per exit country (52 countries, e.g. [`US.txt`](subs/by-country/US.txt), [`DE.txt`](subs/by-country/DE.txt)) | ...`/subs/by-country/<CC>.txt` |
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

The typical aggregator relists whatever it scraped, so 80-95% of entries are dead within hours — this run found **75% unreachable** and **76% unusable** overall. Only tunnels that completed a real HTTPS handshake, returned a verifiable foreign exit IP, and pushed a 1 MB download are counted as working here.

## Disclaimers

- Free proxies are **public and untrusted**. Anything you send through them can be observed or modified by their operators. Never use them for authentication, payments, or anything sensitive.
- All configs are collected from publicly posted sources (GitHub subscriptions and public Telegram channels); no credentials are guessed or brute-forced. Removal requests: open an issue.
- Availability is transient: the working list is re-verified daily, and past performance never guarantees future availability.

