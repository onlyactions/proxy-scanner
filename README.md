# Free Proxy Scanner

> **Really tested** free proxy aggregator. Every proxy below was fetched from public sources, tunneled through with Xray/sing-box, and verified to actually change your egress IP. No CSV-validation theater, no blind relisting.

![working](badges/working.json) ![alive](badges/alive.json) ![speed](badges/speed.json) ![max-speed](badges/max-speed.json) ![countries](badges/countries.json) ![sources](badges/sources.json) ![updated](badges/updated.json)

## What this is

A GitHub Actions pipeline (runs once a day) that fetches free proxy configs from **173 URL subscriptions** and **81 Telegram channels**, parses 9 protocols (vless / vmess / trojan / ss / hysteria2 / hysteria / tuic / socks / http), deduplicates them, and **actually connects through every single one** to separate working proxies from dead ones.

Latest run: **2026-11-16 03:01:45 UTC** — 175748 unique proxies collected from 173/173 URL sources and 81/81 Telegram channels; **4523** endpoints really tested, **1137** reachable, **1100** verified working (egress IP confirmed changed) across **47 exit countries**.

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

All 1100 working proxies are published in [`subs/all.txt`](subs/all.txt); this table shows the top 25.

| # | proxy | proto | server | exit | latency | speed | engine |
|---|-------|-------|--------|------|---------|-------|--------|
| 1 | #1 ❓ ?? → 🇺🇸 US | `trojan` | `66.163.122.81:443` | United States | 144 ms | 83.4 Mbps | xray |
| 2 | #2 ❓ ?? → 🇺🇸 US | `ss` | `172.111.38.72:19999` | United States | 148 ms | 80.0 Mbps | xray |
| 3 | #3 ❓ ?? → 🇺🇸 US | `trojan` | `66.163.122.46:443` | United States | 82 ms | 79.1 Mbps | xray |
| 4 | #4 ❓ ?? → 🇺🇸 US | `ss` | `45.86.51.18:19999` | United States | 118 ms | 77.7 Mbps | xray |
| 5 | #5 ❓ ?? → 🇺🇸 US | `trojan` | `66.163.122.25:443` | United States | 81 ms | 70.3 Mbps | xray |
| 6 | #6 ❓ ?? → 🇺🇸 US | `vless` | `172.233.201.244:61635` | United States | 45 ms | 64.7 Mbps | xray |
| 7 | #7 ❓ ?? → 🇺🇸 US | `ss` | `185.230.230.18:19999` | United States | 75 ms | 60.6 Mbps | xray |
| 8 | #8 ❓ ?? → 🇺🇸 US | `trojan` | `66.163.122.113:443` | United States | 80 ms | 59.7 Mbps | xray |
| 9 | #9 ❓ ?? → 🇺🇸 US | `trojan` | `66.163.122.30:443` | United States | 75 ms | 58.6 Mbps | xray |
| 10 | #10 ❓ ?? → 🇺🇸 US | `vmess` | `172.111.38.246:1270` | United States | 107 ms | 58.4 Mbps | xray |
| 11 | #11 ❓ ?? → 🇺🇸 US | `vmess` | `38.107.226.5:1270` | United States | 103 ms | 55.9 Mbps | xray |
| 12 | #12 ❓ ?? → 🇺🇸 US | `trojan` | `66.163.122.26:443` | United States | 74 ms | 55.3 Mbps | xray |
| 13 | #13 🇺🇸 US → 🇺🇸 US | `vmess` | `67.220.95.3:18000` | United States | 5570 ms | 54.8 Mbps | xray |
| 14 | #14 🇺🇸 US → 🇺🇸 US | `vless` | `47.253.226.114:443` | United States | 48 ms | 54.3 Mbps | xray |
| 15 | #15 ❓ ?? → 🇺🇸 US | `vless` | `185.95.231.156:443` | United States | 110 ms | 51.8 Mbps | xray |
| 16 | #16 ❓ ?? → 🇺🇸 US | `trojan` | `66.163.122.7:443` | United States | 81 ms | 50.1 Mbps | xray |
| 17 | #17 🇺🇸 US → 🇺🇸 US | `vless` | `169.40.42.74:443` | United States | 147 ms | 47.3 Mbps | xray |
| 18 | #18 ❓ ?? → 🇺🇸 US | `vless` | `45.138.100.248:10443` | United States | 68 ms | 47.2 Mbps | singbox |
| 19 | #19 ❓ ?? → 🇺🇸 US | `ss` | `149.40.57.134:19999` | United States | 111 ms | 46.2 Mbps | xray |
| 20 | #20 ❓ ?? → 🇺🇸 US | `vless` | `45.138.100.248:14430` | United States | 71 ms | 45.3 Mbps | singbox |
| 21 | #21 🇺🇸 US → 🇺🇸 US | `vless` | `159.89.87.21:28190` | United States | 68 ms | 45.3 Mbps | xray |
| 22 | #22 🇺🇸 US → 🇺🇸 US | `vless` | `169.40.42.163:443` | United States | 192 ms | 43.6 Mbps | xray |
| 23 | #23 🇺🇸 US → 🇺🇸 US | `vless` | `169.40.42.168:443` | United States | 215 ms | 41.5 Mbps | xray |
| 24 | #24 ❓ ?? → 🇺🇸 US | `ss` | `149.40.57.135:19999` | United States | 130 ms | 41.2 Mbps | xray |
| 25 | #25 ❓ ?? → 🇺🇸 US | `ss` | `193.151.163.18:19999` | United States | 53 ms | 40.4 Mbps | xray |

