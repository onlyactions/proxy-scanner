# Free Proxy Scanner

> **Really tested** free proxy aggregator. Every proxy below was fetched from public sources, tunneled through with Xray/sing-box, and verified to actually change your egress IP. No CSV-validation theater, no blind relisting.

![working](badges/working.json) ![alive](badges/alive.json) ![speed](badges/speed.json) ![max-speed](badges/max-speed.json) ![countries](badges/countries.json) ![sources](badges/sources.json) ![updated](badges/updated.json)

## What this is

A GitHub Actions pipeline (runs once a day) that fetches free proxy configs from **173 URL subscriptions** and **81 Telegram channels**, parses 9 protocols (vless / vmess / trojan / ss / hysteria2 / hysteria / tuic / socks / http), deduplicates them, and **actually connects through every single one** to separate working proxies from dead ones.

Latest run: **2026-11-22 02:56:53 UTC** — 126916 unique proxies collected from 106/106 URL sources and 9/9 Telegram channels; **4516** endpoints really tested, **1213** reachable, **1159** verified working (egress IP confirmed changed) across **45 exit countries**.

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

All 1159 working proxies are published in [`subs/all.txt`](subs/all.txt); this table shows the top 25.

| # | proxy | proto | server | exit | latency | speed | engine |
|---|-------|-------|--------|------|---------|-------|--------|
| 1 | #1 ❓ ?? → 🇺🇸 US | `trojan` | `66.163.122.46:443` | United States | 230 ms | 139.3 Mbps | xray |
| 2 | #2 ❓ ?? → 🇺🇸 US | `ss` | `45.86.51.18:19999` | United States | 153 ms | 128.2 Mbps | xray |
| 3 | #3 ❓ ?? → 🇺🇸 US | `ss` | `185.230.230.18:19999` | United States | 113 ms | 124.5 Mbps | xray |
| 4 | #4 ❓ ?? → 🇺🇸 US | `vmess` | `67.220.95.49:1270` | United States | 226 ms | 112.2 Mbps | xray |
| 5 | #5 🇺🇸 US → 🇺🇸 US | `ss` | `37.19.198.236:443` | United States | 162 ms | 86.1 Mbps | xray |
| 6 | #6 ❓ ?? → 🇺🇸 US | `ss` | `40.160.86.43:8388` | United States | 100 ms | 78.4 Mbps | xray |
| 7 | #7 🇺🇸 US → 🇺🇸 US | `ss` | `37.19.198.244:443` | United States | 174 ms | 74.7 Mbps | xray |
| 8 | #8 ❓ ?? → 🇺🇸 US | `vmess` | `38.107.226.5:1270` | United States | 198 ms | 73.5 Mbps | xray |
| 9 | #9 ❓ ?? → 🇺🇸 US | `ss` | `135.148.42.13:8388` | United States | 100 ms | 70.5 Mbps | xray |
| 10 | #10 🇺🇸 US → 🇺🇸 US | `ss` | `37.19.198.160:443` | United States | 131 ms | 69.2 Mbps | xray |
| 11 | #11 🇺🇸 US → 🇺🇸 US | `ss` | `37.19.198.243:443` | United States | 119 ms | 69.1 Mbps | xray |
| 12 | #12 ❓ ?? → 🇺🇸 US | `ss` | `200.102.179.179:8388` | United States | 83 ms | 62.4 Mbps | xray |
| 13 | #13 ❓ ?? → 🇺🇸 US | `trojan` | `66.163.122.81:443` | United States | 64 ms | 61.7 Mbps | xray |
| 14 | #14 ❓ ?? → 🇺🇸 US | `hysteria2` | `206.189.229.48:443` | United States | 61 ms | 57.1 Mbps | singbox |
| 15 | #15 ❓ ?? → 🇺🇸 US | `ss` | `193.151.163.18:19999` | United States | 88 ms | 55.9 Mbps | xray |
| 16 | #16 ❓ ?? → 🇺🇸 US | `trojan` | `66.163.122.26:443` | United States | 59 ms | 53.5 Mbps | xray |
| 17 | #17 ❓ ?? → 🇺🇸 US | `vless` | `185.95.231.156:443` | United States | 81 ms | 52.3 Mbps | xray |
| 18 | #18 🇺🇸 US → 🇺🇸 US | `vless` | `169.40.42.163:443` | United States | 202 ms | 51.1 Mbps | xray |
| 19 | #19 ❓ ?? → 🇺🇸 US | `ss` | `149.40.57.134:19999` | United States | 82 ms | 49.6 Mbps | xray |
| 20 | #20 ❓ ?? → 🇺🇸 US | `ss` | `149.40.57.133:19999` | United States | 79 ms | 48.7 Mbps | xray |
| 21 | #21 ❓ ?? → 🇺🇸 US | `trojan` | `ipfast.kingmna.com:443` | United States | 58 ms | 47.4 Mbps | singbox |
| 22 | #22 ❓ ?? → 🇺🇸 US | `vless` | `172.233.197.105:62867` | United States | 23 ms | 47.2 Mbps | xray |
| 23 | #23 🇺🇸 US → 🇺🇸 US | `ss` | `15.204.246.107:8888` | United States | 201 ms | 46.7 Mbps | xray |
| 24 | #24 ❓ ?? → 🇺🇸 US | `vless` | `45.138.100.248:14430` | United States | 55 ms | 45.0 Mbps | singbox |
| 25 | #25 ❓ ?? → 🇺🇸 US | `vless` | `45.138.100.248:10443` | United States | 46 ms | 45.0 Mbps | singbox |

