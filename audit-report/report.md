---
title: PasswordStore Audit Report
author: Gopinho
date: Dec 11, 2023
header-includes:
  - \usepackage{titling}
  - \usepackage{graphicx}
---

\begin{titlepage}
\centering
\begin{figure}[h]
\centering
\includegraphics[width=0.5\textwidth]{logo.pdf}
\end{figure}
\vspace\*{2cm}
{\Huge\bfseries Protocol Audit Report\par}
\vspace{1cm}
{\Large Version 1.0\par}
\vspace{2cm}
{\Large\itshape Cyfrin.io\par}
\vfill
{\large \today\par}
\end{titlepage}

\maketitle

<!-- Your report starts here! -->

Prepared by: [Gurpreet](https://profileos.vercel.app)
Lead Researcher:

- Gurpreet

# Table of Contents

- [Table of Contents](#table-of-contents)
- [Protocol Summary](#protocol-summary)
- [Disclaimer](#disclaimer)
- [Risk Classification](#risk-classification)
- [Audit Details](#audit-details)
  - [Scope](#scope)
  - [Roles](#roles)
- [Executive Summary](#executive-summary)
  - [Issues found](#issues-found)
- [Findings](#findings)
  - [High](#high)
    - [\[H-1\] Password stored on-chain as storage can be accessed by anyone - passwords are not private.](#h-1-password-stored-on-chain-as-storage-can-be-accessed-by-anyone---passwords-are-not-private)
    - [\[H-2\] `PasswordStore::getPassword` function does not have any access control, meaning any user could change the password.](#h-2-passwordstoregetpassword-function-does-not-have-any-access-control-meaning-any-user-could-change-the-password)
  - [Informational](#informational)
    - [\[I-1\] The `PasswordStore::getPassword` natspec indicates a parameter that doesn't exist, causing the natspec to be incorrect](#i-1-the-passwordstoregetpassword-natspec-indicates-a-parameter-that-doesnt-exist-causing-the-natspec-to-be-incorrect)

# Protocol Summary

Protocol stores owners passwords for them.

# Disclaimer

The Gurpreet(gopinho) team makes all effort to find as many vulnerabilities in the code in the given time period, but holds no responsibilities for the findings provided in this document. A security audit by the team is not an endorsement of the underlying business or product. The audit was time-boxed and the review of the code was solely on the security aspects of the Solidity implementation of the contracts.

# Risk Classification

|            |        | Impact |        |     |
| ---------- | ------ | ------ | ------ | --- |
|            |        | High   | Medium | Low |
|            | High   | H      | H/M    | M   |
| Likelihood | Medium | H/M    | M      | M/L |
|            | Low    | M      | M/L    | L   |

We use the [CodeHawks](https://docs.codehawks.com/hawks-auditors/how-to-evaluate-a-finding-severity) severity matrix to determine severity. See the documentation for more details.

# Audit Details

**The findings in this documents corrosponds to the following commit hash**

```
2e8f81e263b3a9d18fab4fb5c46805ffc10a9990
```

## Scope

```
./src/
#-- PasswordStore.sol
```

## Roles

- Owner: The user who can set the password and read the password.
- Outsiders: No one else should be able to see the password.

# Executive Summary

Manual review including foundry fuzz tests were expended on this contract.

## Issues found

| Severity | Number of issues |
| -------- | ---------------- |
| High     | 2                |
| Medium   | 0                |
| Low      | 0                |
| Info     | 1                |
| Total    | 3                |

# Findings

## High

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

## Informational

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
