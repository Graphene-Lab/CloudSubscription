# CloudSubscription – Architecture Decision Records

## ADR-001 – Blazor Server for the Subscription Portal

**Status:** Accepted

**Context:** The subscription portal needs real-time state updates (payment confirmation, provisioning progress) and server-side access to the PayPal IPN middleware.

**Decision:** Use Blazor Server. The SignalR connection enables real-time UI updates without polling. Server-side execution gives full access to the IPN pipeline.

**Consequences:** Requires a persistent SignalR connection per browser tab. Acceptable for a low-traffic subscription portal.

---

## ADR-002 – PayPal IPN over PayPal REST API

**Status:** Accepted

**Context:** PayPal offers both IPN (legacy webhook) and the modern REST Webhooks API for payment notifications.

**Decision:** Use IPN. It is simpler to integrate, does not require a PayPal developer app or OAuth tokens, and is well-suited for the current use case (subscription confirmation).

**Consequences:** IPN is a legacy system; PayPal has stated it will remain supported for the foreseeable future but recommends the REST API for new integrations. A future migration to REST Webhooks is possible without changing the business logic — only the middleware changes.

---

## ADR-003 – Signed API Calls for Cloud Provisioning

**Status:** Accepted

**Context:** The subscription portal must call the CloudServerUISupport API to create a new cloud instance. This API must be protected against unauthorised callers.

**Decision:** All API calls from CloudSubscription are signed with `ApiPrivateKey`. The CloudServer verifies the signature with the corresponding public key (`ApiPublicKey`).

**Consequences:** The private key must be kept secret. If compromised, an attacker could provision unlimited cloud instances. Key rotation requires updating both the subscription portal config and the cloud server config simultaneously.

---

## ADR-004 – Email QR Code Delivery for Pairing

**Status:** Accepted

**Context:** After provisioning, the customer must pair their device with the new cloud instance. The pairing QR code must be delivered securely.

**Decision:** `ApiCommands.SendQrByEmail()` sends the QR code as an image in an email to the customer's email address (captured from the PayPal IPN `payer_email` field).

**Consequences:** Delivery depends on email deliverability. The QR code is visible to anyone with access to the customer's email. This is acceptable as the QR code only establishes the pairing, not authentication — the cloud uses its own key exchange after pairing.
