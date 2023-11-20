# Slither True and False Positives

[X] Run Slither on the codebases you created in the last two weeks

[X] Document true and false positives that you discovered with the tools

# Section 1 - Bonding Curves and Advanced Token Protocols

## Token with Sanctions

### True Positives
ERC20 Transfer Return Value Check:  
- https://github.com/crytic/slither/wiki/Detector-Documentation#unchecked-transfer

1. `SanctionsTest.setUp() (test/SanctionedTokenTest.t.sol#18-28) ignores return value by sanctionedToken.transfer(bob,BOB_STARTING_AMOUNT) (test/SanctionedTokenTest.t.sol#27)`
2. `SanctionsTest.test_BannedSenderTransferReverts() (test/SanctionedTokenTest.t.sol#56-62) ignores return value by sanctionedToken.transfer(alice,1000000000000000000) (test/SanctionedTokenTest.t.sol#61)`
3. `SanctionsTest.test_BannedReceiverTransferReverts() (test/SanctionedTokenTest.t.sol#64-70) ignores return value by sanctionedToken.transfer(alice,1000000000000000000) (test/SanctionedTokenTest.t.sol#69)`

### False Positives
N/A

### Non-Issues
Conformance to Solidity Naming Conventions:
- https://github.com/crytic/slither/wiki/Detector-Documentation#conformance-to-solidity-naming-conventions

1. Recommending earlier solidity versions: 
- `Pragma version0.8.21 (script/DeploySanctionedToken.s.sol#2) necessitates a version too recent to be trusted. Consider deploying with 0.6.12/0.7.6/0.8.16`
2. Recommending constant variables and test function names to be in mixed case:
- `Function SanctionsTest.test_BobBalance() (test/SanctionedTokenTest.t.sol#38-40) is not in mixedCase`
- `Variable SanctionsTest.BOB_STARTING_AMOUNT (test/SanctionedTokenTest.t.sol#10) is not in mixedCase`

## Token with God Mode

### True Positives
1. Unchecked transfer:
- `SanctionsTest.setUp() (test/GodModeTokenTest.t.sol#18-28) ignores return value by godModeToken.transfer(bob,BOB_STARTING_AMOUNT) (test/GodModeTokenTest.t.sol#27)`

### False Positives
N/A

### Non-Issues
1. Same as SanctionedToken.sol

## Bonding Curve Token

### True Positives
1. . unused-state-variable: https://github.com/crytic/slither/wiki/Detector-Documentation#unused-state-variable
- `BondingCurveTokenTest.NAME (test/BondingCurveTokenTest.t.sol#16) is never used in BondingCurveTokenTest (test/BondingCurveTokenTest.t.sol#9-111)`
- `BondingCurveTokenTest.SYMBOL (test/BondingCurveTokenTest.t.sol#17) is never used in BondingCurveTokenTest (test/BondingCurveTokenTest.t.sol#9-111)`

