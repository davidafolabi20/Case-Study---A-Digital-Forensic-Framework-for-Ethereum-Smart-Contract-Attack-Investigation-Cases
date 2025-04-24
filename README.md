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
In this section, the functionality of the smart contract and the invocation flow are examined. The smart contract is called **VaultCore.sol**, and it is responsible for handling deposits and withdrawals of supported assets. They include **mint(,)**, which increases OUSD supply when assets are deposited into the vault. **Redeem()**, which decreases the supply of OUSD when assets are withdrawn from the vault. **Allocate()**, a function that redistributes assets between vaults, and **rebase()**, this function adjusts the total supply of OUSD. Additionally, the contract facilitates temporary lending of assets known as flash loans, which is executed via the **flash()** function. The code snippets for some of these features can be seen below. The contract also interacts with multiple external contracts, including price oracles and yield strategies.

### Major Functions
  * Minting: The `mint()` function allows users to deposit supported assets and receive OUSD in return. Line 42 - 115.
  * Redeeming OUSD: The `redeem()` function enables users to burn OUSD tokens and receive underlying assets in return. Line   119 - 249.
  * Rebase: This adjusts the total supply of OUSD based on the total value of assets in the vault. Line 361 - 390
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

According to Figure 2 above, the contract interacts with the ‘dYdX solo margin’ to set up a flash loan worth 70,000 WETH, which acts as capital for the exploitation. Utilisation of flash loans is a typical practice in Ethereum smart contract attacks, it allows entities to temporarily access large sums of funds without leveraging any collateral.

### Token Manipulation
<p align="center">
  <img src=https://github.com/user-attachments/assets/97e9bc89-8404-47e8-8cbe-29bff4086737 alt="Origin Protocol Exploit">
</p>
<p align="center">
  <em>Figure 3: Token Manipulation</em>
</p>

‘CALL Wrapped Ether (WETH).withdraw(uint256)(70,000,000,000,000,000,000,000) → ()’

‘CALL Uniswap V2: Router 2.swapExactETHForTokens(uint256,address[],address,uint256)(amountOutMin:1, path:[Wrapped Ether (WETH), Tether: USDT Stablecoin], to:0x47c5d8459404054f4216472acccd37bb7240fdfe2, deadline:1,605,574,859) → (amounts:[7,500,000,000,000,000,000,000, 7,855,911,51])’

The next step involved price manipulation, with a two-stage token conversion of 17,500 WETH to USDT and 52,500 WETH to DAI. These swaps were executed via Uniswap’s V2 Router through a function called swapExactETHForTokens. These swaps impacted the WETH-USDT and WETH-DAI liquidity pools and, in effect, manipulated token prices, refer to Figure 3.

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

  * Contract Deployment: The second transaction carried out by the attacker is a smart contract that is now known as the        contract used by the attacker to carry out the exploit, see Figure 9.
<p align="center">
  <img src=https://github.com/user-attachments/assets/dd851971-e331-451a-b441-406729b7b66c alt="Origin Protocol Exploit">
</p>
<p align="center">
  <em>Figure 9: The first 4 transactions carried out by the attacker’s address</em>
</p>

  * Failed exploit attempt: The transaction called a method named ‘Do Hard Work’, however, this method failed to execute,       see Figure 9 above.
  * Successful exploit: Subsequently, the ‘Do Hard Work’ method was executed, which marked the start of the exploit, as         seen in Figure 9.
  * The transaction logs reveal that the attacker’s address interacted with several decentralised finance (DeFi) protocols      and services, including Ren BTC Gateway, Curve Finance, Uniswap, and the smart contract that was deployed. These            transactions include token exchanges and transfers across many platforms, see Figure 10.
<p align="center">
  <img src=https://github.com/user-attachments/assets/d148917b-63ca-4a26-adb9-9ed45cda59eb alt="Origin Protocol Exploit">
</p>
<p align="center">
  <em>Figure 10: These transactions represent the attacker’s address interacting with different DeFi protocols</em>
</p>

  * More transaction records: The transaction details below show that the exploiter’s address executed a function via the       smart contract it created. In this transaction, the smart contract borrowed, supplied, and withdrew tokens worth large       sums of money, this can be seen in the screenshot in Figure 11 below.
<p align="center">
  <img src=https://github.com/user-attachments/assets/0cc0c446-9eaf-4646-989d-974d42e3b080 alt="Origin Protocol Exploit">
