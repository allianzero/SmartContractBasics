# SmartContractBasics

Smart contract basics: what they do, how they run, and how to read one. A smart contract is a program stored on a blockchain that runs automatically when its conditions are met — no bank, company, or person needed to execute it.

> This guide is educational only, not financial or legal advice. Code is not law, and a contract that works as written can still do something its users did not expect.

## 1. What a smart contract is

A traditional contract is a document enforced by courts. A smart contract is code enforced by a blockchain network:

- It lives at an **address** on the blockchain, like a wallet, but controlled by code instead of a person.
- It holds assets (tokens, coins) and rules for moving them.
- When someone sends it a transaction that matches its rules, it executes automatically, and every node in the network verifies the result.
- Once deployed, its code generally cannot be changed — unless it was deliberately designed to be upgradeable (see section 6).

**Example in plain language:** a vending machine is a mechanical smart contract. You put in coins, select an item, and the machine delivers it without a shopkeeper. A blockchain smart contract does the same thing, but with digital assets, and with thousands of independent computers verifying that the machine behaved correctly.

## 2. How smart contracts run

1. **Deployment:** a developer writes the contract (commonly in Solidity for Ethereum-compatible chains), compiles it, and sends a deployment transaction. The network assigns it an address.
2. **Interaction:** a user (or another contract) sends a transaction to that address, calling one of its **functions** — for example, `transfer`, `swap`, or `mint` — with some parameters.
3. **Execution:** every validating node runs the same code with the same inputs and must reach the same result. This is what makes the result trustworthy without a central authority.
4. **Gas:** the user pays a fee based on how much computation and storage the execution uses. More complex operations cost more gas.
5. **State change:** if execution succeeds, the contract's stored data and balances update permanently on the blockchain. If it fails (for example, a condition is not met), the state does not change — but gas is still charged for the work attempted.

Contracts can call other contracts, which is how DeFi apps combine: a lending contract can call a price oracle, a DEX, and a token contract in a single transaction.

## 3. The most common contract types

### Tokens
- **ERC-20 / BEP-20 (fungible tokens):** the standard for interchangeable tokens. Defines functions like `transfer`, `balanceOf`, and `approve`.
- **ERC-721 (NFTs):** unique tokens, each with its own ID and owner.
- **ERC-1155:** a multi-token standard that can handle fungible and non-fungible tokens in one contract.

### Decentralised exchanges and pools
Contracts that hold token pairs and let users swap, add liquidity, or remove it, according to a pricing formula.

### Lending protocols
Contracts that accept collateral, issue loans, charge interest, and liquidate undercollateralised positions automatically.

### DAOs and governance
Contracts that hold a treasury and execute decisions voted on by token holders.

### Oracles and bridges
Contracts that bring outside data (prices) onto the chain, or lock and release assets across chains. Both are common points of failure — see SmartContractSecurity.

## 4. How to read a contract (beginner level)

You do not need to be a programmer to do basic checks on an explorer like Etherscan or BscScan:

1. **Is the contract verified?** Verified means the published source code matches the deployed code. Unverified code cannot be read — treat it as a black box you should not trust with significant funds.
2. **Read the "Read Contract" tab.** This shows public data: total supply, owner address, balances, and settings. Look for anything surprising, such as an owner who can pause transfers.
3. **Read the "Write Contract" tab.** This lists the functions users can call. Dangerous functions to notice: `mint` (create new tokens), `burn`, `pause`, `setFee`, `transferOwnership`, and any function only the owner can call.
4. **Check the owner.** Is the owner a normal wallet, a multisig (multiple signers required), a DAO contract, or a burned address (meaning ownership was renounced)? A single wallet owner can change the rules unilaterally.
5. **Check recent transactions.** Is the contract actually used by many distinct addresses, or only by a few? Sudden bursts of activity around launch can be bots.
6. **Check the token holders.** Extreme concentration in a few wallets is a manipulation risk.

If you cannot find verified source code, or the owner functions look dangerous and unexplained, do not interact with the contract with meaningful funds.

## 5. A tiny example, explained line by line

This is a simplified token, not production code — it is here to show structure, not to be deployed:

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;

