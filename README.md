# socks5 proxy server: how to get a stable SOCKS5 endpoint without babysitting free IP lists

Most people typing "socks5 proxy server" into a search box want one of three things: a working list of SOCKS5 addresses, an explanation of why SOCKS5 keeps showing up in tool settings next to "HTTP", or a paid endpoint that survives more than twenty minutes of real traffic. If you're in the third group, the free-list phase is usually where a week disappears.

This is a practical walkthrough of the protocol, what actually breaks in practice, and how a paid residential setup like 9Proxy hands you a SOCKS5 endpoint you can paste into a scraper, a browser, or an antidetect profile.

## What a SOCKS5 proxy server actually is

SOCKS5 is a protocol, not a product. It's defined in RFC 1928, and it does one job: it takes a TCP connection from your app and relays it to a destination, without caring what's inside that connection.

That last part is the whole point. An HTTP proxy reads your request headers, and often rewrites them — stripping or adding `X-Forwarded-For`, forcing a `User-Agent`, blocking non-HTTP traffic. SOCKS5 does none of that. It just pipes bytes. That's why every open-source implementation you'll find (Rust, Go, Node, Dart) exposes the same small set of commands: `CONNECT`, `BIND`, and `UDP ASSOCIATE`, plus address types for IPv4, IPv6 and raw domain names.

Two details matter more than they look:

- **Authentication is a separate handshake.** Username/password is method `0x02`. A lot of SOCKS5 servers in the wild run with no auth at all, because it's simpler to test.
- **Port 1080 is the convention, not the law.** Self-hosted servers usually listen there; commercial providers hand you a host and a port from their own pool, often in the 60000 range for local forwarding setups.

The practical consequence: anything that speaks TCP — Python's `requests`, Scrapy, a fingerprint browser, `curl` — can go through SOCKS5 without being rewritten. A quick sanity check on a local server looks like this:

bash
curl --socks5-hostname user:pass@127.0.0.1:1080 https://ipinfo.io


If that returns an IP that isn't yours, the relay works.

## The free SOCKS5 list problem

Public SOCKS5 lists get indexed heavily, and there's a reason they're free. Three things reliably go wrong:

1. **They die fast.** Hours, sometimes minutes. Open proxies get scanned, abused, and shut down constantly.
2. **You don't know who's on the other end.** Every request you send passes through a stranger's machine. Credentials, session cookies, API keys — all of it is visible to whoever runs that server if the traffic isn't end-to-end encrypted.
3. **They're shared with everyone else.** The IP you just tested is probably already on a dozen blocklists from the last person who used it.

That's the honest tradeoff. Free SOCKS5 is fine for a one-off geo-check on a throwaway endpoint. It's a bad foundation for anything involving logins, payment flows, or work you get paid for.

