# Free Proxy Scanner

> **Really tested** free proxy aggregator. Every proxy below was fetched from public sources, tunneled through with Xray/sing-box, and verified to actually change your egress IP. No CSV-validation theater, no blind relisting.

![working](badges/working.json) ![alive](badges/alive.json) ![speed](badges/speed.json) ![max-speed](badges/max-speed.json) ![countries](badges/countries.json) ![sources](badges/sources.json) ![updated](badges/updated.json)

## What this is

A GitHub Actions pipeline (runs once a day) that fetches free proxy configs from **173 URL subscriptions** and **81 Telegram channels**, parses 9 protocols (vless / vmess / trojan / ss / hysteria2 / hysteria / tuic / socks / http), deduplicates them, and **actually connects through every single one** to separate working proxies from dead ones.

Latest run: **2026-11-06 10:02:03 UTC** — 178576 unique proxies collected from 173/173 URL sources and 81/81 Telegram channels; **4846** endpoints really tested, **900** reachable, **837** verified working (egress IP confirmed changed) across **49 exit countries**.

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

All 837 working proxies are published in [`subs/all.txt`](subs/all.txt); this table shows the top 25.

| # | proxy | proto | server | exit | latency | speed | engine |
|---|-------|-------|--------|------|---------|-------|--------|
| 1 | #1 ❓ ?? → 🇺🇸 US | `trojan` | `216.152.155.131:443` | United States | 226 ms | 49.1 Mbps | xray |
| 2 | #2 🇺🇸 US → 🇺🇸 US | `ss` | `156.146.38.167:443` | United States | 149 ms | 47.0 Mbps | xray |
| 3 | #3 ❓ ?? → 🇺🇸 US | `ss` | `216.106.178.169:19999` | United States | 246 ms | 46.6 Mbps | xray |
| 4 | #4 ❓ ?? → 🇺🇸 US | `ss` | `38.64.138.135:19999` | United States | 59 ms | 42.0 Mbps | xray |
| 5 | #5 ❓ ?? → 🇺🇸 US | `ss` | `167.88.62.203:19999` | United States | 53 ms | 39.3 Mbps | xray |
| 6 | #6 ❓ ?? → 🇺🇸 US | `vmess` | `38.64.138.187:1270` | United States | 56 ms | 38.6 Mbps | xray |
| 7 | #7 ❓ ?? → 🇺🇸 US | `vmess` | `162.251.61.46:1270` | United States | 75 ms | 36.5 Mbps | xray |
| 8 | #8 🇺🇸 US → 🇺🇸 US | `ss` | `156.146.38.170:443` | United States | 58 ms | 36.2 Mbps | xray |
| 9 | #9 ❓ ?? → 🇺🇸 US | `ss` | `167.17.67.171:19999` | United States | 73 ms | 35.5 Mbps | xray |
| 10 | #10 ❓ ?? → 🇺🇸 US | `trojan` | `216.152.155.176:443` | United States | 222 ms | 35.3 Mbps | xray |
| 11 | #11 ❓ ?? → 🇺🇸 US | `vmess` | `167.88.62.51:1270` | United States | 76 ms | 34.9 Mbps | xray |
| 12 | #12 ❓ ?? → 🇺🇸 US | `vmess` | `167.88.62.56:1270` | United States | 72 ms | 34.6 Mbps | xray |
| 13 | #13 ❓ ?? → 🇺🇸 US | `vmess` | `38.64.138.135:1270` | United States | 88 ms | 34.1 Mbps | xray |
| 14 | #14 ❓ ?? → 🇺🇸 US | `ss` | `167.88.62.56:19999` | United States | 60 ms | 32.9 Mbps | xray |
| 15 | #15 🇺🇸 US → 🇺🇸 US | `vless` | `195.211.98.43:443` | United States | 279 ms | 28.7 Mbps | xray |
| 16 | #16 ❓ ?? → 🇺🇸 US | `ss` | `172.111.38.72:19999` | United States | 165 ms | 28.2 Mbps | xray |
| 17 | #17 ❓ ?? → 🇺🇸 US | `trojan` | `66.163.114.114:443` | United States | 472 ms | 27.3 Mbps | xray |
| 18 | #18 🇨🇦 CA → 🇦🇺 AU | `vmess` | `23.162.200.198:18000` | Australia | 282 ms | 26.7 Mbps | xray |
| 19 | #19 ❓ ?? → 🇺🇸 US | `ss` | `38.128.66.32:19999` | United States | 95 ms | 26.5 Mbps | xray |
| 20 | #20 ❓ ?? → 🇺🇸 US | `trojan` | `170.101.101.53:443` | United States | 354 ms | 26.2 Mbps | xray |
| 21 | #21 ❓ ?? → 🇺🇸 US | `ss` | `193.151.163.18:19999` | United States | 154 ms | 25.9 Mbps | xray |
| 22 | #22 ❓ ?? → 🇺🇸 US | `ss` | `38.91.100.39:19999` | United States | 138 ms | 25.8 Mbps | xray |
| 23 | #23 🇺🇸 US → 🇺🇸 US | `ss` | `15.204.246.132:7307` | United States | 134 ms | 25.4 Mbps | xray |
| 24 | #24 ❓ ?? → 🇺🇸 US | `ss` | `38.22.17.219:19999` | United States | 254 ms | 25.3 Mbps | xray |
| 25 | #25 ❓ ?? → 🇺🇸 US | `ss` | `38.121.43.231:19999` | United States | 285 ms | 25.1 Mbps | xray |