</p>
<p align="center">
  <em>Figure 11: An elaborate transaction involving borrowing, supplying, and withdrawal of funds</em>
</p>

  * The token swaps were followed by large deposits into Tornado Cash, these deposits were made in 1s, 10s and 100s with no     definitive pattern to the deposits, as seen in Figure 12. This is a clear indication of the attacker obfuscating the        trail of funds.
<p align="center">
  <img src=https://github.com/user-attachments/assets/cf2fea0d-6c7e-449f-9646-2d74855a0666 alt="Origin Protocol Exploit">
</p>
<p align="center">
  <em>Figure 12: These transactions represent huge deposits of ETH to Tornado Cash</em>
</p>

  * Last notable transaction - Self Destruct:
<p align="center">
  <img src=https://github.com/user-attachments/assets/e92a369d-5b83-481d-a6c2-3bc56d568202 alt="Origin Protocol Exploit">
</p>
<p align="center">
  <em>Figure 13: The last transaction carried out by the attacker was a self-destruct call to the contract they created</em>
</p>

  The last major transaction recorded in this exploit sequence involves the attacker calling a self-destruct method on the    smart contract that was used to perpetrate the exploit. This method essentially removes the bytecode of the smart           contract from the blockchain, which will make future investigation of its functionality more difficult. From the            timestamp in this transaction, it can be seen that the exploit took place within 15 minutes from the initial exploit        transaction to the last, see Figure 13 above.

  The smart contract deployed by the attacker’s address recorded 17 transactions of which 15 were involved in the exploit. 
  The records also show that the attacker’s address dominated the interactions, initiating 16 out of 17 transactions, see     Figure 14.
<p align="center">
  <img src=https://github.com/user-attachments/assets/79a06cf5-150c-4bfb-827c-4499e200c083 alt="Origin Protocol Exploit">
</p>
<p align="center">
  <em>Figure 14: The transaction records of the smart contract deployed by the attacker showing a section of all 17 transactions it executed</em>
</p>

### Fund Flow
This section examines the movement of funds linked with the Origin Protocol exploit. In this section, the attacker's sophisticated chain of transactions is traced, beginning with the first transaction through token swaps, consolidations, and obfuscation efforts.

