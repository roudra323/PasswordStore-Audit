### [H-1] Storing the password on-chain makes it visible to anyone & no longer private

**Description:** All data stored on-chain is visible to anyone, and can be read directly from the blockchain. The `PasswordStore::s_password` is intended to be a private variable and only accessed through the `PasswordStore::getPassword()` function, which is intended to be only called by the owner of the contract.

We show one such method of reading any data off chain below.

**Impact:** Anyone can read the private password, severely breaking the functionality of the protocol.

**Proof of Concept:** (Proof Of Code)

The below test case shows that anyone can read the password directly from the blockchain.

1) Start local anvil blockchain

```shell
make anvil
```
2) Deploy the contract with the password "myPassword"

```shell
make deploy
```
3) print the bytes32 representation which is in memory slot 1 of the password from the contract address
```shell
cast storage 0x5FbDB2315678afecb367f032d93F642f64180aa3 1 --rpc-url http://127.0.0.1:8545
```
4) Convert the bytes32 representation to string
```shell
cast parse-bytes32-string 0x6d7950617373776f726400000000000000000000000000000000000000000014
```


**Recommended Mitigation:** 
 Consider redesigning the contract architecture. One approach could involve encrypting the password off-chain and storing the encrypted version on-chain. Users would need to manage another password off-chain to decrypt the on-chain data. Additionally, removing the view function may prevent accidental exposure of sensitive information.

### [H-2] `PasswordStore::setPassword()` has no access control, meaning a non-owner can overwrite the password

**Description:** The `PasswordStore::setPassword()` function is declared as `external`, but it lacks access control, contrary to the intended purpose of allowing only the owner to set the password.

```solidity
    function setPassword(string memory newPassword) external {
@>    // @audit any user can set a password
        s_password = newPassword;
        emit SetNetPassword();
    }
```

**Impact:** Any user can change the password, compromising the protocol's security.

**Proof of Concept:** Include the following test in the `PasswordStore.t.sol` test file:
<details>
<summary>Code</summary>

```solidity
    function test_anyone_can_set_password(address randomAddress) public {
        vm.assume(owner != randomAddress);
        vm.prank(randomAddress);
        string memory expectedPassword = "myNewPassword";
        passwordStore.setPassword(expectedPassword);

        vm.prank(owner);
        string memory actualPassword = passwordStore.getPassword();
        assertEq(actualPassword, expectedPassword);
    }
```

</details>

**Recommended Mitigation:** Implement access control in the `setPassword()` function to ensure only the contract owner can change the password.

### [I-1] The `PasswordStore::getPassword` natspec indicates a parameter that doesn't exist, causing the natspec to be incorrect

**Description:** The NatSpec documentation for the `PasswordStore::getPassword` function incorrectly mentions a parameter `newPassword`, which does not exist in the function signature.
```javascript
/*
     * @notice This allows only the owner to retrieve the password.
     * @param newPassword The new password to set.
     */
    function getPassword() external view returns (string memory) 
```

**Impact:** The natspec is incorrect and could cause confusion for developers.

**Recommended Mitigation:** Remove the `newPassword` parameter from the natspec.

```diff
- * @param newPassword The new password to set.
```