# First Flight #2: Puppy Raffle - Findings Report

# Table of contents
- ## [Contest Summary](#contest-summary)
- ## [Results Summary](#results-summary)
- ## High Risk Findings
    - ### [H-01. Reentrancy In Refund Will Drain Funds](#H-01)
    - ### [H-02. NFTs and ETH Can Be Bruned](#H-02)
    - ### [H-03. `refund` steals funds](#H-03)
- ## Medium Risk Findings
    - ### [M-01. `enterRaffle` function will run out of gas](#M-01)
    - ### [M-02. Outdated Pragma](#M-02)
- ## Low Risk Findings
    - ### [L-01. Pragma Is Not Specified Correctly](#L-01)


# <a id='contest-summary'></a>Contest Summary

### Sponsor: First Flight #2

### Dates: Oct 25th, 2023 - Nov 1st, 2023

[See more contest details here](https://www.codehawks.com/contests/clo383y5c000jjx087qrkbrj8)

# <a id='results-summary'></a>Results Summary

### Number of findings:
   - High: 3
   - Medium: 2
   - Low: 1


# High Risk Findings

## <a id='H-01'></a>H-01. Reentrancy In Refund Will Drain Funds            

### Relevant GitHub Links
	
https://github.com/Cyfrin/2023-10-Puppy-Raffle/blob/main/src/PuppyRaffle.sol#L96-#L104

## Summary
The `refund` function can be exploited by a Reentrancy attack and drain all the ETH from the Raffle.

## Vulnerability Details
The `refund` function resets the `players[playerIndex]` AFTER it sends ether to the msg.sender. This opens up a reentrancy attack vector. Assuming msg.sender a smart contract and has entered the raffle and knows it's index within the players array. The contract can use a `recieve` function that calls `refund` it is allowed to reenter the function and receive extra ether. They can repeat this until they have drain the raffle of all ether.
 
```javascript
function refund(uint256 playerIndex) public {
        address playerAddress = players[playerIndex];
        require(playerAddress == msg.sender, "PuppyRaffle: Only the player can refund");
        require(playerAddress != address(0), "PuppyRaffle: Player already refunded, or is not active");

        payable(msg.sender).sendValue(entranceFee);

        players[playerIndex] = address(0);
        emit RaffleRefunded(playerAddress);
    }
```
## Impact
Loss of all ether locked in the `PuppyRaffle` contract

## Tools Used
Manual Review

## Recommendations
Use the Checks, Effects, Interactions flow with functions that send ether. For this function move the `players[playerIndex] = address(0);` before the line `payable(msg.sender).sendValue(entranceFee);`

```diff
function refund(uint256 playerIndex) public {
        address playerAddress = players[playerIndex];
        require(playerAddress == msg.sender, "PuppyRaffle: Only the player can refund");
        require(playerAddress != address(0), "PuppyRaffle: Player already refunded, or is not active");

      + players[playerIndex] = address(0);
        payable(msg.sender).sendValue(entranceFee);

      - players[playerIndex] = address(0);
        emit RaffleRefunded(playerAddress);
    }
```
## <a id='H-02'></a>H-02. NFTs and ETH Can Be Bruned            

### Relevant GitHub Links
	
https://github.com/Cyfrin/2023-10-Puppy-Raffle/blob/main/src/PuppyRaffle.sol#L103

## Summary
NFTs and ETH can be burned if player calls refund

## Vulnerability Details
When a player calls `refund` the address at their index in the `players` array is set to `address(0)`. If the winner index happens to be address(0) the winning funds and NFT will be burned and lost forever

## Impact
Loss of NFTs and ETH

## Tools Used
Manual Review

## Recommendations
When deleting something from an array it is best to shift the index you want to delete to the end of the array and use .pop to remove it.
## <a id='H-03'></a>H-03. `refund` steals funds            

### Relevant GitHub Links
	
https://github.com/Cyfrin/2023-10-Puppy-Raffle/blob/main/src/PuppyRaffle.sol#L96-#L98

## Summary
The `refund` function sends the `entranceFee` to the player at the index of the array and if they are the message sender. However that player may not be the one who bought in.
```
function refund(uint256 playerIndex) public {
        address playerAddress = players[playerIndex];
        require(playerAddress == msg.sender, "PuppyRaffle: Only the player can refund");
        require(playerAddress != address(0), "PuppyRaffle: Player already refunded, or is not active");

        payable(msg.sender).sendValue(entranceFee);
```

## Vulnerability Details
To enter a raffle an array of addresses is given and the entry fee is paid for by the `msg.sender` not necessarly by the player who calls for the `refund`. A player who did not pay for the `entranceFee` can calim a refund and basically steal the funds from the raffle and will likey hurt the feelings of the player who bought them their entrance to the raffle. On the flip side, someone who paid for multiple entries will only get a partial refund from calling this function. 
 
## Impact
Loss of funds from the raffle pool and hurt feelings.

## Tools Used
Manual Review

## Recommendations
Track the address that paid for a player's entrance and refund that player if a player wishes to leave the raffle.
		
# Medium Risk Findings

## <a id='M-01'></a>M-01. `enterRaffle` function will run out of gas            

### Relevant GitHub Links
	
https://github.com/Cyfrin/2023-10-Puppy-Raffle/blob/main/src/PuppyRaffle.sol#L79-#L89

## Summary
The `enterRaffle` function itterates over arrays three times and will revert if they become too long.

## Vulnerability Details
Looping through arrays is gas intensive. The `enterRaffle` function does this three times. Once over the array passed by the user and twice over the `player` array to check for duplicates. 
```solidity
function enterRaffle(address[] memory newPlayers) public payable {
        require(msg.value == entranceFee * newPlayers.length, "PuppyRaffle: Must send enough to enter raffle");
        for (uint256 i = 0; i < newPlayers.length; i++) {
            players.push(newPlayers[i]);
        }

        // Check for duplicates
        for (uint256 i = 0; i < players.length - 1; i++) {
            for (uint256 j = i + 1; j < players.length; j++) {
                require(players[i] != players[j], "PuppyRaffle: Duplicate player");
            }
        }
```
 
## Impact
If either of these arrays is or becomes too large it will run out of gas. If a user passed an array of address who wishes to enter and it is too long they will have to split it into multiple function calls. But then when the `players` array becomes too long no new players will be able to enter the raffle

## Tools Used
Manual Review

## Recommendations
Only allow players to enter themselves into the raffle and track their entry with a bool. This would also disallow users from registering unwilling accounts into the raffle.
## <a id='M-02'></a>M-02. Outdated Pragma            

### Relevant GitHub Links
	
https://github.com/Cyfrin/2023-10-Puppy-Raffle/blob/main/src/PuppyRaffle.sol#L2

## Summary
Puppy Raffle uses an outdated Solidity pragma which can lean to mathmatical errors

## Vulnerability Details
The Puppy Raffle contract uses `pragma solidity ^0.7.6;` whihch is outdated. 

## Impact
The major issue is that this version of Solidity does not check for arithmatic over/underflows at the language level and can cause issues with mathmatical operations

## Tools Used
Manual Review

## Recommendations
Use at least pragma 0.8.0 but it is best to use a version closer to the most recently updated

# Low Risk Findings

## <a id='L-01'></a>L-01. Pragma Is Not Specified Correctly            

### Relevant GitHub Links
	
https://github.com/Cyfrin/2023-10-Puppy-Raffle/blob/main/src/PuppyRaffle.sol#L2

## Summary
The Puppy Raffle contract uses a floating pragma

## Vulnerability Details
Using floating pragmas can lead to damaged or non fuctional contracts

## Impact
Floating pragmas allow for potential inconsistencies and vulnerabilities due to differences between Solidity compiler versions. It is possible these contracts get deployed with an outdated compiler version, or on a diffrent chain, which could introduce bugs that negatively affect the Puppy Raffle.

## Tools Used
Manual Review

## Recommendations
Fix the solidity pragma to what was ued in testing the contract.