## Exit locations — which countries work

Every working proxy, grouped by the country of its **verified exit IP** (phase 2), with a dedicated subscription file per country. Currently 49 countries have at least one working exit.

| country | working | avg speed | best latency | share | list |
|---------|--------:|----------:|-------------:|------:|------|
| 🇺🇸 United States | 231 | 16.7 Mbps | 53 ms | 28% | [`US.txt`](subs/by-country/US.txt) |
| 🇳🇱 Netherlands | 125 | 6.3 Mbps | 361 ms | 15% | [`NL.txt`](subs/by-country/NL.txt) |
| 🇩🇪 Germany | 62 | 6.1 Mbps | 366 ms | 7% | [`DE.txt`](subs/by-country/DE.txt) |
| 🇸🇬 Singapore | 48 | 3.0 Mbps | 596 ms | 6% | [`SG.txt`](subs/by-country/SG.txt) |
| 🇯🇵 Japan | 46 | 3.3 Mbps | 420 ms | 5% | [`JP.txt`](subs/by-country/JP.txt) |
| 🇨🇦 Canada | 38 | 5.1 Mbps | 191 ms | 5% | [`CA.txt`](subs/by-country/CA.txt) |
| 🇫🇷 France | 31 | 5.4 Mbps | 365 ms | 4% | [`FR.txt`](subs/by-country/FR.txt) |
| 🇬🇧 United Kingdom | 27 | 5.4 Mbps | 342 ms | 3% | [`GB.txt`](subs/by-country/GB.txt) |
| 🇵🇱 Poland | 24 | 4.9 Mbps | 432 ms | 3% | [`PL.txt`](subs/by-country/PL.txt) |
| 🇮🇹 Italy | 17 | 6.1 Mbps | 402 ms | 2% | [`IT.txt`](subs/by-country/IT.txt) |
| 🇰🇷 South Korea | 16 | 2.9 Mbps | 709 ms | 2% | [`KR.txt`](subs/by-country/KR.txt) |
| 🇸🇪 Sweden | 14 | 4.5 Mbps | 421 ms | 2% | [`SE.txt`](subs/by-country/SE.txt) |
| 🇪🇸 Spain | 13 | 4.9 Mbps | 489 ms | 2% | [`ES.txt`](subs/by-country/ES.txt) |
| 🇹🇷 Turkey | 11 | 4.2 Mbps | 761 ms | 1% | [`TR.txt`](subs/by-country/TR.txt) |
| 🇫🇮 Finland | 11 | 2.9 Mbps | 657 ms | 1% | [`FI.txt`](subs/by-country/FI.txt) |
| 🇦🇺 Australia | 10 | 10.8 Mbps | 109 ms | 1% | [`AU.txt`](subs/by-country/AU.txt) |
| 🇨🇭 Switzerland | 9 | 4.1 Mbps | 401 ms | 1% | [`CH.txt`](subs/by-country/CH.txt) |
| 🇲🇩 Moldova | 9 | 2.2 Mbps | 919 ms | 1% | [`MD.txt`](subs/by-country/MD.txt) |
| 🇭🇰 Hong Kong | 8 | 3.0 Mbps | 726 ms | 1% | [`HK.txt`](subs/by-country/HK.txt) |
| 🇮🇳 India | 8 | 2.8 Mbps | 1001 ms | 1% | [`IN.txt`](subs/by-country/IN.txt) |
| 🇷🇸 Serbia | 8 | 2.3 Mbps | 1076 ms | 1% | [`RS.txt`](subs/by-country/RS.txt) |
| 🇷🇴 Romania | 7 | 2.9 Mbps | 720 ms | 1% | [`RO.txt`](subs/by-country/RO.txt) |
| 🇧🇬 Bulgaria | 7 | 2.9 Mbps | 884 ms | 1% | [`BG.txt`](subs/by-country/BG.txt) |
| 🇧🇾 BY | 6 | 5.6 Mbps | 553 ms | 1% | [`BY.txt`](subs/by-country/BY.txt) |
| 🇮🇷 Iran | 6 | 3.0 Mbps | 1289 ms | 1% | [`IR.txt`](subs/by-country/IR.txt) |
| 🇱🇻 Latvia | 4 | 4.4 Mbps | 625 ms | 0% | [`LV.txt`](subs/by-country/LV.txt) |
| 🇦🇹 Austria | 3 | 5.1 Mbps | 418 ms | 0% | [`AT.txt`](subs/by-country/AT.txt) |
| 🇨🇿 Czechia | 3 | 5.3 Mbps | 586 ms | 0% | [`CZ.txt`](subs/by-country/CZ.txt) |
| 🇺🇦 Ukraine | 3 | 3.5 Mbps | 743 ms | 0% | [`UA.txt`](subs/by-country/UA.txt) |
| 🇿🇦 South Africa | 3 | 2.3 Mbps | 886 ms | 0% | [`ZA.txt`](subs/by-country/ZA.txt) |
| 🇰🇿 Kazakhstan | 3 | 3.1 Mbps | 1102 ms | 0% | [`KZ.txt`](subs/by-country/KZ.txt) |
| 🇹🇼 Taiwan | 3 | 1.2 Mbps | 694 ms | 0% | [`TW.txt`](subs/by-country/TW.txt) |
| 🇲🇽 Mexico | 2 | 13.4 Mbps | 335 ms | 0% | [`MX.txt`](subs/by-country/MX.txt) |
| 🇷🇺 Russia | 2 | 3.0 Mbps | 4986 ms | 0% | [`RU.txt`](subs/by-country/RU.txt) |
| 🇮🇩 Indonesia | 2 | 2.8 Mbps | 922 ms | 0% | [`ID.txt`](subs/by-country/ID.txt) |
| 🇦🇱 AL | 2 | 0.6 Mbps | 687 ms | 0% | [`AL.txt`](subs/by-country/AL.txt) |
| 🇻🇳 Vietnam | 2 | - | 1085 ms | 0% | [`VN.txt`](subs/by-country/VN.txt) |
| 🇦🇪 UAE | 2 | - | 2055 ms | 0% | [`AE.txt`](subs/by-country/AE.txt) |
| 🇮🇱 Israel | 1 | 5.9 Mbps | 630 ms | 0% | [`IL.txt`](subs/by-country/IL.txt) |
| 🇧🇷 Brazil | 1 | 5.8 Mbps | 652 ms | 0% | [`BR.txt`](subs/by-country/BR.txt) |
| 🇱🇹 Lithuania | 1 | 5.2 Mbps | 825 ms | 0% | [`LT.txt`](subs/by-country/LT.txt) |
| 🇪🇪 Estonia | 1 | 4.5 Mbps | 638 ms | 0% | [`EE.txt`](subs/by-country/EE.txt) |
| 🇹🇭 Thailand | 1 | 4.0 Mbps | 870 ms | 0% | [`TH.txt`](subs/by-country/TH.txt) |
| 🇶🇦 Qatar | 1 | 3.8 Mbps | 1047 ms | 0% | [`QA.txt`](subs/by-country/QA.txt) |
| 🇮🇪 Ireland | 1 | 2.6 Mbps | 6795 ms | 0% | [`IE.txt`](subs/by-country/IE.txt) |
| 🇵🇭 Philippines | 1 | 0.9 Mbps | 1938 ms | 0% | [`PH.txt`](subs/by-country/PH.txt) |
| 🇳🇴 Norway | 1 | - | 469 ms | 0% | [`NO.txt`](subs/by-country/NO.txt) |
| 🇸🇦 Saudi Arabia | 1 | - | 1063 ms | 0% | [`SA.txt`](subs/by-country/SA.txt) |
| 🇲🇾 Malaysia | 1 | - | 1339 ms | 0% | [`MY.txt`](subs/by-country/MY.txt) |

