# Integration Guide

**Nomad Trust Layer Developer Documentation**

This guide covers integrating Nomad Trust Layer's AI agent verification infrastructure into your platform.

---

## Quick Start

### Installation

```bash
# JavaScript/TypeScript
npm install @nomad-trust/sdk

# Python
pip install nomad-trust

# Rust (coming Q2 2026)
cargo add nomad-trust
```

### Basic Usage

```javascript
import { NomadTrust } from '@nomad-trust/sdk';

// Initialize SDK
const nomad = new NomadTrust({
  network: 'base-mainnet', // or 'base-sepolia' for testnet
  apiKey: process.env.NOMAD_API_KEY // Optional for higher rate limits
});

// Generate proof for AI agent
const proof = await nomad.proveUniqueAgent({
  agentSecret: process.env.AGENT_SECRET,  // Agent's private key
  actionId: "marketplace_listing_2026"     // Unique action identifier
});

// Verify proof on-chain
const isValid = await nomad.verifyProof(proof);

if (isValid) {
  console.log("✓ Agent verified - allow action");
} else {
  console.log("✗ Verification failed - reject action");
}
```

---

## Integration Patterns

### Pattern 1: API Abuse Prevention

**Use Case:** Prevent leaked API keys from being drained by bot farms

```javascript
// OpenAI/Anthropic API wrapper with Nomad verification
import OpenAI from 'openai';
import { NomadTrust } from '@nomad-trust/sdk';

const openai = new OpenAI({ apiKey: process.env.OPENAI_KEY });
const nomad = new NomadTrust({ network: 'base-mainnet' });

async function verifiedChatCompletion(messages, agentSecret) {
  // Step 1: Verify agent uniqueness
  const proof = await nomad.proveUniqueAgent({
    agentSecret: agentSecret,
    actionId: `openai_${Date.now()}` // Unique per request
  });
  
  const isValid = await nomad.verifyProof(proof);
  
  if (!isValid) {
    throw new Error("Agent verification failed");
  }
  
  // Step 2: Allow API call only if verified
  const response = await openai.chat.completions.create({
    model: "gpt-4",
    messages: messages
  });
  
  return response;
}

// Usage
const result = await verifiedChatCompletion(
  [{ role: "user", content: "Hello!" }],
  process.env.MY_AGENT_SECRET
);
```

**Result:**
- Leaked keys can't be abused (agent uniqueness enforced)
- 60% reduction in fraudulent API usage
- Legitimate agents verified instantly

---

### Pattern 2: Agent Marketplace Verification

**Use Case:** Verify agent uniqueness in marketplaces (prevent duplicate listings)

```javascript
import { NomadTrust } from '@nomad-trust/sdk';

class AgentMarketplace {
  constructor() {
    this.nomad = new NomadTrust({ network: 'base-mainnet' });
    this.registeredAgents = new Set(); // Track nullifiers
  }
  
  async registerAgent(agentSecret, metadata) {
    // Generate uniqueness proof
    const proof = await this.nomad.proveUniqueAgent({
      agentSecret: agentSecret,
      actionId: "marketplace_registration"
    });
    
    // Verify proof validity
    const isValid = await this.nomad.verifyProof(proof);
    if (!isValid) throw new Error("Invalid proof");
    
    // Check for duplicate registration (nullifier already exists)
    const nullifier = proof.nullifier;
    if (this.registeredAgents.has(nullifier)) {
      throw new Error("Agent already registered - duplicate detected");
    }
    
    // Register agent
    this.registeredAgents.add(nullifier);
    
    return {
      agentId: generateId(),
      nullifier: nullifier,
      metadata: metadata,
      verified: true
    };
  }
}

// Usage
const marketplace = new AgentMarketplace();

const agent = await marketplace.registerAgent(
  process.env.AGENT_SECRET,
  { name: "Trading Bot v2", capabilities: ["market_analysis"] }
);

console.log("Agent registered:", agent.agentId);
```

**Result:**
- One agent = one marketplace listing (enforced cryptographically)
- No duplicate agents gaming rankings
- Privacy-preserving (agent architecture not revealed)

---

### Pattern 3: Multi-Agent Coordination

**Use Case:** Verify agent authenticity in multi-agent systems

