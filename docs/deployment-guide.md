# CloudSubscription – Deployment Guide

## System Requirements

| Component | Minimum |
|---|---|
| OS | Ubuntu 22.04 LTS / Debian 12 or Windows Server 2022 |
| RAM | 256 MB |
| .NET Runtime | 9.0 (or self-contained publish) |
| Ports | 80 (HTTP), 443 (HTTPS) |
| External dependency | PayPal merchant account with IPN enabled |

## Prerequisites

1. A **PayPal Business account** with IPN notifications enabled.
2. A running **CloudServerWebUI** (or CloudServerUISupport API endpoint) to receive provisioning calls.
3. A DNS record pointing to this server's IP.

## Installation on Linux

```bash
# 1. Publish
dotnet publish CloudSubscription/CloudSubscription.csproj \
  -c Release -r linux-x64 --self-contained \
  -o /opt/cloudsubscription

# 2. Configure
nano /opt/cloudsubscription/appsettings.json
```

```json
{
  "PayPalBusinessEmail": "merchant@yourdomain.com",
  "ApiEndpoint":         "https://cloudserver.yourdomain.com",
  "ApiPrivateKey":       "<your-api-private-key>"
}
```

```bash
# 3. Install as systemd service
sudo bash /opt/cloudsubscription/install.sh
sudo systemctl enable cloudsubscription
sudo systemctl start cloudsubscription
```

## HTTPS

On first start, HTTPS is configured automatically if a valid domain resolves to this server and port 80 is reachable. Alternatively, place a reverse proxy (nginx/Caddy) in front.

## PayPal IPN Configuration

1. Log in to PayPal → Account Settings → Notifications → Instant Payment Notifications.
2. Set the Notification URL to: `https://subscription.yourdomain.com/`
3. Enable IPN.
4. Test with the [PayPal IPN Simulator](https://developer.paypal.com/developer/ipnSimulator/).

## Environment Variables (alternative to appsettings.json)

```bash
export PayPalBusinessEmail="merchant@yourdomain.com"
export ApiEndpoint="https://cloudserver.yourdomain.com"
export ApiPrivateKey="<your-api-private-key>"
```

## Logs

```bash
sudo journalctl -u cloudsubscription -f
```

## Updating

Re-publish and restart the service:

```bash
dotnet publish ... -o /opt/cloudsubscription
sudo systemctl restart cloudsubscription
```

## Firewall

```bash
sudo ufw allow 80/tcp
sudo ufw allow 443/tcp
```
