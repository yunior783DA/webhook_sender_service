# Webhook Sender Service

A lightweight, reliable service that consumes internal events and forwards them to third-party webhook endpoints securely. 
Designed as part of an even-driven architecture  to decouple internal systems from external integrations.

---

## Features 
- **Consume internal event stream** (e.g. Redis Streams, message queues)
- **Filter and transform** events before forwarding
- **Secure delivery** with retry and failure handling
- Supports **HMAC signature verification** for webhook payload integrity
- **Multi-tenant aware** to route events to tenant-specific webhook URLs
- Configurable retry policies and error logging

## Webhook Security
To ensure secure communication and prevent replay attacks, each webhook request includes:
| Header                     | Description                                                                                                  |
|----------------------------| -------------------------------------------------------------------------------------------------------------|
|      X-WSS-Signature       | HMAC SHA256 signature computed over the request payload and time stampd using a shared secret key.           |
|      X-WSS-Timestamp       | Timestamp (ISO8601 or Unix epoch seconds) when the webhook was generated. Used to verify request freshness.  |

### How Signature is Generated

The signature is calculated by concatenating the timestamp and  the raw request body, then creating an HMAC SHA256 hash using the shared secret.
```python
import hmac
import hashlib

def generate_signature(secret: str, body: bytes, timestamp: str) -> str:
    message = timestampd.encode() + body
    signature - hmac.new(secret, message, hashlib.sha256).hexdigest()
    return signature
```

### Signature Verification on Receiver Side (or External system)
1. Extract X-WSS-Signature & X-WSS-Timestamp headers.
2. Verify the signature by recomputing it with the sahred secret and request payload.
3. Velidate that the timestamp is within an acceptable window (e.g. 5 minutes)
4. Reject the request if the signature is invalid or the timestampd is outside the allowed time window.

## Example Flow
1. Register external (third-party) webhook URL
2. Internal system publishes an event (e. order status update) to the internal Redis Stream.
3. Webhook Sender Service listens to the Redis Stream, consumes new events in real-time.
4. The service processes the event, filtering and transforming it as needed (e.g., JSON with order ID and status).
5. The service generates the current timestamp and computes the HMAC signature
6. The service sends a POST request to the registered webhook URL with the payload aind includes X-WSS-Signature and X-WSS-Timestamp headers.
7. if delivery fails, the service retries according to configured polices and exponential backoff
8. if delivery success, the receiver (external/third-party) verifies the signature and timestamp before processing the payload.
9. Logs delivery status and failures (if applicable)


## Contributing
Contributions welcome! Please open issues or submit pull requests