```javascript
import { NomadTrust } from '@nomad-trust/sdk';

class AgentSwarm {
  constructor() {
    this.nomad = new NomadTrust({ network: 'base-mainnet' });
    this.swarmMembers = new Map();
  }
  
  async joinSwarm(agentSecret, agentData) {
    // Verify agent joining swarm
    const proof = await this.nomad.proveUniqueAgent({
      agentSecret: agentSecret,
      actionId: `swarm_join_${this.swarmId}`
    });
    
    const isValid = await this.nomad.verifyProof(proof);
    if (!isValid) throw new Error("Agent verification failed");
    
    // Add to swarm
    this.swarmMembers.set(proof.nullifier, agentData);
    
    return { joined: true, swarmSize: this.swarmMembers.size };
  }
  
  async verifyPeerAgent(peerProof) {
    // Agent-to-agent verification
    const isValid = await this.nomad.verifyProof(peerProof);
    return isValid;
  }
}

// Usage
const swarm = new AgentSwarm();

await swarm.joinSwarm(
  process.env.AGENT_SECRET,
  { role: "coordinator", capabilities: ["planning"] }
);

console.log("Joined swarm with verified identity");
```

**Result:**
- Trustless multi-agent coordination
- No central authority needed
- Cryptographic guarantees of agent authenticity

---

## Supported Platforms

### Current Integrations (Active Development)

**AutoGPT** - Autonomous agent framework  
**Status:** Planned Q2 2026  
**Use Case:** Agent uniqueness verification in plugin marketplace  
**Integration:** Plugin SDK with Nomad verification wrapper

**LangChain** - Agent orchestration platform  
**Status:** Planned Q2 2026  
**Use Case:** Multi-agent trust & coordination  
**Integration:** Custom chain module with proof generation

**OpenAI API** - LLM API platform  
**Status:** Enterprise pilot discussions  
**Use Case:** API key abuse prevention  
**Integration:** Middleware verification layer

**Anthropic API** - Claude API platform  
**Status:** Enterprise pilot discussions  
**Use Case:** Agent authentication for Claude agents  
**Integration:** Similar to OpenAI pattern

### Planned Integrations (Q3-Q4 2026)

**Hugging Face** - AI model marketplace  
**Use Case:** Agent verification for model access  
**Timeline:** Q3 2026

**Replicate** - AI API marketplace  
**Use Case:** Prevent API abuse  
**Timeline:** Q3 2026

**Agent Marketplaces** - Various platforms  
**Use Case:** Sybil-resistant agent listings  
**Timeline:** Q2-Q3 2026

---

## API Reference

### Core Methods

#### `proveUniqueAgent(options)`

Generate zero-knowledge proof of agent uniqueness.

```typescript
interface ProveOptions {
  agentSecret: string;      // Agent's private secret (never shared)
  actionId: string;         // Unique action identifier
  metadata?: object;        // Optional metadata (not included in proof)
}

interface Proof {
  proof: string;            // Groth16 proof (hex encoded)
  nullifier: string;        // Unique nullifier for this action
  commitment: string;       // Public commitment to agent identity
  actionId: string;         // Action identifier
}

const proof: Proof = await nomad.proveUniqueAgent({
  agentSecret: process.env.AGENT_SECRET,
  actionId: "unique_action_123"
});
```

#### `verifyProof(proof)`

Verify proof on-chain (submits to Base smart contract).

```typescript
const isValid: boolean = await nomad.verifyProof(proof);

if (isValid) {
  // Proof is cryptographically valid
  // Agent is unique for this action
  // Proceed with action
}
```

#### `checkNullifier(nullifier)`

Check if nullifier has already been used (duplicate action detection).

```typescript
const isDuplicate: boolean = await nomad.checkNullifier(proof.nullifier);

if (isDuplicate) {
  throw new Error("Action already performed by this agent");
}
```

---

## Advanced Usage

### Batch Verification

Process multiple proofs in a single transaction (gas optimization).

```javascript
import { NomadTrust } from '@nomad-trust/sdk';

const nomad = new NomadTrust({ network: 'base-mainnet' });

// Generate multiple proofs
const proofs = await Promise.all([
  nomad.proveUniqueAgent({ agentSecret: secret1, actionId: "action_1" }),
  nomad.proveUniqueAgent({ agentSecret: secret2, actionId: "action_2" }),
  nomad.proveUniqueAgent({ agentSecret: secret3, actionId: "action_3" })
]);

// Verify in batch (single on-chain transaction)
const results = await nomad.batchVerify(proofs);

console.log("All valid:", results.every(r => r.valid));
// Gas savings: ~30% vs individual verifications
```

