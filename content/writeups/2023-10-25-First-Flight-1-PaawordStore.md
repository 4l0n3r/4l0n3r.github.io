+++
title = "First Flight 1 : PasswordStore"
date = "2023-10-25"
+++

## [H1] Storing password on-chain makes it visible to anyone

**Description:**
The data that we stored at `PasswordStore:s_password` is intended to be private and only visible to the owner. since it was placed on chain, everything that we store on-chain is visible to everyone.

**Impact:**
Anyone can read the private data, breaking the protocol rule.

**Proof of Concept:**
1. Start the local chain
```shell
$ make anvil
```
2. Deploy the contract
```shell
$ make deploy
```
3. Get the storage slot data in which our password got stored
```shell
$ cast storage <contract_address> 1
```
4. Parse hex to string
```shell
$ cast parse-bytes32-string <Output form the above command>
```

**Recommended Mitigation:**
It would be a best practice to not store any private data on-chain since the chain is public to everyone. We should rethink about the architecture because we are going to eliminate the password from the chain. One think we could do is to, encrypt the password off-chain before we store it on-chain. this will lead us to store one key off-chain to decode the encrypted password.

<br><br>

---

## [H2] `PasswordStore::setPassword` has no access control, vulnerable to non-owner to change it.

**Description:**
The `PasswordStore::setPassword` function is intended to trigger only by the owner to change the `PasswordStore::s_password`. Since the function doesn't have any access control, it allows even a non-owner to change the password.

**Impact:**
Anyone can change the stored password, breaking the contract functional rule.

**Proof Of Concept:**
Let's write a fuzz test to see whether non-owner is able to change the password or not.

```bibtex
    function testAnyoneCanSetPassword(address nonOwner) external {
        string memory newPassword = "newPassword";
        vm.assume(nonOwner != owner );

        vm.prank(nonOwner);
        passwordStore.setPassword(newPassword);

        vm.prank(owner);
        string memory updatePassword = passwordStore.getPassword();

        assertEq(newPassword,updatePassword);
    }
```
passing this test tells us that a non-user can update the password.

**Mitigation:**
Add the below lines to make sure only owner can trigger the function.
```bibtex
    if (msg.sender != s_owner) {
        revert PasswordStore__NotOwner();
    }
```

<br><br>

---

## [L1] Initialization Timeframe Vulnerability

**Description:**
The PasswordStore contract exhibits an initialization timeframe vulnerability. This means that there is a period between contract deployment and the explicit call to setPassword during which the password remains in its default state. During this initialization timeframe, the contract's password is effectively empty and can be considered a security gap.

**Impact:**
The impact of this vulnerability is that during the initialization timeframe, the contract's password is left empty, potentially exposing the contract to unauthorized access or unintended behavior.

**Mitigation:**
To mitigate the initialization timeframe vulnerability, consider setting a password value during the contract's deployment

---

## [I1] Netspec indicates a parameter that doesn't exist, causing netspec incorrect.

**Description:**
The `PasswordStore::getPassword` function is not expecting any parameter where in the netSpec says it should be getPassword(string)

**Impact:**
False documentation making confusion for consumers

**Mitigation:**
Remove the unnecessary netSpec line.
