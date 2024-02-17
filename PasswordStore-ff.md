# First Flight #1: PasswordStore - Findings Report

# Table of contents
- ## [Contest Summary](#contest-summary)
- ## [Results Summary](#results-summary)
- ## High Risk Findings
    - ### [H-01. Passwords Can Be Read By Anyone](#H-01)
    - ### [H-02. Missing Access Control](#H-02)

- ## Low Risk Findings
    - ### [L-01. Typo in event name](#L-01)


# <a id='contest-summary'></a>Contest Summary

### Sponsor: First Flight #1

### Dates: Oct 18th, 2023 - Oct 25th, 2023

[See more contest details here](https://www.codehawks.com/contests/clnuo221v0001l50aomgo4nyn)

# <a id='results-summary'></a>Results Summary

### Number of findings:
   - High: 2
   - Medium: 0
   - Low: 1


# High Risk Findings

## <a id='H-01'></a>H-01. Passwords Can Be Read By Anyone            

### Relevant GitHub Links
	
https://github.com/Cyfrin/2023-10-PasswordStore/blob/main/src/PasswordStore.sol#L14

## Summary
Storing sensitive information directly on-chain poses significant privacy and security risks. 

## Vulnerability Details
The `PasswordStore` contract saves passwords, `s_password`, in a storage slot within the contract. This data is stored on-chain, essentially making it public information. Tagging the passwords as `private` only prevents other smart contracts from viewing the `s_password` value, not the rest of the world.

## Impact
Passwords stored on-chian are not secret and can be read and used by anyone.

## Tools Used
Manual Review

## Recommendations
Salt & Hash. Do not store plain text passwords on-chain. Instead create a unique salt for each password. Hash the salt and the password, store and use that result to verify passwords.
## <a id='H-02'></a>H-02. Missing Access Control            

### Relevant GitHub Links
	
https://github.com/Cyfrin/2023-10-PasswordStore/blob/main/src/PasswordStore.sol#L23-#L28

## Summary
Critical `setPassword` Function Lacks Access Control

## Vulnerability Details
Per the developer's notes, the `setPassword` function is intended to be callable only by the owner. However, this function currently lacks any access control checks, making it callable by anyone.
 
## Impact
Anyone can change the password.

## Tools Used
Manual review

## Recommendations
1. Create an `onlyOwner` modifier and use it on functions that should be callable only by the owner. 
```
modifier onlyOwner() {
     if (owner() != _msgSender()) {
          revert PasswordStore__NotOwner();
   }
   _;
}
```

2. Consider utilizing OpenZeppelin's `Ownable` contract, which already provides the `onlyOwner` modifier.

3. Use the same logic in the `getPassword` function within the `setPassword` function.
```
if (msg.sender != s_owner) {
            revert PasswordStore__NotOwner();
        }
```

		


# Low Risk Findings

## <a id='L-01'></a>L-01. Typo in event name            

### Relevant GitHub Links
	
https://github.com/Cyfrin/2023-10-PasswordStore/blob/main/src/PasswordStore.sol#L16

## Summary
Typo Of Event Name

## Vulnerability Details
The event `SetNetPassword` should likely be `SetNewPassword`.

## Impact
Developer confusion

## Tools Used
Manual review

## Recommendations
Change `SetNetPassword` to `SetNewPassword`


