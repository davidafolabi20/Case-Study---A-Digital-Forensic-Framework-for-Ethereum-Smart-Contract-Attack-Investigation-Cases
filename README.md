# Case Study - A Digital Forensic Framework for Ethereum Smart Contract Attack Investigation Cases

The case study used here is the Origin protocol exploit. It was used to validate the forensic framework developed as part of the original work.

We shall now follow the phases as originally described.

## Code Analysis
I've uploaded a sample of the vulnerable code as `VaultCore.sol` in this repository. Throughout this analysis, I'll reference specific line numbers and major functions from this file to illustrate the vulnerability and explain how the exploit was executed.

Key functions such as `mint()`, `redeem()`, and `rebase()` will be examined in detail, as they played critical roles in the execution of this exploit.

To better understand the technical details of this case study, I recommend examining the code alongside this analysis.

## Origin Protocol Exploit
### Background and Information Gathering
Origin Protocol is a blockchain project focused on building a decentralized e-commerce ecosystem. One of their key offerings was Origin USD (OUSD), a yield-bearing stablecoin. On November 17, 2020, the OUSD smart contract was exploited, resulting in approximately $7 million in losses.

This exploit was first reported on the 17th of November 2020 by the co-founder of Origin Protocol via his X account, see the figure below. The initial response to this was to halt the exploited contract and reassure concerned investors and users of consistent updates while the team investigated the exploit. The transaction hash of the first transaction involved in the exploit was included in the medium report attached to the post. 
<p align="center">
  <img src="https://github.com/user-attachments/assets/bff007fb-9187-4531-baad-4c3cf1623336" alt="Origin Protocol Exploit">
</p>
<p align="center">
  <em>Figure 1: Matthew Liu responds to the Origin Dollar exploit via X</em>
</p>

