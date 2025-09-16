# 📘 Bonding Launch Factory — Documentation

## 1. Overview

The `BondingLaunchFactory` is a smart contract system that enables **permissionless token launches** with a built-in **bonding curve** and **final Uniswap V2 listing**.
It consists of:

* **LaunchToken** — ERC-20 token deployed for each project.
* **LaunchPool** — bonding curve pool that manages buys, sells, and final migration to Uniswap LP.
* **BondingLaunchFactory** — factory contract that deploys tokens and pools, collects fees, and tracks launches.

This system removes the need for manual liquidity setup and guarantees a fair, curve-driven initial distribution.

---

## 2. Key Concepts

### ⚖️ Bonding

* **Bonding** means all trades (buys/sells) happen against a **constant product AMM (CPAMM)** pool owned by the factory.
* Users supply the **reserve token** (e.g., WETH, RBAT) to buy the launch token.
* The pool charges a **1% fee** on each trade, routed to the **treasury**.
* Sellers can exit by swapping back their launch tokens for reserve tokens.

### 🎯 Target Market Cap

* Each launch sets a **target market cap (in reserve token terms)**.
* While the pool is below this cap, trades occur on the bonding curve.
* Once the cap is reached, the pool is **finalized** → all liquidity is added to Uniswap V2.

### 🪙 Listing on Uniswap

* When finalized, **all reserves and tokens in the pool** are sent to the Uniswap V2 router.
* LP tokens are minted to the specified **LP recipient** (for your frontend, this defaults to a **dead wallet** for renounced LP).
* After this, trading happens on Uniswap, not the bonding curve.

---

## 3. Fees & Costs

| Fee Type        | Amount                          | Destination | Purpose                         |
| --------------- | ------------------------------- | ----------- | ------------------------------- |
| **Deploy Fee**  | `?? ETH` (\~\$20 ) | Treasury    | Required to create a token/pool |
| **Trading Fee** | `1%`                            | Treasury    | Applied on every buy/sell       |
| **Gas Fees**    | Varies                          | Validators  | Normal Ethereum gas             |

**Notes:**

* Deploy fee is configurable by factory owner.
* Deploy fee is refunded if excess ETH is sent.
* Treasury address: `0xedeb5bf895eb0315cc65cb31a84ffd92dc06e854`.

---

## 4. Launching a Token (Frontend Flow)

### Step-by-Step

1. **Open Launch Page**
   → Connect wallet via AppKit.
   → Review deploy cost (20 USD equivalent).

2. **Fill Launch Form**

   * Token Name: e.g., "Example Token"
   * Symbol: e.g., "EXM"
   * Initial Supply: amount to mint to pool
   * Initial Reserve: amount of reserve token (WETH/RBAT) to back pool
   * Target Market Cap: threshold at which Uniswap LP is created
   * LP Recipient: auto-set to **dead wallet** (for renounced LP ownership)

3. **Pay Deployment Fee**
   → 1 ETH (deployFeeWei) is sent to treasury.

4. **Factory Actions (on launch)**

   * Deploys new ERC-20 (`LaunchToken`)
   * Deploys new bonding pool (`LaunchPool`)
   * Mints tokens to the pool
   * Transfers reserve tokens to the pool
   * Indexes new launch in factory storage

5. **Frontend Confirmation**
   → Shows deployed token contract + bonding curve dashboard.

---

## 5. What Happens After Launch

### While Bonding

* Users can buy tokens directly from the bonding pool.
* Price rises automatically as reserves grow.
* Fees (1%) are deducted each trade and routed to treasury.

### On Finalization

* If target market cap is reached:

  * Pool pauses.
  * All reserves and tokens migrate to Uniswap.
  * LP tokens minted to **dead wallet** (ensuring no rugpull).
  * Trading continues on Uniswap.

---

## 6. Metadata & Frontend Display

* After launch, **creators can set metadata** for their token:

  * `logoURI` → token logo
  * `description` → project summary
  * `telegram` → social link
  * `twitter` → social link
* Metadata is stored in `BondingLaunchFactory.tokenMeta`.

**Frontend Tabs (Token Details Page):**

1. **Overview** — logo, name, symbol, description, socials.
2. **Chart** — bonding curve price & reserves (live with Multicall3).
3. **Buy** — bonding buy widget.
4. **Transactions** — pool activity.
5. **If Listed** — LP stats from Uniswap (future API).

---

## 7. Differences from Traditional Launches

| Traditional Token Launch           | Bonding Launch                                     |
| ---------------------------------- | -------------------------------------------------- |
| Token deployed manually            | One-click deploy via factory                       |
| Liquidity manually added to DEX    | Liquidity auto-migrated once target cap is reached |
| No guaranteed fair price discovery | Bonding curve ensures gradual price discovery      |
| Liquidity may be controlled by dev | LP sent to **dead wallet**, trustless              |
| Rugpull risk                       | Zero LP control, LP permanently locked             |

This algorithm ensures **fair entry**, **transparent bonding**, and **trustless listing**.

---

## 8. Developer Notes

* **Reserve Token**: `0x0C6eF4f55f315C524C590572625d733491DC0921`
* **Treasury**: `0xedeb5bf895eb0315cc65cb31a84ffd92dc06e854`
* **Uniswap Router**: `0x83641dBab18AF4cd14ac23F6257f3269a5693204`
* **Deploy Fee**: `1e18 wei` (approx ETH of  ≈ 20 USD )

### Frontend Integration

* Use `launchesLength()` and `launches(id)` to fetch launches.
* Use `poolForToken(token)` to fetch pool address.
* Use `tokenMeta(token)` to fetch metadata.
* Use Multicall3 to read:

  * `LaunchPool.reserves()`
  * `LaunchPool.currentMarketCap()`
  * `LaunchPool.listed()`

---

✅ **Summary**:
The Bonding Launch Factory creates a **trustless, fair-launch environment** where tokens are sold on a bonding curve until a target market cap is reached. At that point, liquidity is migrated to Uniswap and LP is permanently locked in a dead wallet. Costs are minimal (≈ \$20 deploy fee + gas + 1% trading fee), and the frontend provides a professional DEX-like token details page with metadata, charts, and buy/sell functionality.

---