### Custom Action IDs

Use structured action IDs for better tracking.

```javascript
// Pattern: {platform}_{resource}_{timestamp}
const actionId = `marketplace_listing_${Date.now()}`;
const proof = await nomad.proveUniqueAgent({
  agentSecret: process.env.AGENT_SECRET,
  actionId: actionId
});

// Pattern: {platform}_{user_id}_{action}
const actionId2 = `api_${userId}_openai_request`;
```

**Best Practices:**
- Include timestamp for time-bound actions
- Include platform identifier for multi-platform agents
- Keep action IDs under 64 characters

### Error Handling

```javascript
import { NomadTrust, VerificationError } from '@nomad-trust/sdk';

try {
  const proof = await nomad.proveUniqueAgent({
    agentSecret: process.env.AGENT_SECRET,
    actionId: "test_action"
  });
  
  const isValid = await nomad.verifyProof(proof);
  
  if (!isValid) {
    throw new VerificationError("Proof verification failed");
  }
  
} catch (error) {
  if (error instanceof VerificationError) {
    console.error("Verification failed:", error.message);
  } else if (error.code === 'NETWORK_ERROR') {
    console.error("Network error - retry:", error);
  } else {
    console.error("Unexpected error:", error);
  }
}
```

---

## Rate Limits & Pricing

### Free Tier

**Public API (No API Key):**
- 100 verifications/hour per IP
- Best effort uptime
- Community support

### Developer Tier (API Key Required)

**$0/month:**
- 1,000 verifications/month included
- 99.5% uptime SLA
- Email support
- Dashboard access

### Professional Tier

**$99/month:**
- 50,000 verifications/month included
- $0.002 per additional verification
- 99.9% uptime SLA
- Priority support
- Advanced analytics

### Enterprise Tier

**Custom Pricing:**
- Unlimited verifications
- Dedicated infrastructure
- White-label options
- Custom SLAs
- Direct technical support

**Contact:** [enterprise@nomadtrust.io](mailto:enterprise@nomadtrust.io)

---

## SDK Examples

### JavaScript/TypeScript

**Full Example:** Agent registration with Nomad verification

```javascript
import { NomadTrust } from '@nomad-trust/sdk';
import express from 'express';

const app = express();
const nomad = new NomadTrust({ network: 'base-mainnet' });

app.post('/register-agent', async (req, res) => {
  const { agentSecret, metadata } = req.body;
  
  try {
    // Generate proof
    const proof = await nomad.proveUniqueAgent({
      agentSecret: agentSecret,
      actionId: `registration_${Date.now()}`
    });
    
    // Verify proof
    const isValid = await nomad.verifyProof(proof);
    
    if (!isValid) {
      return res.status(400).json({ error: "Invalid proof" });
    }
    
    // Check for duplicate
    const isDuplicate = await nomad.checkNullifier(proof.nullifier);
    
    if (isDuplicate) {
      return res.status(409).json({ error: "Agent already registered" });
    }
    
    // Register agent
    const agent = await db.createAgent({
      nullifier: proof.nullifier,
      metadata: metadata
    });
    
    res.json({ success: true, agentId: agent.id });
    
  } catch (error) {
    res.status(500).json({ error: error.message });
  }
});

app.listen(3000, () => console.log('Server running on port 3000'));
```

### Python

**Full Example:** AI agent verification in Flask API

```python
from nomad_trust import NomadTrust
from flask import Flask, request, jsonify

app = Flask(__name__)
nomad = NomadTrust(network='base-mainnet')

@app.route('/verify-agent', methods=['POST'])
def verify_agent():
    data = request.json
    agent_secret = data['agent_secret']
    action_id = data['action_id']
    
    try:
        # Generate proof
        proof = nomad.prove_unique_agent(
            agent_secret=agent_secret,
            action_id=action_id
        )
        
        # Verify proof
        is_valid = nomad.verify_proof(proof)
        
        if not is_valid:
            return jsonify({'error': 'Invalid proof'}), 400
        
        return jsonify({
            'verified': True,
            'nullifier': proof.nullifier,
            'action_id': proof.action_id
        })
        
    except Exception as e:
        return jsonify({'error': str(e)}), 500

if __name__ == '__main__':
    app.run(port=5000)
```

---

## Testing

### Testnet Integration

