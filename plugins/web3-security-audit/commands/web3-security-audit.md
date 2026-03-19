---
allowed-tools: Bash(gh issue view:*), Bash(gh search:*), Bash(gh issue list:*), Bash(gh pr comment:*), Bash(gh pr diff:*), Bash(gh pr view:*), Bash(gh pr list:*), Bash(cat:*), Bash(find:*), Bash(grep:*), Bash(git diff:*), Bash(git log:*), Bash(git show:*), mcp__github_inline_comment__create_inline_comment
description: Web3 smart contract and dApp security audit based on the OWASP SCS Top 10 and Web3 Attack Vectors Top 15
---

Perform a Web3 security audit on the given smart contract code, pull request, or repository files.

**Agent assumptions (applies to all agents and subagents):**
- All tools are functional and will work without error. Do not test tools or make exploratory calls. Make sure this is clear to every subagent that is launched.
- Only call a tool if it is required to complete the task. Every tool call should have a clear purpose.
- You are a Web3 security expert specializing in smart contract vulnerabilities and blockchain application security.

To do this, follow these steps precisely:

1. Launch a haiku agent to determine the audit scope:
   - If running on a pull request: check if the PR is closed or a draft and skip if so.
   - Identify all smart contract files and Web3-related code in scope. Look for:
     - Solidity files (`.sol`)
     - Vyper files (`.vy`)
     - Rust smart contract files (e.g., Anchor/Solana programs, CosmWasm, Ink!)
     - Move files (`.move`)
     - Cairo files (`.cairo`)
     - JavaScript/TypeScript files containing Web3 interactions (ethers.js, web3.js, viem, wagmi)
     - Configuration files for deployment, proxy upgrades, or governance
     - Frontend dApp code that interacts with wallets or contracts
   - If no Web3-related files are found, stop and report: "No Web3 or smart contract code found in scope."

2. Launch a haiku agent to gather context:
   - Collect the project structure and any security-related documentation
   - Identify the blockchain platform(s) targeted (Ethereum, Solana, etc.)
   - Note any existing security tools configured (Slither, Mythril, Foundry tests, etc.)
   - Identify imported libraries and dependencies (OpenZeppelin, Solmate, etc.)

3. Launch a sonnet agent to summarize the code under audit:
   - For PRs: summarize the changes with focus on security-relevant modifications
   - For direct file audit: summarize the contract architecture, external interactions, and state management

