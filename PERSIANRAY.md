# awg-ios

PersianRay iOS fork layout (same idea as `psiphon-ios`):

- Tree root = [amneziawg-go v3.1.20260814](https://github.com/amnezia-vpn/amneziawg-go) (already in this machine's Go module cache).
- `cmd/persianray` = SOCKS sidecar + **injected** policy lists + parallel ping.

Do **not** put domain lists in this Go code. The iOS app sends `ad_exact` / `ad_suffix` / … from `PolicyDomains.psiphonHostLists()`.

## Download (if you need a clean copy)

```bash
git clone --branch v3.1.20260814 --depth 1 https://github.com/amnezia-vpn/amneziawg-go.git awg-ios
```

Keep `cmd/persianray/` from this repo on top of that tree.

## Policy vs ping

| Call | `apply_policy` | Ads / IR bypass / Safe Search / adult |
|------|----------------|----------------------------------------|
| `AwgStart` (Connect) | `true` + lists from Swift | yes |
| `AwgPing` | forced off | **never** |

Ping runs up to `parallel` in-process tunnels (max 8) and `GET /generate_204` through each SOCKS.

## Hop + Amnezia spoofing

`hop: true` starts **two** WireGuard devices in this process:

1. **Outer** — hop account, `endpoint` = clean Cloudflare IP. Junk (`jc`/`jmin`/`jmax`) and `i1`/`i1_sni` apply **only here** (Amnezia spoofing). Hop without spoofing still uses this device with `jc=0`.
2. **Inner** — primary Warp Plus account. UDP goes through the outer netstack to `inner_endpoint` / `inner_endpoints` (default `188.114.97.170:2408`). SOCKS exits via the inner stack.

`hop_shield: true` (Connect only): ip-api country on outer vs inner; retry up to 4 inner hosts. Ping sets `hop_shield: false`.

`AwgSetHopInner("host:port")` swaps the inner session without restarting the outer.

Rebuild `Awg.xcframework` after this change (`awg-apple`).

## Build (Mac)

```bash
cd awg-ios/MobileLibrary/iOS
chmod +x build-awg-framework.sh
./build-awg-framework.sh
# → awg-ios/build/Awg.xcframework
```

GitHub Action: `awg-apple` (workflow_dispatch). Drop the artifact into `persianray-ios/Vendor/Awg/` later.