2. block-timestamp: https://github.com/crytic/slither/wiki/Detector-Documentation#block-timestamp
- _Low severity given Proof of Stake block addition timing and nature of time check in contract_
- `BondingCurveToken.sellTokens(uint256) (src/BondingCurveToken.sol#111-128) uses timestamp for comparisons`
        Dangerous comparisons:
        - require(bool,string)(block.timestamp >= lastPurchaseTime[msg.sender] + sellLockupTime,Tokens are locked up) (src/BondingCurveToken.sol#114)

### False Positives
reentrancy-vulnerability: 
- _Not a strict vulnerability but still worth correcting_
- `Reentrancy in BondingCurveToken.buyTokens(uint256) (src/BondingCurveToken.sol#81-104)`:
        External calls:
        - (sent) = address(msg.sender).call{value: excess}() (src/BondingCurveToken.sol#99)
        Event emitted after the call(s):
        - TokensPurchased(msg.sender,tokenAmount,cost) (src/BondingCurveToken.sol#103)``
- `Reentrancy in BondingCurveToken.sellTokens(uint256) (src/BondingCurveToken.sol#111-128)`:
        External calls:
        - (sent) = address(msg.sender).call{value: revenue}() (src/BondingCurveToken.sol#124)
        Event emitted after the call(s):
        - TokensSold(msg.sender,tokenAmount,revenue) (src/BondingCurveToken.sol#127)
- `Reentrancy in BondingCurveToken.withdrawExcess() (src/BondingCurveToken.sol#141-150)`:
        External calls:
        - (sent) = address(owner()).call{value: excess}() (src/BondingCurveToken.sol#146)
        Event emitted after the call(s):
        - ExcessWithdrawn(owner(),excess) (src/BondingCurveToken.sol#149)

### Non-Issues
1. state-variables-that-could-be-declared-constant
2. conformance-to-solidity-naming-conventions: https://github.com/crytic/slither/wiki/Detector-Documentation#conformance-to-solidity-naming-conventions
3. incorrect-versions-of-solidity: https://github.com/crytic/slither/wiki/Detector-Documentation#incorrect-versions-of-solidity
4. low-level-calls: https://github.com/crytic/slither/wiki/Detector-Documentation#low-level-calls

## Untrusted Escrow
### True Positives
reentrancy: https://github.com/crytic/slither/wiki/Detector-Documentation#reentrancy-vulnerabilities-2
1. `Reentrancy in UntrustedEscrow.deposit(uint256) (src/UntrustedEscrow.sol#54-62):`
        External calls:
        - token.safeTransferFrom(msg.sender,address(this),amount) (src/UntrustedEscrow.sol#57)
        State variables written after the call(s):
        - depositTimestamp = block.timestamp (src/UntrustedEscrow.sol#59)

blocktimestamp: https://github.com/crytic/slither/wiki/Detector-Documentation#block-timestamp
2. `UntrustedEscrow.withdraw() (src/UntrustedEscrow.sol#67-76) uses timestamp for comparisons`
        Dangerous comparisons:
        - require(bool,string)(block.timestamp >= depositTimestamp + WAIT_DURATION,Withdrawal time has not been reached yet) (src/UntrustedEscrow.sol#69)

declaring constants: https://github.com/crytic/slither/wiki/Detector-Documentation#state-variables-that-could-be-declared-constant
3. `BondingCurveTokenTest.WAIT_DURATION (test/UntrustedEscrowTest.t.sol#18) should be constant`
    `DeployTokenAndEscrow.DEFAULT_ANVIL_PRIVATE_KEY (script/DeployTokenAndEscrow.s.sol#11-12) should be constant`

declaring immutables: https://github.com/crytic/slither/wiki/Detector-Documentation#state-variables-that-could-be-declared-immutable
4. `UntrustedEscrow.buyer (src/UntrustedEscrow.sol#18) should be immutable`
   `UntrustedEscrow.seller (src/UntrustedEscrow.sol#21) should be immutable`
   `UntrustedEscrow.token (src/UntrustedEscrow.sol#24) should be immutable`

### False Positives
reentrancy vulnerability: https://github.com/crytic/slither/wiki/Detector-Documentation#reentrancy-vulnerabilities-3
1. `Reentrancy in UntrustedEscrow.deposit(uint256) (src/UntrustedEscrow.sol#54-62):`
        External calls:
        - token.safeTransferFrom(msg.sender,address(this),amount) (src/UntrustedEscrow.sol#57)
        Event emitted after the call(s):
        - TokensDeposited(buyer,amount,depositTimestamp) (src/UntrustedEscrow.sol#61)
2. `Reentrancy in UntrustedEscrow.withdraw() (src/UntrustedEscrow.sol#67-76):`
        External calls:
        - token.safeTransfer(seller,amount) (src/UntrustedEscrow.sol#73)
        Event emitted after the call(s):
        - TokensWithdrawn(seller,amount) (src/UntrustedEscrow.sol#75)


### Non-Issues
1. incorrect-versions-of-solidity: https://github.com/crytic/slither/wiki/Detector-Documentation#incorrect-versions-of-solidity
2. conformance-to-solidity-naming-conventions: https://github.com/crytic/slither/wiki/Detector-Documentation#conformance-to-solidity-naming-conventions

# Section 2 - NFT Variants and Staking

## NFT Staking Ecosystem

### True Positives
1. uninitialized variables: https://github.com/crytic/slither/wiki/Detector-Documentation#uninitialized-state-variables
- `DeployStakedNFT.merkleProof (script/DeployStakedNFT.s.sol#22) is never initialized. It is used in:`
        - DeployStakedNFT.run() (script/DeployStakedNFT.s.sol#26-42)

2. divide before multiply: https://github.com/crytic/slither/wiki/Detector-Documentation#divide-before-multiply
- _This functions as intended within the context of the contract_
- `NFTRewardStaking.claimRewards(uint256) (src/NFTRewardStaking.sol#71-84) performs a multiplication on the result of a division:`
        - periodsPassed = timeElapsed / CLAIM_PERIOD (src/NFTRewardStaking.sol#77)
        - rewards = periodsPassed * REWARDS_PER_PERIOD (src/NFTRewardStaking.sol#78)

3. timestamp for comparison: https://github.com/crytic/slither/wiki/Detector-Documentation#block-timestamp
- _Functions as intended within context of contract_
- `NFTRewardStaking.claimRewards(uint256) (src/NFTRewardStaking.sol#71-84) uses timestamp for comparisons`
        Dangerous comparisons:
        - require(bool,string)(timeElapsed >= CLAIM_PERIOD,Claim too soon) (src/NFTRewardStaking.sol#75)

4. reentrancy in scripts: https://github.com/crytic/slither/wiki/Detector-Documentation#reentrancy-vulnerabilities-2
- `Reentrancy in DeployStakedNFT.run() (script/DeployStakedNFT.s.sol#26-42):`
        External calls:
        - vm.startBroadcast(deployerKey) (script/DeployStakedNFT.s.sol#34)
        - vm.stopBroadcast() (script/DeployStakedNFT.s.sol#38)
        State variables written after the call(s):
        - contractList = ContractList(stakedNFT,erc20Reward,nftRewardStaking) (script/DeployStakedNFT.s.sol#40)
- `Reentrancy in BondingCurveTokenTest.setUp() (test/StakedNFTTest.t.sol#19-26):`
        External calls:
        - contracts = deployer.run() (test/StakedNFTTest.t.sol#21)
        State variables written after the call(s):
        - deployerAddress = deployer.deployerAddress() (test/StakedNFTTest.t.sol#25)
        - erc20Reward = contracts.rewardToken (test/StakedNFTTest.t.sol#23)
        - nftRewardStaking = contracts.nftRewardStaking (test/StakedNFTTest.t.sol#24)
        - stakedNFT = contracts.stakedNFT (test/StakedNFTTest.t.sol#22)

zero check: https://github.com/crytic/slither/wiki/Detector-Documentation#missing-zero-address-validation
- `StakedNFT.constructor(address,bytes32).initialOwner (src/StakedNFT.sol#46) lacks a zero-check on :`
                - ROYALTY_RECEIVER = initialOwner (src/StakedNFT.sol#50)
- `StakedNFT.setRoyalty(address,uint96)._royaltyReceiver (src/StakedNFT.sol#109) lacks a zero-check on :`
                - ROYALTY_RECEIVER = _royaltyReceiver (src/StakedNFT.sol#110)

similar variable names: https://github.com/crytic/slither/wiki/Detector-Documentation#variable-names-too-similar
- `Variable StakedNFT.ROYALTY_FEE (src/StakedNFT.sol#31) is too similar to StakedNFT.setRoyalty(address,uint96)._royaltyFee (src/StakedNFT.sol#109)`
- Variable StakedNFT.ROYALTY_RECEIVER (src/StakedNFT.sol#32) is too similar to StakedNFT.setRoyalty(address,uint96)._royaltyReceiver (src/StakedNFT.sol#109)

declaring constants: https://github.com/crytic/slither/wiki/Detector-Documentation#state-variables-that-could-be-declared-constant
- `DeployStakedNFT.DEFAULT_ANVIL_PRIVATE_KEY (script/DeployStakedNFT.s.sol#18-19) should be constant`
- `DeployStakedNFT.merkleProof (script/DeployStakedNFT.s.sol#22) should be constant` 

declaring immutables: https://github.com/crytic/slither/wiki/Detector-Documentation#state-variables-that-could-be-declared-immutable
- `NFTRewardStaking.rewardToken (src/NFTRewardStaking.sol#19) should be immutable` 
- NFTRewardStaking.stakedNFT (src/NFTRewardStaking.sol#18) should be immutable 
- StakedNFT.merkleRoot (src/StakedNFT.sol#35) should be immutable 

### False Positives
- re-entrancy: https://github.com/crytic/slither/wiki/Detector-Documentation#reentrancy-vulnerabilities-3
- _Event emissions_
- `Reentrancy in NFTRewardStaking.claimRewards(uint256) (src/NFTRewardStaking.sol#71-84):`
        External calls:
        - rewardToken.safeTransfer(msg.sender,rewards) (src/NFTRewardStaking.sol#81)
        Event emitted after the call(s):
        - RewardsClaimed(msg.sender,tokenId,rewards) (src/NFTRewardStaking.sol#83)
- `Reentrancy in StakedNFT.withdraw() (src/StakedNFT.sol#119-127):`
        External calls:
        - (sent) = address(owner()).call{value: balance}() (src/StakedNFT.sol#123)
        Event emitted after the call(s):
        - FundsWithdrawn(owner(),balance) (src/StakedNFT.sol#126)
- `Reentrancy in NFTRewardStaking.withdrawNFT(uint256) (src/NFTRewardStaking.sol#90-97):`
        External calls:
        - stakedNFT.safeTransferFrom(address(this),msg.sender,tokenId) (src/NFTRewardStaking.sol#94)
        Event emitted after the call(s):
        - NFTWithdrawn(msg.sender,tokenId) (src/NFTRewardStaking.sol#96)

### Non-Issues
- solidity versions: https://github.com/crytic/slither/wiki/Detector-Documentation#incorrect-versions-of-solidity
- low level calls: https://github.com/crytic/slither/wiki/Detector-Documentation#low-level-calls
- naming conventions: https://github.com/crytic/slither/wiki/Detector-Documentation#conformance-to-solidity-naming-conventions

## NFT with Merkle Tree Mints

### True Positives
locking ether: https://github.com/crytic/slither/wiki/Detector-Documentation#contracts-that-lock-ether
- `Contract locking ether found:`
        Contract GameNFT (src/GameNFT.sol#11-52) has payable functions:
         - GameNFT.mint(uint256) (src/GameNFT.sol#29-42)
        **But does not have a function to withdraw the ether**

write after write: https://github.com/crytic/slither/wiki/Detector-Documentation#write-after-write
- `PrimeCounter.isPrime(uint256).result (src/PrimeCounter.sol#47) is written in both`
        result = false (src/PrimeCounter.sol#49)
        result = true (src/PrimeCounter.sol#52)


calls inside a loop: https://github.com/crytic/slither/wiki/Detector-Documentation/#calls-inside-a-loop
- `PrimeCounter.countPrimes(address) (src/PrimeCounter.sol#28-40) has external calls inside a loop: tokenId = _nft.tokenOfOwnerByIndex(owner,i) (src/PrimeCounter.sol#33)`

reentrancy: https://github.com/crytic/slither/wiki/Detector-Documentation#reentrancy-vulnerabilities-2
- `Reentrancy in DeployNFTandPrimeCounter.run() (script/DeployNFTandPrimeCounter.s.sol#24-39):`
        External calls:
        - vm.startBroadcast(deployerKey) (script/DeployNFTandPrimeCounter.s.sol#32)
        - vm.stopBroadcast() (script/DeployNFTandPrimeCounter.s.sol#35)
        State variables written after the call(s):
        - contractList = ContractList(gameNFT,primeCounter) (script/DeployNFTandPrimeCounter.s.sol#37)
- `Reentrancy in TestNFTandPrimeCounter.setUp() (test/TestNFTandPrimeCounter.t.sol#18-26):`
        External calls:
        - contracts = deployer.run() (test/TestNFTandPrimeCounter.t.sol#20)
        State variables written after the call(s):
        - deployerAddress = deployer.deployerAddress() (test/TestNFTandPrimeCounter.t.sol#23)
        - gameNFT = contracts.gameNFT (test/TestNFTandPrimeCounter.t.sol#21)
        - primeCounter = contracts.primeCounter (test/TestNFTandPrimeCounter.t.sol#22)

costly operations inside loop: https://github.com/crytic/slither/wiki/Detector-Documentation#costly-operations-inside-a-loop
- `GameNFT.mint(uint256) (src/GameNFT.sol#29-42) has costly operations inside a loop:`
        - remainingSupply -- (src/GameNFT.sol#39)

declaring immutable: https://github.com/crytic/slither/wiki/Detector-Documentation#state-variables-that-could-be-declared-immutable
- `PrimeCounter._nft (src/PrimeCounter.sol#13) should be immutable` 

### False Positives
1. TODO

### Non-Issues
- solidity versions: https://github.com/crytic/slither/wiki/Detector-Documentation#incorrect-versions-of-solidity
- solidity naming conventions: https://github.com/crytic/slither/wiki/Detector-Documentation#conformance-to-solidity-naming-conventions
- declaring constants: https://github.com/crytic/slither/wiki/Detector-Documentation#state-variables-that-could-be-declared-constant

# Section 3 - AMMs

_Unable to get slither to work_

### True Positives
1. TODO

### False Positives
1. TODO