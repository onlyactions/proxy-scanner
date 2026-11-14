# Free Proxy Scanner

> **Really tested** free proxy aggregator. Every proxy below was fetched from public sources, tunneled through with Xray/sing-box, and verified to actually change your egress IP. No CSV-validation theater, no blind relisting.

![working](badges/working.json) ![alive](badges/alive.json) ![speed](badges/speed.json) ![max-speed](badges/max-speed.json) ![countries](badges/countries.json) ![sources](badges/sources.json) ![updated](badges/updated.json)

## What this is

A GitHub Actions pipeline (runs once a day) that fetches free proxy configs from **173 URL subscriptions** and **81 Telegram channels**, parses 9 protocols (vless / vmess / trojan / ss / hysteria2 / hysteria / tuic / socks / http), deduplicates them, and **actually connects through every single one** to separate working proxies from dead ones.

Latest run: **2026-11-14 02:56:30 UTC** — 175340 unique proxies collected from 173/173 URL sources and 81/81 Telegram channels; **4514** endpoints really tested, **1104** reachable, **1067** verified working (egress IP confirmed changed) across **47 exit countries**.

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

All 1067 working proxies are published in [`subs/all.txt`](subs/all.txt); this table shows the top 25.

| # | proxy | proto | server | exit | latency | speed | engine |
|---|-------|-------|--------|------|---------|-------|--------|
| 1 | #1 ❓ ?? → 🇺🇸 US | `vless` | `216.152.155.189:443` | United States | 99 ms | 135.5 Mbps | xray |
| 2 | #2 ❓ ?? → 🇺🇸 US | `trojan` | `216.152.155.54:443` | United States | 128 ms | 115.3 Mbps | xray |
| 3 | #3 ❓ ?? → 🇺🇸 US | `trojan` | `216.152.155.50:443` | United States | 143 ms | 104.7 Mbps | xray |
| 4 | #4 ❓ ?? → 🇺🇸 US | `vmess` | `38.64.138.135:1270` | United States | 96 ms | 98.4 Mbps | xray |
| 5 | #5 ❓ ?? → 🇺🇸 US | `trojan` | `216.152.155.131:443` | United States | 130 ms | 89.5 Mbps | xray |
| 6 | #6 ❓ ?? → 🇺🇸 US | `trojan` | `216.152.155.158:443` | United States | 101 ms | 87.5 Mbps | xray |
| 7 | #7 ❓ ?? → 🇺🇸 US | `ss` | `38.64.138.135:19999` | United States | 49 ms | 75.3 Mbps | xray |
| 8 | #8 ❓ ?? → 🇺🇸 US | `vmess` | `38.128.66.212:1270` | United States | 52 ms | 70.1 Mbps | xray |
| 9 | #9 ❓ ?? → 🇺🇸 US | `trojan` | `216.152.155.23:443` | United States | 39 ms | 69.1 Mbps | xray |
| 10 | #10 ❓ ?? → 🇺🇸 US | `ss` | `216.106.178.169:19999` | United States | 71 ms | 65.8 Mbps | xray |
| 11 | #11 ❓ ?? → 🇺🇸 US | `ss` | `38.128.66.32:19999` | United States | 64 ms | 63.9 Mbps | xray |
| 12 | #12 ❓ ?? → 🇺🇸 US | `ss` | `67.220.73.178:19999` | United States | 54 ms | 62.6 Mbps | xray |
| 13 | #13 ❓ ?? → 🇺🇸 US | `vmess` | `38.128.66.241:1270` | United States | 47 ms | 62.4 Mbps | xray |
| 14 | #14 ❓ ?? → 🇺🇸 US | `vmess` | `162.251.60.97:1270` | United States | 57 ms | 55.0 Mbps | xray |
| 15 | #15 ❓ ?? → 🇺🇸 US | `vmess` | `38.64.138.187:1270` | United States | 42 ms | 55.0 Mbps | xray |
| 16 | #16 ❓ ?? → 🇺🇸 US | `trojan` | `170.101.101.67:443` | United States | 94 ms | 53.7 Mbps | xray |
| 17 | #17 ❓ ?? → 🇺🇸 US | `trojan` | `170.101.101.211:443` | United States | 94 ms | 51.1 Mbps | xray |
| 18 | #18 ❓ ?? → 🇺🇸 US | `trojan` | `170.101.101.53:443` | United States | 162 ms | 51.1 Mbps | xray |
| 19 | #19 ❓ ?? → 🇺🇸 US | `trojan` | `216.152.155.59:443` | United States | 90 ms | 51.0 Mbps | singbox |
| 20 | #20 ❓ ?? → 🇨🇦 CA | `trojan` | `216.152.147.48:443` | Canada | 106 ms | 48.9 Mbps | xray |
| 21 | #21 ❓ ?? → 🇺🇸 US | `vmess` | `38.128.66.249:1270` | United States | 53 ms | 47.3 Mbps | xray |
| 22 | #22 🇺🇸 US → 🇺🇸 US | `vless` | `69.48.201.136:30016` | United States | 121 ms | 47.0 Mbps | singbox |
| 23 | #23 ❓ ?? → 🇨🇦 CA | `vmess` | `134.195.196.154:1270` | Canada | 129 ms | 46.8 Mbps | xray |
| 24 | #24 ❓ ?? → 🇺🇸 US | `trojan` | `170.101.101.19:443` | United States | 94 ms | 45.7 Mbps | xray |
| 25 | #25 ❓ ?? → 🇨🇦 CA | `trojan` | `216.152.147.152:443` | Canada | 181 ms | 45.5 Mbps | xray |

