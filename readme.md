# TACCV4 — PAKE‑Secured Pairing over an Untrusted Relay

**TACCV4** is a *reference implementation* of a human‑verifiable secure pairing protocol built on SPAKE2, X25519, and ChaCha20‑Poly1305, designed to operate safely over an untrusted WebSocket relay. It prioritizes protocol clarity, explicit state machines, downgrade resistance, transcript binding, and auditability.

> ⚠️ **WARNING**: This code is **not production‑ready cryptography**. It is intended for study, experimentation, and protocol review.

---

## Design Goals

* Human‑verifiable pairing (PIN or shared secret)
* Secure operation over an untrusted relay
* Explicit protocol phases and transcript binding
* Downgrade‑resistant profile negotiation
* Deterministic message handling and replay resistance
* Clear failure modes and abort semantics

This project is opinionated: safety > convenience, clarity > cleverness.

---

## Architecture Overview

**Participants**

* *Alice* and *Bob* clients
* Untrusted relay server (FastAPI + WebSockets)

**Protocol Flow**

1. Session creation (relay)
2. Profile negotiation (downgrade‑protected)
3. PAKE (SPAKE2) authentication
4. Handshake (X25519 + transcript binding)
5. Key confirmation
6. Encrypted channel (ChaCha20‑Poly1305)

**Security Properties**

* Password never leaves the endpoint
* Relay cannot impersonate either party
* Transcript hash binds all critical steps
* AEAD AAD binds sender, role, transcript, and sequence

---

## Requirements

### Python

* Python **3.9+** (required)

### Runtime Dependencies

Install via pip:

```bash
pip install websockets fastapi uvicorn[standard] spake2 cryptography structlog
```

### Optional (for relay)

* `uvicorn[standard]` strongly recommended for production‑like testing

---

## Usage

### 1. Run Security Self‑Check

```bash
python taccv4.py check
```

### 2. Start the Relay Server

```bash
python taccv4.py relay --host 127.0.0.1 --port 8000
```

### 3. Create a Session

```http
POST /session
```

Returns a session code and session ID.

### 4. Run Clients

```bash
python taccv4.py client --url ws://localhost:8000 \
  --session ABCD1234 --role alice --pin 123456

python taccv4.py client --url ws://localhost:8000 \
  --session ABCD1234 --role bob --pin 123456
```

---

## What This Code Does Well

* Explicit, readable state machine
* Strong transcript discipline
* Downgrade‑resistant profile negotiation
* Strict base64 and JSON validation
* Human‑verifiable SAS
* Clean separation between relay and crypto logic
* Clear abort semantics and error codes

This is *inspectable crypto*, not magic.

---

## Known Faults, Limitations, and Non‑Goals

This section is intentional. These are **not bugs hidden under the rug** — they are acknowledged design limits.

### 🚫 Not Production‑Ready

* No formal cryptographic proof
* No constant‑time guarantees
* No side‑channel resistance beyond best‑effort

### ⏱ Timing & Side‑Channels

* SPAKE2 failure handling is *not* constant‑time
* Artificial delays reduce but do not eliminate timing leaks

### 🔐 Key Handling

* Keys live in Python memory (no secure enclave / zeroization guarantees)
* Garbage collection timing is nondeterministic

### 🌐 Relay Limitations

* Relay is not hardened for Internet exposure
* In‑memory session storage only
* DoS protections are minimal and heuristic

### 📡 Network Model

* Assumes reliable WebSocket delivery
* No packet loss recovery beyond monotonic sequencing
* No multi‑device or reconnect resume support

### 🔄 Protocol Scope

* No forward secrecy *across sessions* if secrets are reused
* No identity binding beyond shared secret
* No certificate or PKI support by design

### 🧪 Implementation Quality

* Python is not ideal for high‑assurance crypto
* Error handling favors explicit aborts over recovery
* Logging may leak metadata if misconfigured

If you want *battle‑hardened crypto*, use an audited library. If you want to **understand** crypto protocols, this is for you.

---

## Threat Model (Explicit)

**Protected Against**

* Passive network observers
* Malicious relay
* Replay attacks
* Downgrade attacks

**Not Protected Against**

* Compromised endpoints
* Side‑channel attacks
* Malicious OS or Python runtime
* Nation‑state adversaries

---

## License

MIT License

```
MIT License

Copyright (c) 2025

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all
copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
SOFTWARE.
```

---

## Final Note

This project is a *teaching artifact*. It is designed to be read, critiqued, and reasoned about. If you find flaws — good. That means it’s doing its job.
