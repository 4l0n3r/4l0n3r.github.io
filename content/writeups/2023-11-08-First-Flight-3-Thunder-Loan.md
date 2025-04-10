+++
title = "First Flight 3 : Thunder Loan"
date = "2023-10-25"
+++

# High

## [H-1] Storage Collision Due to Variable Order Mismatch

**Description:**
In the `ThunderLoan.sol` contract, the storage variables are declared in the following order:

```solidity
uint256 private s_feePrecision;  
uint256 private s_flashLoanFee; // 0.3% ETH fee
```

However, in the upgraded `ThunderLoanUpgraded.sol` contract, the order is changed:

```solidity
uint256 private s_flashLoanFee; // 0.3% ETH fee  
uint256 public constant FEE_PRECISION = 1e18;
```

Since Solidity relies on the order of variable declaration to assign storage slots, changing the order during an upgrade causes the `s_flashLoanFee` to incorrectly take on the value of `s_feePrecision`. Additionally, the `s_currentlyFlashLoaning` mapping will point to an incorrect storage location, leading to undefined behavior.

### **Impact:**
After upgrading:
- **Incorrect Fees:** The `s_flashLoanFee` will be set to the previous `s_feePrecision` value, leading to users being charged incorrect flash loan fees.
- **Broken Mapping:** The `s_currentlyFlashLoaning` mapping will be misaligned, potentially allowing users to bypass restrictions or causing faulty state tracking.

### **Proof of Code:**
The storage layout difference can be verified by running:

```bash
forge inspect ThunderLoan storage  
forge inspect ThunderLoanUpgraded storage
```

## [H-2] Incorrect `updateExchangeRate` Call in `deposit` Function

**Description:**

In the `deposit` function of the `ThunderLoan` contract, the `updateExchangeRate` function is unnecessarily called with `calculatedFee` immediately after minting.

```solidity
function deposit(IERC20 token, uint256 amount) external revertIfZero(amount) revertIfNotAllowedToken(token) {
    AssetToken assetToken = s_tokenToAssetToken[token];
    uint256 exchangeRate = assetToken.getExchangeRate();
    uint256 mintAmount = (amount * assetToken.EXCHANGE_RATE_PRECISION()) / exchangeRate;

    emit Deposit(msg.sender, token, amount);
    assetToken.mint(msg.sender, mintAmount);

    uint256 calculatedFee = getCalculatedFee(token, amount);
    assetToken.updateExchangeRate(calculatedFee);

    token.safeTransferFrom(msg.sender, address(assetToken), amount);
}
```

The call to `updateExchangeRate` happens **before** the transferred amount is reflected in the asset balance. This leads to incorrect updates of the exchange rate.

**Impact:**

- **Incorrect Withdrawals:** Users may be unable to withdraw correct amounts due to the miscalculated exchange rate.
- **Unfair Reward Distribution:** The updated exchange rate can cause unfair distribution of rewards, as the value is changed without accounting for the actual token balance.


## [H-3] Flashloan and Deposit Exploit Allows Fund Theft

**Description:**  
A critical vulnerability exists where users can exploit the system by calling `ThunderLoan::deposit` instead of `ThunderLoan::repay` after taking a flashloan. This allows malicious users to bypass proper repayment and drain all funds from the protocol.

**Impact:**
- Malicious actors can steal all available funds from the protocol by manipulating the flashloan and deposit mechanism.
- This could result in a complete loss of user deposits and system insolvency.

**Recommended Mitigation:**
- Enforce strict checks to ensure that any outstanding flashloan must be repaid using `ThunderLoan::repay`.
- Implement tracking to verify that all flashloaned funds are returned before allowing new deposits.

---

# Medium

## [M-1] Vulnerability in TSwap Price Oracle Allows Price Manipulation Attacks

**Description:**  
The ThunderLoan protocol relies on the TSwap AMM (Automated Market Maker) for price calculations. TSwap uses a constant product formula to determine token prices based on the pool's reserves. This mechanism is vulnerable to manipulation by users who perform large token swaps within a single transaction, bypassing protocol fees.

**Impact:**  
Malicious users can manipulate token prices during a flash loan, leading to inaccurate pricing. This results in reduced fees for liquidity providers and potential financial loss for the protocol.

**Proof of Concept:**
Within a single transaction, a user can exploit the price calculation as follows:

1. The user initiates a flash loan from ThunderLoan for 1000 tokenA, paying the standard fee (`fee1`).
2. During the flash loan, the user sells 1000 tokenA, significantly reducing its price in the TSwap pool.
3. Instead of repaying the flash loan immediately, the user takes a second flash loan for 1000 tokenA at a much lower price due to the manipulated price calculation:

```solidity
function getPriceInWeth(address token) public view returns (uint256) {
    address swapPoolOfToken = IPoolFactory(s_poolFactory).getPool(token);
    return ITSwapPool(swapPoolOfToken).getPriceOfOnePoolTokenInWeth();
}
```

4. The user repays the first flash loan and subsequently repays the second one at a lower cost.

A proof of concept exists in the audit-data folder, demonstrating this exploit.

**Recommended Mitigation:**
To prevent price manipulation, replace the TSwap-based price calculation with a more robust oracle system. Consider using Chainlink price feeds as the primary oracle, with a Uniswap TWAP (Time-Weighted Average Price) fallback mechanism for enhanced security and accuracy.