## Exit locations — which countries work

Every working proxy, grouped by the country of its **verified exit IP** (phase 2), with a dedicated subscription file per country. Currently 47 countries have at least one working exit.

| country | working | avg speed | best latency | share | list |
|---------|--------:|----------:|-------------:|------:|------|
| 🇺🇸 United States | 320 | 22.3 Mbps | 39 ms | 30% | [`US.txt`](subs/by-country/US.txt) |
| 🇳🇱 Netherlands | 143 | 7.0 Mbps | 310 ms | 13% | [`NL.txt`](subs/by-country/NL.txt) |
| 🇩🇪 Germany | 111 | 5.6 Mbps | 314 ms | 10% | [`DE.txt`](subs/by-country/DE.txt) |
| 🇯🇵 Japan | 80 | 2.3 Mbps | 449 ms | 7% | [`JP.txt`](subs/by-country/JP.txt) |
| 🇨🇦 Canada | 41 | 13.8 Mbps | 83 ms | 4% | [`CA.txt`](subs/by-country/CA.txt) |
| 🇫🇷 France | 41 | 6.8 Mbps | 327 ms | 4% | [`FR.txt`](subs/by-country/FR.txt) |
| 🇸🇬 Singapore | 32 | 3.1 Mbps | 620 ms | 3% | [`SG.txt`](subs/by-country/SG.txt) |
| 🇬🇧 United Kingdom | 31 | 6.8 Mbps | 295 ms | 3% | [`GB.txt`](subs/by-country/GB.txt) |
| 🇪🇸 Spain | 28 | 4.5 Mbps | 392 ms | 3% | [`ES.txt`](subs/by-country/ES.txt) |
| 🇫🇮 Finland | 19 | 4.6 Mbps | 364 ms | 2% | [`FI.txt`](subs/by-country/FI.txt) |
| 🇮🇹 Italy | 18 | 7.0 Mbps | 379 ms | 2% | [`IT.txt`](subs/by-country/IT.txt) |
| 🇦🇺 Australia | 15 | 10.9 Mbps | 100 ms | 1% | [`AU.txt`](subs/by-country/AU.txt) |
| 🇸🇪 Sweden | 15 | 5.8 Mbps | 378 ms | 1% | [`SE.txt`](subs/by-country/SE.txt) |
| 🇨🇭 Switzerland | 14 | 6.4 Mbps | 353 ms | 1% | [`CH.txt`](subs/by-country/CH.txt) |
| 🇵🇱 Poland | 13 | 5.2 Mbps | 523 ms | 1% | [`PL.txt`](subs/by-country/PL.txt) |
| 🇮🇳 India | 13 | 3.2 Mbps | 824 ms | 1% | [`IN.txt`](subs/by-country/IN.txt) |
| 🇧🇾 BY | 12 | 3.7 Mbps | 565 ms | 1% | [`BY.txt`](subs/by-country/BY.txt) |
| 🇰🇷 South Korea | 12 | 3.6 Mbps | 595 ms | 1% | [`KR.txt`](subs/by-country/KR.txt) |
| 🇭🇰 Hong Kong | 12 | 1.4 Mbps | 674 ms | 1% | [`HK.txt`](subs/by-country/HK.txt) |
| 🇹🇷 Turkey | 8 | 4.7 Mbps | 632 ms | 1% | [`TR.txt`](subs/by-country/TR.txt) |
| 🇦🇱 AL | 8 | 2.9 Mbps | 518 ms | 1% | [`AL.txt`](subs/by-country/AL.txt) |
| 🇦🇪 UAE | 7 | 3.0 Mbps | 840 ms | 1% | [`AE.txt`](subs/by-country/AE.txt) |
| 🇷🇺 Russia | 6 | 4.6 Mbps | 617 ms | 1% | [`RU.txt`](subs/by-country/RU.txt) |
| 🇹🇼 Taiwan | 6 | 3.4 Mbps | 537 ms | 1% | [`TW.txt`](subs/by-country/TW.txt) |
| 🇰🇿 Kazakhstan | 6 | 3.1 Mbps | 1535 ms | 1% | [`KZ.txt`](subs/by-country/KZ.txt) |
| 🇦🇹 Austria | 5 | 6.6 Mbps | 396 ms | 0% | [`AT.txt`](subs/by-country/AT.txt) |
| 🇷🇴 Romania | 5 | 5.9 Mbps | 532 ms | 0% | [`RO.txt`](subs/by-country/RO.txt) |
| 🇱🇻 Latvia | 5 | 4.4 Mbps | 625 ms | 0% | [`LV.txt`](subs/by-country/LV.txt) |
| 🇮🇷 Iran | 5 | 4.6 Mbps | 1195 ms | 0% | [`IR.txt`](subs/by-country/IR.txt) |
| 🇿🇦 South Africa | 4 | 3.9 Mbps | 863 ms | 0% | [`ZA.txt`](subs/by-country/ZA.txt) |
| 🇨🇿 Czechia | 3 | 6.2 Mbps | 502 ms | 0% | [`CZ.txt`](subs/by-country/CZ.txt) |
| 🇧🇷 Brazil | 3 | 6.2 Mbps | 601 ms | 0% | [`BR.txt`](subs/by-country/BR.txt) |
| 🇺🇦 Ukraine | 3 | 5.2 Mbps | 453 ms | 0% | [`UA.txt`](subs/by-country/UA.txt) |
| 🇲🇾 Malaysia | 3 | 4.4 Mbps | 871 ms | 0% | [`MY.txt`](subs/by-country/MY.txt) |
| 🇬🇷 Greece | 3 | 1.9 Mbps | 649 ms | 0% | [`GR.txt`](subs/by-country/GR.txt) |
| 🇳🇬 NG | 3 | - | 1654 ms | 0% | [`NG.txt`](subs/by-country/NG.txt) |
| 🇱🇰 LK | 2 | 8.9 Mbps | 682 ms | 0% | [`LK.txt`](subs/by-country/LK.txt) |
| 🇹🇭 Thailand | 2 | 2.7 Mbps | 854 ms | 0% | [`TH.txt`](subs/by-country/TH.txt) |
| 🇻🇳 Vietnam | 2 | 1.0 Mbps | 1322 ms | 0% | [`VN.txt`](subs/by-country/VN.txt) |
| 🇲🇽 Mexico | 1 | 16.3 Mbps | 225 ms | 0% | [`MX.txt`](subs/by-country/MX.txt) |
| 🇧🇪 Belgium | 1 | 7.6 Mbps | 442 ms | 0% | [`BE.txt`](subs/by-country/BE.txt) |
| 🇨🇱 Chile | 1 | 6.9 Mbps | 497 ms | 0% | [`CL.txt`](subs/by-country/CL.txt) |
| 🇪🇪 Estonia | 1 | 6.5 Mbps | 445 ms | 0% | [`EE.txt`](subs/by-country/EE.txt) |
| 🇮🇱 Israel | 1 | 6.4 Mbps | 614 ms | 0% | [`IL.txt`](subs/by-country/IL.txt) |
| 🇧🇬 Bulgaria | 1 | 6.2 Mbps | 630 ms | 0% | [`BG.txt`](subs/by-country/BG.txt) |
| 🇳🇴 Norway | 1 | 1.4 Mbps | 403 ms | 0% | [`NO.txt`](subs/by-country/NO.txt) |
| 🇸🇦 Saudi Arabia | 1 | 0.6 Mbps | 2277 ms | 0% | [`SA.txt`](subs/by-country/SA.txt) |