## Exit locations — which countries work

Every working proxy, grouped by the country of its **verified exit IP** (phase 2), with a dedicated subscription file per country. Currently 45 countries have at least one working exit.

| country | working | avg speed | best latency | share | list |
|---------|--------:|----------:|-------------:|------:|------|
| 🇺🇸 United States | 356 | 18.4 Mbps | 23 ms | 31% | [`US.txt`](subs/by-country/US.txt) |
| 🇳🇱 Netherlands | 174 | 7.3 Mbps | 258 ms | 15% | [`NL.txt`](subs/by-country/NL.txt) |
| 🇩🇪 Germany | 144 | 6.1 Mbps | 277 ms | 12% | [`DE.txt`](subs/by-country/DE.txt) |
| 🇯🇵 Japan | 91 | 2.1 Mbps | 480 ms | 8% | [`JP.txt`](subs/by-country/JP.txt) |
| 🇨🇦 Canada | 49 | 14.0 Mbps | 61 ms | 4% | [`CA.txt`](subs/by-country/CA.txt) |
| 🇫🇷 France | 35 | 6.9 Mbps | 303 ms | 3% | [`FR.txt`](subs/by-country/FR.txt) |
| 🇸🇬 Singapore | 34 | 3.1 Mbps | 674 ms | 3% | [`SG.txt`](subs/by-country/SG.txt) |
| 🇪🇸 Spain | 33 | 5.0 Mbps | 339 ms | 3% | [`ES.txt`](subs/by-country/ES.txt) |
| 🇬🇧 United Kingdom | 23 | 8.2 Mbps | 250 ms | 2% | [`GB.txt`](subs/by-country/GB.txt) |
| 🇨🇭 Switzerland | 21 | 7.3 Mbps | 342 ms | 2% | [`CH.txt`](subs/by-country/CH.txt) |
| 🇮🇹 Italy | 17 | 7.7 Mbps | 336 ms | 1% | [`IT.txt`](subs/by-country/IT.txt) |
| 🇭🇰 Hong Kong | 17 | 3.0 Mbps | 828 ms | 1% | [`HK.txt`](subs/by-country/HK.txt) |
| 🇦🇺 Australia | 16 | 11.0 Mbps | 67 ms | 1% | [`AU.txt`](subs/by-country/AU.txt) |
| 🇫🇮 Finland | 16 | 5.9 Mbps | 351 ms | 1% | [`FI.txt`](subs/by-country/FI.txt) |
| 🇸🇪 Sweden | 13 | 6.5 Mbps | 351 ms | 1% | [`SE.txt`](subs/by-country/SE.txt) |
| 🇵🇱 Poland | 12 | 6.9 Mbps | 429 ms | 1% | [`PL.txt`](subs/by-country/PL.txt) |
| 🇷🇺 Russia | 11 | 6.8 Mbps | 371 ms | 1% | [`RU.txt`](subs/by-country/RU.txt) |
| 🇧🇾 BY | 11 | 7.0 Mbps | 414 ms | 1% | [`BY.txt`](subs/by-country/BY.txt) |
| 🇹🇼 Taiwan | 11 | 3.5 Mbps | 785 ms | 1% | [`TW.txt`](subs/by-country/TW.txt) |
| 🇰🇷 South Korea | 10 | 3.9 Mbps | 639 ms | 1% | [`KR.txt`](subs/by-country/KR.txt) |
| 🇰🇿 Kazakhstan | 9 | 3.4 Mbps | 1201 ms | 1% | [`KZ.txt`](subs/by-country/KZ.txt) |
| 🇦🇹 Austria | 6 | 7.7 Mbps | 350 ms | 1% | [`AT.txt`](subs/by-country/AT.txt) |
| 🇮🇳 India | 6 | 2.0 Mbps | 772 ms | 1% | [`IN.txt`](subs/by-country/IN.txt) |
| 🇧🇷 Brazil | 4 | 7.2 Mbps | 552 ms | 0% | [`BR.txt`](subs/by-country/BR.txt) |
| 🇦🇪 UAE | 4 | 2.4 Mbps | 924 ms | 0% | [`AE.txt`](subs/by-country/AE.txt) |
| 🇦🇱 AL | 3 | 3.8 Mbps | 509 ms | 0% | [`AL.txt`](subs/by-country/AL.txt) |
| 🇪🇪 Estonia | 3 | 5.6 Mbps | 539 ms | 0% | [`EE.txt`](subs/by-country/EE.txt) |
| 🇱🇻 Latvia | 3 | 6.3 Mbps | 587 ms | 0% | [`LV.txt`](subs/by-country/LV.txt) |
| 🇺🇦 Ukraine | 3 | 6.7 Mbps | 637 ms | 0% | [`UA.txt`](subs/by-country/UA.txt) |
| 🇿🇦 South Africa | 3 | 3.8 Mbps | 836 ms | 0% | [`ZA.txt`](subs/by-country/ZA.txt) |
| 🇳🇬 NG | 3 | - | 4053 ms | 0% | [`NG.txt`](subs/by-country/NG.txt) |
| 🇨🇿 Czechia | 2 | 5.7 Mbps | 494 ms | 0% | [`CZ.txt`](subs/by-country/CZ.txt) |
| 🇮🇷 Iran | 2 | 6.5 Mbps | 448 ms | 0% | [`IR.txt`](subs/by-country/IR.txt) |
| 🇨🇱 Chile | 2 | 7.1 Mbps | 477 ms | 0% | [`CL.txt`](subs/by-country/CL.txt) |
| 🇻🇳 Vietnam | 2 | 1.1 Mbps | 1158 ms | 0% | [`VN.txt`](subs/by-country/VN.txt) |
| 🇳🇴 Norway | 1 | 9.3 Mbps | 5989 ms | 0% | [`NO.txt`](subs/by-country/NO.txt) |
| 🇩🇰 Denmark | 1 | 7.7 Mbps | 479 ms | 0% | [`DK.txt`](subs/by-country/DK.txt) |
| 🇷🇴 Romania | 1 | 7.4 Mbps | 543 ms | 0% | [`RO.txt`](subs/by-country/RO.txt) |
| 🇹🇷 Turkey | 1 | 7.3 Mbps | 749 ms | 0% | [`TR.txt`](subs/by-country/TR.txt) |
| 🇨🇾 Cyprus | 1 | 7.1 Mbps | 521 ms | 0% | [`CY.txt`](subs/by-country/CY.txt) |
| 🇨🇴 Colombia | 1 | 3.8 Mbps | 1211 ms | 0% | [`CO.txt`](subs/by-country/CO.txt) |
| 🇨🇳 China | 1 | 2.6 Mbps | 1032 ms | 0% | [`CN.txt`](subs/by-country/CN.txt) |
| 🇹🇭 Thailand | 1 | 1.2 Mbps | 891 ms | 0% | [`TH.txt`](subs/by-country/TH.txt) |
| 🇱🇺 Luxembourg | 1 | 1.1 Mbps | 731 ms | 0% | [`LU.txt`](subs/by-country/LU.txt) |
| 🇵🇦 PA | 1 | 0.7 Mbps | 156 ms | 0% | [`PA.txt`](subs/by-country/PA.txt) |