4. Launch 5 agents in parallel to independently audit the code. Each agent should return a list of findings, where each finding includes:
   - A severity level: **Critical**, **High**, **Medium**, **Low**, or **Informational**
   - The OWASP category it falls under (SCS Top 10 or Web3 Attack Vectors Top 15)
   - A description of the vulnerability
   - The affected code location (file and line numbers)
   - A recommended fix

   The 5 agents are:

   **Agent 1: OWASP SCS Top 10 — Access Control & Reentrancy (Opus agent)**
   Audit for the following OWASP Smart Contract Security Top 10 categories:
   - **SCS-1: Access Control Vulnerabilities** — Missing or improper permission checks (`onlyOwner`, role-based access), unprotected admin functions, missing `msg.sender` validation, improper use of `tx.origin`, unprotected `selfdestruct`/`delegatecall`
   - **SCS-5: Reentrancy Attacks** — State changes after external calls, cross-function reentrancy, cross-contract reentrancy, read-only reentrancy, missing reentrancy guards on functions that transfer value or call external contracts
   - **SCS-6: Unchecked External Calls** — Missing return value checks on `call`/`send`/`transfer`, unchecked low-level calls, missing error handling on cross-contract interactions

   **Agent 2: OWASP SCS Top 10 — Logic, Oracles & Math (Opus agent)**
   Audit for the following OWASP Smart Contract Security Top 10 categories:
   - **SCS-2: Price Oracle Manipulation** — Spot price reliance, single-source oracles, TWAP manipulation, flash-loan-vulnerable price feeds, missing staleness checks on Chainlink oracles
   - **SCS-3: Logic Errors** — Flawed reward/distribution calculations, broken invariants, incorrect state machine transitions, governance logic flaws, off-by-one errors in financial logic
   - **SCS-7: Flash Loan Attacks** — Functions vulnerable to flash-loan-amplified exploits, missing same-block protections, governance attacks via flash-borrowed tokens
   - **SCS-8: Integer Overflow/Underflow** — Unsafe arithmetic in Solidity <0.8.0 without SafeMath, unchecked blocks with risky math, precision loss in division, rounding errors in token/share calculations

   **Agent 3: OWASP SCS Top 10 — Input Validation & Upgradeability (Opus agent)**
   Audit for the following OWASP Smart Contract Security Top 10 categories:
   - **SCS-4: Lack of Input Validation** — Missing zero-address checks, missing bounds validation, unvalidated array lengths, missing slippage parameters, unconstrained function arguments
   - **SCS-9: Insecure Randomness** — Use of `block.timestamp`, `block.number`, `blockhash`, or other miner-influenceable values for randomness; predictable seed generation
   - **SCS-10: Denial of Service** — Unbounded loops over dynamic arrays, block gas limit issues, external call failures causing DoS, griefing vectors
   Also audit for proxy and upgradeability vulnerabilities:
   - Uninitialized proxy implementations, storage slot collisions, missing `initializer` modifier, unprotected `upgradeTo` functions, broken storage layout across upgrades

   **Agent 4: Web3 Attack Vectors — Infrastructure & Supply Chain (Sonnet agent)**
   Audit for the following OWASP Web3 Attack Vectors Top 15 categories as they appear in the codebase:
   - **Multisig Hijacking** — Insecure multisig configurations, insufficient signer thresholds, missing timelocks on critical operations
   - **Key/Seed Exposure** — Hardcoded private keys, seed phrases, or mnemonics in source code, config files, or environment files checked into version control
   - **API Key or Cloud Credential Exfiltration** — Exposed RPC provider keys, deployment keys, infura/alchemy keys in source, `.env` files committed to repo
   - **Software Supply Chain Attacks** — Suspicious or unaudited dependencies, npm packages with known vulnerabilities, unverified contract imports
   - **CDN/Frontend Compromise** — Frontend code that loads contracts or ABIs from untrusted sources, missing integrity checks on loaded resources
   - **Compromised Upgrades, Bridges, or Cross-Chain Infrastructure** — Insecure bridge configurations, unprotected relay mechanisms, excessive upgrade permissions

   **Agent 5: Web3 Attack Vectors — Social, Governance & User Safety (Sonnet agent)**
   Audit for the following OWASP Web3 Attack Vectors Top 15 categories as they appear in the codebase:
   - **Phishing with Malicious Frontends** — Missing EIP-712 typed data signing, unclear transaction signing prompts, missing human-readable transaction descriptions
   - **Malicious Governance Proposals** — Unprotected governance execution, missing timelocks, proposals that can bypass safety checks, quorum manipulation
   - **Front-Running via MEV** — Transactions vulnerable to sandwich attacks, missing commit-reveal schemes, missing deadline parameters, missing private mempool usage for sensitive operations
   - **Fraudulent Token/NFT Interactions** — Missing approval hygiene (unlimited approvals), missing `permit` signature validation, ERC-20/721/1155 compliance issues
   - **Blind Signing Risks** — Contract interactions that produce opaque signing requests, missing EIP-712 domain separators, calldata that cannot be parsed by hardware wallets

   **CRITICAL: We only want HIGH SIGNAL findings.** Flag issues where:
   - The vulnerability can lead to direct loss of funds or unauthorized access
   - The vulnerability is clearly present in the code (not speculative)
   - The issue represents a real deviation from security best practices for the specific blockchain platform
   - The OWASP category clearly applies to the code pattern found

   Do NOT flag:
   - Gas optimization suggestions (unless they cause DoS)
   - Code style or naming conventions
   - Theoretical vulnerabilities that require unrealistic conditions
   - Issues in well-audited imported libraries (e.g., OpenZeppelin) unless misused
   - Issues already mitigated by other code in the same project

   Each subagent should receive the full audit context (project type, platform, purpose of the code).