contract SimpleToken {
    // A mapping from address to balance: who owns how much.
    mapping(address => uint256) public balanceOf;

    // Total supply, set once at deployment.
    uint256 public totalSupply;

    // The constructor runs once, when the contract is deployed.
    constructor(uint256 initialSupply) {
        totalSupply = initialSupply;
        balanceOf[msg.sender] = initialSupply;
    }

    // Transfer tokens from the caller to another address.
    function transfer(address to, uint256 amount) public returns (bool) {
        require(balanceOf[msg.sender] >= amount, "Not enough balance");
        balanceOf[msg.sender] -= amount;
        balanceOf[to] += amount;
        return true;
    }
}
```

Reading it:

- `mapping` is a lookup table: address → balance.
- `msg.sender` is whoever called the function.
- `require` is a guard: if the condition fails, the whole transaction reverts and no state changes (gas is still spent).
- There is no `mint` function here, so supply cannot grow after deployment. In a real token, the presence or absence of `mint` is a crucial detail.

Real contracts are longer and more complex, but they are built from the same pieces: stored data, guards, and functions that move assets when conditions are met.

## 6. Upgradeable contracts and admin keys

"Code cannot be changed" is only true for immutable contracts. Many real contracts are **upgradeable**: a proxy contract points to a logic contract, and an admin can point the proxy at new logic.

Consequences:

- The team can fix bugs — good.
- The team can also change the rules, add a mint function, or drain funds — if the admin key is compromised or malicious.
- Check who controls upgrades: a single wallet, a multisig, a timelock (changes take effect only after a delay, giving users time to exit), or a DAO vote.

**Timelocks and multisigs are good signs.** A single, anonymous admin wallet with instant upgrade power is a major risk, and is a common feature of rug pulls.

## 7. Gas and failed transactions

- Every contract interaction costs gas, proportional to computation and storage.
- A transaction that **reverts** (fails a `require` check, for example) still costs gas for the work done before the failure.
- A transaction that runs out of gas also fails and still costs the gas supplied.
- Before confirming an interaction, your wallet usually shows an estimated gas fee. If the estimate is wildly higher than expected, stop and investigate — the contract may be doing something you did not expect, or the network may be congested.

## 8. Common beginner mistakes

| Mistake | Better approach |
|---|---|
| Interacting with an unverified contract | Only use verified contracts from official sources, for meaningful amounts |
| Ignoring owner and upgrade functions | Read the owner, admin, and upgrade controls before trusting a contract |
| Approving unlimited spending without reading | Approve limited amounts; revoke when done |
| Assuming "audited" means "safe" | Audits reduce known risks; they do not guarantee safety, and unaudited changes after the audit are not covered |
| Copying a contract address from a post or DM | Get addresses only from the project's official site |
| Assuming code equals intent | A contract does exactly what it is written to do, including bugs and backdoors the marketing never mentioned |

## 9. Tools for learning more

- **Remix:** a browser-based editor for writing and testing Solidity contracts on test networks, with no installation and no real funds.
- **Testnets:** free test networks (with free test coins from faucets) where you can deploy and break contracts safely.
- **OpenZeppelin Contracts:** widely used, audited building blocks for tokens, access control, and security patterns. Most serious projects build on these rather than writing everything from scratch.
- **Block explorers:** Etherscan, BscScan, and equivalents, for reading verified contracts and transactions.

Start on a testnet. Deploying to a mainnet with real funds before you understand gas, reverts, and access control is how beginners lose money to their own mistakes.

## 10. Frequently asked questions

**Are smart contracts legally binding?**
Sometimes, in some jurisdictions, under some conditions — but this is unsettled law in most places. Do not assume a smart contract gives you the same legal protections as a traditional contract.

**Can a smart contract be hacked if the blockchain is secure?**
Yes. Blockchain security protects the ledger; it does not guarantee that any individual contract's code is correct. Most crypto hacks exploit contract bugs, not the underlying chain.

**What is "immutable" vs "upgradeable"?**
Immutable means the code cannot change after deployment. Upgradeable means an admin can replace the logic. Both have trade-offs: immutability prevents tampering but also prevents bug fixes; upgradeability allows fixes but requires trusting the admin.

**Do I need to learn programming to use smart contracts?**
No, to use them. Yes, to evaluate them deeply. For using them, the explorer checks in section 4 plus the security habits in CryptoSecurityBasics cover most of what a beginner needs.

> Educational content only. This is not financial, legal, or investment advice.