Again, using the first exploit transaction; 
[0xe1c76241dda7c5fcf1988454c621142495640e708e3f8377982f55f8cf2a8401](https://etherscan.io/tx/0xe1c76241dda7c5fcf1988454c621142495640e708e3f8377982f55f8cf2a8401), Blocksec metasleuth provides an elaborate map using this transaction hash as in Figure 15 and Figure 16 below. The image presents an intricate web of transactions, representing the multi-step exploit executed by the attacker. The attack involved interactions with multiple decentralised finance protocols like Uniswap V2, Aave, and dYdX. Several tokens, including DAI, OUSD, WBTC, and WETH, were swapped before consolidation into the native Ethereum token, which was deposited into Tornado Cash. The fund flow in this instance shows a total of 11,804 ETH being stolen in one transaction.
<p align="center">
  <img src=https://github.com/user-attachments/assets/089b5bd7-a7e4-4592-bd87-7bee8b7f7a73 alt="Origin Protocol Exploit">
</p>
<p align="center">
  <em>Figure 15: Transaction Map visualised by BlockSec MetaSleuth</em>
</p>

<p align="center">
  <img src=https://github.com/user-attachments/assets/b85f48fe-4bd0-4dac-a18a-f74c42d35c46 alt="Origin Protocol Exploit">
</p>
<p align="center">
  <em>Figure 16: Transaction Map visualised by BlockSec MetaSleuth, part 2</em>
</p>

### Evidence Correlation
In this stage of the forensic investigation, we will highlight key findings from the forensic process and correlate them with evidence found on-chain, that is, evidence found on the Ethereum explorer (Etherscan).

#### The key findings include:
  * Attacker’s address: The exploit was reported to have been executed from the Ethereum address: [0xb77f7BBAC3264ae7aBC8aEDf2Ec5F4e7cA079F83](https://etherscan.io/address/0xb77f7bbac3264ae7abc8aedf2ec5f4e7ca079f83). This has been confirmed by the multiple transactions seen on Etherscan, see Figure 7 from the Transaction analysis section above. In addition, it was also confirmed by the invocation flow provided by Blocksec MetaSleuth, see Figure 17 below.
<p align="center">
  <img src=https://github.com/user-attachments/assets/a1816bf9-1aed-45aa-a2f9-e4671350788f alt="Origin Protocol Exploit">
</p>
<p align="center">
  <em>Figure 17: Address of the attacker</em>
</p>

  * Malicious Smart Contract: The attacker indeed created a smart contract and used it to execute the exploit on Origin         OUSD. The findings from Etherscan and the invocation flow both point to this fact, see Figure 14 above. In the image        below, the invocation flow shows the attacker making a call to the ‘**doHardWork**’ function residing in the smart          contract they created.
<p align="center">
  <img src=https://github.com/user-attachments/assets/f435dbeb-1000-4e69-870b-6dcc1833062a alt="Origin Protocol Exploit">
</p>
<p align="center">
  <em>Figure 18: At the start of the exploit, the attacker makes a call to a function residing in the contract they created in order to exploit Origin OUSD</em>
</p>
    
  * The time and date of the first exploit was recorded by Etherscan to be Nov-17-2020, 12:47:19 AM UTC. This is confirmed      by Blocksec Phalcon and MetaSleuth, see the figure below.
<p align="center">
  <img src=https://github.com/user-attachments/assets/113abd1a-ba4f-40d1-9841-2e1041e1688d alt="Origin Protocol Exploit">
</p>
<p align="center">
  <em>Figure 19: Date and time of the attack, reported by Blocksec Phalcon</em>
</p>
 
  * Initial Transaction and Actions: The initial transaction can be identified by this transaction hash:                        [0xe1c76241dda7c5fcf1988454c621142495640e708e3f8377982f55f8cf2a8401](https://etherscan.io/tx/0xe1c76241dda7c5fcf1988454c621142495640e708e3f8377982f55f8cf2a8401). From the invocation flow          examination carried out earlier, we can see that the first action was to set up and acquire a flash loan worth              “**70,000 WETH**”, this flash loan was used as a sort of capital to fund the entire exploit, see the section on             Invocation Flow.
  * The next set of actions included a series of token swaps, which had adverse consequences on the liquidity pools and         price oracles of Origin **OUSD**. Most significantly, the attacker mints and immediately redeems an enormous amount of      **OUSD** tokens, which was made possible due to earlier manipulated price oracles. This was an exploitation of the          rebasing logic of the Origin **OUSD**.
  * Finally, the attacker withdraws their profit and repays the flash loan, which was earlier acquired, see Figure 6.
  * After the exploit was concluded, the attacker proceeded to deposit their ill gains to several known fund trail              obfuscators, Tornado Cash and Ren: BTC gateway. This exfiltration of funds did not all occur on the same day, the image     below shows further deposits to the obfuscators **63 days** after the exploit. The funds trail mostly dies at this          point.

**NB**: Ren BTC Gateway; This is an entity that converts a host of tokens from any blockchain into Bitcoin. This indicates      that the attacker offloaded a chunk of the ill gains in exchange for Bitcoin.
<p align="center">
  <img src=https://github.com/user-attachments/assets/91f91178-0bec-43c7-8b22-afb8f228688d alt="Origin Protocol Exploit">
</p>
<p align="center">
  <em>Figure 20: Multiple deposits going out to fund obfuscators</em>
</p>

  * At the moment, the attacker’s wallet holds only approximately $4, see the image below.
<p align="center">
  <img src=https://github.com/user-attachments/assets/162e61d7-1a97-45ed-aef8-52d7d06b0116 alt="Origin Protocol Exploit">
</p>
<p align="center">
  <em>Figure 21: : Current balance of the attacker’s wallet as of 24th April 2025</em>
</p>

### Impact Assessment
All outgoings were extracted into **CSV** format and analysed to determine how much was exfiltrated excluding gas fees expended. Here is a total of each token:
* Ethereum (ETH): **3,331.556 ETH**
* Ethereum was burned for Bitcoin (BTC) seven times, this adds up to **184.29269693 BTC**
  
Considering the price of **ETH** on the day the exploit was carried out, 17th November 2020. This loss is estimated at $4,840,090.63. However, this was not all that was stolen.

* DAI: The official post from Origin protocol reports that a substantial amount of this token was stolen. Unfortunately, we have not been able to find traces of the attacker exfiltrating this token. This might be due to the self-destruct function called by the attacker’s smart contract. The function resulted in the deletion of the bytecode of the smart contract.
