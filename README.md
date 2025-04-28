# knox
most secure messaging app - post-quantum cryptography implemented


## Kyber + X25519 Messaging App

1. Key Exchange and Session Setup

Component	Details
Classical Key Exchange	X25519 (Elliptic Curve Diffie-Hellman on Curve25519)
Post-Quantum Key Exchange	Kyber768 KEM
Hybrid Key Exchange Scheme	Perform both X25519 and Kyber768 key exchanges. Then combine shared secrets using a KDF (e.g., HKDF-SHAKE256).
Combined Shared Secret	`HKDF(SHAKE256, X25519_shared_secret
2. Key Generation and Storage

Component	Details
Identity Keys (long-term)	X25519 key pair + Kyber768 key pair
Ephemeral Keys (per session)	New X25519 ephemeral key pair + New Kyber768 ephemeral key pair for each session or ratchet
Key Storage	- Private keys stored encrypted at rest (e.g., with OS Secure Enclave / Hardware-backed Keystore)
- Public keys are distributed via a Directory Service (anonymous if possible)
3. Session Management

Component	Details
Initial Session Setup	- Initiator (Alice) uses Bob's identity public keys (X25519 + Kyber768)
- Performs hybrid key exchange
Ratcheting	- Double Ratchet based on symmetric keys derived from hybrid secret
- Symmetric key updated after every message (or periodically for group chats)
Key Update	- Ephemeral Kyber768 + X25519 keys for each session refresh (optional for additional security)
4. Encryption

Component	Details
Symmetric Cipher	AES-256-GCM for message payload encryption
Symmetric Key Derivation	HKDF-SHAKE256 over hybrid shared secret
Authenticated Encryption	Mandatory AEAD (authenticated encryption with associated data) using AES-256-GCM
Replay Protection	Message counters included in encrypted metadata, verified at receiver
5. Authentication and Signatures

Component	Details
Classical Signatures	Ed25519 signatures on public key bundles (identity verification)
Post-Quantum Signatures	Falcon-512 signatures on public key bundles (optional now, mandatory later)
Hybrid Authentication	Verify both Ed25519 and Falcon signatures on identity keys
6. Hashing and KDFs

Component	Details
Hash Function	SHAKE256 (extendable-output, quantum-resistant)
KDF	HKDF-SHAKE256
Usage	- Derive session keys from combined secrets
- Derive message keys from root keys
7. Network Protocol

Component	Details
Transport Layer	TLS 1.3, optional additional onion encryption layer
Message Delivery	Push notifications or direct TCP connections with retries
Metadata Protection	Use sealed sender approach + dummy traffic to obscure who is talking to whom
8. Forward Secrecy & Post-Quantum Forward Secrecy

Component	Details
Forward Secrecy	Standard Double Ratchet based forward secrecy
Post-Quantum Forward Secrecy	- Rotate hybrid keys often
- Delete shared secrets and ephemeral keys immediately after use
Compromise Recovery	New session initiates new hybrid key agreement


## Summary architecture 
Identity Setup:
  X25519_keypair + Kyber768_keypair (+ Falcon signature)

Session Initiation:
  Hybrid Key Exchange: X25519 + Kyber768
  -> HKDF(SHAKE256(X25519_secret || Kyber768_secret)) = Master Key

Messaging:
  Derive root keys and chain keys
  Symmetric encryption using AES-256-GCM
  Ratchet forward on every message
