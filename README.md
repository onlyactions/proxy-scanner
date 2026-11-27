# Free Proxy Scanner

> **Really tested** free proxy aggregator. Every proxy below was fetched from public sources, tunneled through with Xray/sing-box, and verified to actually change your egress IP. No CSV-validation theater, no blind relisting.

![working](badges/working.json) ![alive](badges/alive.json) ![speed](badges/speed.json) ![up-speed](badges/up-speed.json) ![max-speed](badges/max-speed.json) ![clean](badges/clean.json) ![countries](badges/countries.json) ![sources](badges/sources.json) ![updated](badges/updated.json)

## What this is

A GitHub Actions pipeline (runs daily) that fetches free proxy configs from **173 URL subscriptions** and **81 Telegram channels**, parses 9 protocols (vless / vmess / trojan / ss / hysteria2 / hysteria / tuic / socks / http), deduplicates them, and **actually connects through every single one** to separate working proxies from dead ones.

Latest run: **2026-11-27 04:26:08 UTC** — 125498 unique proxies collected from 105/105 URL sources and 8/8 Telegram channels; **4521** endpoints really tested, **1181** reachable, **1147** verified working (egress IP confirmed changed) across **43 exit countries** — **412** of them on unflagged ('clean') exit IPs and **321** on residential IPs.

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

