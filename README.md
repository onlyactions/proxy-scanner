# Free Proxy Scanner

> **Really tested** free proxy aggregator. Every proxy below was fetched from public sources, tunneled through with Xray/sing-box, and verified to actually change your egress IP. No CSV-validation theater, no blind relisting.

![working](badges/working.json) ![alive](badges/alive.json) ![speed](badges/speed.json) ![max-speed](badges/max-speed.json) ![countries](badges/countries.json) ![sources](badges/sources.json) ![updated](badges/updated.json)

## What this is

A GitHub Actions pipeline (runs once a day) that fetches free proxy configs from **173 URL subscriptions** and **81 Telegram channels**, parses 9 protocols (vless / vmess / trojan / ss / hysteria2 / hysteria / tuic / socks / http), deduplicates them, and **actually connects through every single one** to separate working proxies from dead ones.

Latest run: **2026-11-19 02:58:06 UTC** — 126201 unique proxies collected from 107/107 URL sources and 9/9 Telegram channels; **4520** endpoints really tested, **1168** reachable, **1134** verified working (egress IP confirmed changed) across **48 exit countries**.

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

All 1134 working proxies are published in [`subs/all.txt`](subs/all.txt); this table shows the top 25.

| # | proxy | proto | server | exit | latency | speed | engine |
|---|-------|-------|--------|------|---------|-------|--------|
| 1 | #1 ❓ ?? → 🇺🇸 US | `ss` | `172.111.38.72:19999` | United States | 63 ms | 73.9 Mbps | xray |
| 2 | #2 ❓ ?? → 🇺🇸 US | `vmess` | `172.111.38.246:1270` | United States | 80 ms | 73.2 Mbps | xray |
| 3 | #3 🇺🇸 US → 🇺🇸 US | `ss` | `37.19.198.236:443` | United States | 109 ms | 72.1 Mbps | xray |
| 4 | #4 ❓ ?? → 🇺🇸 US | `ss` | `40.160.86.43:8388` | United States | 89 ms | 61.7 Mbps | xray |
| 5 | #5 ❓ ?? → 🇺🇸 US | `ss` | `138.197.89.234:28960` | United States | 109 ms | 57.6 Mbps | xray |
| 6 | #6 ❓ ?? → 🇺🇸 US | `ss` | `193.151.163.18:19999` | United States | 70 ms | 55.7 Mbps | xray |
| 7 | #7 🇺🇸 US → 🇺🇸 US | `ss` | `37.19.198.244:443` | United States | 42 ms | 53.9 Mbps | xray |
| 8 | #8 🇺🇸 US → 🇺🇸 US | `vless` | `169.40.42.163:443` | United States | 131 ms | 53.7 Mbps | xray |
| 9 | #9 ❓ ?? → 🇺🇸 US | `vless` | `slot4.ipoxxo.online:443` | United States | 38 ms | 52.9 Mbps | xray |
| 10 | #10 🇺🇸 US → 🇺🇸 US | `ss` | `37.19.198.160:443` | United States | 35 ms | 52.7 Mbps | xray |
| 11 | #11 🇺🇸 US → 🇺🇸 US | `ss` | `37.19.198.243:443` | United States | 34 ms | 50.7 Mbps | xray |
| 12 | #12 ❓ ?? → 🇺🇸 US | `vless` | `45.144.112.55:443` | United States | 135 ms | 49.0 Mbps | xray |
| 13 | #13 ❓ ?? → 🇺🇸 US | `vless` | `45.138.100.248:14430` | United States | 90 ms | 48.1 Mbps | singbox |
| 14 | #14 🇺🇸 US → 🇺🇸 US | `vless` | `169.40.42.90:443` | United States | 137 ms | 45.2 Mbps | xray |
| 15 | #15 ❓ ?? → 🇺🇸 US | `ss` | `149.40.57.134:19999` | United States | 71 ms | 43.6 Mbps | xray |
| 16 | #16 🇺🇸 US → 🇺🇸 US | `vless` | `159.89.87.21:28190` | United States | 48 ms | 43.4 Mbps | xray |
| 17 | #17 ❓ ?? → 🇺🇸 US | `ss` | `135.148.42.13:8388` | United States | 24 ms | 41.1 Mbps | xray |
| 18 | #18 ❓ ?? → 🇺🇸 US | `ss` | `149.40.57.138:19999` | United States | 58 ms | 40.7 Mbps | xray |
| 19 | #19 ❓ ?? → 🇨🇦 CA | `trojan` | `216.152.147.74:443` | Canada | 104 ms | 39.6 Mbps | xray |
| 20 | #20 ❓ ?? → 🇨🇦 CA | `ss` | `15.235.57.6:8080` | Canada | 280 ms | 39.6 Mbps | xray |
| 21 | #21 ❓ ?? → 🇺🇸 US | `vless` | `185.95.231.156:443` | United States | 163 ms | 39.1 Mbps | xray |
| 22 | #22 ❓ ?? → 🇺🇸 US | `vmess` | `142.202.48.87:1270` | United States | 118 ms | 38.9 Mbps | xray |
| 23 | #23 🇺🇸 US → 🇺🇸 US | `ss` | `15.204.247.124:2375` | United States | 91 ms | 38.7 Mbps | xray |
| 24 | #24 ❓ ?? → 🇺🇸 US | `ss` | `136.148.77.250:8388` | United States | 120 ms | 38.5 Mbps | xray |
| 25 | #25 ❓ ?? → 🇺🇸 US | `ss` | `149.40.57.135:19999` | United States | 59 ms | 38.4 Mbps | xray |