A `??` exit means the geo lookup could not resolve the exit IP. Base64 variants of every country file sit next to them as `<CC>.base64.txt`.

## Protocol distribution

| protocol | tested | alive | working | alive rate |
|----------|-------:|------:|--------:|-----------:|
| `vless` | 2781 | 552 | 523 | 20% |
| `ss` | 610 | 250 | 244 | 41% |
| `vmess` | 892 | 149 | 147 | 17% |
| `trojan` | 159 | 116 | 116 | 73% |
| `hysteria2` | 65 | 36 | 36 | 55% |
| `socks` | 2 | 1 | 1 | 50% |
| `http` | 5 | 0 | 0 | 0% |

Per-protocol working lists: [`subs/by-proto/`](subs/by-proto/) (one `vless.txt`, `vmess.txt`, ... per protocol, plus `.base64.txt` variants).

## Engine breakdown

Which core verified each working proxy. The engine fallback means a proxy that failed on its primary core got a second chance on the other one.

| engine | working | note |
|--------|--------:|------|
| Xray-core | 945 | [`allxray.txt`](subs/by-engine/allxray.txt) |
| sing-box | 121 | [`allsingbox.txt`](subs/by-engine/allsingbox.txt) |
| direct (curl) | 1 | plain http/socks, [`alldirect.txt`](subs/by-engine/alldirect.txt) |