All 1147 working proxies are published in [`subs/all.txt`](subs/all.txt); this table shows the top 25. The `name` column is exactly what your client will show (see [How to read a proxy name](#how-to-read-a-proxy-name)).

| # | name | proto | entry server | latency | engine |
|---|------|-------|--------------|---------|--------|
| 1 | #1 🇺🇸 US | GlobalTeleHost AS63023 | ↓72.8 ↑47.6Mbps | 167.88.61.198 | DC·Clean | `trojan` | `167.88.61.85:443` | 144 ms | xray |
| 2 | #2 🇺🇸 US | Alibaba Cloud AS45102 | ↓70.7 ↑39.5Mbps | 47.89.232.46 | DC·Clean | `hysteria2` | `47.89.232.46:8443` | 85 ms | singbox |
| 3 | #3 🇺🇸 US | GTHost AS63023 | ↓65.5 ↑42.2Mbps | 169.197.142.68 | DC·Flag | `ss` | `169.197.142.68:19999` | 191 ms | xray |
| 4 | #4 🇺🇸 US | Akamai Technologies AS63949 | ↓59.0 ↑32.4Mbps | 172.236.227.60 | DC·Clean | `vless` | `172.236.227.60:443` | 94 ms | singbox |
| 5 | #5 🇺🇸 US | Akamai Technologies AS63949 | ↓57.0 ↑37.6Mbps | 172.239.67.142 | DC·Clean | `vless` | `172.239.67.142:443` | 60 ms | singbox |
| 6 | #6 🇺🇸 US | Akamai Technologies AS63949 | ↓53.6 ↑38.8Mbps | 172.235.49.52 | DC·Clean | `vless` | `172.235.49.52:443` | 68 ms | singbox |
| 7 | #7 🇺🇸 US → 🇦🇺 AU | The Constant AS20473 | ↓50.4 ↑27.0Mbps | 45.76.115.243 | DC·Clean | `vmess` | `169.197.142.22:18000` | 168 ms | xray |
| 8 | #8 🇺🇸 US | Akamai Technologies AS63949 | ↓45.4 ↑24.9Mbps | 172.236.233.130 | DC·Clean | `vless` | `172.236.233.130:443` | 130 ms | singbox |
| 9 | #9 🇺🇸 US | GTHost AS63023 | ↓44.9 ↑31.2Mbps | 23.150.152.38 | RES·Flag | `ss` | `23.150.152.38:19999` | 51 ms | xray |
| 10 | #10 🇺🇸 US | Akamai Technologies AS63949 | ↓42.5 ↑29.2Mbps | 172.236.231.45 | DC·Clean | `vless` | `172.236.231.45:443` | 78 ms | singbox |
| 11 | #11 🇺🇸 US | HostPapa AS36352 | ↓42.4 ↑25.5Mbps | 107.175.54.116 | ? | `vmess` | `www.shopify.com:443` | 175 ms | xray |
| 12 | #12 🇺🇸 US | HostPapa AS36352 | ↓42.1 ↑20.9Mbps | 107.175.54.116 | ? | `vmess` | `172.64.145.93:443` | 173 ms | xray |
| 13 | #13 🇺🇸 US | OVH AS16276 | ↓41.8 ↑17.4Mbps | 15.204.97.197 | DC·Flag | `vless` | `15.204.97.197:23576` | 91 ms | xray |
| 14 | #14 🇺🇸 US | The Constant AS20473 | ↓41.5 ↑32.3Mbps | 45.63.53.81 | DC·Flag | `vless` | `45.63.53.81:443` | 79 ms | singbox |
| 15 | #15 🇺🇸 US | OVH AS16276 | ↓41.5 ↑16.2Mbps | 15.204.97.216 | DC·Flag | `vless` | `15.204.97.216:23576` | 107 ms | xray |
| 16 | #16 🇺🇸 US | Dedicated.com AS63018 | ↓40.8 ↑43.3Mbps | 167.160.90.50 | DC·Clean | `ss` | `167.160.90.51:11475` | 47 ms | xray |
| 17 | #17 🇺🇸 US | OVH AS16276 | ↓40.3 ↑25.9Mbps | 15.204.97.206 | DC·Flag | `vless` | `15.204.97.206:23576` | 108 ms | xray |
| 18 | #18 🇺🇸 US | OVH AS16276 | ↓40.2 ↑25.1Mbps | 15.204.97.198 | DC·Flag | `vless` | `15.204.97.198:23576` | 114 ms | xray |
| 19 | #19 🇺🇸 US | DigitalOcean AS14061 | ↓39.7 ↑61.7Mbps | 137.184.89.151 | DC·Flag | `ss` | `137.184.89.151:19999` | 33 ms | xray |
| 20 | #20 🇺🇸 US | GTHost AS63023 | ↓37.7 ↑22.7Mbps | 95.215.201.143 | DC·Clean | `ss` | `95.215.201.18:19999` | 91 ms | xray |
| 21 | #21 🇺🇸 US | GTHost AS63023 | ↓37.5 ↑6.6Mbps | 216.106.186.124 | RES·Flag | `trojan` | `216.106.186.46:443` | 217 ms | xray |
| 22 | #22 🇺🇸 US | HostPapa AS36352 | ↓37.1 ↑20.1Mbps | 107.175.54.116 | ? | `vmess` | `188.114.99.137:443` | 131 ms | xray |
| 23 | #23 🇺🇸 US | GTHost AS63023 | ↓37.1 ↑21.2Mbps | 66.163.112.31 | DC·Flag | `trojan` | `66.163.112.121:443` | 202 ms | xray |
| 24 | #24 🇺🇸 US | Psychz Networks AS40676 | ↓37.1 ↑34.8Mbps | 108.181.0.177 | DC·Flag | `ss` | `108.181.9.202:8388` | 48 ms | xray |
| 25 | #25 🇺🇸 US | Cogent Communications AS197196 | ↓37.0 ↑2.0Mbps | 38.244.5.231 | RES·Clean | `vless` | `38.244.5.231:443` | 72 ms | xray |

## Exit locations — which countries work

Every working proxy, grouped by the country of its **verified exit IP** (phase 2), with a dedicated subscription file per country. Currently 43 countries have at least one working exit.

| country | working | avg speed | best latency | share | list |
|---------|--------:|----------:|-------------:|------:|------|
| 🇺🇸 United States | 326 | 15.0 Mbps | 33 ms | 28% | [`US.txt`](subs/by-country/US.txt) |
| 🇳🇱 Netherlands | 177 | 4.4 Mbps | 444 ms | 15% | [`NL.txt`](subs/by-country/NL.txt) |
| 🇯🇵 Japan | 165 | 3.5 Mbps | 327 ms | 14% | [`JP.txt`](subs/by-country/JP.txt) |
| 🇩🇪 Germany | 97 | 4.5 Mbps | 463 ms | 8% | [`DE.txt`](subs/by-country/DE.txt) |
| 🇫🇷 France | 40 | 5.0 Mbps | 464 ms | 3% | [`FR.txt`](subs/by-country/FR.txt) |
| 🇨🇦 Canada | 35 | 8.5 Mbps | 107 ms | 3% | [`CA.txt`](subs/by-country/CA.txt) |
| 🇬🇧 United Kingdom | 32 | 4.5 Mbps | 421 ms | 3% | [`GB.txt`](subs/by-country/GB.txt) |
| 🇸🇬 Singapore | 32 | 3.6 Mbps | 512 ms | 3% | [`SG.txt`](subs/by-country/SG.txt) |
| 🇭🇰 Hong Kong | 22 | 3.3 Mbps | 77 ms | 2% | [`HK.txt`](subs/by-country/HK.txt) |
| 🇪🇸 Spain | 22 | 3.7 Mbps | 526 ms | 2% | [`ES.txt`](subs/by-country/ES.txt) |
| 🇮🇱 Israel | 20 | 1.9 Mbps | 633 ms | 2% | [`IL.txt`](subs/by-country/IL.txt) |
| 🇨🇭 Switzerland | 19 | 4.2 Mbps | 514 ms | 2% | [`CH.txt`](subs/by-country/CH.txt) |
| 🇦🇺 Australia | 16 | 11.2 Mbps | 91 ms | 1% | [`AU.txt`](subs/by-country/AU.txt) |
| 🇸🇪 Sweden | 16 | 4.6 Mbps | 539 ms | 1% | [`SE.txt`](subs/by-country/SE.txt) |
| 🇮🇹 Italy | 15 | 5.4 Mbps | 490 ms | 1% | [`IT.txt`](subs/by-country/IT.txt) |
| 🇰🇷 South Korea | 14 | 4.5 Mbps | 481 ms | 1% | [`KR.txt`](subs/by-country/KR.txt) |
| 🇷🇺 Russia | 12 | 3.5 Mbps | 658 ms | 1% | [`RU.txt`](subs/by-country/RU.txt) |
| 🇫🇮 Finland | 12 | 2.7 Mbps | 532 ms | 1% | [`FI.txt`](subs/by-country/FI.txt) |
| 🇰🇿 Kazakhstan | 7 | 3.3 Mbps | 874 ms | 1% | [`KZ.txt`](subs/by-country/KZ.txt) |
| 🇮🇳 India | 7 | 2.2 Mbps | 926 ms | 1% | [`IN.txt`](subs/by-country/IN.txt) |
| 🇧🇷 Brazil | 6 | 4.1 Mbps | 574 ms | 1% | [`BR.txt`](subs/by-country/BR.txt) |
| 🇧🇾 BY | 5 | 4.5 Mbps | 832 ms | 0% | [`BY.txt`](subs/by-country/BY.txt) |
| 🇹🇼 Taiwan | 5 | 3.5 Mbps | 601 ms | 0% | [`TW.txt`](subs/by-country/TW.txt) |
| 🇦🇪 UAE | 5 | 2.7 Mbps | 668 ms | 0% | [`AE.txt`](subs/by-country/AE.txt) |
| 🇦🇹 Austria | 4 | 5.4 Mbps | 520 ms | 0% | [`AT.txt`](subs/by-country/AT.txt) |
| 🇨🇿 Czechia | 4 | 4.4 Mbps | 703 ms | 0% | [`CZ.txt`](subs/by-country/CZ.txt) |
| 🇿🇦 South Africa | 4 | 2.9 Mbps | 986 ms | 0% | [`ZA.txt`](subs/by-country/ZA.txt) |
| 🇻🇳 Vietnam | 4 | 1.2 Mbps | 982 ms | 0% | [`VN.txt`](subs/by-country/VN.txt) |
| 🇵🇱 Poland | 3 | 3.7 Mbps | 685 ms | 0% | [`PL.txt`](subs/by-country/PL.txt) |
| 🇹🇷 Turkey | 3 | 4.3 Mbps | 709 ms | 0% | [`TR.txt`](subs/by-country/TR.txt) |
| 🇱🇻 Latvia | 2 | 4.1 Mbps | 787 ms | 0% | [`LV.txt`](subs/by-country/LV.txt) |
| 🇹🇭 Thailand | 2 | 3.8 Mbps | 742 ms | 0% | [`TH.txt`](subs/by-country/TH.txt) |
| 🇦🇱 AL | 2 | 2.7 Mbps | 661 ms | 0% | [`AL.txt`](subs/by-country/AL.txt) |
| 🇮🇪 Ireland | 2 | 0.6 Mbps | 420 ms | 0% | [`IE.txt`](subs/by-country/IE.txt) |
| 🇲🇾 Malaysia | 2 | - | 543 ms | 0% | [`MY.txt`](subs/by-country/MY.txt) |
| 🇷🇴 Romania | 1 | 4.9 Mbps | 3188 ms | 0% | [`RO.txt`](subs/by-country/RO.txt) |
| 🇧🇬 Bulgaria | 1 | 4.6 Mbps | 868 ms | 0% | [`BG.txt`](subs/by-country/BG.txt) |
| 🇮🇷 Iran | 1 | 4.0 Mbps | 763 ms | 0% | [`IR.txt`](subs/by-country/IR.txt) |
| 🇬🇷 Greece | 1 | 3.8 Mbps | 5553 ms | 0% | [`GR.txt`](subs/by-country/GR.txt) |
| 🇨🇳 China | 1 | 1.4 Mbps | 1155 ms | 0% | [`CN.txt`](subs/by-country/CN.txt) |
| 🇪🇪 Estonia | 1 | 0.9 Mbps | 836 ms | 0% | [`EE.txt`](subs/by-country/EE.txt) |
| 🇵🇦 PA | 1 | 0.6 Mbps | 6078 ms | 0% | [`PA.txt`](subs/by-country/PA.txt) |
| 🇳🇴 Norway | 1 | - | 586 ms | 0% | [`NO.txt`](subs/by-country/NO.txt) |

A `??` exit means the geo lookup could not resolve the exit IP. Base64 variants of every country file sit next to them as `<CC>.base64.txt`.

## Protocol distribution

| protocol | tested | alive | working | alive rate |
|----------|-------:|------:|--------:|-----------:|
| `vless` | 2753 | 525 | 502 | 19% |
| `ss` | 639 | 354 | 350 | 55% |
| `vmess` | 933 | 162 | 156 | 17% |
| `trojan` | 129 | 93 | 92 | 72% |
| `hysteria2` | 63 | 45 | 45 | 71% |
| `socks` | 2 | 2 | 2 | 100% |
| `http` | 2 | 0 | 0 | 0% |

Per-protocol working lists: [`subs/by-proto/`](subs/by-proto/) (one `vless.txt`, `vmess.txt`, ... per protocol, plus `.base64.txt` variants).

## Engine breakdown

Which core verified each working proxy. The engine fallback means a proxy that failed on its primary core got a second chance on the other one.

| engine | working | note |
|--------|--------:|------|
| Xray-core | 1026 | [`allxray.txt`](subs/by-engine/allxray.txt) |
| sing-box | 119 | [`allsingbox.txt`](subs/by-engine/allsingbox.txt) |
| direct (curl) | 2 | plain http/socks, [`alldirect.txt`](subs/by-engine/alldirect.txt) |

## Speed & latency profile

| speed tier | proxies |
|-----------|--------:|
| >= 5 Mbps | 551 |
| 1 - 5 Mbps | 311 |
| 0.25 - 1 Mbps | 91 |
| < 0.25 Mbps | 0 |
| unmeasured | 194 |

Latency (phase-1 HTTPS round trip): median **624 ms**, p90 **2857 ms**. 34 proxies passed connectivity but failed egress verification (leaked the runner IP or broke on the exit check) and are excluded.

## Source health

112 active, 142 auto-retired (10 fetch failures / 10 all-dead runs / 10 days without anything new). Best contributors this run:

| source | kind | tested | alive | new this run |
|--------|------|-------:|------:|-------------:|
| `EV` | url | 1006 | 899 | 149 |
| `NK` | url | 1003 | 654 | 4783 |
| `SK` | url | 964 | 617 | 0 |
| `EV-VL` | url | 514 | 421 | 3039 |
| `HP` | url | 418 | 382 | 18 |
| `EV-SS` | url | 341 | 334 | 246 |
| `RD-SS` | url | 556 | 319 | 319 |
| `AG-PL` | url | 494 | 312 | 678 |
| `F0` | url | 288 | 285 | 13 |
| `Ni` | url | 358 | 276 | 84 |
| `AN` | url | 265 | 259 | 42 |
| `ET` | url | 329 | 244 | 1084 |
| `HC` | url | 1175 | 198 | 166 |
| `RD-VL` | url | 2178 | 195 | 3451 |
| `LA` | url | 786 | 171 | 14650 |
| `ST-VL` | url | 2154 | 148 | 483 |
| `RD-VM` | url | 565 | 125 | 4029 |
| `F18` | url | 103 | 91 | 43 |
| `HC-VM` | url | 456 | 87 | 907 |
| `ST-VM` | url | 546 | 82 | 29 |

## History

working per run (last 30): ▆█▇▆▆▆▆▆█▆▆▇█████████████▇▇█▇█

| date (UTC) | tested | alive | working | avg ↓ | max ↓ | avg ↑ | clean |
|------------|-------:|------:|--------:|----:|----:|----:|------:|
| 2026-11-27 05:07 | 4521 | 1181 | 1147 | 7.6M | 72.8M | 4.9M | 412 |
| 2026-11-26 05:02 | 4523 | 1027 | 968 | 10.5M | 141.6M | 7.4M | 295 |
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

## Subscription files

Everything under [`subs/`](subs/) is regenerated every run. Plain lists are one proxy URI per line; `.base64.txt` variants are the same list base64-encoded for clients that need the classic sub format.

| file | contents | direct subscription URL |
|------|----------|--------------------------|
| [`all.txt`](subs/all.txt) | ALL working proxies, ranked by speed (1147) | `https://raw.githubusercontent.com/onlyactions/proxy-scanner/main/subs/all.txt` |
| [`all.base64.txt`](subs/all.base64.txt) | same, base64 | `https://raw.githubusercontent.com/onlyactions/proxy-scanner/main/subs/all.base64.txt` |
| [`top150.txt`](subs/top150.txt) | top 150 ranked (150) | `https://raw.githubusercontent.com/onlyactions/proxy-scanner/main/subs/top150.txt` |
| [`top150.base64.txt`](subs/top150.base64.txt) | same, base64 | `https://raw.githubusercontent.com/onlyactions/proxy-scanner/main/subs/top150.base64.txt` |
| [`by-proto/`](subs/by-proto/) | one list per protocol (6 protocols) | ...`/subs/by-proto/<proto>.txt` |
| [`by-engine/allxray.txt`](subs/by-engine/allxray.txt) | verified on Xray (1026) | `https://raw.githubusercontent.com/onlyactions/proxy-scanner/main/subs/by-engine/allxray.txt` |
| [`by-engine/allsingbox.txt`](subs/by-engine/allsingbox.txt) | verified on sing-box (119) | `https://raw.githubusercontent.com/onlyactions/proxy-scanner/main/subs/by-engine/allsingbox.txt` |
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

The typical aggregator relists whatever it scraped, so 80-95% of entries are dead within hours — this run found **74% unreachable** and **75% unusable** overall. Only tunnels that completed a real HTTPS handshake, returned a verifiable foreign exit IP, and moved a 1 MB download **and** a 512 KB upload through the tunnel are counted as working here.

## Disclaimers

- Free proxies are **public and untrusted**. Anything you send through them can be observed or modified by their operators. Never use them for authentication, payments, or anything sensitive.
- All configs are collected from publicly posted sources (GitHub subscriptions and public Telegram channels); no credentials are guessed or brute-forced. Removal requests: open an issue.
- Availability is transient: the working list is re-verified daily, and past performance never guarantees future availability.

