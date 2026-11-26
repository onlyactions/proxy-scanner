# Free Proxy Scanner

> **Really tested** free proxy aggregator. Every proxy below was fetched from public sources, tunneled through with Xray/sing-box, and verified to actually change your egress IP. No CSV-validation theater, no blind relisting.

![working](badges/working.json) ![alive](badges/alive.json) ![speed](badges/speed.json) ![up-speed](badges/up-speed.json) ![max-speed](badges/max-speed.json) ![clean](badges/clean.json) ![countries](badges/countries.json) ![sources](badges/sources.json) ![updated](badges/updated.json)

## What this is

A GitHub Actions pipeline (runs daily) that fetches free proxy configs from **173 URL subscriptions** and **81 Telegram channels**, parses 9 protocols (vless / vmess / trojan / ss / hysteria2 / hysteria / tuic / socks / http), deduplicates them, and **actually connects through every single one** to separate working proxies from dead ones.

Latest run: **2026-11-26 04:23:13 UTC** — 124750 unique proxies collected from 105/105 URL sources and 8/8 Telegram channels; **4523** endpoints really tested, **1027** reachable, **968** verified working (egress IP confirmed changed) across **41 exit countries** — **295** of them on unflagged ('clean') exit IPs and **270** on residential IPs.

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

All 968 working proxies are published in [`subs/all.txt`](subs/all.txt); this table shows the top 25. The `name` column is exactly what your client will show (see [How to read a proxy name](#how-to-read-a-proxy-name)).

| # | name | proto | entry server | latency | engine |
|---|------|-------|--------------|---------|--------|
| 1 | #1 🇺🇸 US | GTHost AS63023 | ↓141.6 ↑25.4Mbps | 66.163.122.30 | DC·Flag | `trojan` | `66.163.122.46:443` | 145 ms | xray |
| 2 | #2 🇺🇸 US | GTHost AS63023 | ↓135.7 ↑57.4Mbps | 45.86.51.144 | DC·Flag | `ss` | `45.86.51.18:19999` | 102 ms | xray |
| 3 | #3 🇺🇸 US | GTHost AS63023 | ↓120.2 ↑0.5Mbps | 172.111.38.72 | DC·Flag | `ss` | `172.111.38.72:19999` | 93 ms | xray |
| 4 | #4 🇺🇸 US → 🇦🇺 AU | The Constant AS20473 | ↓104.0 ↑0.4Mbps | 45.76.115.243 | DC·Clean | `vmess` | `67.220.95.3:18000` | 4532 ms | xray |
| 5 | #5 🇺🇸 US | GTHost AS63023 | ↓95.1 ↑118.5Mbps | 66.163.122.165 | DC·Flag | `trojan` | `66.163.122.26:443` | 150 ms | xray |
| 6 | #6 🇺🇸 US | OVH AS16276 | ↓86.6 ↑106.9Mbps | 135.148.42.13 | DC·Flag | `ss` | `135.148.42.13:8388` | 21 ms | xray |
| 7 | #7 🇺🇸 US | Washington DC AS63023 | ↓85.2 ↑119.6Mbps | 185.230.230.142 | DC·Flag | `ss` | `185.230.230.18:19999` | 56 ms | xray |
| 8 | #8 🇺🇸 US | GTHost AS63023 | ↓78.4 ↑78.8Mbps | 66.163.122.81 | DC·Flag | `trojan` | `66.163.122.7:443` | 61 ms | xray |
| 9 | #9 🇺🇸 US | GTHost AS63023 | ↓77.1 ↑61.1Mbps | 38.107.226.5 | RES·Flag | `vmess` | `38.107.226.5:1270` | 86 ms | xray |
| 10 | #10 🇺🇸 US | GTHost AS63023 | ↓75.3 ↑46.8Mbps | 172.111.38.246 | DC·Flag | `vmess` | `172.111.38.246:1270` | 133 ms | xray |
| 11 | #11 🇺🇸 US | Baykov Ilya Sergeevich AS41745 | ↓70.0 ↑0.6Mbps | 45.138.100.248 | RES·Flag | `vless` | `45.138.100.248:10443` | 41 ms | singbox |
| 12 | #12 🇺🇸 US | OVH AS16276 | ↓69.4 ↑85.3Mbps | 40.160.86.43 | DC·Clean | `ss` | `40.160.86.43:8388` | 96 ms | xray |
| 13 | #13 🇺🇸 US | GTHost AS63023 | ↓65.6 ↑83.3Mbps | 66.163.122.46 | DC·Flag | `trojan` | `66.163.122.30:443` | 62 ms | xray |
| 14 | #14 🇺🇸 US | Baykov Ilya Sergeevich AS41745 | ↓63.2 ↑40.4Mbps | 185.95.231.156 | RES·Flag | `vless` | `185.95.231.156:443` | 77 ms | xray |
| 15 | #15 🇺🇸 US | Baykov Ilya Sergeevich AS41745 | ↓63.2 ↑31.4Mbps | 45.138.100.248 | RES·Flag | `vless` | `45.138.100.248:14430` | 48 ms | singbox |
| 16 | #16 🇺🇸 US | Datacamp AS212238 | ↓60.4 ↑56.5Mbps | 37.19.198.160 | DC·Flag | `ss` | `37.19.198.160:443` | 34 ms | xray |
| 17 | #17 🇺🇸 US | GTHost AS63023 | ↓60.3 ↑40.3Mbps | 66.163.122.113 | DC·Flag | `trojan` | `66.163.122.25:443` | 61 ms | xray |
| 18 | #18 🇺🇸 US | Datacamp AS212238 | ↓59.1 ↑55.5Mbps | 37.19.198.243 | DC·Flag | `ss` | `37.19.198.243:443` | 43 ms | xray |
| 19 | #19 🇺🇸 US | Clouvider AS62240 | ↓56.5 ↑25.1Mbps | 185.195.57.168 | DC·Flag | `vless` | `169.40.42.90:443` | 197 ms | xray |
| 20 | #20 🇺🇸 US | Datacamp AS212238 | ↓54.7 ↑59.9Mbps | 37.19.198.236 | DC·Flag | `ss` | `37.19.198.236:443` | 41 ms | xray |
| 21 | #21 🇺🇸 US | Datacamp AS212238 | ↓52.2 ↑0.2Mbps | 37.19.198.244 | DC·Flag | `ss` | `37.19.198.244:443` | 41 ms | xray |
| 22 | #22 🇺🇸 US | Wedeploy AS210699 | ↓48.5 ↑45.4Mbps | 200.102.179.179 | DC·Clean | `ss` | `200.102.179.179:8388` | 91 ms | xray |
| 23 | #23 🇺🇸 US | New Jersey AS63023 | ↓46.4 ↑30.4Mbps | 193.151.163.139 | DC·Flag | `ss` | `193.151.163.18:19999` | 36 ms | xray |
| 24 | #24 🇺🇸 US | Datacamp AS212238 | ↓44.3 ↑28.8Mbps | 149.40.57.135 | DC·Flag | `ss` | `149.40.57.135:19999` | 122 ms | xray |
| 25 | #25 🇺🇸 US | GTHost AS63023 | ↓43.2 ↑25.7Mbps | 38.91.100.69 | DC·Flag | `vmess` | `38.91.100.69:1270` | 71 ms | xray |

## Exit locations — which countries work

Every working proxy, grouped by the country of its **verified exit IP** (phase 2), with a dedicated subscription file per country. Currently 41 countries have at least one working exit.

| country | working | avg speed | best latency | share | list |
|---------|--------:|----------:|-------------:|------:|------|
| 🇺🇸 United States | 270 | 21.7 Mbps | 21 ms | 28% | [`US.txt`](subs/by-country/US.txt) |
| 🇳🇱 Netherlands | 164 | 7.4 Mbps | 253 ms | 17% | [`NL.txt`](subs/by-country/NL.txt) |
| 🇩🇪 Germany | 115 | 6.7 Mbps | 288 ms | 12% | [`DE.txt`](subs/by-country/DE.txt) |
| 🇯🇵 Japan | 78 | 1.6 Mbps | 475 ms | 8% | [`JP.txt`](subs/by-country/JP.txt) |
| 🇨🇦 Canada | 42 | 11.1 Mbps | 59 ms | 4% | [`CA.txt`](subs/by-country/CA.txt) |
| 🇫🇷 France | 32 | 8.0 Mbps | 265 ms | 3% | [`FR.txt`](subs/by-country/FR.txt) |
| 🇸🇬 Singapore | 30 | 3.0 Mbps | 694 ms | 3% | [`SG.txt`](subs/by-country/SG.txt) |
| 🇬🇧 United Kingdom | 25 | 6.2 Mbps | 249 ms | 3% | [`GB.txt`](subs/by-country/GB.txt) |
| 🇪🇸 Spain | 21 | 5.7 Mbps | 302 ms | 2% | [`ES.txt`](subs/by-country/ES.txt) |
| 🇸🇪 Sweden | 21 | 5.1 Mbps | 342 ms | 2% | [`SE.txt`](subs/by-country/SE.txt) |
| 🇭🇰 Hong Kong | 18 | 3.4 Mbps | 661 ms | 2% | [`HK.txt`](subs/by-country/HK.txt) |
| 🇨🇭 Switzerland | 17 | 5.4 Mbps | 343 ms | 2% | [`CH.txt`](subs/by-country/CH.txt) |
| 🇦🇺 Australia | 15 | 14.8 Mbps | 113 ms | 2% | [`AU.txt`](subs/by-country/AU.txt) |
| 🇮🇹 Italy | 14 | 8.1 Mbps | 325 ms | 1% | [`IT.txt`](subs/by-country/IT.txt) |
| 🇧🇾 BY | 11 | 6.4 Mbps | 477 ms | 1% | [`BY.txt`](subs/by-country/BY.txt) |
| 🇷🇺 Russia | 10 | 4.4 Mbps | 621 ms | 1% | [`RU.txt`](subs/by-country/RU.txt) |
| 🇰🇷 South Korea | 8 | 2.8 Mbps | 919 ms | 1% | [`KR.txt`](subs/by-country/KR.txt) |
| 🇵🇱 Poland | 7 | 4.4 Mbps | 478 ms | 1% | [`PL.txt`](subs/by-country/PL.txt) |
| 🇮🇳 India | 7 | 2.0 Mbps | 943 ms | 1% | [`IN.txt`](subs/by-country/IN.txt) |
| 🇫🇮 Finland | 6 | 5.3 Mbps | 363 ms | 1% | [`FI.txt`](subs/by-country/FI.txt) |
| 🇰🇿 Kazakhstan | 6 | 3.5 Mbps | 987 ms | 1% | [`KZ.txt`](subs/by-country/KZ.txt) |
| 🇹🇼 Taiwan | 6 | 2.0 Mbps | 838 ms | 1% | [`TW.txt`](subs/by-country/TW.txt) |
| 🇮🇱 Israel | 5 | 2.6 Mbps | 652 ms | 1% | [`IL.txt`](subs/by-country/IL.txt) |
| 🇨🇾 Cyprus | 4 | 5.7 Mbps | 461 ms | 0% | [`CY.txt`](subs/by-country/CY.txt) |
| 🇨🇿 Czechia | 4 | 6.4 Mbps | 488 ms | 0% | [`CZ.txt`](subs/by-country/CZ.txt) |
| 🇧🇷 Brazil | 4 | 6.0 Mbps | 426 ms | 0% | [`BR.txt`](subs/by-country/BR.txt) |
| 🇦🇹 Austria | 3 | 6.2 Mbps | 347 ms | 0% | [`AT.txt`](subs/by-country/AT.txt) |
| 🇦🇱 AL | 3 | 4.5 Mbps | 492 ms | 0% | [`AL.txt`](subs/by-country/AL.txt) |
| 🇹🇷 Turkey | 3 | 3.6 Mbps | 636 ms | 0% | [`TR.txt`](subs/by-country/TR.txt) |
| 🇿🇦 South Africa | 3 | 3.7 Mbps | 820 ms | 0% | [`ZA.txt`](subs/by-country/ZA.txt) |
| 🇹🇭 Thailand | 3 | 2.4 Mbps | 1185 ms | 0% | [`TH.txt`](subs/by-country/TH.txt) |
| 🇧🇬 Bulgaria | 2 | 6.2 Mbps | 780 ms | 0% | [`BG.txt`](subs/by-country/BG.txt) |
| 🇺🇦 Ukraine | 2 | 4.8 Mbps | 914 ms | 0% | [`UA.txt`](subs/by-country/UA.txt) |
| 🇦🇪 UAE | 2 | 2.4 Mbps | 1576 ms | 0% | [`AE.txt`](subs/by-country/AE.txt) |
| 🇮🇷 Iran | 1 | 8.3 Mbps | 388 ms | 0% | [`IR.txt`](subs/by-country/IR.txt) |
| 🇱🇻 Latvia | 1 | 6.8 Mbps | 515 ms | 0% | [`LV.txt`](subs/by-country/LV.txt) |
| 🇷🇴 Romania | 1 | 6.6 Mbps | 5303 ms | 0% | [`RO.txt`](subs/by-country/RO.txt) |
| 🇨🇳 China | 1 | 1.2 Mbps | 2487 ms | 0% | [`CN.txt`](subs/by-country/CN.txt) |
| 🇻🇳 Vietnam | 1 | 0.9 Mbps | 4327 ms | 0% | [`VN.txt`](subs/by-country/VN.txt) |
| 🇳🇴 Norway | 1 | 0.7 Mbps | 4920 ms | 0% | [`NO.txt`](subs/by-country/NO.txt) |
| 🇲🇾 Malaysia | 1 | - | 997 ms | 0% | [`MY.txt`](subs/by-country/MY.txt) |

A `??` exit means the geo lookup could not resolve the exit IP. Base64 variants of every country file sit next to them as `<CC>.base64.txt`.

## Protocol distribution

| protocol | tested | alive | working | alive rate |
|----------|-------:|------:|--------:|-----------:|
| `vless` | 2703 | 521 | 475 | 19% |
| `ss` | 530 | 242 | 237 | 46% |
| `vmess` | 1092 | 143 | 137 | 13% |
| `trojan` | 129 | 70 | 69 | 54% |
| `hysteria2` | 65 | 49 | 48 | 75% |
| `socks` | 2 | 2 | 2 | 100% |
| `http` | 2 | 0 | 0 | 0% |

Per-protocol working lists: [`subs/by-proto/`](subs/by-proto/) (one `vless.txt`, `vmess.txt`, ... per protocol, plus `.base64.txt` variants).

## Engine breakdown

Which core verified each working proxy. The engine fallback means a proxy that failed on its primary core got a second chance on the other one.

| engine | working | note |
|--------|--------:|------|
| Xray-core | 834 | [`allxray.txt`](subs/by-engine/allxray.txt) |
| sing-box | 132 | [`allsingbox.txt`](subs/by-engine/allsingbox.txt) |
| direct (curl) | 2 | plain http/socks, [`alldirect.txt`](subs/by-engine/alldirect.txt) |

## Speed & latency profile

| speed tier | proxies |
|-----------|--------:|
| >= 5 Mbps | 555 |
| 1 - 5 Mbps | 234 |
| 0.25 - 1 Mbps | 69 |
| < 0.25 Mbps | 0 |
| unmeasured | 110 |

Latency (phase-1 HTTPS round trip): median **461 ms**, p90 **3525 ms**. 59 proxies passed connectivity but failed egress verification (leaked the runner IP or broke on the exit check) and are excluded.

## Source health

113 active, 141 auto-retired (10 fetch failures / 10 all-dead runs / 10 days without anything new). Best contributors this run:

| source | kind | tested | alive | new this run |
|--------|------|-------:|------:|-------------:|
| `SK` | url | 1040 | 637 | 5977 |
| `NK` | url | 978 | 559 | 588 |
| `EV` | url | 611 | 479 | 1112 |
| `HP` | url | 473 | 402 | 132 |
| `AG-PL` | url | 481 | 281 | 875 |
| `F0` | url | 254 | 247 | 21 |
| `RD-SS` | url | 462 | 230 | 338 |
| `EV-SS` | url | 224 | 212 | 0 |
| `HC` | url | 1160 | 210 | 261 |
| `AN` | url | 214 | 203 | 36 |
| `RD-VL` | url | 2125 | 190 | 3220 |
| `KA` | url | 2987 | 181 | 7724 |
| `EV-VL` | url | 279 | 174 | 0 |
| `ST-VL` | url | 2100 | 153 | 658 |
| `ET` | url | 228 | 115 | 1251 |
| `RD-VM` | url | 570 | 110 | 4017 |
| `LA` | url | 645 | 92 | 13857 |
| `AZN` | url | 123 | 73 | 172 |
| `SB` | url | 2067 | 73 | 6 |
| `Ni` | url | 172 | 73 | 4 |

## History

working per run (last 30): ▆▆█▇▆▆▆▆▆█▆▆▇█████████████▇▇█▇

| date (UTC) | tested | alive | working | avg ↓ | max ↓ | avg ↑ | clean |
|------------|-------:|------:|--------:|----:|----:|----:|------:|
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
| 2026-11-10 03:31 | 4511 | 1098 | 1052 | 8.2M | 76.8M | - | - |

## Subscription files

Everything under [`subs/`](subs/) is regenerated every run. Plain lists are one proxy URI per line; `.base64.txt` variants are the same list base64-encoded for clients that need the classic sub format.

| file | contents | direct subscription URL |
|------|----------|--------------------------|
| [`all.txt`](subs/all.txt) | ALL working proxies, ranked by speed (968) | `https://raw.githubusercontent.com/onlyactions/proxy-scanner/main/subs/all.txt` |
| [`all.base64.txt`](subs/all.base64.txt) | same, base64 | `https://raw.githubusercontent.com/onlyactions/proxy-scanner/main/subs/all.base64.txt` |
| [`top150.txt`](subs/top150.txt) | top 150 ranked (150) | `https://raw.githubusercontent.com/onlyactions/proxy-scanner/main/subs/top150.txt` |
| [`top150.base64.txt`](subs/top150.base64.txt) | same, base64 | `https://raw.githubusercontent.com/onlyactions/proxy-scanner/main/subs/top150.base64.txt` |
| [`by-proto/`](subs/by-proto/) | one list per protocol (6 protocols) | ...`/subs/by-proto/<proto>.txt` |
| [`by-engine/allxray.txt`](subs/by-engine/allxray.txt) | verified on Xray (834) | `https://raw.githubusercontent.com/onlyactions/proxy-scanner/main/subs/by-engine/allxray.txt` |
| [`by-engine/allsingbox.txt`](subs/by-engine/allsingbox.txt) | verified on sing-box (132) | `https://raw.githubusercontent.com/onlyactions/proxy-scanner/main/subs/by-engine/allsingbox.txt` |
| [`by-country/`](subs/by-country/) | one list per exit country (41 countries, e.g. [`US.txt`](subs/by-country/US.txt), [`DE.txt`](subs/by-country/DE.txt)) | ...`/subs/by-country/<CC>.txt` |
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

The typical aggregator relists whatever it scraped, so 80-95% of entries are dead within hours — this run found **77% unreachable** and **79% unusable** overall. Only tunnels that completed a real HTTPS handshake, returned a verifiable foreign exit IP, and moved a 1 MB download **and** a 512 KB upload through the tunnel are counted as working here.

## Disclaimers

- Free proxies are **public and untrusted**. Anything you send through them can be observed or modified by their operators. Never use them for authentication, payments, or anything sensitive.
- All configs are collected from publicly posted sources (GitHub subscriptions and public Telegram channels); no credentials are guessed or brute-forced. Removal requests: open an issue.
- Availability is transient: the working list is re-verified daily, and past performance never guarantees future availability.

