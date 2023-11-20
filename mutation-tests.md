# Mutation Testing

[X] Ensure the ERC721 / ERC20 / Staking application has 100% line and branch coverage

[X] Run vertigo-rs to do mutation testing on the ERC721 / ERC20 / Staking application. What mutations do you discover?

### Hints for Vertigo

- use `--sample-ratio 0.1` when testing so you don’t have to wait a long time for it to finish. Then let it run the entire set of mutations without that argument
- use `--output my_file` to save the output because some shells don’t show the output


## Vertigo

Vertigo mutation testing results show that out of 29 mutations tested, 19 were killed, indicating a mutation score of approximately 65.5%. The testing encountered warnings but completed the campaign run.

### Survivors

4 mutations survived:

1. **ERC20Reward.sol** - The `onlyOwner` modifier was removed from the `mint` function, which did not kill the mutant, suggesting a potential oversight in access control.
   - File: `ERC20Reward.sol`
   - Line: 12
   - Original: `function mint(address to, uint256 amount) external onlyOwner {`
   - Mutated: `function mint(address to, uint256 amount) external {`

2. **StakedNFT.sol** - The condition `s_tokenCounter < MAX_SUPPLY` was changed to `<=`, which also survived, indicating an edge case in the supply check.
   - File: `StakedNFT.sol`
   - Line: 80
   - Original: `require(s_tokenCounter < MAX_SUPPLY, "Max supply reached");`
   - Mutated: `require(s_tokenCounter <= MAX_SUPPLY, "Max supply reached");`

3. **StakedNFT.sol** - A line setting the default royalty was removed, and the mutant lived, highlighting a missing assertion or test for royalty settings.
   - File: `StakedNFT.sol`
   - Line: 102
   - Original: `_setDefaultRoyalty(ROYALTY_RECEIVER, ROYALTY_FEE);`
   - Mutated: *(line removed)*

4. **NFTRewardStaking.sol** - The subtraction operation in the calculation of `timeElapsed` was changed to addition, which survived, indicating a lack of validation for time-based logic.
   - File: `NFTRewardStaking.sol`
   - Line: 68
   - Original: `uint256 timeElapsed = block.timestamp - lastClaimed[tokenId];`
   - Mutated: `uint256 timeElapsed = block.timestamp + lastClaimed[tokenId];`

These survivors suggest areas in the code that may require additional testing or improved logic to handle mutated cases.

## Raw Output

python3 vertigo-rs/vertigo.py run
[*] Starting mutation testing
[*] Starting analysis on project
[+] Foundry project detected
[+] If this is taking a while, vertigo-rs is probably installing dependencies in your project
[*] Initializing campaign run 
[*] Checking validity of project
[+] The project is valid
[*] Storing compilation results
[*] Running analysis on 29 mutants
  3%|███▎                                                                                           | 1/29 [00:03<01:51,  3.97s/mutant]WARNING:root:Error: 
  7%|██████▌                                                                                        | 2/29 [00:06<01:18,  2.89s/mutant]WARNING:root:Error: 
 38%|███████████████████████████████████▋                                                          | 11/29 [00:40<01:11,  3.97s/mutant]WARNING:root:Error: 
 41%|██████████████████████████████████████▉                                                       | 12/29 [00:42<00:58,  3.44s/mutant]WARNING:root:Error: 
 83%|█████████████████████████████████████████████████████████████████████████████▊                | 24/29 [01:28<00:19,  3.97s/mutant]WARNING:root:Error: 
 97%|██████████████████████████████████████████████████████████████████████████████████████████▊   | 28/29 [01:42<00:03,  3.83s/mutant]WARNING:root:Error: 
100%|██████████████████████████████████████████████████████████████████████████████████████████████| 29/29 [01:45<00:00,  3.62s/mutant]
[*] Done with campaign run
[+] Report:
Mutation testing report:
Number of mutations:    29
Killed:                 19 / 29

Mutations:

[+] Survivors
Mutation:
    File: /Users/longoria/Desktop/user/code/solidity-advanced/rs-wk02-nft/01_NFTStakingEco/src/ERC20Reward.sol
    Line nr: 12
    Result: Lived
    Original line:
             function mint(address to, uint256 amount) external onlyOwner {

    Mutated line:
             function mint(address to, uint256 amount) external  {

Mutation:
    File: /Users/longoria/Desktop/user/code/solidity-advanced/rs-wk02-nft/01_NFTStakingEco/src/StakedNFT.sol
    Line nr: 80
    Result: Lived
    Original line:
                 require(s_tokenCounter < MAX_SUPPLY, "Max supply reached");

    Mutated line:
                 require(s_tokenCounter <= MAX_SUPPLY, "Max supply reached");

Mutation:
    File: /Users/longoria/Desktop/user/code/solidity-advanced/rs-wk02-nft/01_NFTStakingEco/src/StakedNFT.sol
    Line nr: 102
    Result: Lived
    Original line:
                 _setDefaultRoyalty(ROYALTY_RECEIVER, ROYALTY_FEE);

    Mutated line:
                 

Mutation:
    File: /Users/longoria/Desktop/user/code/solidity-advanced/rs-wk02-nft/01_NFTStakingEco/src/NFTRewardStaking.sol
    Line nr: 68
    Result: Lived
    Original line:
                 uint256 timeElapsed = block.timestamp - lastClaimed[tokenId];

    Mutated line:
                 uint256 timeElapsed = block.timestamp + lastClaimed[tokenId];

[*] Done! 