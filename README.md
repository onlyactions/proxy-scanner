# Free Proxy Scanner

> **Really tested** free proxy aggregator. Every proxy below was fetched from public sources, tunneled through with Xray/sing-box, and verified to actually change your egress IP. No CSV-validation theater, no blind relisting.

![working](badges/working.json) ![alive](badges/alive.json) ![speed](badges/speed.json) ![max-speed](badges/max-speed.json) ![countries](badges/countries.json) ![sources](badges/sources.json) ![updated](badges/updated.json)

## What this is

A GitHub Actions pipeline (runs once a day) that fetches free proxy configs from **173 URL subscriptions** and **81 Telegram channels**, parses 9 protocols (vless / vmess / trojan / ss / hysteria2 / hysteria / tuic / socks / http), deduplicates them, and **actually connects through every single one** to separate working proxies from dead ones.

Latest run: **2026-11-20 02:55:12 UTC** — 126647 unique proxies collected from 107/107 URL sources and 9/9 Telegram channels; **4517** endpoints really tested, **1203** reachable, **1171** verified working (egress IP confirmed changed) across **49 exit countries**.

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

All 1171 working proxies are published in [`subs/all.txt`](subs/all.txt); this table shows the top 25.

| # | proxy | proto | server | exit | latency | speed | engine |
|---|-------|-------|--------|------|---------|-------|--------|
| 1 | #1 ❓ ?? → 🇺🇸 US | `trojan` | `216.152.155.23:443` | United States | 191 ms | 61.0 Mbps | xray |
| 2 | #2 ❓ ?? → 🇺🇸 US | `trojan` | `216.152.155.54:443` | United States | 190 ms | 55.1 Mbps | xray |
| 3 | #3 ❓ ?? → 🇺🇸 US | `ss` | `67.220.73.178:19999` | United States | 54 ms | 47.6 Mbps | xray |
| 4 | #4 ❓ ?? → 🇺🇸 US | `vmess` | `162.251.60.97:1270` | United States | 129 ms | 46.5 Mbps | xray |
| 5 | #5 ❓ ?? → 🇺🇸 US | `vmess` | `38.128.66.241:1270` | United States | 116 ms | 44.3 Mbps | xray |
| 6 | #6 ❓ ?? → 🇺🇸 US | `ss` | `167.88.62.203:19999` | United States | 62 ms | 43.6 Mbps | xray |
| 7 | #7 ❓ ?? → 🇺🇸 US | `ss` | `167.88.62.56:19999` | United States | 96 ms | 42.5 Mbps | xray |
| 8 | #8 🇺🇸 US → 🇺🇸 US | `ss` | `156.146.38.169:443` | United States | 63 ms | 41.9 Mbps | xray |
| 9 | #9 ❓ ?? → 🇺🇸 US | `vless` | `38.107.232.36:80` | United States | 130 ms | 41.5 Mbps | singbox |
| 10 | #10 🇺🇸 US → 🇺🇸 US | `ss` | `156.146.38.170:443` | United States | 120 ms | 40.1 Mbps | xray |
| 11 | #11 ❓ ?? → 🇺🇸 US | `vmess` | `167.88.62.134:1270` | United States | 72 ms | 39.3 Mbps | xray |
| 12 | #12 🇺🇸 US → 🇺🇸 US | `ss` | `156.146.38.168:443` | United States | 64 ms | 38.7 Mbps | xray |
| 13 | #13 ❓ ?? → 🇺🇸 US | `vmess` | `162.251.61.46:1270` | United States | 63 ms | 37.1 Mbps | xray |
| 14 | #14 ❓ ?? → 🇺🇸 US | `trojan` | `170.101.101.67:443` | United States | 139 ms | 35.4 Mbps | xray |
| 15 | #15 ❓ ?? → 🇺🇸 US | `trojan` | `216.152.155.59:443` | United States | 65 ms | 34.2 Mbps | singbox |
| 16 | #16 ❓ ?? → 🇺🇸 US | `trojan` | `170.101.101.131:443` | United States | 125 ms | 32.8 Mbps | xray |
| 17 | #17 ❓ ?? → 🇺🇸 US | `trojan` | `66.163.114.106:443` | United States | 435 ms | 31.7 Mbps | xray |
| 18 | #18 ❓ ?? → 🇺🇸 US | `trojan` | `216.152.155.176:443` | United States | 119 ms | 31.5 Mbps | xray |
| 19 | #19 ❓ ?? → 🇺🇸 US | `trojan` | `66.163.114.132:443` | United States | 133 ms | 31.2 Mbps | xray |
| 20 | #20 ❓ ?? → 🇺🇸 US | `trojan` | `66.163.114.45:443` | United States | 131 ms | 30.9 Mbps | xray |
| 21 | #21 ❓ ?? → 🇺🇸 US | `trojan` | `66.163.114.126:443` | United States | 177 ms | 30.6 Mbps | xray |
| 22 | #22 ❓ ?? → 🇺🇸 US | `vmess` | `167.88.62.56:1270` | United States | 114 ms | 30.3 Mbps | xray |
| 23 | #23 ❓ ?? → 🇺🇸 US | `vmess` | `38.68.135.131:1270` | United States | 71 ms | 30.2 Mbps | xray |
| 24 | #24 ❓ ?? → 🇺🇸 US | `trojan` | `66.163.114.166:443` | United States | 134 ms | 30.1 Mbps | xray |
| 25 | #25 ❓ ?? → 🇺🇸 US | `trojan` | `170.101.101.112:443` | United States | 132 ms | 30.0 Mbps | xray |

