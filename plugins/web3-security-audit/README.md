# Web3 Security Audit Plugin

Web3 smart contract and dApp vulnerability hunter based on the [OWASP Smart Contract Security Top 10](https://scs.owasp.org/sctop10) and [OWASP Web3 Attack Vectors Top 15](https://scs.owasp.org/sctop10/Web3-Attack-Vectors-Top15/).

## Overview

The Web3 Security Audit Plugin automates security auditing of smart contracts and decentralized applications by launching multiple specialized agents in parallel. Each agent focuses on specific OWASP vulnerability categories, and findings are validated with confidence scoring to filter out false positives.

## Commands

### `/web3-security-audit`

Performs a comprehensive Web3 security audit on smart contract code or a pull request.

**What it does:**
1. Identifies Web3/smart contract files in scope (Solidity, Vyper, Rust, Move, Cairo, JS/TS with Web3 libs)
2. Gathers project context (platform, dependencies, existing security tooling)
3. Summarizes the code under audit
4. Launches 5 parallel agents to independently audit against OWASP categories:
   - **Agent #1**: SCS Top 10 — Access Control, Reentrancy, Unchecked External Calls (SCS-1, SCS-5, SCS-6)
   - **Agent #2**: SCS Top 10 — Oracle Manipulation, Logic Errors, Flash Loans, Arithmetic (SCS-2, SCS-3, SCS-7, SCS-8)
   - **Agent #3**: SCS Top 10 — Input Validation, Randomness, DoS, Proxy/Upgradeability (SCS-4, SCS-9, SCS-10)
   - **Agent #4**: Web3 Attack Vectors — Infrastructure & Supply Chain (Multisig, Key Exposure, API Keys, Supply Chain, CDN, Bridges)
   - **Agent #5**: Web3 Attack Vectors — Social, Governance & User Safety (Phishing, Governance, MEV, Token Safety, Blind Signing)
5. Validates each finding with dedicated subagents
6. Filters findings below 75 confidence threshold
7. Outputs a structured security audit report (to terminal by default, or as PR comment with `--comment` flag)

**Usage:**
```bash
/web3-security-audit [--comment]
```

**Options:**
- `--comment`: Post the audit report as inline comments on the pull request (default: outputs to terminal only)

**Example workflow:**
```bash
# Audit smart contracts in the current project:
/web3-security-audit

# Audit a PR and post findings as inline comments:
/web3-security-audit --comment

# Claude will:
# - Identify all smart contract and Web3 code
# - Launch 5 audit agents covering all OWASP categories
# - Validate and score each finding
# - Output findings ≥75 confidence with OWASP classification
```

## OWASP Coverage

### Smart Contract Security Top 10 (SCS Top 10)

| ID     | Category                        | Description                                                  |
|--------|---------------------------------|--------------------------------------------------------------|
| SCS-1  | Access Control Vulnerabilities  | Missing/improper permission checks, unprotected admin functions |
| SCS-2  | Price Oracle Manipulation       | Weak oracle mechanisms, flash-loan-vulnerable price feeds    |
| SCS-3  | Logic Errors                    | Flawed business logic, broken invariants, miscalculations    |
| SCS-4  | Lack of Input Validation        | Missing zero-address checks, unbounded parameters            |
| SCS-5  | Reentrancy Attacks              | State changes after external calls, cross-function reentrancy |
| SCS-6  | Unchecked External Calls        | Missing return value checks on low-level calls               |
| SCS-7  | Flash Loan Attacks              | Functions vulnerable to flash-loan-amplified exploits        |
| SCS-8  | Integer Overflow/Underflow      | Unsafe arithmetic, precision loss, rounding errors           |
| SCS-9  | Insecure Randomness             | Predictable on-chain randomness sources                      |
| SCS-10 | Denial of Service               | Unbounded loops, gas limit issues, griefing vectors          |

### Web3 Attack Vectors Top 15

| #  | Attack Vector                                           | Description                                                      |
|----|---------------------------------------------------------|------------------------------------------------------------------|
| 1  | Multisig Hijacking                                      | Insecure multisig configs, insufficient thresholds               |
| 2  | Key/Seed Theft via Seed Phrases                         | Hardcoded keys/seeds in source or config                         |
| 3  | API Key or Cloud Credential Exfiltration                | Exposed RPC/deployment keys                                      |
| 4  | Malicious Browser Extensions or Wallet Plugins          | Untrusted wallet interactions                                    |
| 5  | Phishing with Malicious Frontends                       | Missing EIP-712, unclear signing prompts                         |
| 6  | Social Engineering and Impersonation                    | Impersonation in project communications                          |
| 7  | Software Supply Chain Attacks                           | Unaudited dependencies, malicious packages                       |
| 8  | CDN/Frontend Compromise                                 | Untrusted ABI/contract loading, missing integrity checks         |
| 9  | Cloud Provider Escalation                               | Exposed cloud credentials, misconfigurations                     |
| 10 | Compromised Upgrades, Bridges, or Cross-Chain Infra     | Insecure bridge/relay/upgrade mechanisms                         |
| 11 | Malicious Governance Proposals                          | Unprotected governance, missing timelocks                        |
| 12 | Spam/Griefing in Decentralized Infra                    | Flooding attacks on decentralized services                       |
| 13 | Fraudulent NFT Drops, Token Mints, or Airdrops          | Malicious token interactions, unlimited approvals                |
| 14 | Front-Running via RPC Providers or MEV Bots             | Sandwich attacks, missing commit-reveal schemes                  |
| 15 | Ledger/Hardware Wallet Supply Chain and Blind Signing    | Opaque signing requests, missing domain separators               |

## Severity Levels

Findings are classified by severity:

- **Critical**: Direct, exploitable loss of funds or complete access control bypass
- **High**: Significant vulnerability that could lead to fund loss under realistic conditions
- **Medium**: Vulnerability requiring specific conditions or limited impact
- **Low**: Minor issue or deviation from best practices with minimal security impact
- **Informational**: Observation or recommendation for improved security posture

## Confidence Scoring

Each finding is independently validated and scored 0-100:

- **0**: Not confident, likely false positive
- **25**: Somewhat confident, might be real
- **50**: Moderately confident, real but may be mitigated
- **75**: Highly confident, real and exploitable
- **100**: Absolutely certain, clearly vulnerable

Findings below 75 confidence are filtered out by default.

## Audit Report Format

```markdown
## Web3 Security Audit

**Scope:** contracts/Token.sol, contracts/Vault.sol
**Platform:** Ethereum (Solidity 0.8.x)
**Standards:** OWASP SCS Top 10, OWASP Web3 Attack Vectors Top 15

### Critical Findings

1. **[SCS-5: Reentrancy]** Withdraw function updates balance after external call

   https://github.com/owner/repo/blob/abc123.../contracts/Vault.sol#L45-L52

   **Fix:** Apply checks-effects-interactions pattern or add `nonReentrant` modifier.

### High Findings

2. **[SCS-1: Access Control]** Missing access control on `setOracle()` allows anyone to change price feed

   https://github.com/owner/repo/blob/abc123.../contracts/Vault.sol#L30-L35

   **Fix:** Add `onlyOwner` modifier.

### Summary
- Total findings: 2 (1 Critical, 1 High)
- OWASP SCS Top 10 categories covered: SCS-1, SCS-5
```

## False Positive Filtering

The following are NOT flagged:
- Pre-existing issues in unmodified code (when auditing a PR)
- Patterns protected by modifiers, guards, or checks elsewhere in the codebase
- Issues in well-audited libraries (OpenZeppelin, Solmate) unless misused
- Theoretical attacks requiring protocol-level changes or miner collusion
- Gas inefficiencies that do not cause security issues
- Issues covered by configured automated tools

## Supported Platforms & Languages

| Platform  | Language(s)               | File Extensions       |
|-----------|---------------------------|-----------------------|
| Ethereum  | Solidity, Vyper           | `.sol`, `.vy`         |
| Solana    | Rust (Anchor)             | `.rs`                 |
| Cosmos    | Rust (CosmWasm)           | `.rs`                 |
| Polkadot  | Rust (Ink!)               | `.rs`                 |
| Aptos/Sui | Move                      | `.move`               |
| StarkNet  | Cairo                     | `.cairo`              |
| Frontend  | JavaScript, TypeScript    | `.js`, `.ts`, `.tsx`  |

## Installation

This plugin is included in the Claude Code repository. The command is automatically available when using Claude Code.

## Best Practices

### Using `/web3-security-audit`
- Run before deploying any smart contract to mainnet
- Use on all PRs that modify contract logic or Web3 interactions
- Review findings as a starting point — always pair with manual expert review
- Run alongside existing tools (Slither, Mythril, Foundry tests) for defense in depth

### When to use
- All PRs modifying smart contracts or Web3 code
- Before contract deployment or upgrade
- After adding new external integrations (oracles, bridges, DEXs)
- When implementing new DeFi mechanisms (lending, staking, governance)

### When not to use
- Non-Web3 codebases with no smart contract or blockchain code
- Closed or draft PRs (automatically skipped)
- Pure frontend changes with no wallet/contract interaction

## Workflow Integration

### Standard audit workflow:
```bash
# Develop smart contract changes
# Run local audit (outputs to terminal)
/web3-security-audit

# Review findings and apply fixes
# Re-run to verify fixes

# Optionally post as PR comment
/web3-security-audit --comment

# Proceed with additional tooling (Slither, Mythril, etc.)
# Deploy when ready
```

### As part of CI/CD:
```bash
# Trigger on PR creation or update
/web3-security-audit --comment
```

## Requirements

- Git repository with smart contract or Web3 code
- GitHub CLI (`gh`) installed and authenticated (for PR comment features)

## Troubleshooting

### Audit takes too long

**Issue**: Agents are slow on large contracts

**Solution**:
- Normal for complex contracts — 5 agents run in parallel
- Consider auditing specific files rather than entire repositories
- Split large PRs into focused changes

### Too many false positives

**Issue**: Audit flags issues that aren't real

**Solution**:
- Default threshold is 75 (filters most false positives)
- Check if flagged patterns are protected elsewhere in the codebase
- Well-audited library usage is automatically excluded

### No Web3 code detected

**Issue**: Audit reports no smart contract code found

**Solution**:
- Ensure smart contract files use standard extensions (`.sol`, `.vy`, `.rs`, `.move`, `.cairo`)
- Ensure Web3 JavaScript/TypeScript files import known Web3 libraries
- Check that the correct files are in scope

## OWASP References

- [OWASP Smart Contract Security Top 10](https://scs.owasp.org/sctop10)
- [OWASP Web3 Attack Vectors Top 15](https://scs.owasp.org/sctop10/Web3-Attack-Vectors-Top15/)

## Version

1.0.0
