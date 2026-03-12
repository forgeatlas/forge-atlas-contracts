Executive Summary: Forge Atlas Smart Contracts

**Objective**

Implement a modular smart contract platform where:

The protocol layer is minimal and revenue-generating

Game economies are composed via explicit module deployment
No contract assumes another exists unless explicitly wired
Governance and tokenization are optional and introduced only after revenue exists

**Core Rules**

Protocol contracts never depend on game modules
Game modules may depend on protocol contracts
All value routing is explicit and configurable
Each ‘Layer’ Module is a separate Github repo, functional on its own. Important for audit and compliance

**Enforcement Rule**

Any constraint described in this document must be enforced at the smart contract level.
If a behavior is not explicitly allowed, it must be impossible on-chain.
Protocol contracts never mint NFTs directly. All NFT mint execution occurs through the Immutable Minting API.

**Layer 0 — Protocol**
Managed by Forge Atlas Team. Provides shared infrastructure only.

**0.1) ENERGY Token**
Protocol-issued settlement and liquidity unit used for protocol accounting, DEX pairing, and game developer settlement.
Backed by approved stablecoins held by the protocol treasury
Stablecoin treasury deploys collateral to yield venues (e.g. AAVE on Base)
Yield accrues to the protocol treasury only
ENERGY is a fully collateralized protocol settlement token minted via deposits of approved stablecoins and redeemable under protocol-defined constraints. Redemption can be paused or disabled at any time by the admin.
ENERGY is redeemable by any wallet, subject to a mandatory 48 hour timelock, per wallet withdrawal limits, and a rolling protocol wide redemption cap calculated as a percentage of total protocol TVL over each 48-hour window.
Redemptions that exceed per wallet limits may only be finalized to whitelisted destination wallets, and all redemptions, including those to whitelisted destinations, remain subject to protocol wide caps, rate limits, and pause controls.
ENERGY is issued on Immutable and is not bridgeable
ENERGY can be purchased via Crossmint SDK, Transak SDK

Constraints
Fully collateralized treasury principal
Yield is additive only (no rebasing)
No ENERGY staking or refinery
ENERGY is not a gameplay asset

**0.2) Protocol Admin & Routing**
Central configuration authority for protocol-level parameters and value routing.
Define protocolWallet
Define protocol fee percentages
Changes are timelocked 48 hours
Route:
Minted ENERGY yield → protocolWallet
DEX protocol fees → protocolWallet
Launchpad fees → protocolWallet
Expose read-only configuration to modules
No protocol routing function transfers value to end users; all protocol-level value flows route exclusively to the protocolWallet.


Modules may not override protocol routing rules.

**0.3) Native DEX (Uniswap v4)**
Shared DEX deployed once.
ENERGY-paired pools only
No pools by default
Dynamic fee support required (via v4 hooks)

Default Fee
0.30% total
80% → LPs
20% → protocol


Fee decay logic must be supported for fair launches in gameFactory.
Layer 1 — gameFactory
Permissionless. Each module is deployable separately and optional. Used by any team to deploy and manage independent game economies.

**1.1) Game Admin**
Per-game configuration:
Set teamWallet (team-controlled, multisig recommended)
Configure revenue routing:
teamWallet (100% default)
Additional destination addresses (optional, capped, percentage based; must sum to 100%)

Design constraints:
gameFactory is infrastructure only; it does not enforce economics or revenue splits
No implied yield, dividends, or profit guarantees
All routing parameters are defined by the deploying team

Revenue sources:
FMT chests (in-game Chest that grant access to mint or receive FMT; non-transferable)
Shop items (ENERGY / FMT; no fund custody by gameFactory)
LP fees (ENERGY ↔ govToken pairing; team-deployed, optional)

System boundary:
No platform-level pooling of user funds

**1.2) Free Mint Token (FMT)**
Game-scoped, non-transferable activity token. Free to mint via gameplay or in-game actions.
FMT is used to trigger configurable probabilistic in-game outcomes (probabilistic or deterministic) 
Default implementation uses the Forge Atlas hosted backend to issue signed attestations for outcome resolution and mint authorization (dev identity and rate limits). Teams may self host an equivalent signer.
FMT is burned on spend
Each FMT burn generates LUCK
Mint parameters are configurable per game (with defaults)

Team minting:
Team may mint FMT for operational or future module needs
All team minting is subject to a 48-hour timelock
Team minting is optional and can be disabled per game

Design constraints:
FMT has no external redemption
FMT is not transferable
FMT does not require shops, NFTs, governance, or liquidity

**1.3) Shop Module**
Optional module for selling game items. Supported item types:
Consumables
Collectibles
Equipment
Mystery boxes
Battle passes

Pricing:
Items are priced in FMT, ENERGY, or backend-minted
Optional recipient address for item delivery
Supports gifting and no-login purchase flows

Minting and delivery
When shop items produce NFTs, protocol contracts emit mint authorization events and mint execution must occur through the Immutable Minting API operated by the protocol backend
NFTs produced by shop purchases must use Immutable preset ERC721 or ERC1155 collections

Custody and transferability
Optionality: During restricted mint phases NFTs are minted to a protocol controlled vault contract instead of the user wallet
Vault custody prevents user trading until the protocol tradingEnabled flag is activated
After trading is enabled, users may withdraw NFTs from the vault to their wallet

Scarcity controls:
Global supply caps
Wallet allowlists
Per-address mint limits
Incremental or phased cap expansion
Non-transferable optionality


LUCK generation:
FMT spend generates LUCK
ENERGY spend generates LUCK
LUCK generation rates are configurable per game


Revenue handling:
Revenue is routed according to per-game admin configuration
Shop module does not custody user funds