## Exit locations — which countries work

Every working proxy, grouped by the country of its **verified exit IP** (phase 2), with a dedicated subscription file per country. Currently 47 countries have at least one working exit.

| country | working | avg speed | best latency | share | list |
|---------|--------:|----------:|-------------:|------:|------|
| 🇺🇸 United States | 312 | 19.7 Mbps | 39 ms | 28% | [`US.txt`](subs/by-country/US.txt) |
| 🇳🇱 Netherlands | 152 | 7.3 Mbps | 279 ms | 14% | [`NL.txt`](subs/by-country/NL.txt) |
| 🇩🇪 Germany | 129 | 6.4 Mbps | 278 ms | 12% | [`DE.txt`](subs/by-country/DE.txt) |
| 🇯🇵 Japan | 88 | 2.4 Mbps | 476 ms | 8% | [`JP.txt`](subs/by-country/JP.txt) |
| 🇨🇦 Canada | 46 | 14.2 Mbps | 85 ms | 4% | [`CA.txt`](subs/by-country/CA.txt) |
| 🇬🇧 United Kingdom | 36 | 7.0 Mbps | 258 ms | 3% | [`GB.txt`](subs/by-country/GB.txt) |
| 🇪🇸 Spain | 36 | 5.5 Mbps | 326 ms | 3% | [`ES.txt`](subs/by-country/ES.txt) |
| 🇫🇷 France | 34 | 7.8 Mbps | 297 ms | 3% | [`FR.txt`](subs/by-country/FR.txt) |
| 🇸🇬 Singapore | 33 | 3.0 Mbps | 677 ms | 3% | [`SG.txt`](subs/by-country/SG.txt) |
| 🇫🇮 Finland | 24 | 4.9 Mbps | 352 ms | 2% | [`FI.txt`](subs/by-country/FI.txt) |
| 🇸🇪 Sweden | 19 | 5.4 Mbps | 379 ms | 2% | [`SE.txt`](subs/by-country/SE.txt) |
| 🇨🇭 Switzerland | 16 | 7.7 Mbps | 326 ms | 1% | [`CH.txt`](subs/by-country/CH.txt) |
| 🇮🇹 Italy | 15 | 7.9 Mbps | 331 ms | 1% | [`IT.txt`](subs/by-country/IT.txt) |
| 🇵🇱 Poland | 15 | 6.7 Mbps | 349 ms | 1% | [`PL.txt`](subs/by-country/PL.txt) |
| 🇭🇰 Hong Kong | 12 | 2.8 Mbps | 407 ms | 1% | [`HK.txt`](subs/by-country/HK.txt) |
| 🇷🇺 Russia | 11 | 5.3 Mbps | 383 ms | 1% | [`RU.txt`](subs/by-country/RU.txt) |
| 🇮🇳 India | 11 | 3.3 Mbps | 796 ms | 1% | [`IN.txt`](subs/by-country/IN.txt) |
| 🇰🇷 South Korea | 10 | 4.6 Mbps | 627 ms | 1% | [`KR.txt`](subs/by-country/KR.txt) |
| 🇦🇱 AL | 10 | 1.9 Mbps | 1373 ms | 1% | [`AL.txt`](subs/by-country/AL.txt) |
| 🇧🇾 BY | 9 | 6.9 Mbps | 483 ms | 1% | [`BY.txt`](subs/by-country/BY.txt) |
| 🇹🇷 Turkey | 8 | 3.5 Mbps | 649 ms | 1% | [`TR.txt`](subs/by-country/TR.txt) |
| 🇷🇴 Romania | 6 | 5.3 Mbps | 573 ms | 1% | [`RO.txt`](subs/by-country/RO.txt) |
| 🇮🇷 Iran | 6 | 5.0 Mbps | 793 ms | 1% | [`IR.txt`](subs/by-country/IR.txt) |
| 🇰🇿 Kazakhstan | 6 | 3.5 Mbps | 1479 ms | 1% | [`KZ.txt`](subs/by-country/KZ.txt) |
| 🇦🇪 UAE | 5 | 2.2 Mbps | 926 ms | 0% | [`AE.txt`](subs/by-country/AE.txt) |
| 🇨🇿 Czechia | 4 | 5.5 Mbps | 441 ms | 0% | [`CZ.txt`](subs/by-country/CZ.txt) |
| 🇦🇺 Australia | 4 | 3.6 Mbps | 849 ms | 0% | [`AU.txt`](subs/by-country/AU.txt) |
| 🇿🇦 South Africa | 4 | 4.1 Mbps | 812 ms | 0% | [`ZA.txt`](subs/by-country/ZA.txt) |
| 🇹🇼 Taiwan | 4 | 3.4 Mbps | 825 ms | 0% | [`TW.txt`](subs/by-country/TW.txt) |
| 🇹🇭 Thailand | 4 | 1.9 Mbps | 1508 ms | 0% | [`TH.txt`](subs/by-country/TH.txt) |
| 🇳🇬 NG | 4 | - | 1832 ms | 0% | [`NG.txt`](subs/by-country/NG.txt) |
| 🇺🇦 Ukraine | 3 | 6.5 Mbps | 435 ms | 0% | [`UA.txt`](subs/by-country/UA.txt) |
| 🇦🇹 Austria | 3 | 8.1 Mbps | 351 ms | 0% | [`AT.txt`](subs/by-country/AT.txt) |
| 🇧🇷 Brazil | 3 | 7.0 Mbps | 514 ms | 0% | [`BR.txt`](subs/by-country/BR.txt) |
| 🇪🇪 Estonia | 3 | 6.1 Mbps | 491 ms | 0% | [`EE.txt`](subs/by-country/EE.txt) |
| 🇱🇻 Latvia | 2 | 5.5 Mbps | 643 ms | 0% | [`LV.txt`](subs/by-country/LV.txt) |
| 🇱🇰 LK | 2 | 2.7 Mbps | 1533 ms | 0% | [`LK.txt`](subs/by-country/LK.txt) |
| 🇲🇾 Malaysia | 2 | - | 1879 ms | 0% | [`MY.txt`](subs/by-country/MY.txt) |
| 🇮🇪 Ireland | 1 | 8.2 Mbps | 228 ms | 0% | [`IE.txt`](subs/by-country/IE.txt) |
| 🇷🇸 Serbia | 1 | 7.4 Mbps | 577 ms | 0% | [`RS.txt`](subs/by-country/RS.txt) |
| 🇳🇴 Norway | 1 | 7.2 Mbps | 380 ms | 0% | [`NO.txt`](subs/by-country/NO.txt) |
| 🇨🇱 Chile | 1 | 6.1 Mbps | 458 ms | 0% | [`CL.txt`](subs/by-country/CL.txt) |
| 🇧🇬 Bulgaria | 1 | 6.0 Mbps | 574 ms | 0% | [`BG.txt`](subs/by-country/BG.txt) |
| 🇬🇷 Greece | 1 | 5.2 Mbps | 637 ms | 0% | [`GR.txt`](subs/by-country/GR.txt) |
| 🇸🇦 Saudi Arabia | 1 | 0.9 Mbps | 932 ms | 0% | [`SA.txt`](subs/by-country/SA.txt) |
| 🇻🇳 Vietnam | 1 | - | 1167 ms | 0% | [`VN.txt`](subs/by-country/VN.txt) |
| 🇱🇹 Lithuania | 1 | - | 1574 ms | 0% | [`LT.txt`](subs/by-country/LT.txt) |

