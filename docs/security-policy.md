# CloudSubscription – Security Policy

## Payment Security

- All PayPal IPN messages are **verified** against PayPal's IPN verification endpoint before any action is taken. Unverified messages are silently discarded.
- Cloud provisioning only occurs after a confirmed, successful payment status (`payment_status=Completed`).

## API Request Signing

Calls to `CloudServerUISupport.ApiCommands` are signed with `ApiPrivateKey`. The receiving server validates the signature with the corresponding public key. The private key must never be committed to source control.

## Sensitive Configuration

The following settings are sensitive and must be protected:

| Key | Risk if exposed |
|---|---|
| `ApiPrivateKey` | Attacker could provision unlimited cloud instances |
| `PayPalBusinessEmail` | Lower risk, but could be used for phishing |

Use environment variables or a secrets manager (e.g., Azure Key Vault, HashiCorp Vault, or `dotnet user-secrets` for development) instead of plain `appsettings.json` in production.

## HTTPS

Always deploy behind HTTPS. The application enforces HTTPS redirection (`app.UseHttpsRedirection()`). Use HSTS in production.

## Reporting Vulnerabilities

Open a private GitHub Security Advisory in the repository. Do not disclose vulnerabilities publicly before a fix is available.
