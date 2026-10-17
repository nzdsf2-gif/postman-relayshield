# Postman + RelayShield

Threat intelligence screening for API developers, delivered as a Postman collection.

Every API developer using Postman tests endpoints. This collection adds the missing check: **is this endpoint, domain, or counterparty associated with known threats?** Screen before you integrate.

## What's inside

- **RelayShield Threat Screening** Postman collection (v2.1 format)
- Pre-request script for automatic screening in your own collections
- Environment template with API key variable

## Import the collection

1. Open Postman
2. Import → paste this repo's `relayshield-threat-screening.postman_collection.json`
3. Set the `relayshield_api_key` environment variable
4. Run any request

## Endpoints covered

| Request | RelayShield API | Use case |
|---|---|---|
| Screen Domain | `POST /v1/metered/domain` | Check a domain before integrating |
| Screen Email | `POST /v1/metered/breach` | Check for breach exposure |
| Screen URL | `POST /v1/metered/domain` | Extract domain, then screen |
| Verify Agent (TAP) | `POST /v1/tap/verify` | Merchant-side agent verification |

## Pre-request script

Drop this into any collection to screen the target domain automatically:

```javascript
// Pre-request Script: RelayShield threat screen
const targetUrl = pm.request.url.toString();
const domain = new URL(targetUrl).hostname;

pm.sendRequest({
    url: 'https://api.relayshield.net/v1/metered/domain',
    method: 'POST',
    header: {
        'Content-Type': 'application/json',
        'X-RS-API-KEY': pm.environment.get('relayshield_api_key')
    },
    body: { mode: 'raw', raw: JSON.stringify({ domain: domain }) }
}, (err, res) => {
    if (err) { console.error('RelayShield screen failed:', err); return; }
    const verdict = res.json().data?.verdict || 'unknown';
    console.log(`RelayShield: ${domain} → ${verdict}`);
    if (verdict === 'malicious') {
        console.warn('BLOCKED: Known threat. Review before proceeding.');
    }
});
```

## Why this matters

Postman is where API integrations start. Threat screening belongs at that starting point, not after you've already wired up a malicious endpoint. This collection makes it one click.

## License

Apache 2.0