A `??` exit means the geo lookup could not resolve the exit IP. Base64 variants of every country file sit next to them as `<CC>.base64.txt`.

## Protocol distribution

| protocol | tested | alive | working | alive rate |
|----------|-------:|------:|--------:|-----------:|
| `vless` | 2760 | 563 | 532 | 20% |
| `ss` | 683 | 257 | 252 | 38% |
| `vmess` | 831 | 153 | 152 | 18% |
| `trojan` | 170 | 118 | 118 | 69% |
| `hysteria2` | 73 | 45 | 45 | 62% |
| `socks` | 1 | 1 | 1 | 100% |
| `http` | 5 | 0 | 0 | 0% |

Per-protocol working lists: [`subs/by-proto/`](subs/by-proto/) (one `vless.txt`, `vmess.txt`, ... per protocol, plus `.base64.txt` variants).

## Engine breakdown

Which core verified each working proxy. The engine fallback means a proxy that failed on its primary core got a second chance on the other one.

| engine | working | note |
|--------|--------:|------|
| Xray-core | 970 | [`allxray.txt`](subs/by-engine/allxray.txt) |
| sing-box | 129 | [`allsingbox.txt`](subs/by-engine/allsingbox.txt) |
| direct (curl) | 1 | plain http/socks, [`alldirect.txt`](subs/by-engine/alldirect.txt) |