## Exit locations — which countries work

Every working proxy, grouped by the country of its **verified exit IP** (phase 2), with a dedicated subscription file per country. Currently 48 countries have at least one working exit.

| country | working | avg speed | best latency | share | list |
|---------|--------:|----------:|-------------:|------:|------|
| 🇺🇸 United States | 337 | 18.2 Mbps | 24 ms | 30% | [`US.txt`](subs/by-country/US.txt) |
| 🇳🇱 Netherlands | 171 | 7.7 Mbps | 250 ms | 15% | [`NL.txt`](subs/by-country/NL.txt) |
| 🇩🇪 Germany | 136 | 6.1 Mbps | 269 ms | 12% | [`DE.txt`](subs/by-country/DE.txt) |
| 🇯🇵 Japan | 85 | 1.9 Mbps | 504 ms | 7% | [`JP.txt`](subs/by-country/JP.txt) |
| 🇨🇦 Canada | 50 | 15.2 Mbps | 55 ms | 4% | [`CA.txt`](subs/by-country/CA.txt) |
| 🇫🇷 France | 40 | 7.0 Mbps | 274 ms | 4% | [`FR.txt`](subs/by-country/FR.txt) |
| 🇬🇧 United Kingdom | 30 | 7.8 Mbps | 255 ms | 3% | [`GB.txt`](subs/by-country/GB.txt) |
| 🇪🇸 Spain | 29 | 5.5 Mbps | 282 ms | 3% | [`ES.txt`](subs/by-country/ES.txt) |
| 🇸🇬 Singapore | 29 | 2.4 Mbps | 728 ms | 3% | [`SG.txt`](subs/by-country/SG.txt) |
| 🇫🇮 Finland | 25 | 6.5 Mbps | 384 ms | 2% | [`FI.txt`](subs/by-country/FI.txt) |
| 🇦🇺 Australia | 17 | 11.7 Mbps | 63 ms | 1% | [`AU.txt`](subs/by-country/AU.txt) |
| 🇸🇪 Sweden | 17 | 6.1 Mbps | 339 ms | 1% | [`SE.txt`](subs/by-country/SE.txt) |
| 🇭🇰 Hong Kong | 16 | 4.3 Mbps | 190 ms | 1% | [`HK.txt`](subs/by-country/HK.txt) |
| 🇮🇹 Italy | 16 | 7.5 Mbps | 320 ms | 1% | [`IT.txt`](subs/by-country/IT.txt) |
| 🇨🇭 Switzerland | 13 | 6.5 Mbps | 347 ms | 1% | [`CH.txt`](subs/by-country/CH.txt) |
| 🇷🇺 Russia | 12 | 5.5 Mbps | 373 ms | 1% | [`RU.txt`](subs/by-country/RU.txt) |
| 🇵🇱 Poland | 11 | 5.6 Mbps | 438 ms | 1% | [`PL.txt`](subs/by-country/PL.txt) |
| 🇧🇾 BY | 9 | 6.3 Mbps | 471 ms | 1% | [`BY.txt`](subs/by-country/BY.txt) |
| 🇰🇷 South Korea | 9 | 3.5 Mbps | 700 ms | 1% | [`KR.txt`](subs/by-country/KR.txt) |
| 🇹🇷 Turkey | 8 | 3.9 Mbps | 529 ms | 1% | [`TR.txt`](subs/by-country/TR.txt) |
| 🇰🇿 Kazakhstan | 8 | 3.0 Mbps | 982 ms | 1% | [`KZ.txt`](subs/by-country/KZ.txt) |
| 🇦🇱 AL | 8 | - | 454 ms | 1% | [`AL.txt`](subs/by-country/AL.txt) |
| 🇹🇼 Taiwan | 7 | 3.4 Mbps | 798 ms | 1% | [`TW.txt`](subs/by-country/TW.txt) |
| 🇷🇴 Romania | 5 | 6.4 Mbps | 570 ms | 0% | [`RO.txt`](subs/by-country/RO.txt) |
| 🇮🇳 India | 4 | 1.7 Mbps | 854 ms | 0% | [`IN.txt`](subs/by-country/IN.txt) |
| 🇦🇪 UAE | 4 | 1.0 Mbps | 911 ms | 0% | [`AE.txt`](subs/by-country/AE.txt) |
| 🇳🇬 NG | 4 | - | 2624 ms | 0% | [`NG.txt`](subs/by-country/NG.txt) |
| 🇨🇿 Czechia | 3 | 6.6 Mbps | 484 ms | 0% | [`CZ.txt`](subs/by-country/CZ.txt) |
| 🇦🇹 Austria | 3 | 7.5 Mbps | 355 ms | 0% | [`AT.txt`](subs/by-country/AT.txt) |
| 🇱🇹 Lithuania | 3 | 5.8 Mbps | 523 ms | 0% | [`LT.txt`](subs/by-country/LT.txt) |
| 🇧🇷 Brazil | 3 | 6.3 Mbps | 552 ms | 0% | [`BR.txt`](subs/by-country/BR.txt) |
| 🇿🇦 South Africa | 3 | 3.4 Mbps | 823 ms | 0% | [`ZA.txt`](subs/by-country/ZA.txt) |
| 🇪🇪 Estonia | 2 | 4.6 Mbps | 342 ms | 0% | [`EE.txt`](subs/by-country/EE.txt) |
| 🇺🇦 Ukraine | 2 | 4.9 Mbps | 2090 ms | 0% | [`UA.txt`](subs/by-country/UA.txt) |
| 🇲🇾 Malaysia | 2 | - | 1303 ms | 0% | [`MY.txt`](subs/by-country/MY.txt) |
| 🇲🇽 Mexico | 1 | 12.5 Mbps | 262 ms | 0% | [`MX.txt`](subs/by-country/MX.txt) |
| 🇱🇻 Latvia | 1 | 7.2 Mbps | 887 ms | 0% | [`LV.txt`](subs/by-country/LV.txt) |
| 🇳🇴 Norway | 1 | 7.2 Mbps | 6423 ms | 0% | [`NO.txt`](subs/by-country/NO.txt) |
| 🇨🇾 Cyprus | 1 | 6.7 Mbps | 1344 ms | 0% | [`CY.txt`](subs/by-country/CY.txt) |
| 🇧🇬 Bulgaria | 1 | 6.3 Mbps | 595 ms | 0% | [`BG.txt`](subs/by-country/BG.txt) |
| 🇨🇱 Chile | 1 | 6.3 Mbps | 684 ms | 0% | [`CL.txt`](subs/by-country/CL.txt) |
| 🇮🇷 Iran | 1 | 5.8 Mbps | 716 ms | 0% | [`IR.txt`](subs/by-country/IR.txt) |
| 🇬🇷 Greece | 1 | 5.7 Mbps | 620 ms | 0% | [`GR.txt`](subs/by-country/GR.txt) |
| 🇹🇭 Thailand | 1 | 3.3 Mbps | 6023 ms | 0% | [`TH.txt`](subs/by-country/TH.txt) |
| 🇻🇳 Vietnam | 1 | 1.6 Mbps | 1245 ms | 0% | [`VN.txt`](subs/by-country/VN.txt) |
| 🇮🇩 Indonesia | 1 | 0.7 Mbps | 2732 ms | 0% | [`ID.txt`](subs/by-country/ID.txt) |
| 🇮🇪 Ireland | 1 | - | 210 ms | 0% | [`IE.txt`](subs/by-country/IE.txt) |
| 🇧🇭 BH | 1 | - | 2916 ms | 0% | [`BH.txt`](subs/by-country/BH.txt) |