Here are key information we have been able to gather from this.
* Initial Transaction hash; [0xe1c76241dda7c5fcf1988454c621142495640e708e3f8377982f55f8cf2a8401](https://etherscan.io/tx/0xe1c76241dda7c5fcf1988454c621142495640e708e3f8377982f55f8cf2a8401)
* Timestamp; Nov-17-2020, 12:47:19 AM UTC.
* This transaction hash reveals the address that initiated the transaction [0xb77f7BBAC3264ae7aBC8aEDf2Ec5F4e7cA079F83](https://etherscan.io/address/0xb77f7bbac3264ae7abc8aedf2ec5f4e7ca079f83). As well as the address that executed the transaction [0x47C3d84394043a4f42F6422AcCD27bB7240FDFE2](https://etherscan.io/address/0x47c3d84394043a4f42f6422accd27bb7240fdfe2).

Further examining the logs on etherscan revealed that the attacker with the Ethereum address - **0xb77f7BBAC3264ae7aBC8aEDf2Ec5F4e7cA079F83** created the contract they used to attack Origin's USD contract (**0x47C3d84394043a4f42F6422AcCD27bB7240FDFE2**). This occurred about 7 minutes before he first exploited the Origin USD contract; Nov-17-2020, 12:40:56 AM UTC.
* Transaction hash of the contract creation; [0x646be3dd18187636b09bd61a15d43da0a1ce36af44d21e45f160ecf04a5a3e09](https://etherscan.io/tx/0x646be3dd18187636b09bd61a15d43da0a1ce36af44d21e45f160ecf04a5a3e09).

## Smart Contract Analysis
In this section, the functionality of the smart contract and the invocation flow are examined. The smart contract is called **VaultCore.sol**, and it is responsible for handling deposits and withdrawals of supported assets. They include **mint(,)** which increases OUSD supply when assets are deposited into the vault. **Redeem()**, which decreases the supply of OUSD when assets are withdrawn from the vault. **Allocate()**, a function that redistributes assets between vaults, and **rebase()**, this function adjusts the total supply of OUSD. Additionally, the contract facilitates temporary lending of assets known as flash loans, this is executed via the **flash()** function. The code snippets for some of these features can be seen below. The contract also interacts with multiple external contracts, including price oracles and yield strategies.

### Major Functions
  * Minting; The `mint()` function allows users to deposit supported assets and receive OUSD in return. Line 42 - 115.
  * Redeeming OUSD; The `redeem()` function enables users to burn OUSD tokens and receive underlying assets in return. Line   119 - 249.
  * Rebase; This adjusts the total supply of OUSD based on the total value of assets in the vault. Line 361 - 390
  * Asset Allocation: The `allocate()` function distributes assets from the vault to various strategies according to their   target weights. Line 299 - 356

## Invocation Flow
The invocation flow analysis is an essential phase in our digital forensic framework. It provides detailed information about the types and sequence of interactions that occurred throughout the exploitation of the smart contract. This section covers the technical mechanics of the attack as well as the individual vulnerability exploited. The invocation flow was visualized by supplying the transaction hash to Blocksec Phalcon. In addition, the transaction used in this invocation flow has a transaction hash of [0xe1c76241dda7c5fcf1988454c621142495640e708e3f8377982f55f8cf2a8401](https://etherscan.io/tx/0xe1c76241dda7c5fcf1988454c621142495640e708e3f8377982f55f8cf2a8401).

### Initial Interaction and Flash Loan Acquisition
<p align="center">
  <img src=https://github.com/user-attachments/assets/41237423-1b6b-402d-a0c6-998c7bfc4af8 alt="Origin Protocol Exploit">
</p>
<p align="center">
  <em>Figure 2: Initial Interaction and flash loan acquisition.</em>
</p>

“CALL | [Receiver] 0x47c5d8459404e5a6f42164427accd27bb7240fdfe2.doHardWork (uint256)(threshold:950) > ()”
The exploit is initiated with a call from the attacker’s custom contract. The function name ‘doHardWork’ suggests a specifically designed contract for this exploit, as seen in the screenshot above.

“CALL dYdX: Solo Margin.operate(uint256) (accounts:[{owner:0x47c5d8459404054f4216472acccd37bb7240fdfe2, number:1}], actions:[{actionType:1, accountId:0, amount:{sign:false, denomination:0, ref:0, value:70,000.000,000,000,000,000,000}, primaryMarketId:0, secondaryMarketId:0, otherAddress:0x0000000000000000000000000000000000000000, otherAccountId:0, data:0x}]) → ()”

According to Figure 2 above, the contract interacts with the ‘dYdX solo margin’ to set up a flash loan worth 70,000 WETH which acts as capital for the exploitation. Utilisation of flash loans is a typical practice in Ethereum smart contract attacks, it allows entities to temporarily access large sums of funds without leveraging any collateral.

### Token Manipulation
<p align="center">
  <img src=https://github.com/user-attachments/assets/97e9bc89-8404-47e8-8cbe-29bff4086737 alt="Origin Protocol Exploit">
</p>
<p align="center">
  <em>Figure 3: Token Manipulation</em>
</p>

‘CALL Wrapped Ether (WETH).withdraw(uint256)(70,000,000,000,000,000,000,000) → ()’

‘CALL Uniswap V2: Router 2.swapExactETHForTokens(uint256,address[],address,uint256)(amountOutMin:1, path:[Wrapped Ether (WETH), Tether: USDT Stablecoin], to:0x47c5d8459404054f4216472acccd37bb7240fdfe2, deadline:1,605,574,859) → (amounts:[7,500,000,000,000,000,000,000, 7,855,911,51])’

The next step involved price manipulation, with a two-stage token conversion of 17,500 WETH to USDT and 52,500 WETH to DAI. These swaps were executed via Uniswap’s V2 Router through a function called swapExactETHForTokens. These swaps impacted the WETH-USDT and WETH-DAI liquidity pools and in effect manipulated token prices, refer to Figure 3.

### Exploitation of OUSD due to poor rebasing logic
<p align="center">
  <img src=https://github.com/user-attachments/assets/9dcbd52f-088e-4f1a-a26c-99f49e8ddbee alt="Origin Protocol Exploit">
</p>
<p align="center">
  <em>Figure 4: Exploitation of OUSD due to vulnerable rebasing logic</em>
</p>

During the next stage of the exploit, the attacker mints and immediately redeems even larger amounts of OUSD tokens by making calls to the `mintMultiple()` function. This exploits the rebasing mechanism of the contract and shows the key vulnerability that was exploited. Via this mechanism, the attacker was able to mint large sums of OUSD at prices that have been manipulated and then redeem the token at a profit.

### Profit Extraction and Flash Loan Repayment
<p align="center">
  <img src=https://github.com/user-attachments/assets/a8ccf257-a11a-4b60-9bfd-9dc5567cdb81 alt="Origin Protocol Exploit">
</p>
<p align="center">
  <em>Figure 5: Profit Extraction</em>
</p>

<p align="center">
  <img src=https://github.com/user-attachments/assets/cc48fe2f-a0a0-4d45-ac04-9662c3ba56d9 alt="Origin Protocol Exploit">
</p>
<p align="center">
  <em>Figure 6: Flash Loan Repayment</em>
</p>

CALL | Wrapped Ether (WETH).withdraw (uint256)(wad:47,976,521,942,995,684,479,041) > ()”

The attacker proceeds to withdraw their profit as seen in the figure above. In this transaction alone, “47,976.521942995684470041 ETH” was withdrawn. After this, the attacker proceeded to repay the flash loan of “70,0000.000000000000000002” wrapped ETH (WETH).

“CALL | value: 70000.000000000000000002 Wrapped Ether (WETH).deposit () > ()”

This invocation flow reveals the mechanism of the vulnerability, illustrating a complex attack that employed flash loans, price manipulation and vulnerabilities in the rebasing logic used by Origin protocol. This vulnerability was used over several different transactions within a few minutes on the 10th of November to drain the Origin contract of enormous amounts of funds. 

### Transaction Analysis
<p align="center">
  <img src=https://github.com/user-attachments/assets/793c766b-1885-41d4-8f7a-e8099e1510ae alt="Origin Protocol Exploit">
</p>
<p align="center">
  <em>Figure 7: Etherscan shows the total number of transactions carried out from the attacker’s address</em>
</p>

Using Etherscan, transaction records of the address found to be the exploiter’s address can be examined. The attacker has carried out 131 transactions using the address: [0xb77f7BBAC3264ae7aBC8aEDf2Ec5F4e7cA079F83](https://etherscan.io/address/0xb77f7bbac3264ae7abc8aedf2ec5f4e7ca079f83), see Figure 7 above. Examining the initial transactions related to the exploit reveals a sequence of events.
  * First Transaction: The records show that the attacker first executed an ‘approve’ method on the same day the exploit was carried out, granting permission for token transfers, see Figure 8 below. Transaction Hash: [0xd95a59e64aabfa932620e7770fcd55d904381fe195282202b2fd464ac44a165b](https://etherscan.io/tx/0xd95a59e64aabfa932620e7770fcd55d904381fe195282202b2fd464ac44a165b).
<p align="center">
  <img src=https://github.com/user-attachments/assets/d7116606-a752-4b29-b40c-ffeea4a30467 alt="Origin Protocol Exploit">
</p>
<p align="center">
  <em>Figure 8: Details of the first transaction carried out by the attacker’s address</em>
</p>
