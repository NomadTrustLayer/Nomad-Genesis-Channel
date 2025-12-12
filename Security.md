# Security Policy

## Reporting a Vulnerability

**Nomad Trust Layer takes security seriously.** We appreciate responsible disclosure of security vulnerabilities.

### How to Report

**DO NOT create a public GitHub issue for security vulnerabilities.**

Instead, please email: **security@nomadtrust.com**

Include:

* **Description** - Clear explanation of the vulnerability
* **Impact** - Potential consequences if exploited
* **Steps to reproduce** - Detailed reproduction steps
* **Affected components** - Which parts of the system are vulnerable
* **Suggested fix** - If you have recommendations
* **Your contact info** - For follow-up questions

### What to Expect

* **Acknowledgment** - Within 48 hours
* **Initial assessment** - Within 5 business days
* **Regular updates** - Throughout investigation
* **Resolution timeline** - Depends on severity and complexity
* **Credit** - Public recognition (if desired) after fix is deployed

### Scope

**In scope:**

* Zero-knowledge proof circuits (Circom)
* Smart contracts (Solidity)
* Nomad Channel routing logic
* Fee distribution mechanisms
* Frontend proof generation libraries
* API endpoints (once public)

**Out of scope:**

* Social engineering attacks
* Physical security
* Third-party dependencies (report to them directly)
* Already-disclosed vulnerabilities
* Denial of service attacks

### Severity Levels

**Critical** - Immediate threat to user funds or privacy
* Response: < 24 hours
* Fix target: < 7 days

**High** - Significant security impact
* Response: < 48 hours  
* Fix target: < 14 days

**Medium** - Moderate security impact
* Response: < 5 days
* Fix target: < 30 days

**Low** - Minor security impact
* Response: < 7 days
* Fix target: Next release cycle

## Security Measures

### Circuit Security

* Built on audited **Semaphore Protocol**
* Simplified circuit reduces attack surface
* Independent security audits scheduled (Q1 2026)
* Formal verification of critical constraints
* Comprehensive test coverage

### Smart Contract Security

* Solidity best practices
* Multi-sig admin controls (3-of-5)
* Time-locked upgrades (48-hour delay)
* Emergency pause mechanism
* Automated testing and continuous integration
* Formal verification of critical functions
* Professional security audits (Q1 2026)

### Key Management

* Client-side key generation only
* Keys never transmitted to servers
* Encrypted browser storage
* User-controlled backups
* No custodial access

### Infrastructure Security

* DDoS protection
* Rate limiting
* Input validation and sanitization
* Secure API authentication
* Encrypted data transmission (TLS 1.3)
* Regular security patches

### Financial Security

* Non-custodial architecture
* Direct payments to node holder wallets
* No ability to seize or redirect earnings
* Transparent on-chain accounting
* Immutable ownership records

## Audit History

### Planned Audits (Q1 2026)

**Circuit Audit:**
* Auditor: Trail of Bits, Veridise, or equivalent
* Scope: Custom Semaphore circuits, constraint analysis
* Timeline: Q1 2026

**Smart Contract Audit:**
* Auditor: OpenZeppelin, Consensys Diligence, or equivalent
* Scope: Nomad Channel contracts, fee distribution, routing logic
* Timeline: Q1 2026

**Penetration Testing:**
* Scope: Full platform security assessment
* Timeline: Q1 2026

### Bug Bounty Program

**Coming Q2 2026:**

* $50,000-$100,000 total pool
* Rewards based on severity
* Public recognition for responsible researchers
* Details to be announced

## Security Best Practices

### For Node Holders

* **Secure your wallet** - Use hardware wallet for large holdings
* **Backup your keys** - Store recovery phrases securely
* **Verify contracts** - Always verify contract addresses before interacting
* **Enable 2FA** - On all associated accounts (email, exchanges, etc.)
* **Be wary of phishing** - We will never DM you asking for keys or credentials

### For Developers

* **Review code carefully** - Before submitting pull requests
* **Test thoroughly** - Include security-focused test cases
* **Follow standards** - Adhere to established security patterns
* **Document security considerations** - Explain security-relevant design decisions
* **Use secure dependencies** - Keep dependencies updated

### For Users

* **Verify authenticity** - Always access via official links
* **Protect your identity_secret** - Never share with anyone
* **Use secure devices** - Avoid public computers for sensitive operations
* **Keep software updated** - Use latest browser and wallet versions

## Disclosure Policy

Once a vulnerability is fixed:

* **Coordinated disclosure** - Public announcement after patch is deployed
* **Security advisory** - Published on GitHub and website
* **Credit to reporter** - Public recognition (if desired)
* **Vulnerability details** - Shared transparently with community

## Security Updates

Subscribe to security updates:

* **GitHub Security Advisories** - Watch this repository
* **Email notifications** - security@nomadtrust.com (announce-only list coming soon)
* **Twitter** - [@nomadtrustlayer](https://twitter.com/nomadtrustlayer)
* **Discord** - #security-announcements channel (coming soon)

## Contact

* **Security issues:** security@nomadtrust.com
* **General inquiries:** team@nomadtrust.com
* **Website:** [nomadtrust.com](https://nomadtrust.com)

---

**Thank you for helping keep Nomad Trust Layer secure!**

*Built by [0L1 Labs](https://0l1labs.com)*