A `??` exit means the geo lookup could not resolve the exit IP. Base64 variants of every country file sit next to them as `<CC>.base64.txt`.

## Protocol distribution

| protocol | tested | alive | working | alive rate |
|----------|-------:|------:|--------:|-----------:|
| `vless` | 2782 | 579 | 548 | 21% |
| `ss` | 570 | 270 | 270 | 47% |
| `vmess` | 954 | 164 | 162 | 17% |
| `trojan` | 151 | 119 | 119 | 79% |
| `hysteria2` | 59 | 34 | 33 | 58% |
| `socks` | 2 | 2 | 2 | 100% |
| `http` | 2 | 0 | 0 | 0% |

Per-protocol working lists: [`subs/by-proto/`](subs/by-proto/) (one `vless.txt`, `vmess.txt`, ... per protocol, plus `.base64.txt` variants).

## Engine breakdown

Which core verified each working proxy. The engine fallback means a proxy that failed on its primary core got a second chance on the other one.

| engine | working | note |
|--------|--------:|------|
| Xray-core | 997 | [`allxray.txt`](subs/by-engine/allxray.txt) |
| sing-box | 135 | [`allsingbox.txt`](subs/by-engine/allsingbox.txt) |
| direct (curl) | 2 | plain http/socks, [`alldirect.txt`](subs/by-engine/alldirect.txt) |

