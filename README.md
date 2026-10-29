# Blue Knight Panel · Hostless

A Node.js proxy panel adapted for [Hostless](https://hostless.cloud/), with VLESS, VMess, Trojan, subscriptions, engine recovery, and resource monitoring.

**[YouTube · @BlueKnight-Net](https://www.youtube.com/@BlueKnight-Net)** · **[Telegram · @BlueKnight_Net](https://t.me/BlueKnight_Net)**

## Deploy on Hostless

1. Sign in to [Hostless](https://hostless.cloud/) and create an **App** connected to this GitHub repository.
2. Select the **`blue-knight-net`** branch.
3. Use these build and startup settings:

   | Setting | Value |
   | --- | --- |
   | Build system | Detect automatically |
   | Working directory | `/` |
   | Build command | Leave empty |
   | Start command | `npm start` |
   | Internal port | Use the `PORT` supplied by Hostless |

4. Attach a persistent volume at **`/data`** to keep panel settings and downloaded engines across deployments. Choose its capacity in the hosting dashboard; storage may incur charges.
5. Keep the default TCP health check, or use an HTTP check on **`/knight/login`**.
6. Deploy. The first startup downloads sing-box and Xray-core from GitHub, so allow time for both engines to install and start.
7. Open **`https://YOUR-APP.hostless.app/knight`**, create your panel password, and open **Subscriptions** to import a link into your client.

The included [`hostless.yaml`](hostless.yaml) declares automatic builds and `npm start`. The app binds to `0.0.0.0` and reads Hostless's `PORT`; do not change the internal port to 443.

See the official [Hostless Node.js deployment guide](https://docs.hostless.cloud/javascript/express) for platform setup.

## Public URLs and configs

For a generated `*.hostless.app` domain, the panel automatically generates **HTTPS/TLS links on public port 443**, without pinning its internal self-signed certificate. The internal container port can be different.

Open the panel once after deployment so it can learn and save the public domain. Refresh your client subscription after changing a domain, UUID, or deployment configuration.

- **VLESS, VMess, and Trojan over WebSocket:** served by sing-box.
- **HTTPUpgrade:** served by sing-box; requires support from your client and the hosting proxy.
- **VLESS-XHTTP:** served by Xray-core; requires a client with XHTTP support.
- **Shadowsocks over WebSocket:** served by Xray-core where generated and supported by the client.

A working web panel does not by itself confirm that the proxy engines are running. Use a real connection or URL test in your client.

## Automatic recovery

- Each engine has bounded crash-restart retries with backoff. The retry counter resets after a stable five-minute run.
- Engine watchdogs check local TCP listeners every minute and request a restart after three consecutive failed checks.
- A self-ping checks the panel every 15 minutes once its public domain is known.
- IPv4-only domain resolution is selected automatically when no public IPv6 interface is detected.
- Internal engine ports are assigned uniquely to avoid collisions between transports.

**The self-ping cannot wake a suspended container or restart a stopped Node.js process.** Availability still depends on the host's lifecycle and health checks.

## Usage tab

The Usage tab includes per-protocol traffic counters and live CPU/RAM readings. When container metrics are available, they include the proxy engines; fallback readings are labelled as panel-process readings.

**App data storage** measures files in the data directory, refreshed every 30 seconds. It does not report shared host-disk usage or invent a storage quota. Check the hosting dashboard for your purchased capacity and available allowance.

Expiry and credit cards show previously saved values when present. They are not connected to Hostless billing; unknown values are shown as **Not provided**.

## Persistent data

Keep the `/data` volume. It stores the panel login, UUID, subscription token, certificates, traffic counters, and engine files. Without persistent storage, redeployment may reset these values and require new client configs.

If you use another mount path, set `BK_DATA_DIR` to that path. Attaching an empty volume over an existing data directory hides its previous contents, so back up existing data before switching storage.

## Optional overrides

Normal deployment on a generated Hostless domain does not require these variables. Explicit values override automatic behavior.

| Variable | Purpose |
| --- | --- |
| `PANEL_PASSWORD` | Set a fixed panel password, at least 8 characters |
| `UUID` | Keep a fixed client UUID |
| `SUB_TOKEN` | Keep a fixed subscription token |
| `BK_DATA_DIR` | Choose the persistent data directory; default `/data` when writable |
| `DOMAIN` | Specify the public domain, especially for a custom domain |
| `PUBLIC_BASE_URL` | Override the public panel/subscription base URL, including its scheme |
| `LINK_TLS=true` | Generate links for an HTTPS edge; useful for custom domains |
| `LINK_PORT=443` | Override the public client port independently of the internal `PORT` |
| `BK_IPV4_ONLY=true` or `false` | Override IPv4-only domain resolution |
| `ENGINE_WATCHDOG=false` | Disable the additional listener watchdogs |
| `SELF_PING=false` | Disable the 15-minute self-ping |

Keep passwords and subscription tokens private. Configure secrets in your hosting dashboard rather than committing them to Git.

## Troubleshooting

- **Panel opens, configs fail:** check runtime logs for successful sing-box and Xray startup. WebSocket configs need sing-box; XHTTP needs Xray.
- **`bad path` errors:** update your client subscription and use the matching transport. VMess WebSocket uses a path ending in `vmess`; VMess HTTPUpgrade uses `vmess-hu`.
- **IPv6 `network is unreachable`:** set `BK_IPV4_ONLY=true` if automatic interface detection does not match your host's routing. Clients sending literal IPv6 destinations still need to use IPv4 or domain destinations.
- **Deployment is queued or fails Hostless-file validation:** inspect the deployment configuration and platform error before changing proxy settings. Confirm the selected branch and the included `hostless.yaml`.
- **Credentials or configs reset after redeployment:** verify that the persistent volume is attached and the startup log points to its data directory.

## Blue Knight Net

- [Watch tutorials on YouTube](https://www.youtube.com/@BlueKnight-Net)
- [Follow updates on Telegram](https://t.me/BlueKnight_Net)
