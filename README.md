# zk-HumanProof Protocol Specification (Draft)

## Overview

The zk-HumanProof Protocol is a privacy-preserving standard for web request verification. It enables websites to cryptographically verify that incoming requests originate from legitimate, human users—without collecting personal data—through zero-knowledge proofs (ZKPs).

This protocol is intended to mitigate abusive behaviors such as automated scraping, spam, and bandwidth exhaustion by requiring proof of humanness or effort without revealing identity or tracking users.

## Objectives

- Allow websites to verify a user is human or has expended effort (e.g., proof-of-work).
- Prevent abuse from bots and automated crawlers.
- Preserve user privacy through zero-knowledge proofs.
- Be lightweight and browser-compatible.
- Be extensible to multiple types of verification mechanisms.

## Threat Model

- Large-scale scraping by AI crawlers.
- Botnets spamming endpoints or harvesting public data.
- Abuse of public APIs leading to high bandwidth charges.
- Malicious actors bypassing traditional CAPTCHAs or rate limits.

## Proof Types

### 1. zk-CAPTCHA
- User solves a CAPTCHA challenge in the browser.
- A zk-SNARK or zk-STARK is generated as proof of completion.
- The proof does not reveal CAPTCHA content or user identity.

### 2. zk-Proof-of-Work (PoW)
- The browser performs a lightweight computational task.
- A ZK proof is generated showing valid solution to the task.
- This limits scalability of automated abuse.

### 3. zk-Credential
- A user is issued an anonymous credential after passing a one-time verification.
- Credential can be reused across multiple domains without leaking identity.
- Based on constructs like Semaphore or MACI.

## Protocol Flow

### Client-side
1. Request for verification is made by the server (challenge issued).
2. User completes the verification task (CAPTCHA, PoW, etc.).
3. Browser generates a zero-knowledge proof.
4. Proof is attached to subsequent request headers (e.g., `X-Human-Proof`).

### Server-side
1. Server receives request with proof header.
2. Server validates the proof using the appropriate verifier circuit.
3. If valid, request proceeds. If invalid or missing, request is rejected.

## API Specification (Example)

### Client Request Header
```js
POST /api/resource HTTP/1.1
X-Human-Proof: <base64-encoded zk-SNARK proof>
Content-Type: application/json

{"query": "..."}
```

### Server Verification (Pseudocode)
```js
app.post('/api/data', async (req, res) => {
  const proof = req.headers['x-human-proof'];
  const valid = await verifyZKProof(proof);
  if (!valid) return res.status(403).send('Bot blocked');
  res.send(fetchRequestedData());
});
```

## Proof Lifespan & Replay Protection
- Proofs must include a nonce or timestamp to prevent reuse.
- Optional expiration windows (e.g., valid for 10 minutes).
- Servers may cache seen proofs to prevent replay attacks.

## Privacy Considerations
- No user-identifiable information is stored or transmitted.
- zk-SNARKs/zk-STARKs guarantee verifiability without leakage.
- Anonymous credentials may be revoked via nullifiers.

## Browser Integration Goals

This protocol is designed with the long-term goal of **native browser integration**:

- No extensions should be required for normal use.
- Browsers implement the core proof-generation logic as part of the browser engine.
- Standardized APIs allow websites to interact with the proof system in a secure and consistent way.

## Proposed Navigator API

A browser-native interface for requesting human verification:

```js
const proof = await navigator.zkProof.generate({
  challenge: "abcdefg12345",
  type: "zkCaptcha"
});
```

- `challenge`: A server-issued challenge string.
- `type`: The type of proof required (`zkCaptcha`, `zkPoW`, `zkCredential`).
- Returns: A base64-encoded zero-knowledge proof.

## Long-Term Vision

- Integration with browser engines (Chromium, Firefox, WebKit).
- Standardization through W3C Privacy CG or Web Platform Incubator.
- Coordination with privacy-preserving trust models (Privacy Pass, WebAuthn).
- Enable caching and trust delegation across trusted sites without re-verification.

## Extensions & Future Work
- Support for WebAssembly-based provers for browser-native ZK generation.
- Integration with DID (Decentralized Identity) frameworks.
- Delegation to trusted privacy-preserving CAPTCHA providers.
- **Crawl Policy Extensions**: Introduce a new standard file, `crawler.txt`, as a modern, machine-readable complement or replacement for `robots.txt`. This file, written in YAML or JSON, could specify:
  - Allowed/disallowed paths for crawler access.
  - Request frequency or rate limits per endpoint.
  - Accepted verification mechanisms (e.g., ZK proof types).
  - Contact and usage policy metadata.

### crawler.txt (Example in YAML)
```yaml
allow:
  - /docs/
  - /blog/
disallow:
  - /private/
rate_limit:
  requests_per_minute: 10
  burst_limit: 5
zk_proof_required: true
contact: admin@example.com
```

## License
This specification is released under the MIT License.

---

*This is a community draft. Feedback, discussion, and pull requests are welcome.*