A `??` exit means the geo lookup could not resolve the exit IP. Base64 variants of every country file sit next to them as `<CC>.base64.txt`.

## Protocol distribution

| protocol | tested | alive | working | alive rate |
|----------|-------:|------:|--------:|-----------:|
| `vless` | 3091 | 411 | 368 | 13% |
| `ss` | 607 | 243 | 232 | 40% |
| `vmess` | 868 | 113 | 109 | 13% |
| `trojan` | 197 | 92 | 89 | 47% |
| `hysteria2` | 77 | 40 | 39 | 52% |
| `http` | 5 | 0 | 0 | 0% |
| `socks` | 1 | 1 | 0 | 100% |

Per-protocol working lists: [`subs/by-proto/`](subs/by-proto/) (one `vless.txt`, `vmess.txt`, ... per protocol, plus `.base64.txt` variants).

## Engine breakdown

Which core verified each working proxy. The engine fallback means a proxy that failed on its primary core got a second chance on the other one.

| engine | working | note |
|--------|--------:|------|
| Xray-core | 743 | [`allxray.txt`](subs/by-engine/allxray.txt) |
| sing-box | 94 | [`allsingbox.txt`](subs/by-engine/allsingbox.txt) |
| direct (curl) | 0 | plain http/socks, [`alldirect.txt`](subs/by-engine/alldirect.txt) |