## Speed & latency profile

| speed tier | proxies |
|-----------|--------:|
| >= 5 Mbps | 662 |
| 1 - 5 Mbps | 279 |
| 0.25 - 1 Mbps | 48 |
| < 0.25 Mbps | 0 |
| unmeasured | 111 |

Latency (phase-1 HTTPS round trip): median **464 ms**, p90 **3003 ms**. 37 proxies passed connectivity but failed egress verification (leaked the runner IP or broke on the exit check) and are excluded.

## Source health

178 active, 76 auto-retired (10 fetch failures / 10 all-dead runs / 10 days without anything new). Best contributors this run:

| source | kind | tested | alive | new this run |
|--------|------|-------:|------:|-------------:|
| `NK` | url | 1171 | 726 | 7212 |
| `SK` | url | 1104 | 685 | 286 |
| `EV` | url | 630 | 554 | 90 |
| `HP` | url | 389 | 361 | 80 |
| `AG-PL` | url | 468 | 343 | 636 |
| `RD-VL` | url | 2226 | 319 | 3833 |
| `HC` | url | 1192 | 277 | 169 |
| `KA` | url | 2850 | 275 | 10494 |
| `RD-SS` | url | 558 | 252 | 386 |
| `F0` | url | 251 | 244 | 1 |
| `EV-SS` | url | 241 | 229 | 125 |
| `ST-VL` | url | 2090 | 208 | 737 |
| `EV-VL` | url | 233 | 180 | 659 |
| `SB` | url | 2274 | 177 | 0 |
| `LA` | url | 785 | 164 | 14343 |
| `RD-VM` | url | 587 | 141 | 3828 |
| `Ni` | url | 211 | 133 | 23 |
| `ST-VM` | url | 593 | 128 | 30 |
| `SB-VL` | url | 1927 | 127 | 0 |
| `TK` | url | 109 | 81 | 1 |

## History

working per run (last 28): ▁▁▆▆▆▇▆▆▆█▇▆▇▆▆▆█▆▆▇████████

| date (UTC) | tested | alive | working | avg | max |
|------------|-------:|------:|--------:|----:|----:|
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
| 2026-10-02 21:49 | 4500 | 890 | 861 | 8.9M | 68.7M |
| 2026-10-02 17:14 | 4500 | 847 | 807 | 8.6M | 88.1M |
| 2026-10-02 11:49 | 4500 | 861 | 798 | 7.9M | 85.3M |

## Subscription files

Everything under [`subs/`](subs/) is regenerated every run. Plain lists are one proxy URI per line; `.base64.txt` variants are the same list base64-encoded for clients that need the classic sub format.

| file | contents | direct subscription URL |
|------|----------|--------------------------|
| [`all.txt`](subs/all.txt) | ALL working proxies, ranked by speed (1100) | `https://raw.githubusercontent.com/onlyactions/proxy-scanner/main/subs/all.txt` |
| [`all.base64.txt`](subs/all.base64.txt) | same, base64 | `https://raw.githubusercontent.com/onlyactions/proxy-scanner/main/subs/all.base64.txt` |
| [`top150.txt`](subs/top150.txt) | top 150 ranked (150) | `https://raw.githubusercontent.com/onlyactions/proxy-scanner/main/subs/top150.txt` |
| [`top150.base64.txt`](subs/top150.base64.txt) | same, base64 | `https://raw.githubusercontent.com/onlyactions/proxy-scanner/main/subs/top150.base64.txt` |
| [`by-proto/`](subs/by-proto/) | one list per protocol (6 protocols) | ...`/subs/by-proto/<proto>.txt` |
| [`by-engine/allxray.txt`](subs/by-engine/allxray.txt) | verified on Xray (970) | `https://raw.githubusercontent.com/onlyactions/proxy-scanner/main/subs/by-engine/allxray.txt` |
| [`by-engine/allsingbox.txt`](subs/by-engine/allsingbox.txt) | verified on sing-box (129) | `https://raw.githubusercontent.com/onlyactions/proxy-scanner/main/subs/by-engine/allsingbox.txt` |
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

The typical aggregator relists whatever it scraped, so 80-95% of entries are dead within hours — this run found **75% unreachable** and **76% unusable** overall. Only tunnels that completed a real HTTPS handshake, returned a verifiable foreign exit IP, and pushed a 1 MB download are counted as working here.

## Disclaimers

- Free proxies are **public and untrusted**. Anything you send through them can be observed or modified by their operators. Never use them for authentication, payments, or anything sensitive.
- All configs are collected from publicly posted sources (GitHub subscriptions and public Telegram channels); no credentials are guessed or brute-forced. Removal requests: open an issue.
- Availability is transient: the working list is re-verified daily, and past performance never guarantees future availability.

