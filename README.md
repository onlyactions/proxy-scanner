# Free Proxy Scanner

> **Really tested** free proxy aggregator. Every proxy below was fetched from public sources, tunneled through with Xray/sing-box, and verified to actually change your egress IP. No CSV-validation theater, no blind relisting.

![working](badges/working.json) ![alive](badges/alive.json) ![speed](badges/speed.json) ![up-speed](badges/up-speed.json) ![max-speed](badges/max-speed.json) ![clean](badges/clean.json) ![countries](badges/countries.json) ![sources](badges/sources.json) ![updated](badges/updated.json)

## What this is

A GitHub Actions pipeline (runs daily) that fetches free proxy configs from **173 URL subscriptions** and **81 Telegram channels**, parses 9 protocols (vless / vmess / trojan / ss / hysteria2 / hysteria / tuic / socks / http), deduplicates them, and **actually connects through every single one** to separate working proxies from dead ones.

Latest run: **2026-11-25 04:27:02 UTC** — 125594 unique proxies collected from 106/106 URL sources and 9/9 Telegram channels; **4511** endpoints really tested, **1108** reachable, **1053** verified working (egress IP confirmed changed) across **44 exit countries** — **341** of them on unflagged ('clean') exit IPs and **323** on residential IPs.

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

All 1053 working proxies are published in [`subs/all.txt`](subs/all.txt); this table shows the top 25. The `name` column is exactly what your client will show (see [How to read a proxy name](#how-to-read-a-proxy-name)).

| # | name | proto | entry server | latency | engine |
|---|------|-------|--------------|---------|--------|
| 1 | #1 🇺🇸 US | GTHost AS63023 | ↓76.6 ↑48.2Mbps | 38.143.66.146 | RES·Flag | `ss` | `38.143.66.146:19999` | 148 ms | xray |
| 2 | #2 🇺🇸 US | OVH AS16276 | ↓63.0 ↑19.8Mbps | 15.204.97.197 | DC·Flag | `vless` | `15.204.97.197:23576` | 191 ms | xray |
| 3 | #3 🇺🇸 US | OVH AS16276 | ↓59.5 ↑41.4Mbps | 15.204.97.219 | DC·Flag | `vless` | `15.204.97.219:23576` | 198 ms | xray |
| 4 | #4 🇺🇸 US | OVH AS16276 | ↓47.2 ↑41.4Mbps | 15.204.97.198 | DC·Flag | `vless` | `15.204.97.198:23576` | 64 ms | xray |
| 5 | #5 🇨🇦 CA | Datacamp AS212238 | ↓45.8 ↑46.7Mbps | 149.22.95.183 | DC·Flag | `ss` | `149.22.95.183:443` | 62 ms | xray |
| 6 | #6 🇺🇸 US → 🇦🇺 AU | The Constant AS20473 | ↓44.7 ↑58.7Mbps | 45.76.115.243 | DC·Clean | `vmess` | `84.17.41.2:18000` | 32 ms | xray |
| 7 | #7 🇺🇸 US | OVH AS16276 | ↓42.9 ↑14.3Mbps | 15.204.97.206 | DC·Flag | `vless` | `15.204.97.206:23576` | 84 ms | xray |
| 8 | #8 🇺🇸 US | OVH AS16276 | ↓42.3 ↑27.5Mbps | 15.204.97.214 | DC·Flag | `vless` | `15.204.97.214:23576` | 115 ms | xray |
| 9 | #9 🇺🇸 US | OVH AS16276 | ↓40.9 ↑35.1Mbps | 15.204.97.195 | DC·Flag | `vless` | `15.204.97.195:23576` | 63 ms | xray |
| 10 | #10 🇺🇸 US | OVH AS16276 | ↓39.3 ↑39.4Mbps | 15.204.97.209 | DC·Flag | `vless` | `15.204.97.209:23576` | 61 ms | xray |
| 11 | #11 🇺🇸 US | Alibaba Cloud AS45102 | ↓29.0 ↑24.2Mbps | 47.89.232.46 | DC·Clean | `hysteria2` | `47.89.232.46:8443` | 152 ms | singbox |
| 12 | #12 🇺🇸 US | GlobalTeleHost AS63023 | ↓27.9 ↑16.6Mbps | 167.88.61.20 | DC·Flag | `trojan` | `167.88.61.175:443` | 192 ms | xray |
| 13 | #13 🇺🇸 US | GlobalTeleHost AS63023 | ↓27.8 ↑15.4Mbps | 167.88.61.85 | DC·Clean | `trojan` | `167.88.61.193:443` | 313 ms | xray |
| 14 | #14 🇺🇸 US | DigitalOcean AS14061 | ↓27.1 ↑16.9Mbps | 137.184.89.151 | DC·Flag | `ss` | `137.184.89.151:19999` | 91 ms | xray |
| 15 | #15 🇺🇸 US | GlobalTeleHost AS63023 | ↓26.9 ↑15.0Mbps | 167.88.61.159 | DC·Flag | `trojan` | `167.88.61.198:443` | 139 ms | xray |
| 16 | #16 🇺🇸 US | GTHost AS63023 | ↓26.2 ↑14.9Mbps | 169.197.142.68 | DC·Flag | `vmess` | `169.197.142.68:1270` | 135 ms | xray |
| 17 | #17 🇺🇸 US | OVH AS16276 | ↓25.1 ↑40.1Mbps | 15.204.97.216 | DC·Flag | `vless` | `15.204.97.216:23576` | 70 ms | xray |
| 18 | #18 🇺🇸 US | GTT Americ AS48529 | ↓24.8 ↑18.7Mbps | 69.17.3.150 | RES·Flag | `vless` | `69.17.3.150:2001` | 139 ms | singbox |
| 19 | #19 🇺🇸 US | GlobalTeleHost AS63023 | ↓24.7 ↑16.4Mbps | 167.88.61.193 | DC·Flag | `trojan` | `167.88.61.85:443` | 120 ms | xray |
| 20 | #20 🇺🇸 US | GTHost AS63023 | ↓24.7 ↑14.9Mbps | 169.197.142.68 | DC·Flag | `ss` | `169.197.142.68:19999` | 100 ms | xray |
| 21 | #21 🇺🇸 US | GlobalTeleHost AS63023 | ↓24.5 ↑8.3Mbps | 167.88.61.52 | DC·Clean | `trojan` | `167.88.61.20:443` | 190 ms | xray |
| 22 | #22 🇺🇸 US | GTHost AS63023 | ↓24.1 ↑7.0Mbps | 23.150.152.65 | RES·Flag | `vmess` | `23.150.152.65:1270` | 162 ms | xray |
| 23 | #23 🇺🇸 US | GTHost AS63023 | ↓23.4 ↑10.9Mbps | 23.150.152.38 | RES·Flag | `ss` | `23.150.152.38:19999` | 104 ms | xray |
| 24 | #24 🇺🇸 US | NetLab Global AS979 | ↓23.3 ↑9.9Mbps | 154.12.49.239 | RES·Clean | `vless` | `154.12.49.239:80` | 141 ms | singbox |
| 25 | #25 🇺🇸 US | 2E TELEKOMUNIKASYON AS209604 | ↓23.2 ↑18.7Mbps | 179.254.94.184 | RES·Clean | `hysteria2` | `179.254.94.184:8443` | 151 ms | singbox |

## Exit locations — which countries work

Every working proxy, grouped by the country of its **verified exit IP** (phase 2), with a dedicated subscription file per country. Currently 44 countries have at least one working exit.

| country | working | avg speed | best latency | share | list |
|---------|--------:|----------:|-------------:|------:|------|
| 🇺🇸 United States | 280 | 12.4 Mbps | 61 ms | 27% | [`US.txt`](subs/by-country/US.txt) |
| 🇩🇪 Germany | 168 | 3.6 Mbps | 469 ms | 16% | [`DE.txt`](subs/by-country/DE.txt) |
| 🇳🇱 Netherlands | 153 | 5.0 Mbps | 428 ms | 15% | [`NL.txt`](subs/by-country/NL.txt) |
| 🇯🇵 Japan | 88 | 2.7 Mbps | 303 ms | 8% | [`JP.txt`](subs/by-country/JP.txt) |
| 🇨🇦 Canada | 39 | 7.9 Mbps | 62 ms | 4% | [`CA.txt`](subs/by-country/CA.txt) |
| 🇫🇷 France | 37 | 5.0 Mbps | 453 ms | 4% | [`FR.txt`](subs/by-country/FR.txt) |
| 🇸🇬 Singapore | 30 | 4.3 Mbps | 491 ms | 3% | [`SG.txt`](subs/by-country/SG.txt) |
| 🇬🇧 United Kingdom | 26 | 4.5 Mbps | 430 ms | 2% | [`GB.txt`](subs/by-country/GB.txt) |
| 🇨🇭 Switzerland | 23 | 3.5 Mbps | 500 ms | 2% | [`CH.txt`](subs/by-country/CH.txt) |
| 🇮🇱 Israel | 20 | 1.4 Mbps | 1299 ms | 2% | [`IL.txt`](subs/by-country/IL.txt) |
| 🇭🇰 Hong Kong | 19 | 3.9 Mbps | 596 ms | 2% | [`HK.txt`](subs/by-country/HK.txt) |
| 🇸🇪 Sweden | 17 | 4.0 Mbps | 509 ms | 2% | [`SE.txt`](subs/by-country/SE.txt) |
| 🇦🇺 Australia | 16 | 9.5 Mbps | 32 ms | 2% | [`AU.txt`](subs/by-country/AU.txt) |
| 🇪🇸 Spain | 15 | 4.7 Mbps | 536 ms | 1% | [`ES.txt`](subs/by-country/ES.txt) |
| 🇮🇹 Italy | 14 | 5.2 Mbps | 514 ms | 1% | [`IT.txt`](subs/by-country/IT.txt) |
| 🇧🇾 BY | 11 | 5.2 Mbps | 664 ms | 1% | [`BY.txt`](subs/by-country/BY.txt) |
| 🇷🇺 Russia | 9 | 3.8 Mbps | 845 ms | 1% | [`RU.txt`](subs/by-country/RU.txt) |
| 🇹🇼 Taiwan | 8 | 5.0 Mbps | 551 ms | 1% | [`TW.txt`](subs/by-country/TW.txt) |
| 🇵🇱 Poland | 8 | 3.4 Mbps | 705 ms | 1% | [`PL.txt`](subs/by-country/PL.txt) |
| 🇰🇷 South Korea | 7 | 5.1 Mbps | 528 ms | 1% | [`KR.txt`](subs/by-country/KR.txt) |
| 🇰🇿 Kazakhstan | 7 | 2.7 Mbps | 1230 ms | 1% | [`KZ.txt`](subs/by-country/KZ.txt) |
| 🇱🇻 Latvia | 6 | 4.9 Mbps | 724 ms | 1% | [`LV.txt`](subs/by-country/LV.txt) |
| 🇫🇮 Finland | 5 | 2.7 Mbps | 741 ms | 0% | [`FI.txt`](subs/by-country/FI.txt) |
| 🇮🇳 India | 5 | 1.9 Mbps | 1158 ms | 0% | [`IN.txt`](subs/by-country/IN.txt) |
| 🇱🇹 Lithuania | 4 | 5.1 Mbps | 763 ms | 0% | [`LT.txt`](subs/by-country/LT.txt) |
| 🇨🇾 Cyprus | 4 | 4.2 Mbps | 731 ms | 0% | [`CY.txt`](subs/by-country/CY.txt) |
| 🇧🇷 Brazil | 4 | 4.6 Mbps | 786 ms | 0% | [`BR.txt`](subs/by-country/BR.txt) |
| 🇦🇹 Austria | 3 | 4.2 Mbps | 530 ms | 0% | [`AT.txt`](subs/by-country/AT.txt) |
| 🇨🇿 Czechia | 3 | 4.5 Mbps | 683 ms | 0% | [`CZ.txt`](subs/by-country/CZ.txt) |
| 🇿🇦 South Africa | 3 | 3.2 Mbps | 1042 ms | 0% | [`ZA.txt`](subs/by-country/ZA.txt) |
| 🇻🇳 Vietnam | 3 | 0.6 Mbps | 1042 ms | 0% | [`VN.txt`](subs/by-country/VN.txt) |
| 🇦🇱 AL | 2 | 5.0 Mbps | 666 ms | 0% | [`AL.txt`](subs/by-country/AL.txt) |
| 🇺🇦 Ukraine | 2 | 3.0 Mbps | 880 ms | 0% | [`UA.txt`](subs/by-country/UA.txt) |
| 🇮🇷 Iran | 2 | 4.4 Mbps | 685 ms | 0% | [`IR.txt`](subs/by-country/IR.txt) |
| 🇹🇷 Turkey | 2 | - | 1164 ms | 0% | [`TR.txt`](subs/by-country/TR.txt) |
| 🇦🇪 UAE | 2 | - | 2531 ms | 0% | [`AE.txt`](subs/by-country/AE.txt) |
| 🇹🇭 Thailand | 1 | 4.6 Mbps | 3335 ms | 0% | [`TH.txt`](subs/by-country/TH.txt) |
| 🇪🇪 Estonia | 1 | 2.2 Mbps | 1081 ms | 0% | [`EE.txt`](subs/by-country/EE.txt) |
| 🇮🇪 Ireland | 1 | 1.1 Mbps | 3921 ms | 0% | [`IE.txt`](subs/by-country/IE.txt) |
| 🇳🇴 Norway | 1 | 0.6 Mbps | 2484 ms | 0% | [`NO.txt`](subs/by-country/NO.txt) |
| 🇷🇴 Romania | 1 | - | 763 ms | 0% | [`RO.txt`](subs/by-country/RO.txt) |
| 🇮🇩 Indonesia | 1 | - | 850 ms | 0% | [`ID.txt`](subs/by-country/ID.txt) |
| 🇲🇾 Malaysia | 1 | - | 1561 ms | 0% | [`MY.txt`](subs/by-country/MY.txt) |
| 🇬🇷 Greece | 1 | - | 5840 ms | 0% | [`GR.txt`](subs/by-country/GR.txt) |

A `??` exit means the geo lookup could not resolve the exit IP. Base64 variants of every country file sit next to them as `<CC>.base64.txt`.

## Protocol distribution

| protocol | tested | alive | working | alive rate |
|----------|-------:|------:|--------:|-----------:|
| `vless` | 2736 | 580 | 542 | 21% |
| `ss` | 542 | 244 | 235 | 45% |
| `vmess` | 1018 | 159 | 152 | 16% |
| `trojan` | 139 | 73 | 73 | 53% |
| `hysteria2` | 72 | 50 | 49 | 69% |
| `socks` | 2 | 2 | 2 | 100% |
| `http` | 2 | 0 | 0 | 0% |

Per-protocol working lists: [`subs/by-proto/`](subs/by-proto/) (one `vless.txt`, `vmess.txt`, ... per protocol, plus `.base64.txt` variants).

## Engine breakdown

Which core verified each working proxy. The engine fallback means a proxy that failed on its primary core got a second chance on the other one.

| engine | working | note |
|--------|--------:|------|
| Xray-core | 907 | [`allxray.txt`](subs/by-engine/allxray.txt) |
| sing-box | 144 | [`allsingbox.txt`](subs/by-engine/allsingbox.txt) |
| direct (curl) | 2 | plain http/socks, [`alldirect.txt`](subs/by-engine/alldirect.txt) |

## Speed & latency profile

| speed tier | proxies |
|-----------|--------:|
| >= 5 Mbps | 483 |
| 1 - 5 Mbps | 403 |
| 0.25 - 1 Mbps | 52 |
| < 0.25 Mbps | 0 |
| unmeasured | 115 |

Latency (phase-1 HTTPS round trip): median **702 ms**, p90 **4501 ms**. 55 proxies passed connectivity but failed egress verification (leaked the runner IP or broke on the exit check) and are excluded.

## Source health

115 active, 139 auto-retired (10 fetch failures / 10 all-dead runs / 10 days without anything new). Best contributors this run:

| source | kind | tested | alive | new this run |
|--------|------|-------:|------:|-------------:|
| `NK` | url | 1021 | 615 | 5727 |
| `SK` | url | 1013 | 611 | 809 |
| `EV` | url | 622 | 486 | 80 |
| `HP` | url | 468 | 408 | 173 |
| `HC` | url | 1249 | 262 | 98 |
| `AN` | url | 270 | 255 | 99 |
| `AG-PL` | url | 464 | 251 | 986 |
| `Ni` | url | 355 | 251 | 39 |
| `F0` | url | 254 | 241 | 18 |
| `RD-SS` | url | 466 | 227 | 465 |
| `RD-VL` | url | 2189 | 223 | 3253 |
| `EV-SS` | url | 236 | 216 | 10 |
| `ST-VL` | url | 2113 | 197 | 774 |
| `ET` | url | 288 | 195 | 1840 |
| `EV-VL` | url | 295 | 190 | 822 |
| `RD-VM` | url | 545 | 125 | 4006 |
| `SB` | url | 2170 | 121 | 3 |
| `SB-VL` | url | 1960 | 116 | 0 |
| `LA` | url | 693 | 106 | 15129 |
| `ST-VM` | url | 539 | 97 | 43 |

## History

working per run (last 30): ▆▆▆█▇▆▆▆▆▆█▆▆▇█████████████▇▇█

| date (UTC) | tested | alive | working | avg ↓ | max ↓ | avg ↑ | clean |
|------------|-------:|------:|--------:|----:|----:|----:|------:|
| 2026-11-25 05:08 | 4511 | 1108 | 1053 | 6.5M | 76.6M | 4.0M | 341 |
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

## Subscription files

Everything under [`subs/`](subs/) is regenerated every run. Plain lists are one proxy URI per line; `.base64.txt` variants are the same list base64-encoded for clients that need the classic sub format.

| file | contents | direct subscription URL |
|------|----------|--------------------------|
| [`all.txt`](subs/all.txt) | ALL working proxies, ranked by speed (1053) | `https://raw.githubusercontent.com/onlyactions/proxy-scanner/main/subs/all.txt` |
| [`all.base64.txt`](subs/all.base64.txt) | same, base64 | `https://raw.githubusercontent.com/onlyactions/proxy-scanner/main/subs/all.base64.txt` |
| [`top150.txt`](subs/top150.txt) | top 150 ranked (150) | `https://raw.githubusercontent.com/onlyactions/proxy-scanner/main/subs/top150.txt` |
| [`top150.base64.txt`](subs/top150.base64.txt) | same, base64 | `https://raw.githubusercontent.com/onlyactions/proxy-scanner/main/subs/top150.base64.txt` |
| [`by-proto/`](subs/by-proto/) | one list per protocol (6 protocols) | ...`/subs/by-proto/<proto>.txt` |
| [`by-engine/allxray.txt`](subs/by-engine/allxray.txt) | verified on Xray (907) | `https://raw.githubusercontent.com/onlyactions/proxy-scanner/main/subs/by-engine/allxray.txt` |
| [`by-engine/allsingbox.txt`](subs/by-engine/allsingbox.txt) | verified on sing-box (144) | `https://raw.githubusercontent.com/onlyactions/proxy-scanner/main/subs/by-engine/allsingbox.txt` |
| [`by-country/`](subs/by-country/) | one list per exit country (44 countries, e.g. [`US.txt`](subs/by-country/US.txt), [`DE.txt`](subs/by-country/DE.txt)) | ...`/subs/by-country/<CC>.txt` |
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

The typical aggregator relists whatever it scraped, so 80-95% of entries are dead within hours — this run found **75% unreachable** and **77% unusable** overall. Only tunnels that completed a real HTTPS handshake, returned a verifiable foreign exit IP, and moved a 1 MB download **and** a 512 KB upload through the tunnel are counted as working here.

## Disclaimers

- Free proxies are **public and untrusted**. Anything you send through them can be observed or modified by their operators. Never use them for authentication, payments, or anything sensitive.
- All configs are collected from publicly posted sources (GitHub subscriptions and public Telegram channels); no credentials are guessed or brute-forced. Removal requests: open an issue.
- Availability is transient: the working list is re-verified daily, and past performance never guarantees future availability.

