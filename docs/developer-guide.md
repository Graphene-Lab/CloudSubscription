# CloudSubscription – Developer Guide

## Running Locally

1. Set **CloudSubscription** as the startup project.
2. Configure `appsettings.Development.json`:

```json
{
  "PayPalBusinessEmail": "sandbox@example.com",
  "ApiEndpoint": "https://localhost:7001",
  "ApiPrivateKey": "<your-dev-private-key>"
}
```

3. For local PayPal IPN testing, use [ngrok](https://ngrok.com/) to expose a public URL and configure it in your PayPal sandbox account.

## Project Structure

```
CloudSubscription/
├── Program.cs                         – App setup, PayPal IPN middleware registration
├── Settings.cs                        – Static settings loaded from appsettings.json
├── Events.cs                          – Payment event handlers
├── Panels/
│   ├── CreateNewSubscription.cs       – Subscription creation logic (UI panel)
│   └── LoginCredential.cs             – Login panel
├── Components/
│   ├── Pages/
│   │   ├── Home.razor                 – Landing / plan selection page
│   │   ├── Cancel.razor               – PayPal cancellation page
│   │   └── Nav.razor                  – Navigation panel
│   └── Layout/
│       ├── MainLayout.razor
│       └── NavMenu.razor
└── appsettings.json
```

## Adding a New Subscription Tier

1. Update the Home page (`Components/Pages/Home.razor`) to add a new PayPal button/form with the correct amount and item name.
2. In `Events.OnPaymentCompleted`, map the `item_name` from the IPN payload to the corresponding storage size.
3. Update `ApiCommands.CreateSubscription()` call with the correct `storageSpaceGb` value.

## PayPal IPN Verification

The `PayPal.PayPalIpnMiddleware` automatically verifies IPN messages against PayPal's servers before calling `OnPaymentCompleted`. Never provision a cloud instance from an unverified IPN.

## Deployment

```bash
dotnet publish -c Release -r linux-x64 --self-contained -o /opt/cloudsubscription
sudo bash install.sh
sudo systemctl start cloudsubscription
```

Set `ASPNETCORE_URLS` to `http://*:80;https://*:443` in the systemd unit file or `appsettings.json`.