**1.4) Multiplayer Module (FMT Only)**
Optional module for multiplayer match settlement using FMT. Flow
Players deposit FMT into on-chain escrow
Contract escrows FMT for the duration of the match
Forge Atlas Backend submits signed match outcome
Contract verifies signature and settles escrow

Rules:
FMT only (no ENERGY or external assets)
All FMT spent or settled is burned
FMT burns generate LUCK

Supported features:
Up to 16 players
Replay protection via nonces or match IDs
Timeouts with deterministic refunds
Optional team-funded bonus pools (FMT only)
AI participants (capped, explicitly identified)

**1.5) NFT Module **
Optional module for managing game-scoped NFT collections. Capabilities: 
Launch new NFT collections or migrate existing collections from other chains
Configure supply, metadata, and media references
NFTs are passive by default unless explicitly referenced by other modules
Collections are intended primarily for standalone or PFP style assets and may exist independently of the Shop module

Minting:
Mint pricing is set in FMT or ENERGY (never both)
Mint parameters are configurable per collection


Design constraints:
NFTs do not affect protocol or game economics unless explicitly wired
NFTs confer no implicit gameplay, financial, or governance effects
Staking for NFTs is not supported
Collection contracts must remain standard ERC721 or ERC1155 implementations deployed via Immutable Minting API to maintain marketplace compatibility

**1.6) govToken Launch & Liquidity**
Optional module for launching a per-game token with controlled initial liquidity.

Token deployment:
Optional per-game govToken deployment
Total supply defined and fully minted at deploy
Allocation recipients configurable at deploy:
teamWallet (required)
optional additional recipients (e.g. community rewards treasury, emissions escrow)

Liquidity initialization:
Liquidity pools are explicit deployments on the protocol DEX
ENERGY-paired liquidity only
Minimum token allocation for liquidity ( ≥5% of total supply)
Optional fair launch (up to 100% of supply allocated to liquidity)
Minimum ENERGY seeding requirement (configurable)

Liquidity properties:
Liquidity locked permanently (non-withdrawable)
Full-range liquidity by default

Launch protections:
High initial launch fee with time-based decay
Anti-sniper mechanics implemented via hooks (e.g. Uniswap v4)

Fees:
Factory fee (e.g. 1%) routed to protocol

Design constraints:
Token launch does not imply staking, yield, or revenue sharing
Liquidity provides market initialization only
No protocol or game revenue is routed to token holders by default
Downstream modules consume tokens via explicit deposits only

Distribution (post-deploy):
Token distribution to users is handled by optional Layer 2 modules.
Layer 2 — Advanced Economic Modules
Standalone deployments. Explicitly opt-in. Not required for game or token launch. 
Layer 2 modules extend game economies beyond launch primitives. 
All modules are independently deployable and consume assets only via explicit deposits or revenue routing. 
No Layer 2 module has mint authority over tokens.

**2.1) Airdrop & Claim Module**
Optional module for post-deploy token distribution.

Distribution methods:
Airdrop or claimdrop to a list of wallet addresses
Airdrop or claimdrop to NFT holders

Claim controls:
Claims may be required or optional
Claim windows configurable per distribution
NFTs may optionally be required to be transferred or burned to claim

Constraints:
Distribution uses pre-minted token supply only
No minting authority
Distribution logic is deterministic per configuration

**2.2) Activity Rewards Distribution Module (LUCK-Based)**
Optional module for distributing pre-minted token supply based on gameplay activity. 

Funding:
Module deployment is optional
Tokens must be explicitly deposited into the module
No minting authority

Eligibility:
Users who earn LUCK qualify for rewards
LUCK determines eligibility and ranking only
No random selection; all rewards are deterministic

Distribution logic:
Tier-based allocation defined per deployment
Rankings determine reward amounts within each tier
Epoch-based seasons with configurable duration (Epoch 1 - 72 Hours)
Developers control season and epoch configuration (Season 1 - 100 Epoch)

Claims:
Rewards must be claimed at the end of each season
No automatic airdrops
Claim window configurable (for example 1–30 epochs)
Unclaimed rewards are recycled within the module

Supply control:
Rewards are distributed from deposited token balances only
Emission rate is reduced by a configurable halving schedule
After a configured number of epochs, emission amounts are reduced by 50%
Additional tokens may be deposited explicitly by the team

Constraints:
No minting authority
No guaranteed rewards or returns
Distribution is activity-based and deterministic

**2.3) Buyback Module**
Optional module for market-based token recycling.
Funding:
Receives ENERGY via explicit deposits or revenue routing
No minting authority

Execution:
ENERGY is swapped for govToken using on-chain liquidity
Swap execution parameters configurable per deployment
Execution may be manual, scheduled, or threshold-based

Token handling:
Purchased tokens are routed to one or more destinations:
Activity Rewards Distribution module
Staking rewards pool
Burn address

Constraints:
No guaranteed buyback schedule or price support
No obligation to execute swaps
All operations are explicit and auditable


**2.4) Refinery Module (FMT Only)**
Optional module for applying risk-based transformations to FMT.
Operation:
Players deposit FMT into the Refinery
Contract applies a configurable VRF outcome resulting in more or less FMT
All logic operates exclusively on FMT

Rules:
FMT only
No LUCK is generated by Refinery interactions
Refinery does not mint FMT during operation

Liquidity:
Initial liquidity seeded at deploy via FMT mint (subject to timelock)
Liquidity pool permanently locked and non-withdrawable
Pool acts as a buffer for Refinery outcomes

Fees:
Adjustable fee (default configurable)
Fees accrue to the locked FMT pool

Constraints:
No ENERGY or other assets supported
No withdrawals from the liquidity pool
No implicit guarantees on outcomes