5. For each finding from the previous step, launch parallel subagents to validate the finding. These validation agents should:
   - Verify the vulnerable code pattern actually exists at the stated location
   - Confirm the OWASP category is correctly assigned
   - Check if the vulnerability is mitigated elsewhere in the codebase (e.g., by a modifier, inherited contract, or external guard)
   - Assign a confidence score from 0-100
   Use Opus subagents for Critical/High severity findings and Sonnet agents for Medium/Low/Informational findings.

6. Filter out any findings that:
   - Were not validated in step 5
   - Have a confidence score below 75
   - Are mitigated by other code in the project

7. Output a structured security audit report to the terminal:

   If findings were found:
   ```
   ## Web3 Security Audit

   **Scope:** [files/PR audited]
   **Platform:** [Ethereum/Solana/etc.]
   **Standards:** OWASP SCS Top 10, OWASP Web3 Attack Vectors Top 15

   ### Critical Findings
   [List critical findings with OWASP category, description, location, and fix]

   ### High Findings
   [List high findings]

   ### Medium Findings
   [List medium findings]

   ### Low / Informational Findings
   [List low findings]

   ### Summary
   - Total findings: X (Y Critical, Z High, ...)
   - OWASP SCS Top 10 categories covered: [list]
   - Web3 Attack Vectors Top 15 categories covered: [list]
   ```

   If no findings were found:
   ```
   ## Web3 Security Audit

   No vulnerabilities found. Audited against OWASP SCS Top 10 and Web3 Attack Vectors Top 15.
   ```

   If `--comment` argument was NOT provided, stop here. Do not post any GitHub comments.

   If `--comment` argument IS provided and NO findings were found, post a summary comment using `gh pr comment` and stop.

   If `--comment` argument IS provided and findings were found, continue to step 8.

8. Create a list of all comments that you plan on leaving. This is only for you to make sure you are comfortable with the comments. Do not post this list anywhere.

9. Post inline comments for each finding using `mcp__github_inline_comment__create_inline_comment` with `confirmed: true`. For each comment:
   - Prefix with severity and OWASP category: e.g., `**[Critical | SCS-1: Access Control]**`
   - Provide a brief description of the vulnerability
   - For small, self-contained fixes, include a committable suggestion block
   - For larger fixes (6+ lines, structural changes, or changes spanning multiple locations), describe the vulnerability and recommended remediation without a suggestion block
   - Include a link to the relevant OWASP page for reference

   **IMPORTANT: Only post ONE comment per unique finding. Do not post duplicate comments.**

Use this list when evaluating findings in Steps 4 and 5 (these are false positives, do NOT flag):

- Pre-existing issues in code that was not modified (when auditing a PR)
- Patterns that look vulnerable but are protected by modifiers, guards, or checks elsewhere
- Issues in battle-tested library code (OpenZeppelin, Solmate, etc.) unless the library is misused
- Theoretical attacks that require protocol-level changes or miner collusion
- Gas inefficiencies that do not cause security issues
- Issues already covered by well-configured automated tools (Slither findings, compiler warnings)

OWASP Reference Links (use in comments when citing categories):
- SCS Top 10: https://scs.owasp.org/sctop10
- Web3 Attack Vectors Top 15: https://scs.owasp.org/sctop10/Web3-Attack-Vectors-Top15/

Notes:

- Use gh CLI to interact with GitHub (e.g., fetch pull requests, create comments). Do not use web fetch.
- Create a todo list before starting.
- You must cite the OWASP category for each finding.
- If no findings are found and `--comment` argument is provided, post a comment with the following format:

---

## Web3 Security Audit

No vulnerabilities found. Audited against OWASP SCS Top 10 and Web3 Attack Vectors Top 15.

---

- When linking to code in inline comments, follow the following format precisely, otherwise the Markdown preview won't render correctly: https://github.com/owner/repo/blob/c21d3c10bc8e898b7ac1a2d745bdc9bc4e423afe/contracts/Token.sol#L10-L15
  - Requires full git sha
  - You must provide the full sha. Commands like `https://github.com/owner/repo/blob/$(git rev-parse HEAD)/foo/bar` will not work, since your comment will be directly rendered in Markdown.
  - Repo name must match the repo you're auditing
  - # sign after the file name
  - Line range format is L[start]-L[end]
  - Provide at least 1 line of context before and after, centered on the line you are commenting about
