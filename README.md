# Case Study - A Digital Forensic Framework for Ethereum Smart Contract Attack Investigation Cases

The case study used here is the Origin protocol exploit. It was used to validate the forensic framework developed as part of the original work.

We shall now follow the phases as originally described.

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

### Smart Contract Analysis
In this section, the functionality of the smart contract and the invocation flow are examined. The smart contract is called VaultCore.sol, and it is responsible for handling deposits and withdrawals of supported assets. They include mint(,) which increases OUSD supply when assets are deposited into the vault. Redeem(), which decreases the supply of OUSD when assets are withdrawn from the vault. Allocate(), a function that redistributes assets between vaults, and rebase(), this function adjusts the total supply of OUSD. Additionally, the contract facilitates temporary lending of assets known as flash loans, this is executed via the flash() function. The code snippets for some of these features can be seen below. The contract also interacts with multiple external contracts, including price oracles and yield strategies.

#### Major Functions
**Minting**
'''
function mint(address _asset, uint256 _amount)
external
whenNotDepositPaused
{
 // Implementation
}
'''
