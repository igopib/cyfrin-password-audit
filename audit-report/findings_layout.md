### [H-1] Password stored on-chain as storage can be accessed by anyone - passwords are not private.

**Description:** All data stored on blockchains can be retrieved by anyone. Data in any solidity visibility that is in a storage is never private. The `PasswordStore::s_password` variable is intended to be a private variable and only accessible by the `PasswordStore::s_owner` using `PasswordStore::getPassword` function.

Below is one such example to read data off chain.

**Impact:** Anyone can read the password of the owner, which completely breaks the functionality of the smart contract.

**Proof of Concept:**

1. Create a locally running chain

```bash
make anvil
```

2. Deploy the contract to the chain

```
make deploy
```

3. Run the storage tool

We use `1` because that's the storage slot of `s_password` in the contract.

```
cast storage <ADDRESS_HERE> 1 --rpc-url http://127.0.0.1:8545
```

You'll get an output that looks like this:

`0x6d7950617373776f726400000000000000000000000000000000000000000014`

You can then parse that hex to a string with:

```
cast parse-bytes32-string 0x6d7950617373776f726400000000000000000000000000000000000000000014
```

And get an output of:

```
myPassword
```

**Recommended Mitigation:**
ity: HIGH

### [H-2] `PasswordStore::getPassword` function does not have any access control, meaning any user could change the password.

**Description:** The function `PasswordStore::getPassword` is intended to be called only by the `PasswordStore::s_owner` user, but infact the function has no checks on who calls the function and could be called by anyone.

```javaScript
    function setPassword(string memory newPassword) external {
 @>     // @audit - There is no access control

        s_password = newPassword;
        emit SetNetPassword();
    }
```

**Impact:** The function is only supposed to be called by the owner, but in this case it severly breaks the logic of the protocol since anyone can call it up.

**Proof of Concept:** Add the following to the`PasswordStore.t.sol` test file

<details>

```javascript
    function test_everyone_can_set_password(address randomAddress) public {
        vm.assume(randomAddress != owner);
        vm.prank(randomAddress);
        string memory expectedPassword = "myNewPassword";
        passwordStore.setPassword(expectedPassword);

        vm.prank(owner);
        string memory actualPassword = passwordStore.getPassword();

        assertEq(actualPassword, expectedPassword);
    }
```

</details>

**Recommended Mitigation:** Following function will add access control to the function.

```javascript
         if (s_owner != msg.sender) {
             revert PasswordStore__NotOwner();
       }
```

# Low Risk Findings

## <a id='L-01'></a>L-01. Initialization Timeframe Vulnerability

_Submitted by [dianivanov](/profile/clo3cuadr0017mp08rvq00v4e)._

### Relevant GitHub Links

https://github.com/Cyfrin/2023-10-PasswordStore/blob/main/src/PasswordStore.sol

## Summary

The PasswordStore contract exhibits an initialization timeframe vulnerability. This means that there is a period between contract deployment and the explicit call to setPassword during which the password remains in its default state. It's essential to note that even after addressing this issue, the password's public visibility on the blockchain cannot be entirely mitigated, as blockchain data is inherently public as already stated in the "Storing password in blockchain" vulnerability.

## Vulnerability Details

The contract does not set the password during its construction (in the constructor). As a result, when the contract is initially deployed, the password remains uninitialized, taking on the default value for a string, which is an empty string.

During this initialization timeframe, the contract's password is effectively empty and can be considered a security gap.

## Impact

The impact of this vulnerability is that during the initialization timeframe, the contract's password is left empty, potentially exposing the contract to unauthorized access or unintended behavior.

## Tools Used

No tools used. It was discovered through manual inspection of the contract.

## Recommendations

To mitigate the initialization timeframe vulnerability, consider setting a password value during the contract's deployment (in the constructor). This initial value can be passed in the constructor parameters.

### [I-1] The `PasswordStore::getPassword` natspec indicates a parameter that doesn't exist, causing the natspec to be incorrect

**Description:**

```javascript
    /*
     * @notice This allows only the owner to retrieve the password.
@>   * @param newPassword The new password to set.
     */
    function getPassword() external view returns (string memory) {
```

The natspec for the function `PasswordStore::getPassword` indicates it should have a parameter with the signature `getPassword(string)`. However, the actual function signature is `getPassword()`.

**Impact:** The natspec is incorrect.

**Recommended Mitigation:** Remove the incorrect natspec line.

```diff
-     * @param newPassword The new password to set.
```
