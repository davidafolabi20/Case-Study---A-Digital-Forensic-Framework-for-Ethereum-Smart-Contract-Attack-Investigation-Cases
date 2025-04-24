# This section provides a shorter summary of the Origin protocol exploit forensic analysis.
## Incident Overview
<p align="center">
  <img src=https://github.com/user-attachments/assets/d6e955ff-48b7-486d-9376-814929385094 alt="Origin Protocol Exploit">
</p>
<p align="center">
  <em>Figure 1:Exploit timeline, part 1</em>
</p>

<p align="center">
  <img src=https://github.com/user-attachments/assets/4a653648-b16b-4055-8919-a9bf16283405 alt="Origin Protocol Exploit">
</p>
<p align="center">
  <em>Figure 2:Exploit timeline, part 2</em>
</p>

Additional details include the estimated loss reported by Origin protocol; $7.5 million. The exploit itself was initiated and completed within 15 minutes, and the smart contract was made to ‘**self-destruct**’ 7 minutes after it was deployed. Finally, the attacker continued to obfuscate funds for up to 63 days after the exploit was carried out.

## Key Findings
* The attacker utilized a flash loan of **70,000 WETH** to initiate the exploit.
  * Price oracles were manipulated via large token swaps.
  * The key vulnerability to the success of the exploit was the **OUSD** rebasing mechanism.
* Critical transactions:
  * Initial exploit transaction hash: [0xe1c76241dda7c5fcf1988454c621142495640e708e3f8377982f55f8cf2a8401](https://etherscan.io/tx/0xe1c76241dda7c5fcf1988454c621142495640e708e3f8377982f55f8cf2a8401).
  * Attacker's address: [0xb77f7BBAC3264ae7aBC8aEDf2Ec5F4e7cA079F83](https://etherscan.io/address/0xb77f7bbac3264ae7abc8aedf2ec5f4e7ca079f83).
  * Malicious contract address: [0x47C3d84394043a4f42F6422AcCD27bB7240FDFE2](https://etherscan.io/address/0x47c3d84394043a4f42f6422accd27bb7240fdfe2).
* Exploit Mechanism:
  * Large amounts of **OUSD** tokens were minted at manipulated prices.
  * **OUSD** was immediately redeemed for more valuable underlying assets.
  * The rebasing function was then exploited to artificially inflate the **OUSD** token 
supply.

## Fund Flow
  * It is estimated that **3,331.556 ETH** was stolen, and an additional amount was converted 
to bitcoin (BTC), in **BTC** it adds up to **184.29269693**.
  * Tornado cash and **Ren BTC** gateway were used to obfuscate the funds.

Following the thorough investigation of both the DFX finance and Origin protocol case studies, it can be seen that the previously developed digital forensic methodology for Ethereum smart contract exploits is robust and complete in its present form. The framework’s ability to address the complexity of both exploits demonstrates its adaptability. As a result, no substantial changes to the developed framework are deemed essential at this time. **This concluding paragraph is as seen in the original work. The link will be added to this repo after publication**