## Speed & latency profile

| speed tier | proxies |
|-----------|--------:|
| >= 5 Mbps | 684 |
| 1 - 5 Mbps | 260 |
| 0.25 - 1 Mbps | 63 |
| < 0.25 Mbps | 0 |
| unmeasured | 127 |

Latency (phase-1 HTTPS round trip): median **435 ms**, p90 **3410 ms**. 34 proxies passed connectivity but failed egress verification (leaked the runner IP or broke on the exit check) and are excluded.

## Source health

116 active, 138 auto-retired (10 fetch failures / 10 all-dead runs / 10 days without anything new). Best contributors this run:

| source | kind | tested | alive | new this run |
|--------|------|-------:|------:|-------------:|
| `SK` | url | 1134 | 695 | 6954 |
| `EV` | url | 777 | 642 | 825 |
| `NK` | url | 1020 | 588 | 0 |
| `HP` | url | 533 | 470 | 79 |
| `AG-PL` | url | 548 | 370 | 722 |
| `RD-VL` | url | 2282 | 330 | 3637 |
| `HC` | url | 1235 | 298 | 267 |
| `RD-SS` | url | 517 | 270 | 600 |
| `F0` | url | 268 | 255 | 12 |
| `Ni` | url | 313 | 253 | 76 |
| `EV-VL` | url | 349 | 252 | 0 |
| `EV-SS` | url | 264 | 243 | 0 |
| `ST-VL` | url | 2100 | 197 | 642 |
| `ET` | url | 248 | 172 | 827 |
| `RD-VM` | url | 596 | 144 | 3811 |
| `KA` | url | 2520 | 137 | 5650 |
| `ST-VM` | url | 596 | 127 | 38 |
| `LA` | url | 671 | 112 | 14481 |
| `SB` | url | 2049 | 101 | 33 |
| `SB-VL` | url | 1902 | 92 | 0 |