A `??` exit means the geo lookup could not resolve the exit IP. Base64 variants of every country file sit next to them as `<CC>.base64.txt`.

## Protocol distribution

| protocol | tested | alive | working | alive rate |
|----------|-------:|------:|--------:|-----------:|
| `vless` | 2744 | 602 | 561 | 22% |
| `ss` | 574 | 278 | 272 | 48% |
| `vmess` | 979 | 178 | 171 | 18% |
| `trojan` | 151 | 105 | 105 | 70% |
| `hysteria2` | 64 | 48 | 48 | 75% |
| `socks` | 2 | 2 | 2 | 100% |
| `http` | 2 | 0 | 0 | 0% |

Per-protocol working lists: [`subs/by-proto/`](subs/by-proto/) (one `vless.txt`, `vmess.txt`, ... per protocol, plus `.base64.txt` variants).

## Engine breakdown

Which core verified each working proxy. The engine fallback means a proxy that failed on its primary core got a second chance on the other one.

| engine | working | note |
|--------|--------:|------|
| Xray-core | 1023 | [`allxray.txt`](subs/by-engine/allxray.txt) |
| sing-box | 134 | [`allsingbox.txt`](subs/by-engine/allsingbox.txt) |
| direct (curl) | 2 | plain http/socks, [`alldirect.txt`](subs/by-engine/alldirect.txt) |