## Speed & latency profile

| speed tier | proxies |
|-----------|--------:|
| >= 5 Mbps | 392 |
| 1 - 5 Mbps | 192 |
| 0.25 - 1 Mbps | 43 |
| < 0.25 Mbps | 0 |
| unmeasured | 210 |

Latency (phase-1 HTTPS round trip): median **577 ms**, p90 **3287 ms**. 63 proxies passed connectivity but failed egress verification (leaked the runner IP or broke on the exit check) and are excluded.

## Source health

254 active, 0 auto-retired (10 fetch failures / 10 all-dead runs / 10 days without anything new). Best contributors this run:

| source | kind | tested | alive | new this run |
|--------|------|-------:|------:|-------------:|
| `EV` | url | 647 | 501 | 1100 |
| `SK` | url | 837 | 456 | 8776 |
| `NK` | url | 825 | 441 | 201 |
| `HC` | url | 1303 | 267 | 267 |
| `AG-PL` | url | 425 | 257 | 969 |
| `RD-VL` | url | 2267 | 244 | 3667 |
| `RD-SS` | url | 516 | 229 | 379 |
| `Ni` | url | 346 | 221 | 224 |
| `F0` | url | 223 | 210 | 4 |
| `EV-SS` | url | 225 | 205 | 0 |
| `ST-VL` | url | 2175 | 202 | 716 |
| `EV-VL` | url | 280 | 185 | 0 |
| `ET` | url | 283 | 183 | 1959 |
| `KA` | url | 2696 | 169 | 3785 |
| `SB` | url | 2233 | 162 | 33 |
| `EP-ALL` | url | 1325 | 124 | 0 |
| `SB-VL` | url | 1983 | 124 | 0 |
| `RD-VM` | url | 624 | 108 | 3458 |
| `MS-C` | url | 158 | 107 | 152 |
| `LA` | url | 711 | 105 | 15147 |

## History

working per run (last 19): ▁▁▆▆▇█▇▆▇█▇▆▇▆▇▇█▇▇

| date (UTC) | tested | alive | working | avg | max |
|------------|-------:|------:|--------:|----:|----:|
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
| 2026-09-29 21:53 | 4500 | 880 | 852 | 6.5M | 61.8M |

## Subscription files

Everything under [`subs/`](subs/) is regenerated every run. Plain lists are one proxy URI per line; `.base64.txt` variants are the same list base64-encoded for clients that need the classic sub format.

| file | contents | direct subscription URL |
|------|----------|--------------------------|
| [`all.txt`](subs/all.txt) | ALL working proxies, ranked by speed (837) | `https://raw.githubusercontent.com/onlyactions/proxy-scanner/main/subs/all.txt` |
| [`all.base64.txt`](subs/all.base64.txt) | same, base64 | `https://raw.githubusercontent.com/onlyactions/proxy-scanner/main/subs/all.base64.txt` |
| [`top150.txt`](subs/top150.txt) | top 150 ranked (150) | `https://raw.githubusercontent.com/onlyactions/proxy-scanner/main/subs/top150.txt` |
| [`top150.base64.txt`](subs/top150.base64.txt) | same, base64 | `https://raw.githubusercontent.com/onlyactions/proxy-scanner/main/subs/top150.base64.txt` |
| [`by-proto/`](subs/by-proto/) | one list per protocol (5 protocols) | ...`/subs/by-proto/<proto>.txt` |
| [`by-engine/allxray.txt`](subs/by-engine/allxray.txt) | verified on Xray (743) | `https://raw.githubusercontent.com/onlyactions/proxy-scanner/main/subs/by-engine/allxray.txt` |
| [`by-engine/allsingbox.txt`](subs/by-engine/allsingbox.txt) | verified on sing-box (94) | `https://raw.githubusercontent.com/onlyactions/proxy-scanner/main/subs/by-engine/allsingbox.txt` |
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

The typical aggregator relists whatever it scraped, so 80-95% of entries are dead within hours — this run found **81% unreachable** and **83% unusable** overall. Only tunnels that completed a real HTTPS handshake, returned a verifiable foreign exit IP, and pushed a 1 MB download are counted as working here.

## Disclaimers

- Free proxies are **public and untrusted**. Anything you send through them can be observed or modified by their operators. Never use them for authentication, payments, or anything sensitive.
- All configs are collected from publicly posted sources (GitHub subscriptions and public Telegram channels); no credentials are guessed or brute-forced. Removal requests: open an issue.
- Availability is transient: the working list is re-verified daily, and past performance never guarantees future availability.

