# CloudSubscription – System Overview

## Purpose

CloudSubscription is a **Blazor Server web application** (.NET 9) that handles the commercial subscription workflow for CloudBox users. It integrates with **PayPal IPN** (Instant Payment Notification) to automatically provision new cloud instances upon successful payment.

## Architecture

```
Customer Browser
	  │
	  ▼ HTTPS
CloudSubscription (Blazor Server)
	  │
	  ├─ PayPal IPN Middleware  ──→  Events.OnPaymentCompleted
	  │                                    │
	  │                                    ▼
	  │                          CloudServerUISupport.ApiCommands
	  │                               .CreateSubscription()
	  │                                    │
	  │                                    ▼
	  │                          CloudServer (remote, via API)
	  │
	  └─ Pages
		   ├─ Home    – Subscription plans and payment buttons
		   ├─ Nav     – Navigation
		   └─ Cancel  – Payment cancellation landing page
```

## Subscription Flow

1. User visits the Home page and selects a plan.
2. User is redirected to PayPal.
3. PayPal sends an IPN POST to `/ipn` (handled by `PayPal.PayPalIpnMiddleware`).
4. `Events.OnPaymentCompleted` is invoked with payment details.
5. `ApiCommands.CreateSubscription()` provisions a new cloud instance on the CloudServer.
6. A QR code is sent to the customer's email for device pairing.

## Configuration (`appsettings.json`)

| Key | Description |
|---|---|
| `PayPalBusinessEmail` | PayPal merchant account email |
| `ApiEndpoint` | URL of the CloudServerUISupport API endpoint |
| `ApiPrivateKey` | Private key to sign API requests |

## Technology Stack

- ASP.NET Core 9 + Blazor Server
- Bootstrap 5
- `PayPal` library (in-solution NuGet)
- `UISupportBlazor` / `UISupportGeneric`

## Dependencies

- `PayPal` (in-solution project)
- `CloudServerUISupport` (API commands)
- `UISupportBlazor`
- `SystemExtra`