## Speed & latency profile

| speed tier | proxies |
|-----------|--------:|
| >= 5 Mbps | 603 |
| 1 - 5 Mbps | 262 |
| 0.25 - 1 Mbps | 52 |
| < 0.25 Mbps | 0 |
| unmeasured | 150 |

Latency (phase-1 HTTPS round trip): median **497 ms**, p90 **3709 ms**. 37 proxies passed connectivity but failed egress verification (leaked the runner IP or broke on the exit check) and are excluded.

## Source health

254 active, 0 auto-retired (10 fetch failures / 10 all-dead runs / 10 days without anything new). Best contributors this run:

| source | kind | tested | alive | new this run |
|--------|------|-------:|------:|-------------:|
| `NK` | url | 1091 | 643 | 7516 |
| `EV` | url | 714 | 575 | 67 |
| `SK` | url | 1004 | 565 | 212 |
| `HP` | url | 474 | 427 | 61 |
| `KA` | url | 2910 | 321 | 10045 |
| `RD-VL` | url | 2333 | 309 | 3495 |
| `AG-PL` | url | 518 | 269 | 666 |
| `HC` | url | 1279 | 262 | 151 |
| `EV-VL` | url | 362 | 246 | 633 |
| `RD-SS` | url | 576 | 244 | 413 |
| `F0` | url | 248 | 238 | 5 |
| `Ni` | url | 294 | 225 | 74 |
| `EV-SS` | url | 233 | 219 | 140 |
| `ST-VL` | url | 2160 | 177 | 676 |
| `RD-VM` | url | 671 | 139 | 3727 |
| `LA` | url | 728 | 132 | 13947 |
| `SB` | url | 2094 | 131 | 0 |
| `EP-ALL` | url | 1325 | 124 | 0 |
| `ST-VM` | url | 647 | 109 | 8 |
| `WD` | url | 105 | 97 | 105 |