## Exit locations — which countries work

Every working proxy, grouped by the country of its **verified exit IP** (phase 2), with a dedicated subscription file per country. Currently 49 countries have at least one working exit.

| country | working | avg speed | best latency | share | list |
|---------|--------:|----------:|-------------:|------:|------|
| 🇺🇸 United States | 345 | 15.6 Mbps | 52 ms | 29% | [`US.txt`](subs/by-country/US.txt) |
| 🇳🇱 Netherlands | 178 | 5.6 Mbps | 329 ms | 15% | [`NL.txt`](subs/by-country/NL.txt) |
| 🇩🇪 Germany | 148 | 5.4 Mbps | 342 ms | 13% | [`DE.txt`](subs/by-country/DE.txt) |
| 🇯🇵 Japan | 91 | 2.8 Mbps | 413 ms | 8% | [`JP.txt`](subs/by-country/JP.txt) |
| 🇨🇦 Canada | 50 | 11.4 Mbps | 116 ms | 4% | [`CA.txt`](subs/by-country/CA.txt) |
| 🇫🇷 France | 43 | 5.4 Mbps | 354 ms | 4% | [`FR.txt`](subs/by-country/FR.txt) |
| 🇸🇬 Singapore | 42 | 3.6 Mbps | 590 ms | 4% | [`SG.txt`](subs/by-country/SG.txt) |
| 🇪🇸 Spain | 29 | 4.0 Mbps | 422 ms | 2% | [`ES.txt`](subs/by-country/ES.txt) |
| 🇫🇮 Finland | 29 | 4.0 Mbps | 447 ms | 2% | [`FI.txt`](subs/by-country/FI.txt) |
| 🇬🇧 United Kingdom | 27 | 7.2 Mbps | 305 ms | 2% | [`GB.txt`](subs/by-country/GB.txt) |
| 🇸🇪 Sweden | 17 | 5.4 Mbps | 435 ms | 1% | [`SE.txt`](subs/by-country/SE.txt) |
| 🇮🇹 Italy | 17 | 6.5 Mbps | 410 ms | 1% | [`IT.txt`](subs/by-country/IT.txt) |
| 🇨🇭 Switzerland | 16 | 4.9 Mbps | 402 ms | 1% | [`CH.txt`](subs/by-country/CH.txt) |
| 🇭🇰 Hong Kong | 14 | 4.2 Mbps | 153 ms | 1% | [`HK.txt`](subs/by-country/HK.txt) |
| 🇵🇱 Poland | 14 | 5.2 Mbps | 557 ms | 1% | [`PL.txt`](subs/by-country/PL.txt) |
| 🇰🇷 South Korea | 11 | 5.0 Mbps | 596 ms | 1% | [`KR.txt`](subs/by-country/KR.txt) |
| 🇦🇺 Australia | 10 | 12.9 Mbps | 125 ms | 1% | [`AU.txt`](subs/by-country/AU.txt) |
| 🇷🇺 Russia | 10 | 4.4 Mbps | 500 ms | 1% | [`RU.txt`](subs/by-country/RU.txt) |
| 🇧🇾 BY | 10 | 6.0 Mbps | 556 ms | 1% | [`BY.txt`](subs/by-country/BY.txt) |
| 🇮🇳 India | 7 | 3.0 Mbps | 871 ms | 1% | [`IN.txt`](subs/by-country/IN.txt) |
| 🇹🇼 Taiwan | 6 | 4.0 Mbps | 700 ms | 1% | [`TW.txt`](subs/by-country/TW.txt) |
| 🇧🇷 Brazil | 5 | 5.3 Mbps | 647 ms | 0% | [`BR.txt`](subs/by-country/BR.txt) |
| 🇰🇿 Kazakhstan | 5 | 3.2 Mbps | 1237 ms | 0% | [`KZ.txt`](subs/by-country/KZ.txt) |
| 🇦🇹 Austria | 4 | 6.7 Mbps | 443 ms | 0% | [`AT.txt`](subs/by-country/AT.txt) |
| 🇹🇷 Turkey | 4 | 5.5 Mbps | 670 ms | 0% | [`TR.txt`](subs/by-country/TR.txt) |
| 🇻🇳 Vietnam | 4 | 3.3 Mbps | 1084 ms | 0% | [`VN.txt`](subs/by-country/VN.txt) |
| 🇪🇪 Estonia | 3 | 5.6 Mbps | 480 ms | 0% | [`EE.txt`](subs/by-country/EE.txt) |
| 🇨🇿 Czechia | 3 | 3.4 Mbps | 585 ms | 0% | [`CZ.txt`](subs/by-country/CZ.txt) |
| 🇦🇱 AL | 3 | 2.6 Mbps | 569 ms | 0% | [`AL.txt`](subs/by-country/AL.txt) |
| 🇿🇦 South Africa | 3 | 3.5 Mbps | 907 ms | 0% | [`ZA.txt`](subs/by-country/ZA.txt) |
| 🇱🇻 Latvia | 2 | 5.9 Mbps | 665 ms | 0% | [`LV.txt`](subs/by-country/LV.txt) |
| 🇺🇦 Ukraine | 2 | 4.5 Mbps | 1150 ms | 0% | [`UA.txt`](subs/by-country/UA.txt) |
| 🇮🇪 Ireland | 2 | 1.0 Mbps | 290 ms | 0% | [`IE.txt`](subs/by-country/IE.txt) |
| 🇳🇬 NG | 2 | - | 3181 ms | 0% | [`NG.txt`](subs/by-country/NG.txt) |
| 🇵🇦 PA | 1 | 16.9 Mbps | 242 ms | 0% | [`PA.txt`](subs/by-country/PA.txt) |
| 🇳🇴 Norway | 1 | 7.8 Mbps | 1586 ms | 0% | [`NO.txt`](subs/by-country/NO.txt) |
| 🇨🇱 Chile | 1 | 6.4 Mbps | 529 ms | 0% | [`CL.txt`](subs/by-country/CL.txt) |
| 🇷🇴 Romania | 1 | 5.6 Mbps | 1125 ms | 0% | [`RO.txt`](subs/by-country/RO.txt) |
| 🇧🇬 Bulgaria | 1 | 5.4 Mbps | 710 ms | 0% | [`BG.txt`](subs/by-country/BG.txt) |
| 🇱🇺 Luxembourg | 1 | 5.4 Mbps | 875 ms | 0% | [`LU.txt`](subs/by-country/LU.txt) |
| 🇨🇾 Cyprus | 1 | 4.7 Mbps | 663 ms | 0% | [`CY.txt`](subs/by-country/CY.txt) |
| 🇲🇾 Malaysia | 1 | 4.4 Mbps | 642 ms | 0% | [`MY.txt`](subs/by-country/MY.txt) |
| 🇹🇭 Thailand | 1 | 3.9 Mbps | 843 ms | 0% | [`TH.txt`](subs/by-country/TH.txt) |
| 🇦🇪 UAE | 1 | 0.7 Mbps | 1173 ms | 0% | [`AE.txt`](subs/by-country/AE.txt) |
| 🇬🇷 Greece | 1 | 0.7 Mbps | 991 ms | 0% | [`GR.txt`](subs/by-country/GR.txt) |
| 🇲🇽 Mexico | 1 | - | 244 ms | 0% | [`MX.txt`](subs/by-country/MX.txt) |
| 🇮🇷 Iran | 1 | - | 793 ms | 0% | [`IR.txt`](subs/by-country/IR.txt) |
| 🇨🇳 China | 1 | - | 958 ms | 0% | [`CN.txt`](subs/by-country/CN.txt) |
| 🇮🇩 Indonesia | 1 | - | 4863 ms | 0% | [`ID.txt`](subs/by-country/ID.txt) |

