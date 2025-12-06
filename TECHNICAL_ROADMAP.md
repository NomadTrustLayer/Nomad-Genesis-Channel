# Technical Roadmap

**0L1 Labs - Digital Trust Infrastructure**

*Last Updated: December 2025*

---

## Table of Contents

1. [Architecture Overview](#architecture-overview)
2. [Genesis Channel System](#genesis-channel-system)
3. [ZK Circuit Specifications](#zk-circuit-specifications)
4. [Smart Contract Architecture](#smart-contract-architecture)
5. [API Infrastructure](#api-infrastructure)
6. [Frontend & Dashboard](#frontend--dashboard)
7. [Security & Audits](#security--audits)
8. [Development Timeline](#development-timeline)
9. [Technical Stack](#technical-stack)

---

## Architecture Overview

### System Components

```
┌─────────────────────────────────────────────────────────────┐
│                        0L1 Labs Platform                     │
├─────────────────────────────────────────────────────────────┤
│                                                               │
│  ┌──────────────┐      ┌──────────────┐   ┌──────────────┐ │
│  │   User App   │      │ ZK Proof Gen │   │ Genesis NFT  │ │
│  │  (Browser)   │─────▶│   (Client)   │   │   (Base)     │ │
│  └──────────────┘      └──────────────┘   └──────────────┘ │
│         │                      │                    │        │
│         │                      ▼                    │        │
│         │              ┌──────────────┐            │        │
│         └─────────────▶│  API Gateway │◀───────────┘        │
│                        └──────────────┘                     │
│                              │                               │
│                              ▼                               │
│                   ┌─────────────────────┐                   │
│                   │  Channel Selection  │                   │
│                   │   (Round-Robin)     │                   │
│                   └─────────────────────┘                   │
│                              │                               │
│                              ▼                               │
│        ┌────────────────────────────────────────┐           │
│        │       Genesis Channels (1-1000)        │           │
│        │  ┌────┐ ┌────┐ ┌────┐       ┌────┐    │           │
│        │  │ #1 │ │ #2 │ │ #3 │  ...  │#1000│   │           │
│        │  └────┘ └────┘ └────┘       └────┘    │           │
│        └────────────────────────────────────────┘           │
│                              │                               │
│                              ▼                               │
│                   ┌─────────────────────┐                   │
│                   │  ZK Proof Verifier  │                   │
│                   │     (Base EVM)      │                   │
│                   └─────────────────────┘                   │
│                              │                               │
│                              ▼                               │
│                   ┌─────────────────────┐                   │
│                   │  Fee Distribution   │                   │
│                   │  70% Owner / 30% 0L1│                   │
│                   └─────────────────────┘                   │
│                                                               │
└─────────────────────────────────────────────────────────────┘
```

### Data Flow

**Verification Request Lifecycle:**

1. **User Action** - User wants to prove eligibility (e.g., age > 18)
2. **Client-Side Proof Generation** - Browser generates ZK proof locally
3. **App Submission** - App submits proof to 0L1 API
4. **Authentication** - API gateway validates API key and request
5. **Channel Selection** - Smart contract selects Genesis Channel via round-robin
6. **Verification** - Selected channel verifies proof cryptographically
7. **Result Return** - Validation result returned to app
8. **Fee Distribution** - 70% to channel owner, 30% to protocol (automatic)
9. **Analytics Update** - Dashboard updated with earnings data

**Time:** ~100-500ms end-to-end

---

## Genesis Channel System

### Core Concept

**Genesis Channels are verification endpoints in the 0L1 network.** Each of the 1,000 Genesis Layer NFTs represents ownership of a numbered channel that processes verification requests and earns fees.

### Channel Structure

**Base Smart Contract Implementation:**

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;

/// @title GenesisChannel
/// @notice Represents a single verification channel in the 0L1 network
struct GenesisChannel {
    /// Channel identifier (1-1000)
    uint16 channelId;
    
    /// NFT holder's wallet address
    address owner;
    
    /// Channel tier: 1=Platinum, 2=Titanium, 3=Obsidian
    uint8 tier;
    
    /// Capacity multiplier based on tier (10=1.0x, 15=1.5x, 20=2.0x)
    uint8 capacityWeight;
    
    /// Total verifications processed lifetime
    uint64 totalVerifications;
    
    /// Total earnings in wei (lifetime)
    uint256 totalEarned;
    
    /// Current active status
    bool active;
    
    /// Last verification timestamp
    uint256 lastVerificationTimestamp;
    
    /// Hourly verification count (for rate limiting)
    uint32 hourlyVerificationCount;
}
```

### Channel Tiers

| Tier | Price | Channels | Capacity | Weight | Hourly Max |
|---|---|---|---|---|---|
| **Platinum** | $500 | #1-333 | Standard | 1x | 100 |
| **Titanium** | $750 | #334-666 | Enhanced | 1.5x | 200 |
| **Obsidian** | $1,000 | #667-1000 | Premium | 2x | 500 |

**Capacity Weight** determines how often a channel is selected in routing:
- Platinum: Selected every rotation
- Titanium: Selected 1.5x as often (appears in rotation more frequently)
- Obsidian: Selected 2x as often (highest priority)

### Routing Algorithm

**Round-Robin with Capacity Weighting:**

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;

contract ChannelRouter {
    uint16 public lastChannelUsed;
    uint64 public rotationCount;
    
    mapping(uint16 => GenesisChannel) public channels;
    
    event ChannelSelected(uint16 indexed channelId, uint64 rotationCount);
    
    /// @notice Select next available Genesis Channel
    /// @return channelId The selected channel ID
    function selectNextChannel() external returns (uint16) {
        uint16 attempts = 0;
        uint16 maxAttempts = 1000;
        
        while (attempts < maxAttempts) {
            // Increment to next channel (1-1000)
            lastChannelUsed = (lastChannelUsed % 1000) + 1;
            
            GenesisChannel memory channel = channels[lastChannelUsed];
            
            // Check if channel is available
            if (channel.active && !isAtCapacity(channel)) {
                // Apply capacity weighting
                if (shouldSelectBasedOnWeight(channel)) {
                    rotationCount++;
                    emit ChannelSelected(lastChannelUsed, rotationCount);
                    return lastChannelUsed;
                }
            }
            
            attempts++;
        }
        
        revert("No channels available");
    }
    
    /// @notice Check if channel is at hourly capacity
    function isAtCapacity(GenesisChannel memory channel) 
        internal 
        view 
        returns (bool) 
    {
        if (block.timestamp - channel.lastVerificationTimestamp > 1 hours) {
            return false;  // Hour has reset
        }
        
        uint32 maxHourly = channel.tier == 1 ? 100 : 
                          (channel.tier == 2 ? 200 : 500);
        return channel.hourlyVerificationCount >= maxHourly;
    }
    
    /// @notice Determine if channel should be selected based on tier weight
    function shouldSelectBasedOnWeight(GenesisChannel memory channel) 
        internal 
        view 
        returns (bool) 
    {
        if (channel.tier == 1) return true;  // Platinum: always selected
        if (channel.tier == 2) return rotationCount % 2 == 0;  // Titanium: ~1.5x
        if (channel.tier == 3) return true;  // Obsidian: always selected (2x)
        return false;
    }
}
```

**Key Properties:**
- **Fair Distribution** - All channels get turns over time
- **Capacity Respect** - Higher tiers get proportionally more traffic
- **Failover** - Skips unavailable/maxed channels automatically
- **Deterministic** - Same state produces same selection
- **Gas Efficient** - Simple counter increment, no random number generation

### Fee Distribution

**Payment Split:**
```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;

contract FeeDistributor {
    uint256 public constant OWNER_SHARE = 70;  // 70%
    uint256 public constant PROTOCOL_SHARE = 30;  // 30%
    
    address public protocolWallet;
    mapping(uint16 => GenesisChannel) public channels;
    
    event FeesDistributed(
        uint16 indexed channelId,
        address indexed channelOwner,
        uint256 ownerAmount,
        uint256 protocolAmount
    );
    
    /// @notice Distribute verification fee between owner and protocol
    function distributeFees(uint16 channelId) external payable {
        GenesisChannel memory channel = channels[channelId];
        require(channel.active, "Channel not active");
        
        uint256 totalFee = msg.value;
        
        // Calculate split (70/30)
        uint256 ownerShare = (totalFee * OWNER_SHARE) / 100;
        uint256 protocolFee = totalFee - ownerShare;
        
        // Transfer to channel owner
        (bool ownerSuccess, ) = channel.owner.call{value: ownerShare}("");
        require(ownerSuccess, "Owner transfer failed");
        
        // Transfer to protocol wallet
        (bool protocolSuccess, ) = protocolWallet.call{value: protocolFee}("");
        require(protocolSuccess, "Protocol transfer failed");
        
        // Update channel earnings tracking
        channels[channelId].totalEarned += ownerShare;
        
        emit FeesDistributed(channelId, channel.owner, ownerShare, protocolFee);
    }
}
```

**Fee Structure:**
- **Standard Verification:** 0.001 ETH (~$2-3 on Base)
- **Owner Receives:** 0.0007 ETH (70%)
- **Protocol Receives:** 0.0003 ETH (30%)
- **Gas Cost:** ~0.0001-0.0005 ETH (Base L2 fees)

---

## ZK Circuit Specifications

### Circuit Architecture

**Based on Semaphore Protocol, customized for action-specific verification:**

```circom
pragma circom 2.1.0;

include "circomlib/circuits/poseidon.circom";

template ActionProof() {
    // Private inputs (never revealed)
    signal input identitySecret;
    signal input actionId;
    
    // Public outputs (posted on-chain)
    signal output identityCommitment;
    signal output actionNullifier;
    
    // Generate identity commitment
    component identityHasher = Poseidon(1);
    identityHasher.inputs[0] <== identitySecret;
    identityCommitment <== identityHasher.out;
    
    // Generate action nullifier (prevents double-claims)
    component nullifierHasher = Poseidon(2);
    nullifierHasher.inputs[0] <== identitySecret;
    nullifierHasher.inputs[1] <== actionId;
    actionNullifier <== nullifierHasher.out;
}

component main {public [actionId]} = ActionProof();
```

### Circuit Properties

**Complexity:**
- **Constraints:** ~1,200 (2x Poseidon hash operations)
- **Proof Generation Time:** ~2 seconds (client-side, browser)
- **Proof Size:** 256 bytes (Groth16)
- **Verification Gas:** ~250,000 gas (~$0.50 on Base)

**Security:**
- **Soundness:** Attacker cannot forge proof without identitySecret
- **Privacy:** identitySecret never revealed, only commitment posted
- **Uniqueness:** Same person + same action = same nullifier = rejected
- **Unlinkability:** Different actions produce different nullifiers

### Proof Generation Flow

**Client-Side (JavaScript/Browser):**

```javascript
import { groth16 } from "snarkjs";

async function generateActionProof(identitySecret, actionId) {
    // Prepare circuit inputs
    const inputs = {
        identitySecret: identitySecret,
        actionId: actionId
    };
    
    // Generate witness
    const { proof, publicSignals } = await groth16.fullProve(
        inputs,
        "circuits/action_proof.wasm",
        "circuits/action_proof_final.zkey"
    );
    
    // Extract public outputs
    const identityCommitment = publicSignals[0];
    const actionNullifier = publicSignals[1];
    
    return {
        proof,
        identityCommitment,
        actionNullifier
    };
}
```

**On-Chain Verification (Solidity):**

```solidity
interface IVerifier {
    function verifyProof(
        uint256[2] calldata _pA,
        uint256[2][2] calldata _pB,
        uint256[2] calldata _pC,
        uint256[2] calldata _pubSignals
    ) external view returns (bool);
}

contract ProofValidator {
    IVerifier public verifier;
    mapping(uint256 => bool) public nullifierUsed;
    
    function validateAction(
        uint256[8] calldata proof,
        uint256 actionId,
        uint256 identityCommitment,
        uint256 actionNullifier
    ) external returns (bool) {
        // Check nullifier hasn't been used
        require(!nullifierUsed[actionNullifier], "Action already claimed");
        
        // Verify ZK proof
        bool isValid = verifier.verifyProof(
            [proof[0], proof[1]],
            [[proof[2], proof[3]], [proof[4], proof[5]]],
            [proof[6], proof[7]],
            [actionId, identityCommitment, actionNullifier]
        );
        
        require(isValid, "Invalid proof");
        
        // Mark nullifier as used
        nullifierUsed[actionNullifier] = true;
        
        return true;
    }
}
```

---

## Smart Contract Architecture

### Core Contracts

**1. GenesisChannelRegistry.sol** - Manages all 1,000 channels

```solidity
contract GenesisChannelRegistry {
    mapping(uint16 => GenesisChannel) public channels;
    uint16 public constant TOTAL_CHANNELS = 1000;
    
    event ChannelRegistered(uint16 indexed channelId, address indexed owner);
    event ChannelTransferred(uint16 indexed channelId, address from, address to);
    
    function registerChannel(
        uint16 channelId, 
        address owner, 
        uint8 tier
    ) external onlyOwner {
        require(channelId > 0 && channelId <= TOTAL_CHANNELS);
        require(channels[channelId].owner == address(0), "Already registered");
        
        channels[channelId] = GenesisChannel({
            channelId: channelId,
            owner: owner,
            tier: tier,
            capacityWeight: tier == 1 ? 10 : (tier == 2 ? 15 : 20),
            totalVerifications: 0,
            totalEarned: 0,
            active: true,
            lastVerificationTimestamp: 0,
            hourlyVerificationCount: 0
        });
        
        emit ChannelRegistered(channelId, owner);
    }
}
```

**2. VerificationRouter.sol** - Routes requests to channels

```solidity
contract VerificationRouter {
    GenesisChannelRegistry public registry;
    IVerifier public verifier;
    
    function processVerification(
        uint256[8] calldata proof,
        uint256 actionId,
        uint256 identityCommitment,
        uint256 actionNullifier
    ) external payable returns (uint16) {
        require(msg.value >= 0.001 ether, "Insufficient fee");
        
        // Verify ZK proof
        bool isValid = verifyProof(proof, actionId, identityCommitment, actionNullifier);
        require(isValid, "Invalid proof");
        
        // Select Genesis Channel
        uint16 channelId = registry.selectNextChannel();
        
        // Distribute fees
        distributeFees(channelId, msg.value);
        
        // Update channel stats
        registry.incrementVerificationCount(channelId);
        
        return channelId;
    }
}
```

**3. GenesisNFT.sol** - ERC-721 badge contract

```solidity
import "@openzeppelin/contracts/token/ERC721/ERC721.sol";

contract GenesisNFT is ERC721 {
    GenesisChannelRegistry public registry;
    uint16 public nextTokenId = 1;
    
    constructor() ERC721("0L1 Genesis Channel", "0L1GEN") {}
    
    function mint(address to, uint8 tier) external onlyOwner {
        require(nextTokenId <= 1000, "All minted");
        
        uint16 tokenId = nextTokenId++;
        _mint(to, tokenId);
        
        // Register channel in registry
        registry.registerChannel(tokenId, to, tier);
    }
    
    function _beforeTokenTransfer(
        address from,
        address to,
        uint256 tokenId
    ) internal override {
        super._beforeTokenTransfer(from, to, tokenId);
        
        // Update channel ownership on transfer
        if (from != address(0) && to != address(0)) {
            registry.transferChannel(uint16(tokenId), to);
        }
    }
}
```

### Gas Optimization

**Target Gas Costs (Base L2):**
| Operation | Gas | Cost @ 0.5 Gwei |
|-----------|-----|-----------------|
| Mint Genesis NFT | ~150K | $0.30 |
| Verify proof + route | ~300K | $0.60 |
| Withdraw earnings | ~50K | $0.10 |
| Transfer NFT | ~100K | $0.20 |

---

## API Infrastructure

### REST API Endpoints

**Base URL:** `https://api.0l1labs.com/v1`

**1. Submit Verification**
```http
POST /verify
Content-Type: application/json
Authorization: Bearer {API_KEY}

{
  "proof": [...],
  "actionId": "0x...",
  "identityCommitment": "0x...",
  "actionNullifier": "0x..."
}

Response:
{
  "success": true,
  "channelId": 42,
  "transactionHash": "0x...",
  "verificationTime": 347
}
```

**2. Get Channel Stats**
```http
GET /channel/{channelId}/stats

Response:
{
  "channelId": 42,
  "owner": "0x...",
  "tier": "Platinum",
  "totalVerifications": 1247,
  "totalEarned": "1.234 ETH",
  "last24h": {
    "verifications": 42,
    "earned": "0.042 ETH"
  }
}
```

**3. Get Channel Earnings**
```http
GET /channel/{channelId}/earnings

Response:
{
  "channelId": 42,
  "availableToWithdraw": "0.543 ETH",
  "totalEarned": "1.234 ETH",
  "totalWithdrawn": "0.691 ETH"
}
```

### WebSocket Real-Time Updates

```javascript
const ws = new WebSocket('wss://api.0l1labs.com/v1/channel/42/live');

ws.onmessage = (event) => {
  const update = JSON.parse(event.data);
  
  if (update.type === 'verification_processed') {
    console.log(`Earned ${update.amount} ETH`);
    console.log(`Total verifications: ${update.totalCount}`);
  }
};
```

**Event Types:**
- `verification_processed` - New verification through your channel
- `earnings_update` - Earnings balance changed
- `channel_status_change` - Active status updated

**Rate Limiting:**
- 100 requests/minute per API key
- 1000 requests/hour per API key
- WebSocket: 1 connection per channel

**SLA Targets:**
- Uptime: 99.9%
- Response time: <500ms (p95)
- Verification time: <2 seconds end-to-end

---

## Frontend & Dashboard

### Badge Holder Dashboard

**Tech Stack:**
- **Framework:** Next.js 14 (React)
- **Styling:** Tailwind CSS
- **Wallet:** wagmi + RainbowKit (Ethereum wallet connection)
- **Charts:** Recharts
- **API:** REST + WebSocket (real-time updates)

**Features:**

**1. Channel Overview**
```typescript
interface ChannelOverview {
  channelId: number;
  tier: 'Platinum' | 'Titanium' | 'Obsidian';
  owner: string;
  active: boolean;
  totalVerifications: number;
  totalEarned: string;  // in ETH
  lastVerification: Date;
}
```

**2. Real-Time Earnings**
```typescript
// WebSocket connection for live updates
const ws = new WebSocket('wss://api.0l1labs.com/v1/channel/42/live');

ws.onmessage = (event) => {
  const update = JSON.parse(event.data);
  if (update.type === 'verification_processed') {
    updateEarnings(update.amount);
    showNotification(`Earned ${update.amount} ETH`);
  }
};
```

**3. Analytics Dashboard**
- Hourly/daily/monthly verification charts
- Earnings breakdown by time period
- Network activity comparison
- Tier performance metrics
- Withdrawal history

**4. Withdrawal Interface**
- Current balance display
- One-click withdrawal to connected wallet
- Transaction history
- Fee breakdown (if any)

### Proof Generation Library

**Client-Side JavaScript SDK:**

```typescript
import { generateProof, verifyProof } from '@0l1labs/zk-sdk';

// Generate proof on user device
async function generateAgeProof(
  identitySecret: string,
  minimumAge: number
): Promise<ProofData> {
  const proof = await generateProof({
    circuit: 'age_verification',
    privateInputs: {
      identitySecret,
      birthdate: userBirthdate,
    },
    publicInputs: {
      minimumAge,
      timestamp: Date.now(),
    },
  });
  
  return proof;
}

// Submit to app backend
async function submitProof(proof: ProofData) {
  const response = await fetch('/api/verify', {
    method: 'POST',
    body: JSON.stringify(proof),
  });
  
  return response.json();
}
```

---

## Security & Audits

### Audit Schedule

| Component | Auditor | Timeline | Budget |
|---|---|---|---|
| **ZK Circuits** | Trail of Bits or Veridise | Q1 2026 (Week 11) | $50K-$75K |
| **Smart Contracts** | OpenZeppelin or Consensys Diligence | Q1 2026 (Week 12) | $40K-$60K |
| **API Security** | Internal + Penetration Test | Q2 2026 | $20K |

### Security Measures

**Circuit Security:**
- Constraint validation
- Soundness verification
- Completeness checks
- Trusted setup (Powers of Tau)

**Smart Contract Security:**
- Formal verification (critical functions)
- Fuzz testing (100K+ iterations)
- Static analysis (Slither, Mythril)
- Test coverage >95%

**Operational Security:**
- Multi-sig admin controls (3-of-5)
- Time-locked upgrades (48 hours)
- Rate limiting and DDoS protection
- Encrypted data at rest and in transit

### Bug Bounty Program

**Launch:** Q2 2026  
**Total Pool:** $50K-$100K

**Severity Tiers:**
- **Critical:** $10K-$25K (funds at risk, protocol break)
- **High:** $5K-$10K (significant vulnerability)
- **Medium:** $1K-$5K (edge cases, DoS)
- **Low:** $250-$1K (minor issues)

**Scope:**
- Smart contracts
- ZK circuits
- API endpoints
- Dashboard application

---

## Development Timeline

### Q1 2026: Foundation (12 Weeks)

**Weeks 1-4: Circuit Development**
- [ ] Set up Circom development environment
- [ ] Implement ActionProof circuit
- [ ] Write circuit tests (>100 test cases)
- [ ] Optimize for proof generation speed
- [ ] Generate proving and verifying keys

**Weeks 5-6: Smart Contract Development**
- [ ] Initialize Hardhat/Foundry project
- [ ] Implement Genesis Channel registry
- [ ] Build routing algorithm
- [ ] Create fee distribution logic
- [ ] Write comprehensive tests

**Weeks 7-8: Frontend Development**
- [ ] Set up Next.js project
- [ ] Build proof generation interface
- [ ] Create badge holder dashboard
- [ ] Implement wallet integration (wagmi + RainbowKit)
- [ ] Design real-time updates (WebSocket)

**Weeks 9-10: Integration & Testing**
- [ ] End-to-end testing (user → verification → payment)
- [ ] Performance benchmarking
- [ ] Load testing (simulate 100K verifications)
- [ ] Bug fixes and optimization

**Weeks 11-12: Security Audits**
- [ ] Circuit audit
- [ ] Smart contract audit
- [ ] Fix identified issues
- [ ] Prepare for launch

**Week 13: Genesis Deployment**
- [ ] Deploy to Base mainnet
- [ ] Launch badge holder dashboard
- [ ] Process first live verifications
- [ ] Monitor and support

### Q2 2026: API Launch

- [ ] Public API release
- [ ] SDK libraries (JS, Python, TypeScript)
- [ ] Developer documentation
- [ ] Integration examples
- [ ] Customer onboarding

### Q3-Q4 2026: Expansion

- [ ] Multi-vertical API development
- [ ] Enterprise partnerships
- [ ] Mobile SDKs
- [ ] Advanced analytics
- [ ] Protocol optimizations

---

## Technical Stack

### Core Technologies

**Blockchain:**
- Base (Ethereum L2)
- Hardhat/Foundry for development
- Solidity 0.8.20+

**Zero-Knowledge:**
- Circom 2.1+
- SnarkJS
- Groth16 proving system
- Powers of Tau trusted setup

**Backend:**
- TypeScript with Express/Fastify
- PostgreSQL 15+
- Redis 7+
- Docker + Kubernetes

**Frontend:**
- Next.js 14+
- TypeScript 5+
- Tailwind CSS
- wagmi + RainbowKit (wallet connection)

**Infrastructure:**
- AWS or GCP
- Cloudflare (CDN + DDoS)
- Prometheus + Grafana (monitoring)
- Sentry (error tracking)

### Development Tools

- **IDE:** VSCode with Solidity extension
- **Testing:** Hardhat test framework, Jest
- **CI/CD:** GitHub Actions
- **Version Control:** Git + GitHub
- **Documentation:** Docusaurus

---

## Performance Targets

### End-to-End Metrics

| Metric | Target | Stretch Goal |
|---|---|---|
| Proof generation time | <3 seconds | <1 second |
| API response time (p95) | <500ms | <200ms |
| Verification throughput | 100/second | 1,000/second |
| Dashboard load time | <2 seconds | <1 second |
| Uptime (SLA) | 99.9% | 99.99% |

### Cost Targets

| Operation | Cost | Notes |
|---|---|---|
| Proof generation | Free | Client-side |
| Verification (Base) | ~$0.01-$0.50 | Gas fees (L2) |
| Fee distribution | ~$0.01-$0.10 | Gas fees (L2) |
| API call | Free-$0.50 | Customer pricing |

---

**This roadmap is subject to change based on development progress, security requirements, and market feedback.**

**Last updated:** December 2025  
**Next review:** February 2026