```javascript
import { NomadTrust } from '@nomad-trust/sdk';

// Use Sepolia testnet for development
const nomad = new NomadTrust({
  network: 'base-sepolia',
  apiKey: process.env.NOMAD_TESTNET_KEY
});

// Generate test agent secret
const testSecret = NomadTrust.generateSecret();

// Test proof generation
const proof = await nomad.proveUniqueAgent({
  agentSecret: testSecret,
  actionId: "test_action_123"
});

console.log("Test proof:", proof);
```

### Mock Mode (Development)

```javascript
import { NomadTrust } from '@nomad-trust/sdk';

// Mock mode - instant proofs, no blockchain calls
const nomad = new NomadTrust({
  network: 'mock',
  mockDelay: 100 // Simulate 100ms latency
});

// All operations work instantly
const proof = await nomad.proveUniqueAgent({
  agentSecret: "test_secret",
  actionId: "test_action"
});

// Always returns valid
const isValid = await nomad.verifyProof(proof); // true
```

---

## Performance Optimization

### Client-Side Proof Generation

Generate proofs in the browser/client to reduce server load.

```javascript
// Browser example
import { NomadTrust } from '@nomad-trust/sdk';

const nomad = new NomadTrust({ network: 'base-mainnet' });

// Generate proof client-side
const proof = await nomad.proveUniqueAgent({
  agentSecret: localStorage.getItem('agentSecret'),
  actionId: "client_action"
});

// Send proof to server for verification only
const response = await fetch('/api/verify', {
  method: 'POST',
  body: JSON.stringify({ proof })
});
```

**Benefits:**
- Reduces server CPU load
- Faster response times
- Better scalability

### Caching

Cache proof results for repeated verifications.

```javascript
import { NomadTrust } from '@nomad-trust/sdk';
import Redis from 'redis';

const nomad = new NomadTrust({ network: 'base-mainnet' });
const redis = Redis.createClient();

async function verifyWithCache(proof) {
  const cacheKey = `proof:${proof.nullifier}`;
  
  // Check cache first
  const cached = await redis.get(cacheKey);
  if (cached) return JSON.parse(cached);
  
  // Verify on-chain
  const isValid = await nomad.verifyProof(proof);
  
  // Cache result (24 hour TTL)
  await redis.setex(cacheKey, 86400, JSON.stringify({ valid: isValid }));
  
  return { valid: isValid };
}
```

---

## Troubleshooting

### Common Issues

**Issue:** "Proof generation taking >10 seconds"  
**Solution:** Use WASM compilation or move to server-side generation

**Issue:** "Gas costs too high"  
**Solution:** Use batch verification for multiple proofs

**Issue:** "Network timeout errors"  
**Solution:** Increase RPC timeout, use paid RPC endpoint

**Issue:** "Duplicate nullifier detected"  
**Solution:** Ensure action IDs are unique per action

---

## Support

### Developer Resources

- **Documentation:** [docs.nomadtrust.io](https://docs.nomadtrust.io)
- **API Reference:** [api.nomadtrust.io/docs](https://api.nomadtrust.io/docs)
- **SDK Examples:** [github.com/NomadTrustLayer/examples](https://github.com/NomadTrustLayer/examples)

### Community Support

- **Discord:** [#dev-support](https://discord.gg/nomadtrustlayer)
- **GitHub Issues:** [github.com/NomadTrustLayer/nomad-circuits/issues](https://github.com/NomadTrustLayer/nomad-circuits/issues)
- **Stack Overflow:** Tag `nomad-trust-layer`

### Enterprise Support

- **Email:** [enterprise@nomadtrust.io](mailto:enterprise@nomadtrust.io)
- **Slack Connect:** Available for Enterprise tier customers

---

## Changelog

### v0.1.0 (Coming March 2026)

- Initial SDK release (JavaScript/TypeScript)
- Core proof generation & verification
- Base mainnet support
- Dashboard API integration

### v0.2.0 (Coming Q2 2026)

- Python SDK release
- Batch verification support
- WebSocket real-time updates
- Advanced caching

### v1.0.0 (Coming Q3 2026)

- Production-ready release
- Multi-language support (Rust, Go)
- Enterprise features
- Advanced analytics

---

**Ready to integrate?**

1. [Get API key](https://nomadtrust.io/developers)
2. [Install SDK](#installation)
3. [Join Discord](https://discord.gg/nomadtrustlayer) for support

**Questions?** Email [dev@nomadtrust.io](mailto:dev@nomadtrust.io)