A `??` exit means the geo lookup could not resolve the exit IP. Base64 variants of every country file sit next to them as `<CC>.base64.txt`.

## Protocol distribution

| protocol | tested | alive | working | alive rate |
|----------|-------:|------:|--------:|-----------:|
| `vless` | 2762 | 589 | 562 | 21% |
| `ss` | 606 | 295 | 293 | 49% |
| `vmess` | 937 | 162 | 160 | 17% |
| `trojan` | 153 | 118 | 117 | 77% |
| `hysteria2` | 55 | 37 | 37 | 67% |
| `socks` | 2 | 2 | 2 | 100% |
| `http` | 2 | 0 | 0 | 0% |

Per-protocol working lists: [`subs/by-proto/`](subs/by-proto/) (one `vless.txt`, `vmess.txt`, ... per protocol, plus `.base64.txt` variants).

## Engine breakdown

Which core verified each working proxy. The engine fallback means a proxy that failed on its primary core got a second chance on the other one.

| engine | working | note |
|--------|--------:|------|
| Xray-core | 1023 | [`allxray.txt`](subs/by-engine/allxray.txt) |
| sing-box | 146 | [`allsingbox.txt`](subs/by-engine/allsingbox.txt) |
| direct (curl) | 2 | plain http/socks, [`alldirect.txt`](subs/by-engine/alldirect.txt) |

