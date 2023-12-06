# Shell Protocol

## Dev Provided Invariants

### Ocean.sol

A user's balances should only move with their permission
they are msg.sender
they've set approval for msg.sender
they are a contract that was the target of a ComputeInput/Output, and they did not revert the transaction


```
(msg.sender == user) || (isApprovedForAll(user, msg.sender)) || (isContract(user) && !revertedTransaction)
```

Where:

- msg.sender == user: The user is the one initiating the transaction.
- isApprovedForAll(user, msg.sender): The user has set approval for msg.sender.
- isContract(user) && !revertedTransaction: The user is a contract that was the target of a ComputeInputAmount or ComputeOutputAmount interaction and did not revert the transaction.  
    If none of these conditions are met, then ΔbalanceOf(user, token) should be zero, indicating no change in the user's balance:


**Fee Crediting Invariant:**
Fees should be credited to the Ocean owner's ERC-1155 balance

```solidity
balanceOf(owner, token)_{after} == balanceOf(owner, token)_{before} + feeCharged
```


**External Transfer Restriction Invariant:**
Calls to the Ocean cannot cause the Ocean to make external transfers unless a
doInteraction/doMultipleInteractions function is called and a wrap or unwrap interaction is provided.

```solidity
if (!(calledFunction == doInteraction || calledFunction == doMultipleInteractions)) {
    externalTransferAmount == 0
}
```

**Wrapped Token ID Calculation Invariant:**
The way the Ocean calculates wrapped token IDs is correct

```solidity
oceanId == deterministicFunction(externalContract, tokenId)
```

**Minting Restriction Invariant:**
Calls to the Ocean cannot cause it to mint a token without first calling the contract used to calculate its token ID.

```solidity
if (mintAmount > 0) {
    calledContract == externalContract
}
```

**Standards Conformance Invariant:**
The Ocean should conform to all standards that its code claims to (ERC-1155, ERC-165)

```solidity
supportsInterface(interfaceId) == true
```

**No Support for Rebasing or Fee on Transfer Tokens Invariant:**
The Ocean does not support rebasing tokens, fee on transfer tokens

```solidity
supportsRebasingTokens == false
supportsFeeOnTransferTokens == false
```

**Reentrancy Protection Invariant:**
The Ocean ERC-1155 transfer functions are secure and protected with reentrancy checks


**Accurate Balance Tracking Invariant:**
During any do* call, the Ocean accurately tracks balances of the tokens involved throughout the transaction.

```solidity
sum(balanceDeltas) == netTokenBalanceChange
```

**Balance Delta Invariant:**
The Ocean does not provide any guarantees against the underlying token blacklisting the Ocean or any sort of other non-standard behavior





## Other Invariants

**Token Normalization Invariant:**
For ERC-20 tokens wrapped into the Ocean, amounts are adjusted to or from 18 decimals during wrapping or unwrapping.

```solidity
wrapAmount = specifiedAmount * (10 ** (18 - d))
unwrapAmount = specifiedAmount / (10 ** (18 - d))
```

**NORMALIZED_DECIMALS and Wrapped ERC-20 Balances:**
The NORMALIZED_DECIMALS constant normalizes ERC-20 token balances to 18 decimals.

```solidity
normalizedBalance = actualBalance * (10 ** (NORMALIZED_DECIMALS - tokenDecimals))
```

**Unwrap Fee Invariant:**
The unwrap fee is calculated as a division by unwrapFeeDivisor, always rounded down, with a minimum fee of zero.

```solidity
feeCharged = unwrapAmount / unwrapFeeDivisor
```

**ERC-721 Amount Invariant:**
When wrapping or unwrapping ERC-721 tokens, the specifiedAmount must always be 1.

```solidity
specifiedAmount == 1
```

**Primitive Balance Invariant:**
The balance of a primitive contract for its registered tokens remains constant after computeOutputAmount or computeInputAmount interactions.

```solidity
balanceOf(primitive, registeredToken) == constant
```

**Ether Wrap and Unwrap Invariant:**
The amount of Ether wrapped or unwrapped must equal the msg.value provided in the transaction.

```solidity
wrappedEtherAmount == msg.value
unwrappedEtherAmount + feeCharged == msg.value
```

**Token Conservation Invariant:**
The total supply of any token within the Ocean contract must remain constant unless explicitly minted or burned.

```solidity
sum(balanceOf(allUsers, token)) == totalSupply(token)
```

**balanceOf and totalSupply:**
The sum of all users' balances for a token must equal the total supply of that token.

```solidity
sum(balanceOf(allUsers, token)) == totalSupply(token)
```

**\_mint and \_burn Functions and totalSupply:**
totalSupply of a token should increase by the amount minted and decrease by the amount burned.

```solidity
totalSupply(token)_{afterMint} == totalSupply(token)_{beforeMint} + mintAmount
totalSupply(token)_{afterBurn} == totalSupply(token)_{beforeBurn} - burnAmount
```

**Wrap and Unwrap Amount Invariant:**
When wrapping or unwrapping tokens, the input amount plus fees must equal the output amount.

```solidity
wrapAmount + feeCharged == unwrapAmount
```

**Loss of Funds:**
The contract should not lose funds due to rounding errors or incorrect calculations.

```solidity
(wrapAmount + feeCharged < unwrapAmount) || (wrapAmount > unwrapAmount + feeCharged)
```

**Ether Conservation Invariant:**
The contract's Ether balance must match the sum of wrapped Ether balances.

```solidity
contractEtherBalance == sum(balanceOf(allUsers, WRAPPED_ETHER_ID))
```

**Non-negative Balances Invariant:**
The balance of any token for any user must never be negative.

```solidity
balanceOf(user, token) >= 0
```
