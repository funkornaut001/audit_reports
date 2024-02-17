# Foundry DeFi Stablecoin CodeHawks Audit Contest - Findings Report

# Table of contents
- ## [Contest Summary](#contest-summary)
- ## [Results Summary](#results-summary)

- ## Medium Risk Findings
    - ### [M-01. Potential Function Failures due to Non-Standard ERC20 Tokens as Collateral](#M-01)
- ## Low Risk Findings
    - ### [L-01. Use of floating pragma](#L-01)
- ## Gas Optimizations / Informationals
    - ### [G-01. Misleading NatSpec for redeemCollateral function](#G-01)

# <a id='contest-summary'></a>Contest Summary

### Sponsor: Cyfrin

### Dates: Jul 23rd, 2023 - Aug 4th, 2023

[See more contest details here](https://www.codehawks.com/contests/cljx3b9390009liqwuedkn0m0)

# <a id='results-summary'></a>Results Summary

### Number of findings:
   - High: 0
   - Medium: 1
   - Low: 1
  - Gas/Info: 1


		
# Medium Risk Findings

## <a id='M-01'></a>M-01. Potential Function Failures due to Non-Standard ERC20 Tokens as Collateral            

### Relevant GitHub Links
	
https://github.com/Cyfrin/2023-07-foundry-defi-stablecoin/blob/d1c5501aa79320ca0aeaa73f47f0dbc88c7b77e2/src/DSCEngine.sol#L112

https://github.com/Cyfrin/2023-07-foundry-defi-stablecoin/blob/d1c5501aa79320ca0aeaa73f47f0dbc88c7b77e2/src/DSCEngine.sol#L157

https://github.com/Cyfrin/2023-07-foundry-defi-stablecoin/blob/d1c5501aa79320ca0aeaa73f47f0dbc88c7b77e2/src/DSCEngine.sol#L287

## Summary
The DSCEngine.sol contract may face functionality issues if it is deployed with certain ERC20 tokens as approved collateral. These tokens, which do not return a boolean on their transfer methods (e.g. USDT), will cause multiple functions in DSCEngine.sol to fail consistently.

## Vulnerability Details
During the contract deployment, there is no check in the constructor to ensure that the approved collateral tokens strictly adhere to the ERC20 standard. Consequently, it is possible to deploy the contract with tokens that do not return a boolean value on transfer methods, leading to subsequent failures in some of the contract's key functions.

## Impact
The functions `depositCollateral` and `_redeemCollateral` in the DSCEngine.sol contract will not operate as expected when dealing with ERC20 tokens that do not return a boolean on their transfer functions. This could significantly impair the contract's core functionality.

## Tools Used
Manual review

## Recommendations
Consider using the SafeERC20 library from Open Zeppelin and call safeTransfer or safeTransferFrom when transferring ERC20 tokens

# Low Risk Findings

## <a id='L-01'></a>L-01. Use of floating pragma            

### Relevant GitHub Links
	
https://github.com/Cyfrin/2023-07-foundry-defi-stablecoin/blob/d1c5501aa79320ca0aeaa73f47f0dbc88c7b77e2/src/DSCEngine.sol#L24

https://github.com/Cyfrin/2023-07-foundry-defi-stablecoin/blob/d1c5501aa79320ca0aeaa73f47f0dbc88c7b77e2/src/DecentralizedStableCoin.sol#L24

## Details & Impact
All .sol files in src & script are currently using a floating pragma, ^0.8.18, which allows for potential inconsistencies and vulnerabilities due to differences between Solidity compiler versions. It is possible these contracts get deployed with an outdated compiler version that might introduce bugs that negatively affect the stablecoin system.

## Tools Used
Manual review

## Recommendations
Contracts should be deployed with the same compiler version they were tested with. Fix all pragmas to 0.8.19.

# Gas Optimizations / Informationals

## <a id='G/I-01'></a>G/I-01. Misleading NatSpec for redeemCollateral function            

### Relevant GitHub Links
	
https://github.com/Cyfrin/2023-07-foundry-defi-stablecoin/blob/main/src/DSCEngine.sol#L181

## Summary
This comment on the redeemCollateral function is misleading
`* @notice If you have DSC minted, you will not be able to redeem until you burn your DSC`

## Vulnerability Details
The `redeemCollateral` function does not directly require the user to burn DSC to redeem their collateral. Instead, it checks whether the operation would break the health factor. In cases where a user has a high collateralization ratio, they may redeem some of their collateral without burning DSC while keeping their health factor above the threshold. Therefore, the NatSpec comment may inaccurately represent the `redeemCollateral` functionality under certain conditions.

PoC: Add this test to `DSCEngineTest.t.sol` and it passes 
```
    function testCanRedeemCollateralWithSomeDSCMintedAndNotBurnDSC() public {
        //user deposits a large amout of weth and mints a small amount of dsc
        vm.startPrank(user);
        ERC20Mock(weth).approve(address(dsce), 1000);
        dsce.depositCollateralAndMintDsc(weth, 1000, 1);
        //user redeems some collateral without burning any dsc
        dsce.redeemCollateral(weth, 10);
        vm.stopPrank;
    }
```

## Impact
This comment can lead users and auditors to misunderstand how the function works.

## Tools Used
Manual Review

## Recommendations
Remove the NatSpec line or further clarify that the `redeemCollateral` function may revert if the user has too much DSC minted and will need to burn DSC before calling the function again.
