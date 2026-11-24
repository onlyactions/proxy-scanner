# Free Proxy Scanner

> **Really tested** free proxy aggregator. Every proxy below was fetched from public sources, tunneled through with Xray/sing-box, and verified to actually change your egress IP. No CSV-validation theater, no blind relisting.

![working](badges/working.json) ![alive](badges/alive.json) ![speed](badges/speed.json) ![up-speed](badges/up-speed.json) ![max-speed](badges/max-speed.json) ![clean](badges/clean.json) ![countries](badges/countries.json) ![sources](badges/sources.json) ![updated](badges/updated.json)

## What this is

A GitHub Actions pipeline (runs daily) that fetches free proxy configs from **173 URL subscriptions** and **81 Telegram channels**, parses 9 protocols (vless / vmess / trojan / ss / hysteria2 / hysteria / tuic / socks / http), deduplicates them, and **actually connects through every single one** to separate working proxies from dead ones.

Latest run: **2026-11-24 14:31:23 UTC** — 124797 unique proxies collected from 106/106 URL sources and 9/9 Telegram channels; **4518** endpoints really tested, **1039** reachable, **979** verified working (egress IP confirmed changed) across **43 exit countries** — **310** of them on unflagged ('clean') exit IPs and **270** on residential IPs.

## How the testing works (the *really* part)

```mermaid
flowchart LR
  S1[URL subs<br/>+ Telegram] --> F[fetch &<br/>parse]
  F --> C[clean &<br/>dedupe]
  C --> T1[phase 1<br/>connectivity<br/>gstatic 204]
  T1 -->|reachable| T2[phase 2<br/>exit IP + ISP +<br/>reputation check]
  T1 -->|dead| X[drop]
  T2 -->|egress changed| T3[↓ 1 MB + ↑ 0.5 MB<br/>speed test]
  T2 -->|same IP| X2[mark leak]
  T3 --> R[README + subs<br/>+ badges]
```

1. **Phase 1 — connectivity.** Each proxy is loaded as an outbound into a real Xray or sing-box instance (or used directly via `curl -x` for plain http/socks). An HTTPS request to `gstatic.com/generate_204` goes through the tunnel with **DNS resolved on the proxy side**. Only an HTTP 204 counts as reachable.
2. **Phase 2 — egress verification.** An ip-api request through the same tunnel must return an exit IP **different from the runner's own IP**. The same lookup also returns the exit's **country, city, ISP, AS number and IP-reputation flags** (datacenter / residential / mobile, proxy-DB listing) — all of which end up in the proxy's name. Proxies that pass phase 1 but keep your IP are marked as leaks and excluded from the working list.
3. **Throughput, both directions.** Working proxies download 1 MB from and upload 512 KB to `speed.cloudflare.com` through the tunnel; the measured down/up rates are the `↓`/`↑` numbers you see in the names and tables.
4. **Engine fallback.** If a proxy fails on Xray, it is retried on sing-box (and vice versa) before being declared dead — some configs only work on one core.

## How to read a proxy name

Every proxy in the subscription files is named so that the node list in your client already tells you **exactly what you would be connecting to** — entry and exit country, the ISP and AS operating the **verified exit IP**, measured download **and upload** speed, the exit IP itself, and the IP's reputation:

```text
#42 🇺🇸 US → 🇩🇪 DE | Contabo AS141995 | ↓12.0 ↑1.2Mbps | 84.247.155.196 | DC·Clean
  │            │        │               │               │                │
 rank   entry→exit   ISP + AS   measured ↓/↑   verified exit   IP type·reputation
```

| tag | meaning |
|-----|---------|
| `↓` / `↑` | measured download / upload throughput through the live tunnel (Mbps); `?` = test failed or skipped
| `DC` | the exit IP belongs to a datacenter / hosting network
| `RES` | residential IP (not in hosting ranges — usually the best bet for not getting blocked as a bot/VPN)
| `MOB` | mobile / carrier IP (CGNAT — often the "cleanest" class for streaming and sign-ups)
| `Clean` | exit IP is **not** listed in ip-api's proxy/VPN/Tor database
| `Flag` | exit IP **is** listed as a proxy/VPN exit — expect streaming and anti-bot blocks
| `→` | entry country → exit country; a single flag means the tunnel exits where it enters (no cross-country relay)