## History

working per run (last 30): ▁▁▆▆▆▇▆▆▆█▇▆▆▆▆▆█▆▆▇██████████

| date (UTC) | tested | alive | working | avg | max |
|------------|-------:|------:|--------:|----:|----:|
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
| 2026-10-02 21:49 | 4500 | 890 | 861 | 8.9M | 68.7M |

## Subscription files

Everything under [`subs/`](subs/) is regenerated every run. Plain lists are one proxy URI per line; `.base64.txt` variants are the same list base64-encoded for clients that need the classic sub format.

| file | contents | direct subscription URL |
|------|----------|--------------------------|
| [`all.txt`](subs/all.txt) | ALL working proxies, ranked by speed (1134) | `https://raw.githubusercontent.com/onlyactions/proxy-scanner/main/subs/all.txt` |
| [`all.base64.txt`](subs/all.base64.txt) | same, base64 | `https://raw.githubusercontent.com/onlyactions/proxy-scanner/main/subs/all.base64.txt` |
| [`top150.txt`](subs/top150.txt) | top 150 ranked (150) | `https://raw.githubusercontent.com/onlyactions/proxy-scanner/main/subs/top150.txt` |
| [`top150.base64.txt`](subs/top150.base64.txt) | same, base64 | `https://raw.githubusercontent.com/onlyactions/proxy-scanner/main/subs/top150.base64.txt` |
| [`by-proto/`](subs/by-proto/) | one list per protocol (6 protocols) | ...`/subs/by-proto/<proto>.txt` |
| [`by-engine/allxray.txt`](subs/by-engine/allxray.txt) | verified on Xray (997) | `https://raw.githubusercontent.com/onlyactions/proxy-scanner/main/subs/by-engine/allxray.txt` |
| [`by-engine/allsingbox.txt`](subs/by-engine/allsingbox.txt) | verified on sing-box (135) | `https://raw.githubusercontent.com/onlyactions/proxy-scanner/main/subs/by-engine/allsingbox.txt` |
| [`by-country/`](subs/by-country/) | one list per exit country (48 countries, e.g. [`US.txt`](subs/by-country/US.txt), [`DE.txt`](subs/by-country/DE.txt)) | ...`/subs/by-country/<CC>.txt` |
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

The typical aggregator relists whatever it scraped, so 80-95% of entries are dead within hours — this run found **74% unreachable** and **75% unusable** overall. Only tunnels that completed a real HTTPS handshake, returned a verifiable foreign exit IP, and pushed a 1 MB download are counted as working here.

## Disclaimers

- Free proxies are **public and untrusted**. Anything you send through them can be observed or modified by their operators. Never use them for authentication, payments, or anything sensitive.
- All configs are collected from publicly posted sources (GitHub subscriptions and public Telegram channels); no credentials are guessed or brute-forced. Removal requests: open an issue.
- Availability is transient: the working list is re-verified daily, and past performance never guarantees future availability.