## Speed & latency profile

| speed tier | proxies |
|-----------|--------:|
| >= 5 Mbps | 672 |
| 1 - 5 Mbps | 311 |
| 0.25 - 1 Mbps | 62 |
| < 0.25 Mbps | 0 |
| unmeasured | 126 |

Latency (phase-1 HTTPS round trip): median **550 ms**, p90 **3365 ms**. 32 proxies passed connectivity but failed egress verification (leaked the runner IP or broke on the exit check) and are excluded.

## Source health

115 active, 139 auto-retired (10 fetch failures / 10 all-dead runs / 10 days without anything new). Best contributors this run:

| source | kind | tested | alive | new this run |
|--------|------|-------:|------:|-------------:|
| `NK` | url | 1234 | 850 | 6907 |
| `SK` | url | 1116 | 733 | 0 |
| `EV` | url | 675 | 609 | 117 |
| `HP` | url | 534 | 481 | 58 |
| `RD-VL` | url | 2308 | 333 | 3652 |
| `HC` | url | 1303 | 323 | 183 |
| `AG-PL` | url | 478 | 301 | 679 |
| `RD-SS` | url | 538 | 285 | 428 |
| `F0` | url | 271 | 264 | 0 |
| `Ni` | url | 324 | 257 | 122 |
| `KA` | url | 2980 | 249 | 8894 |
| `EV-SS` | url | 236 | 231 | 130 |
| `EV-VL` | url | 276 | 225 | 585 |
| `ST-VL` | url | 2179 | 222 | 832 |
| `ET` | url | 313 | 192 | 1912 |
| `SB` | url | 2213 | 164 | 6 |
| `RD-VM` | url | 584 | 147 | 3888 |
| `EV-VM` | url | 102 | 97 | 60 |
| `HC-VM` | url | 448 | 95 | 916 |
| `SB-VL` | url | 1908 | 90 | 0 |

## History

working per run (last 30): ▁▆▆▆▇▆▆▆█▇▆▆▆▆▆█▆▆▇███████████

| date (UTC) | tested | alive | working | avg | max |
|------------|-------:|------:|--------:|----:|----:|
| 2026-11-20 03:37 | 4517 | 1203 | 1171 | 8.6M | 61.0M |
| 2026-11-19 03:39 | 4520 | 1168 | 1134 | 10.2M | 73.9M |
| 2026-11-17 03:38 | 4521 | 1198 | 1174 | 10.0M | 83.1M |
| 2026-11-16 03:41 | 4523 | 1137 | 1100 | 10.3M | 83.4M |
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

## Subscription files

Everything under [`subs/`](subs/) is regenerated every run. Plain lists are one proxy URI per line; `.base64.txt` variants are the same list base64-encoded for clients that need the classic sub format.

| file | contents | direct subscription URL |
|------|----------|--------------------------|
| [`all.txt`](subs/all.txt) | ALL working proxies, ranked by speed (1171) | `https://raw.githubusercontent.com/onlyactions/proxy-scanner/main/subs/all.txt` |
| [`all.base64.txt`](subs/all.base64.txt) | same, base64 | `https://raw.githubusercontent.com/onlyactions/proxy-scanner/main/subs/all.base64.txt` |
| [`top150.txt`](subs/top150.txt) | top 150 ranked (150) | `https://raw.githubusercontent.com/onlyactions/proxy-scanner/main/subs/top150.txt` |
| [`top150.base64.txt`](subs/top150.base64.txt) | same, base64 | `https://raw.githubusercontent.com/onlyactions/proxy-scanner/main/subs/top150.base64.txt` |
| [`by-proto/`](subs/by-proto/) | one list per protocol (6 protocols) | ...`/subs/by-proto/<proto>.txt` |
| [`by-engine/allxray.txt`](subs/by-engine/allxray.txt) | verified on Xray (1023) | `https://raw.githubusercontent.com/onlyactions/proxy-scanner/main/subs/by-engine/allxray.txt` |
| [`by-engine/allsingbox.txt`](subs/by-engine/allsingbox.txt) | verified on sing-box (146) | `https://raw.githubusercontent.com/onlyactions/proxy-scanner/main/subs/by-engine/allsingbox.txt` |
| [`by-country/`](subs/by-country/) | one list per exit country (49 countries, e.g. [`US.txt`](subs/by-country/US.txt), [`DE.txt`](subs/by-country/DE.txt)) | ...`/subs/by-country/<CC>.txt` |
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