So `#42 🇺🇸 US → 🇩🇪 DE | Contabo AS141995 | ↓12.0 ↑1.2Mbps | 84.247.155.196 | DC·Clean` reads as: rank 42, enter in the US, exit in Germany on a Contabo datacenter address (AS141995) at 12 Mbps down / 1.2 Mbps up, and the exit is not listed in proxy databases.

## Top working proxies

All 979 working proxies are published in [`subs/all.txt`](subs/all.txt); this table shows the top 25. The `name` column is exactly what your client will show (see [How to read a proxy name](#how-to-read-a-proxy-name)).

| # | name | proto | entry server | latency | engine |
|---|------|-------|--------------|---------|--------|
| 1 | #1 🇺🇸 US | Datacamp AS60068 | ↓60.1 ↑57.0Mbps | 84.17.41.2 | ? | `vmess` | `84.17.41.2:18000` | 67 ms | xray |
| 2 | #2 🇨🇦 CA | Datacamp AS212238 | ↓58.9 ↑49.1Mbps | 149.22.95.183 | DC·Flag | `ss` | `149.22.95.183:443` | 72 ms | xray |
| 3 | #3 🇺🇸 US | GTHost AS63023 | ↓58.6 ↑0.4Mbps | 38.143.66.146 | RES·Flag | `ss` | `38.143.66.146:19999` | 35 ms | xray |
| 4 | #4 🇺🇸 US | OVH AS16276 | ↓54.8 ↑19.8Mbps | 15.204.97.198 | DC·Flag | `vless` | `15.204.97.198:23576` | 69 ms | xray |
| 5 | #5 🇺🇸 US | OVH AS16276 | ↓54.5 ↑38.5Mbps | 15.204.97.197 | DC·Flag | `vless` | `15.204.97.197:23576` | 59 ms | xray |
| 6 | #6 🇺🇸 US | OVH AS16276 | ↓53.9 ↑29.9Mbps | 15.204.97.216 | DC·Flag | `vless` | `15.204.97.216:23576` | 60 ms | xray |
| 7 | #7 🇺🇸 US | OVH AS16276 | ↓53.7 ↑38.6Mbps | 15.204.97.195 | DC·Flag | `vless` | `15.204.97.195:23576` | 65 ms | xray |
| 8 | #8 🇺🇸 US | OVH AS16276 | ↓52.6 ↑38.5Mbps | 15.204.97.209 | DC·Flag | `vless` | `15.204.97.209:23576` | 61 ms | xray |
| 9 | #9 🇺🇸 US | OVH AS16276 | ↓50.4 ↑38.7Mbps | 15.204.97.214 | DC·Flag | `vless` | `15.204.97.214:23576` | 73 ms | xray |
| 10 | #10 🇺🇸 US | OVH AS16276 | ↓46.1 ↑39.9Mbps | 15.204.97.206 | DC·Flag | `vless` | `15.204.97.206:23576` | 65 ms | xray |
| 11 | #11 🇺🇸 US | OVH AS16276 | ↓45.5 ↑40.7Mbps | 15.204.97.219 | DC·Flag | `vless` | `15.204.97.219:23576` | 90 ms | xray |
| 12 | #12 🇺🇸 US | DigitalOcean AS14061 | ↓31.2 ↑4.9Mbps | 137.184.89.151 | DC·Flag | `ss` | `137.184.89.151:19999` | 111 ms | xray |
| 13 | #13 🇺🇸 US | Oracle AS31898 | ↓28.8 ↑10.2Mbps | 147.224.54.25 | DC·Clean | `vless` | `147.224.54.25:2053` | 138 ms | xray |
| 14 | #14 🇺🇸 US | GTHost AS63023 | ↓27.4 ↑13.3Mbps | 169.197.142.68 | DC·Flag | `vmess` | `169.197.142.68:1270` | 158 ms | xray |
| 15 | #15 🇺🇸 US | GTHost AS63023 | ↓27.3 ↑15.0Mbps | 169.197.142.68 | DC·Flag | `ss` | `169.197.142.68:19999` | 91 ms | xray |
| 16 | #16 🇺🇸 US | OVH AS16276 | ↓27.0 ↑33.0Mbps | 15.204.97.206 | DC·Flag | `vless` | `ww13.levikogjgfdd.ir:23576` | 285 ms | xray |
| 17 | #17 🇺🇸 US | GlobalTeleHost AS63023 | ↓27.0 ↑15.3Mbps | 167.88.61.20 | DC·Flag | `trojan` | `167.88.61.85:443` | 151 ms | xray |
| 18 | #18 🇺🇸 US | GlobalTeleHost AS63023 | ↓26.1 ↑12.8Mbps | 167.88.61.85 | DC·Clean | `trojan` | `167.88.61.175:443` | 147 ms | xray |
| 19 | #19 🇺🇸 US | GlobalTeleHost AS63023 | ↓25.2 ↑15.3Mbps | 167.88.61.159 | DC·Flag | `trojan` | `167.88.61.52:443` | 128 ms | xray |
| 20 | #20 🇺🇸 US | Oracle AS31898 | ↓24.8 ↑14.6Mbps | 147.224.54.25 | DC·Clean | `vless` | `panel.apexkey.store:2053` | 176 ms | xray |
| 21 | #21 🇺🇸 US | GTHost AS63023 | ↓23.0 ↑12.7Mbps | 23.150.152.65 | RES·Flag | `vmess` | `23.150.152.65:1270` | 147 ms | xray |
| 22 | #22 🇺🇸 US | Akamai Technologies AS63949 | ↓22.9 ↑13.3Mbps | 172.235.49.52 | DC·Clean | `vless` | `172.235.49.52:443` | 171 ms | singbox |
| 23 | #23 🇺🇸 US | Akamai Technologies AS63949 | ↓22.8 ↑11.1Mbps | 172.239.67.142 | DC·Clean | `vless` | `172.239.67.142:443` | 168 ms | singbox |
| 24 | #24 🇺🇸 US | GlobalTeleHost AS63023 | ↓22.8 ↑15.5Mbps | 167.88.61.85 | DC·Clean | `trojan` | `167.88.61.20:443` | 121 ms | xray |
| 25 | #25 🇺🇸 US | GlobalTeleHost AS63023 | ↓22.8 ↑14.4Mbps | 167.88.61.159 | DC·Flag | `trojan` | `167.88.61.193:443` | 123 ms | xray |

## Exit locations — which countries work

Every working proxy, grouped by the country of its **verified exit IP** (phase 2), with a dedicated subscription file per country. Currently 43 countries have at least one working exit.

| country | working | avg speed | best latency | share | list |
|---------|--------:|----------:|-------------:|------:|------|
| 🇺🇸 United States | 265 | 12.1 Mbps | 35 ms | 27% | [`US.txt`](subs/by-country/US.txt) |
| 🇳🇱 Netherlands | 150 | 4.4 Mbps | 426 ms | 15% | [`NL.txt`](subs/by-country/NL.txt) |
| 🇩🇪 Germany | 143 | 3.3 Mbps | 468 ms | 15% | [`DE.txt`](subs/by-country/DE.txt) |
| 🇯🇵 Japan | 78 | 4.2 Mbps | 304 ms | 8% | [`JP.txt`](subs/by-country/JP.txt) |
| 🇨🇦 Canada | 37 | 6.8 Mbps | 72 ms | 4% | [`CA.txt`](subs/by-country/CA.txt) |
| 🇸🇬 Singapore | 37 | 4.4 Mbps | 491 ms | 4% | [`SG.txt`](subs/by-country/SG.txt) |
| 🇫🇷 France | 36 | 4.6 Mbps | 475 ms | 4% | [`FR.txt`](subs/by-country/FR.txt) |
| 🇬🇧 United Kingdom | 28 | 3.7 Mbps | 433 ms | 3% | [`GB.txt`](subs/by-country/GB.txt) |
| 🇨🇭 Switzerland | 21 | 3.0 Mbps | 511 ms | 2% | [`CH.txt`](subs/by-country/CH.txt) |
| 🇪🇸 Spain | 19 | 3.3 Mbps | 530 ms | 2% | [`ES.txt`](subs/by-country/ES.txt) |
| 🇭🇰 Hong Kong | 18 | 3.5 Mbps | 610 ms | 2% | [`HK.txt`](subs/by-country/HK.txt) |
| 🇸🇪 Sweden | 18 | 3.0 Mbps | 554 ms | 2% | [`SE.txt`](subs/by-country/SE.txt) |
| 🇮🇹 Italy | 16 | 3.7 Mbps | 506 ms | 2% | [`IT.txt`](subs/by-country/IT.txt) |
| 🇧🇾 BY | 11 | 4.3 Mbps | 688 ms | 1% | [`BY.txt`](subs/by-country/BY.txt) |
| 🇮🇱 Israel | 10 | 0.7 Mbps | 1515 ms | 1% | [`IL.txt`](subs/by-country/IL.txt) |
| 🇰🇷 South Korea | 8 | 5.3 Mbps | 480 ms | 1% | [`KR.txt`](subs/by-country/KR.txt) |
| 🇫🇮 Finland | 8 | 2.7 Mbps | 709 ms | 1% | [`FI.txt`](subs/by-country/FI.txt) |
| 🇹🇼 Taiwan | 8 | 4.4 Mbps | 553 ms | 1% | [`TW.txt`](subs/by-country/TW.txt) |
| 🇦🇺 Australia | 6 | 4.6 Mbps | 705 ms | 1% | [`AU.txt`](subs/by-country/AU.txt) |
| 🇷🇺 Russia | 6 | 3.4 Mbps | 821 ms | 1% | [`RU.txt`](subs/by-country/RU.txt) |
| 🇰🇿 Kazakhstan | 6 | 2.3 Mbps | 1208 ms | 1% | [`KZ.txt`](subs/by-country/KZ.txt) |
| 🇧🇷 Brazil | 5 | 4.0 Mbps | 798 ms | 1% | [`BR.txt`](subs/by-country/BR.txt) |
| 🇨🇾 Cyprus | 4 | 2.7 Mbps | 877 ms | 0% | [`CY.txt`](subs/by-country/CY.txt) |
| 🇻🇳 Vietnam | 4 | 3.7 Mbps | 887 ms | 0% | [`VN.txt`](subs/by-country/VN.txt) |
| 🇮🇳 India | 4 | 2.3 Mbps | 1360 ms | 0% | [`IN.txt`](subs/by-country/IN.txt) |
| 🇦🇹 Austria | 3 | 3.6 Mbps | 556 ms | 0% | [`AT.txt`](subs/by-country/AT.txt) |
| 🇨🇿 Czechia | 3 | 4.3 Mbps | 793 ms | 0% | [`CZ.txt`](subs/by-country/CZ.txt) |
| 🇵🇱 Poland | 3 | 3.0 Mbps | 1343 ms | 0% | [`PL.txt`](subs/by-country/PL.txt) |
| 🇿🇦 South Africa | 3 | 3.2 Mbps | 998 ms | 0% | [`ZA.txt`](subs/by-country/ZA.txt) |
| 🇦🇪 UAE | 3 | 0.7 Mbps | 2476 ms | 0% | [`AE.txt`](subs/by-country/AE.txt) |
| 🇳🇬 NG | 3 | - | 3152 ms | 0% | [`NG.txt`](subs/by-country/NG.txt) |
| 🇺🇦 Ukraine | 2 | 4.2 Mbps | 1038 ms | 0% | [`UA.txt`](subs/by-country/UA.txt) |
| 🇱🇻 Latvia | 2 | 2.2 Mbps | 2081 ms | 0% | [`LV.txt`](subs/by-country/LV.txt) |
| 🇮🇷 Iran | 2 | 2.1 Mbps | 1052 ms | 0% | [`IR.txt`](subs/by-country/IR.txt) |
| 🇱🇹 Lithuania | 1 | 5.3 Mbps | 792 ms | 0% | [`LT.txt`](subs/by-country/LT.txt) |
| 🇦🇱 AL | 1 | 2.8 Mbps | 4516 ms | 0% | [`AL.txt`](subs/by-country/AL.txt) |
| 🇧🇬 Bulgaria | 1 | 1.4 Mbps | 3496 ms | 0% | [`BG.txt`](subs/by-country/BG.txt) |
| 🇹🇭 Thailand | 1 | 1.2 Mbps | 5592 ms | 0% | [`TH.txt`](subs/by-country/TH.txt) |
| 🇮🇪 Ireland | 1 | - | 401 ms | 0% | [`IE.txt`](subs/by-country/IE.txt) |
| 🇲🇾 Malaysia | 1 | - | 537 ms | 0% | [`MY.txt`](subs/by-country/MY.txt) |
| 🇳🇴 Norway | 1 | - | 1357 ms | 0% | [`NO.txt`](subs/by-country/NO.txt) |
| 🇬🇷 Greece | 1 | - | 2797 ms | 0% | [`GR.txt`](subs/by-country/GR.txt) |
| 🇮🇩 Indonesia | 1 | - | 7441 ms | 0% | [`ID.txt`](subs/by-country/ID.txt) |

A `??` exit means the geo lookup could not resolve the exit IP. Base64 variants of every country file sit next to them as `<CC>.base64.txt`.

## Protocol distribution

| protocol | tested | alive | working | alive rate |
|----------|-------:|------:|--------:|-----------:|
| `vless` | 2668 | 521 | 479 | 20% |
| `ss` | 600 | 246 | 237 | 41% |
| `vmess` | 1023 | 150 | 145 | 15% |
| `trojan` | 148 | 73 | 70 | 49% |
| `hysteria2` | 75 | 47 | 46 | 63% |
| `socks` | 2 | 2 | 2 | 100% |
| `http` | 2 | 0 | 0 | 0% |

Per-protocol working lists: [`subs/by-proto/`](subs/by-proto/) (one `vless.txt`, `vmess.txt`, ... per protocol, plus `.base64.txt` variants).

## Engine breakdown

Which core verified each working proxy. The engine fallback means a proxy that failed on its primary core got a second chance on the other one.

| engine | working | note |
|--------|--------:|------|
| Xray-core | 825 | [`allxray.txt`](subs/by-engine/allxray.txt) |
| sing-box | 152 | [`allsingbox.txt`](subs/by-engine/allsingbox.txt) |
| direct (curl) | 2 | plain http/socks, [`alldirect.txt`](subs/by-engine/alldirect.txt) |

## Speed & latency profile

| speed tier | proxies |
|-----------|--------:|
| >= 5 Mbps | 389 |
| 1 - 5 Mbps | 319 |
| 0.25 - 1 Mbps | 70 |
| < 0.25 Mbps | 0 |
| unmeasured | 201 |

Latency (phase-1 HTTPS round trip): median **820 ms**, p90 **5771 ms**. 60 proxies passed connectivity but failed egress verification (leaked the runner IP or broke on the exit check) and are excluded.

## Source health

115 active, 139 auto-retired (10 fetch failures / 10 all-dead runs / 10 days without anything new). Best contributors this run:

| source | kind | tested | alive | new this run |
|--------|------|-------:|------:|-------------:|
| `NK` | url | 1128 | 606 | 7006 |
| `SK` | url | 1087 | 568 | 253 |
| `EV` | url | 557 | 469 | 1054 |
| `HP` | url | 514 | 449 | 142 |
| `KA` | url | 3139 | 334 | 10899 |
| `AN` | url | 254 | 244 | 17 |
| `RD-SS` | url | 495 | 234 | 334 |
| `HC` | url | 1192 | 232 | 165 |
| `F0` | url | 253 | 232 | 10 |
| `EV-SS` | url | 235 | 215 | 0 |
| `RD-VL` | url | 2138 | 201 | 3613 |
| `AG-PL` | url | 373 | 195 | 753 |
| `ST-VL` | url | 2022 | 154 | 525 |
| `EV-VL` | url | 197 | 153 | 0 |
| `ET` | url | 284 | 148 | 1395 |
| `RD-VM` | url | 551 | 118 | 3978 |
| `SB-VL` | url | 1889 | 96 | 0 |
| `SB` | url | 2114 | 95 | 1 |
| `LA` | url | 697 | 86 | 13893 |
| `AQ` | url | 158 | 82 | 623 |

## History

working per run (last 30): ▇▆▆▆█▇▆▆▆▆▆█▆▆▇█████████████▇▇

| date (UTC) | tested | alive | working | avg ↓ | max ↓ | avg ↑ | clean |
|------------|-------:|------:|--------:|----:|----:|----:|------:|
| 2026-11-24 15:17 | 4518 | 1039 | 979 | 6.4M | 60.1M | 4.1M | 310 |
| 2026-11-24 03:38 | 4517 | 1065 | 1017 | 10.5M | 138.1M | - | - |
| 2026-11-22 03:38 | 4516 | 1213 | 1159 | 10.1M | 139.3M | - | - |
| 2026-11-21 03:33 | 4514 | 1155 | 1120 | 11.2M | 128.3M | - | - |
| 2026-11-20 03:37 | 4517 | 1203 | 1171 | 8.6M | 61.0M | - | - |
| 2026-11-19 03:39 | 4520 | 1168 | 1134 | 10.2M | 73.9M | - | - |
| 2026-11-17 03:38 | 4521 | 1198 | 1174 | 10.0M | 83.1M | - | - |
| 2026-11-16 03:41 | 4523 | 1137 | 1100 | 10.3M | 83.4M | - | - |
| 2026-11-14 03:41 | 4514 | 1104 | 1067 | 10.7M | 135.5M | - | - |
| 2026-11-13 03:31 | 4507 | 1150 | 1119 | 9.4M | 56.3M | - | - |
| 2026-11-12 03:35 | 4512 | 1209 | 1166 | 7.0M | 73.5M | - | - |
| 2026-11-11 03:35 | 4521 | 1141 | 1102 | 10.1M | 108.1M | - | - |
| 2026-11-10 03:31 | 4511 | 1098 | 1052 | 8.2M | 76.8M | - | - |
| 2026-11-09 03:34 | 4509 | 1081 | 1046 | 7.6M | 96.5M | - | - |
| 2026-11-08 03:39 | 4502 | 1098 | 1068 | 7.7M | 107.7M | - | - |

## Subscription files

Everything under [`subs/`](subs/) is regenerated every run. Plain lists are one proxy URI per line; `.base64.txt` variants are the same list base64-encoded for clients that need the classic sub format.

| file | contents | direct subscription URL |
|------|----------|--------------------------|
| [`all.txt`](subs/all.txt) | ALL working proxies, ranked by speed (979) | `https://raw.githubusercontent.com/onlyactions/proxy-scanner/main/subs/all.txt` |
| [`all.base64.txt`](subs/all.base64.txt) | same, base64 | `https://raw.githubusercontent.com/onlyactions/proxy-scanner/main/subs/all.base64.txt` |
| [`top150.txt`](subs/top150.txt) | top 150 ranked (150) | `https://raw.githubusercontent.com/onlyactions/proxy-scanner/main/subs/top150.txt` |
| [`top150.base64.txt`](subs/top150.base64.txt) | same, base64 | `https://raw.githubusercontent.com/onlyactions/proxy-scanner/main/subs/top150.base64.txt` |
| [`by-proto/`](subs/by-proto/) | one list per protocol (6 protocols) | ...`/subs/by-proto/<proto>.txt` |
| [`by-engine/allxray.txt`](subs/by-engine/allxray.txt) | verified on Xray (825) | `https://raw.githubusercontent.com/onlyactions/proxy-scanner/main/subs/by-engine/allxray.txt` |
| [`by-engine/allsingbox.txt`](subs/by-engine/allsingbox.txt) | verified on sing-box (152) | `https://raw.githubusercontent.com/onlyactions/proxy-scanner/main/subs/by-engine/allsingbox.txt` |
| [`by-country/`](subs/by-country/) | one list per exit country (43 countries, e.g. [`US.txt`](subs/by-country/US.txt), [`DE.txt`](subs/by-country/DE.txt)) | ...`/subs/by-country/<CC>.txt` |
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

The typical aggregator relists whatever it scraped, so 80-95% of entries are dead within hours — this run found **77% unreachable** and **78% unusable** overall. Only tunnels that completed a real HTTPS handshake, returned a verifiable foreign exit IP, and moved a 1 MB download **and** a 512 KB upload through the tunnel are counted as working here.

## Disclaimers

- Free proxies are **public and untrusted**. Anything you send through them can be observed or modified by their operators. Never use them for authentication, payments, or anything sensitive.
- All configs are collected from publicly posted sources (GitHub subscriptions and public Telegram channels); no credentials are guessed or brute-forced. Removal requests: open an issue.
- Availability is transient: the working list is re-verified daily, and past performance never guarantees future availability.