## Speed & latency profile

| speed tier | proxies |
|-----------|--------:|
| >= 5 Mbps | 680 |
| 1 - 5 Mbps | 307 |
| 0.25 - 1 Mbps | 68 |
| < 0.25 Mbps | 0 |
| unmeasured | 104 |

Latency (phase-1 HTTPS round trip): median **448 ms**, p90 **3098 ms**. 54 proxies passed connectivity but failed egress verification (leaked the runner IP or broke on the exit check) and are excluded.

## Source health

115 active, 139 auto-retired (10 fetch failures / 10 all-dead runs / 10 days without anything new). Best contributors this run:

| source | kind | tested | alive | new this run |
|--------|------|-------:|------:|-------------:|
| `NK` | url | 1139 | 722 | 5609 |
| `SK` | url | 1076 | 664 | 0 |
| `EV` | url | 699 | 601 | 69 |
| `HP` | url | 470 | 417 | 106 |
| `ET` | url | 452 | 347 | 1937 |
| `AG-PL` | url | 483 | 322 | 747 |
| `RD-VL` | url | 2283 | 308 | 3629 |
| `RD-SS` | url | 513 | 272 | 429 |
| `F0` | url | 280 | 260 | 28 |
| `EV-SS` | url | 264 | 245 | 151 |
| `Ni` | url | 315 | 239 | 113 |
| `HC` | url | 1209 | 229 | 235 |
| `EV-VL` | url | 282 | 223 | 693 |
| `ST-VL` | url | 2129 | 194 | 898 |
| `RD-VM` | url | 601 | 154 | 3936 |
| `LA` | url | 685 | 136 | 14742 |
| `F18` | url | 161 | 128 | 196 |
| `SB` | url | 2054 | 121 | 2 |
| `ST-VM` | url | 568 | 110 | 29 |
| `KA` | url | 2492 | 103 | 5408 |

## History

working per run (last 30): ▆▆▇▆▆▆█▇▆▆▆▆▆█▆▆▇█████████████

| date (UTC) | tested | alive | working | avg | max |
|------------|-------:|------:|--------:|----:|----:|
| 2026-11-22 03:38 | 4516 | 1213 | 1159 | 10.1M | 139.3M |
| 2026-11-21 03:33 | 4514 | 1155 | 1120 | 11.2M | 128.3M |
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

## Subscription files

Everything under [`subs/`](subs/) is regenerated every run. Plain lists are one proxy URI per line; `.base64.txt` variants are the same list base64-encoded for clients that need the classic sub format.

| file | contents | direct subscription URL |
|------|----------|--------------------------|
| [`all.txt`](subs/all.txt) | ALL working proxies, ranked by speed (1159) | `https://raw.githubusercontent.com/onlyactions/proxy-scanner/main/subs/all.txt` |
| [`all.base64.txt`](subs/all.base64.txt) | same, base64 | `https://raw.githubusercontent.com/onlyactions/proxy-scanner/main/subs/all.base64.txt` |
| [`top150.txt`](subs/top150.txt) | top 150 ranked (150) | `https://raw.githubusercontent.com/onlyactions/proxy-scanner/main/subs/top150.txt` |
| [`top150.base64.txt`](subs/top150.base64.txt) | same, base64 | `https://raw.githubusercontent.com/onlyactions/proxy-scanner/main/subs/top150.base64.txt` |
| [`by-proto/`](subs/by-proto/) | one list per protocol (6 protocols) | ...`/subs/by-proto/<proto>.txt` |
| [`by-engine/allxray.txt`](subs/by-engine/allxray.txt) | verified on Xray (1023) | `https://raw.githubusercontent.com/onlyactions/proxy-scanner/main/subs/by-engine/allxray.txt` |
| [`by-engine/allsingbox.txt`](subs/by-engine/allsingbox.txt) | verified on sing-box (134) | `https://raw.githubusercontent.com/onlyactions/proxy-scanner/main/subs/by-engine/allsingbox.txt` |
| [`by-country/`](subs/by-country/) | one list per exit country (45 countries, e.g. [`US.txt`](subs/by-country/US.txt), [`DE.txt`](subs/by-country/DE.txt)) | ...`/subs/by-country/<CC>.txt` |
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