Paid residential SOCKS5 endpoints flip the economics: instead of hunting for IPs, you buy either a fixed number of IPs (with unlimited bandwidth while they're active) or a bandwidth balance you spend across rotating IPs. 9Proxy is one of the providers built around exactly that split, with 20M+ residential IPs across 90+ countries and both HTTP/HTTPS and SOCKS5 support across its product lines.

## SOCKS5 or HTTP? A quick decision

If you only need one rule, use this:

| Situation | Better choice |
| --- | --- |
| Automating a browser (Playwright, antidetect profiles, fingerprint browsers) | SOCKS5 |
| Non-HTTP protocols, SSH, custom TCP clients | SOCKS5 |
| Tools that only expose an "HTTP proxy" field (most SEO rank trackers, some ad-verification platforms) | HTTP/HTTPS |
| iOS device-level proxy without an extra app | HTTP (iOS only supports HTTP natively) |

Pro tip from the provider docs themselves: tools like ixBrowser, MuLogin, XLogin and Hidemium all have a proxy type dropdown where you pick SOCKS5 and paste the endpoint. Same for iOS, except Apple's built-in Wi-Fi proxy settings only do HTTP — getting SOCKS5 system-wide there means an app like Shadowrocket in between.

One caveat that bites people: not every app or browser supports SOCKS authentication. If the field only accepts `IP:port` with no username/password, a no-auth endpoint is your only option — which is precisely why you don't want to build a workflow on public no-auth proxies.

## What to check before paying for SOCKS5

Skip the marketing pages and check these six things:

- **Protocol support, explicitly stated.** Lots of residential providers advertise HTTP/HTTPS and quietly skip SOCKS5, or support it only on certain plan types.
- **How the endpoint arrives.** A remote `host:port` with username/password is portable. A locally forwarded port that requires a desktop app is not — it works great, but it ties you to that machine.
- **Session control.** Rotating per request versus sticky for N minutes. Scraping usually wants rotating; account work wants sticky.
- **Targeting granularity.** Country is table stakes. City, state and ISP filtering is what separates a usable pool from a frustration generator.
- **Validity rules.** Does unused balance expire? What happens when a sticky session ends mid-job?
- **Auth options.** Username/password, IP whitelisting, or both.

This is where the two 9Proxy product models genuinely differ, and the difference matters more for SOCKS5 users than for anyone else.

## How 9Proxy hands you a SOCKS5 endpoint

9Proxy runs two residential models, and they are not the same thing with different units.

**Residential by GB (bandwidth).** You buy gigabytes, then generate endpoints from the dashboard. You get a fixed hostname, a port, a sub-user username and password. Targeting lives inside the username string, like `subaccount-country-us-city-newyork-sst-15-ssid-bot01`. Sticky sessions use `sst` (minutes) and `ssid` (session ID for parallel work), rotating mode just drops those fields. No desktop app needed, and both username/password auth and IP whitelisting are supported. This is the model that behaves like a classic remote SOCKS5 proxy server.

**Residential by IPs.** You buy a pool of IPs with unlimited bandwidth per IP. IPs don't expire when unused, and each one lives anywhere from a few hours up to about 24 hours. The catch: it works through port forwarding in the 9Proxy app (Windows/macOS), so the SOCKS5 endpoint you plug into your tools is usually `localhost:60000`-style, on your own machine. Proxy authentication against that local port is off by default and can be switched on in the Forwarding List.

Both models speak SOCKS5, so the endpoints drop into the same antidetect browsers, scrapers and automation scripts. What changes is where the endpoint lives.

Two features worth knowing about, because they change day-to-day costs:

- **Today List.** Proxies you forwarded in the last 24 hours stay reusable at no extra cost if they come back online. For account farming or parallel scraping, that's real money.
- **Auto Rotation Proxy.** On IP-based plans you can set rotation intervals on selected ports instead of forwarding a new IP by hand every time.

New users can also ask support about a limited trial, subject to availability — worth doing before committing to a large IP package, since it's the only way to test whether the IPs behave well against your specific targets.

## All 9Proxy packages, current rates

9Proxy announced its first price adjustment in three years, effective 1 June 2026 (00:00 UTC). IP-based and bundle prices moved; GB-based pricing was left untouched. Here's the full published package list.

| Plan | Type | What you get | Price | Validity / billing | Buy |
| --- | --- | --- | --- | --- | --- |
| 100 IPs | By IP | 100 residential IPs, unlimited bandwidth per active IP | $24 total ($0.24/IP) | One-off, IPs don't expire | [ Grab the 100 IP package](https://bit.ly/9-Proxy) |
| 500 IPs | By IP | 500 IPs, unlimited bandwidth | $72 ($0.144/IP) | One-off, IPs don't expire | [ Compare the 500 IP tier](https://bit.ly/9-Proxy) |
| 1,000 IPs (+500 bonus) | By IP | 1,500 IPs total, unlimited bandwidth | $126 ($0.084/IP) | One-off, IPs don't expire | [ Check the 1,000+500 IP bundle](https://bit.ly/9-Proxy) |
| 2,500 IPs | By IP | 2,500 IPs, unlimited bandwidth | $210 | One-off, IPs don't expire | [ See the 2,500 IP package](https://bit.ly/9-Proxy) |
| 5,000 IPs | By IP | 5,000 IPs, unlimited bandwidth | $360 | One-off, IPs don't expire | [ Open the 5,000 IP tier](https://bit.ly/9-Proxy) |
| 15,000 IPs | By IP | 15,000 IPs, unlimited bandwidth | $720 | One-off, IPs don't expire | [ View the 15,000 IP package](https://bit.ly/9-Proxy) |
| 25,000 IPs | By IP | 25,000 IPs, unlimited bandwidth | $863 | One-off, IPs don't expire | [ Check the 25,000 IP tier](https://bit.ly/9-Proxy) |
| 50,000 IPs | By IP | 50,000 IPs, unlimited bandwidth | $1,438 | One-off, IPs don't expire | [ See the 50,000 IP package](https://bit.ly/9-Proxy) |
| Business 100,000 IPs | By IP | High-volume IP pool for teams | $2,300 | One-off, IPs don't expire | [ Get the 100,000 IP package](https://bit.ly/9-Proxy) |
| Business 200,000 IPs | By IP | High-volume IP pool for teams | $4,140 | One-off, IPs don't expire | [ Open the 200,000 IP tier](https://bit.ly/9-Proxy) |
| Business 500,000 IPs | By IP | Largest IP pool | $8,625 | One-off, IPs don't expire | [ Check the 500,000 IP package](https://bit.ly/9-Proxy) |
| 5 GB | By GB | 5 GB traffic, unlimited endpoints | $15 ($3.00/GB) | 180 days | [ Start with the 5 GB pack](https://bit.ly/9-Proxy) |
| 50 GB (+5 GB bonus) | By GB | 55 GB traffic, unlimited endpoints | $105 ($2.10/GB) | 180 days | [ Get the 50+5 GB pack](https://bit.ly/9-Proxy) |
| 100 GB | By GB | 100 GB traffic | $150 ($1.50/GB) | 180 days | [ View the 100 GB package](https://bit.ly/9-Proxy) |
| 200 GB | By GB | 200 GB traffic | $200 ($1.00/GB) | 180 days | [ Compare the 200 GB tier](https://bit.ly/9-Proxy) |
| 1,000 GB | By GB | 1,000 GB traffic | $800 ($0.80/GB) | 180 days | [ Open the 1,000 GB package](https://bit.ly/9-Proxy) |
| 2,000 GB | By GB | 2,000 GB traffic | $1,500 ($0.75/GB) | 180 days | [ See the 2,000 GB tier](https://bit.ly/9-Proxy) |
| Enterprise 3,000 GB | By GB | 3,000 GB traffic | $2,160 ($0.72/GB) | Unlimited (no expiry) | [ Check the 3,000 GB enterprise pack](https://bit.ly/9-Proxy) |
| Enterprise 6,000 GB | By GB | 6,000 GB traffic | $4,200 ($0.70/GB) | Unlimited (no expiry) | [ View the 6,000 GB enterprise pack](https://bit.ly/9-Proxy) |
| Enterprise 10,000 GB | By GB | 10,000 GB traffic | $6,800 ($0.68/GB) | Unlimited (no expiry) | [ Open the 10,000 GB enterprise pack](https://bit.ly/9-Proxy) |
| Starter Bundle | IP + GB | 100 IPs + 5 GB | $30 | Bundled traffic valid 180 days | [ Get the Starter Bundle](https://bit.ly/9-Proxy) |
| Popular Bundle | IP + GB | 1,500 IPs + 50 GB | $180 | Bundled traffic valid 180 days | [ See the Popular Bundle](https://bit.ly/9-Proxy) |
| Pro Bundle | IP + GB | 5,000 IPs + 500 GB | $720 | Bundled traffic valid 180 days | [ Check the Pro Bundle](https://bit.ly/9-Proxy) |

A note on the headline numbers you'll see in ads. Claims like "from $0.018/IP" are only reachable at the very top of the IP tiers — the 200,000-IP and 500,000-IP packages. At the 100-IP level you're paying $0.24 per IP. That's not a criticism, it's just how volume pricing works everywhere; check the tier you'll actually buy, not the footer of a banner.

Payment options include credit and bank cards, crypto (USDT, BTC, ETH, LTC, DOGE and others), Alipay, Apple Pay and Google Pay.

## Which package fits an actual SOCKS5 workload

Being blunt about it, because this is the part marketing pages blur:

- **Testing anything at all, first time:** 5 GB for $15. Run your real targets through it, measure your GB burn, then decide.
- **Scrapers and automation where IP continuity doesn't matter:** GB-based. You'll generate unlimited endpoints and pay only for traffic consumed.
- **Multi-account work, social platforms, anything that punishes IP changes:** IP-based. You want the same IP holding steady across a session, and unlimited bandwidth means a big upload doesn't eat your balance.
- **Mixed agency work — some clients need sticky IPs, some need traffic:** a bundle. Starter at $30 and Popular at $180 line up neatly with small and mid-size client loads.
- **Runners at scale:** the 100,000+ IP tiers, or enterprise GB if your pipeline runs continuously and you don't want an expiry clock ticking.

The 180-day validity on GB traffic is the detail that makes the bandwidth model less scary than it sounds. Uneven project work doesn't punish you the way a monthly subscription does.

## Setup, both ways

**Dashboard route (GB-based):** generate the proxy with User-Pass auth, pick a country plus optional state/city/ISP, choose sticky or rotating, copy the connection string in whichever field order your tool wants. Then paste it into your antidetect browser or script and test against `ipinfo.io` before running anything real.

**App route (IP-based):** open the 9Proxy desktop app, filter the proxy list by geolocation, right-click and forward the IP to a port (single, multiple, free ports, or all). Switch to the Forwarding List, hit Test, then copy the local IP and port into your software — selecting SOCKS5 as the proxy type. If you want credential-based access to that local port, enable Proxy Authentication in the Forwarding List; otherwise it accepts connections without a username.

For phones on the same Wi-Fi, you can point them at your computer's LAN IP and the forwarded port. iPhones and iPads need the HTTP setting for the native Wi-Fi proxy, or Shadowrocket with SOCKS5 selected.

## Where it's rough

Worth knowing before you buy:

- **The desktop app is mandatory for IP-based plans.** Third-party reviews flag this as the main friction point — it's fine on one workstation, awkward if you're juggling several machines or want a fully headless setup. If that's your situation, buy GB-based and skip the app entirely.
- **Residential IP lifetimes are variable.** A few hours, up to a day. That's a property of residential networks, not a bug in this provider, but it means sticky sessions aren't infinite sessions.
- **Streaming is a coin flip.** One review noted detection on services like Netflix even where e-commerce targets worked cleanly. If your use case is streaming, test before you buy in bulk.
- **Trials depend on availability.** They're handled through support rather than a one-click button in the dashboard.

Aggregated third-party reviews land around 9/10, with the app requirement and promo-dependent trials as the recurring complaints.

## Quick troubleshooting for SOCKS5 connections

| Symptom | Likely cause |
| --- | --- |
| Connection refused on `localhost:60000` | Port not forwarded, or the app isn't running |
| `curl` returns your own IP | Tool is ignoring the proxy flag — check for `--socks5` vs `--socks5-hostname` |
| Auth rejected | Wrong sub-user password, or the username's targeting string is malformed |
| Empty results, no error | Bandwidth balance exhausted, or the chosen city/ISP combination has no stock |
| Works in browser, fails in script | Your client library doesn't support SOCKS authentication over the endpoint type you generated |

Narrowing targeting with country + state + city + ISP at once is the most common self-inflicted wound. Each filter shrinks the available pool. Target country alone if speed matters, and add filters only where the task genuinely needs them.

## The short version

Free SOCKS5 endpoints are fine for a throwaway test and bad for anything you care about. If you need a SOCKS5 proxy server that behaves predictably, you're choosing between a bandwidth model that never expires mid-month and an IP model with unlimited traffic and reusable IPs.

Start with 5 GB for $15, measure your actual usage against your real targets, and move to IP-based packages or a bundle only if you find yourself repeatedly needing the same address — or if you'd rather [👉 set up a 9Proxy account and test the SOCKS5 endpoint on your own workload](https://bit.ly/9-Proxy) than take a table's word for it.