## History

working per run (last 27): ▁▁▆▆▆▇▆▆▆█▇▆▇▆▆▆█▆▆▇███████

| date (UTC) | tested | alive | working | avg | max |
|------------|-------:|------:|--------:|----:|----:|
| 2026-11-14 03:41 | 4514 | 1104 | 1067 | 10.7M | 135.5M |
| 2026-11-13 03:31 | 4507 | 1150 | 1119 | 9.4M | 56.3M |
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

## Subscription files

Everything under [`subs/`](subs/) is regenerated every run. Plain lists are one proxy URI per line; `.base64.txt` variants are the same list base64-encoded for clients that need the classic sub format.

| file | contents | direct subscription URL |
|------|----------|--------------------------|
| [`all.txt`](subs/all.txt) | ALL working proxies, ranked by speed (1067) | `https://raw.githubusercontent.com/onlyactions/proxy-scanner/main/subs/all.txt` |
| [`all.base64.txt`](subs/all.base64.txt) | same, base64 | `https://raw.githubusercontent.com/onlyactions/proxy-scanner/main/subs/all.base64.txt` |
| [`top150.txt`](subs/top150.txt) | top 150 ranked (150) | `https://raw.githubusercontent.com/onlyactions/proxy-scanner/main/subs/top150.txt` |
| [`top150.base64.txt`](subs/top150.base64.txt) | same, base64 | `https://raw.githubusercontent.com/onlyactions/proxy-scanner/main/subs/top150.base64.txt` |
| [`by-proto/`](subs/by-proto/) | one list per protocol (6 protocols) | ...`/subs/by-proto/<proto>.txt` |
| [`by-engine/allxray.txt`](subs/by-engine/allxray.txt) | verified on Xray (945) | `https://raw.githubusercontent.com/onlyactions/proxy-scanner/main/subs/by-engine/allxray.txt` |
| [`by-engine/allsingbox.txt`](subs/by-engine/allsingbox.txt) | verified on sing-box (121) | `https://raw.githubusercontent.com/onlyactions/proxy-scanner/main/subs/by-engine/allsingbox.txt` |
| [`by-country/`](subs/by-country/) | one list per exit country (47 countries, e.g. [`US.txt`](subs/by-country/US.txt), [`DE.txt`](subs/by-country/DE.txt)) | ...`/subs/by-country/<CC>.txt` |
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

The typical aggregator relists whatever it scraped, so 80-95% of entries are dead within hours — this run found **76% unreachable** and **76% unusable** overall. Only tunnels that completed a real HTTPS handshake, returned a verifiable foreign exit IP, and pushed a 1 MB download are counted as working here.

## Disclaimers

- Free proxies are **public and untrusted**. Anything you send through them can be observed or modified by their operators. Never use them for authentication, payments, or anything sensitive.
- All configs are collected from publicly posted sources (GitHub subscriptions and public Telegram channels); no credentials are guessed or brute-forced. Removal requests: open an issue.
- Availability is transient: the working list is re-verified daily, and past performance never guarantees future availability.

