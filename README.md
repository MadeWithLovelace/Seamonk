# SeaMonk
A gateway application which facilitates a simple->complex transaction incorporating a Smart Contract on the Cardano blockchain. SeaMonk makes it possible for a user to send an amount of ADA in a simple wallet-wallet transaction to a given address and interact with a given smart contract, without the need for the user to include any smart-contract-specific transaction details.

SeaMonk can also be used without a smart contract with some minor adjustments, acting as an active gateway application to "watch for" specified amounts or sending addresses and performing tasks or creating transactions according to criteria. However it is developed with the incorporation of smart contracts in mind, as a way to allow for anyone with any Cardano wallet, to interact with arguably any Smart Contract, according to the criteria setup within SeaMonk and the Smart Contract.

## How To Use
Run seamonk.py the first time and the folder structure and settings file will be created, interactively prompting you for the settings. These settings can be changed either by deleting the profile.json file within the data folder, or by running seamonk with the param/option `reconfigure`.

This app is meant to work with any SmartContract but for all references I make to the smartcontract, the following simple smartcontract for a token swap is the reference point: https://github.com/MadeWithLovelace/CypherMonks/tree/main/NFTSwaps/SmartContracts/TheBeginning

Besides setup/reconfigure, Seamonk can be run in three (3) primary ways: 
    - Log transactions only
    - Deposit Native Token into Smart Contract
    - Payment Action & Logging

Log transactions only updates the transactions log file within the data folder, with all new transactions detected on the address being watched. It checks for new transactions about every 5 seconds.  It is initiated by setting the option `get_transactions`

Deposit, initiated with the option `deposit`, allows the owner of the smart contract to deposit the native tokens to be made available via SeaMonk and the Smart Contract. It prompts and walks through the process step by step.

Payment Action & Logging is the main function of SeaMonk and is initiated by running the configured application without any options. The main functionality will be to compare the transactions log with a payment log, which is updated accordingly when a payment meeting the criteria is detected and recorded. If Whitelist Once is set the whitelist will also be updated to remove the successful payment. And upon detection of a payment, the appropriate transaction is created and submitted to the blockchain, spending the matching/criteria-based utxo(s) for both the watched wallet and the Smart Contract.  For example, if acting as a token exchange, when the app finds a new transaction matching a sender on the whitelist in the expected amount, it will perform the token exchange, sending the tokens required by the settings out of the smart contract to the sender, and any other transactions back to both the watched wallet and smart contract. 

One of the functions of SeaMonk is to replenish the collateral UTxO in the watched wallet, so it can successfully execute another Smart Contract transaction after the current one.  SeaMonk also adds any additional UTxO for a "payment" or "price" to include. This is sometimes different than the expected amount to watch for. For example, in a token giveaway where a user sends just 1.5 ADA to initiate their "take" in the giveaway, you may want to include a large transaction in each exchange, sent from the watch wallet back to the watch wallet as a simple Smart Contract validation technique. This ensures if someone were to drain the smart contract of the tokens in the giveaway, they would need to pay you to do so. This allows a smart contract to be used with little configuration and do the whitelisting on the front-end.

When you whitelist addresses, you can enter a single address from each users wallet and their entire wallet will be whitelisted. SeaMonk compares 103-length Cardano addresses in such a way as to compare the string shared by all addresses in a wallet which utilizes a staking address. This way the user paying into the SeaMonk watch address won't have to fund the specific whitelisted address, unless it is a simple address or they are using Nami for example, where only a single address is shown.

SeaMonk also accounts for change, if needed, back to the Smart Contract. For example, if you want to setup a simple swap contract, this can be accomplished and the settings can be dynamically changed via SeaMonk. Wherein you can set an amount of Native Tokens to be sent to the user/sender who engaged the app, and send the change back into the contract for the next user. This allows both the criteria to have more flexibility with a single contract, and for dynamically and automatically interacting with a smart contract in a more flexible way.

## Running SeaMonk
### Prereq
You'll need to have Cardano node and cli installed/synced on the server/system where you'll run SeaMonk. You'll also want your watched wallet to be on this same system, with its .skey and .vkey files accessible. The smartcontract you'll be using will also need to be on this system.

For now SeaMonk has only been tested on Debian and Ubuntu, but should work flawlessly on any system where you can run Python. The file paths for settings config do need to be full paths, so if your wallet skey files are in a wallet folder in the same folder with SeaMonk, you'll still want to enter the full path to that skey file for example.

Before you run or configure, you'll need to adjust the variables in the top of the cardanotx.py file, to match whether you are using mainnet or testnet and which blockfrost also, mainnet or testnet.

### First-run
On first run, just simply `python3 seamonk.py`, SeaMonk will walk you through configuration and generate a profile.json file with your settings. To change these settings, you can run `python3 seamonk.py reconfigure`

### Deposit
After it's setup, you will need to fund the SmartContract. Running `python3 seamonk.py deposit` will walk you through this process.

### Get Transactions
It's possible to run SeaMonk and configure the "Check" option to true, to allow getting transactions in-line with monitoring and acting on transactions...but it's not recommended at this time, as it may end up missing some transactions it needs to record.  So it's recommended to run 2 instances of SeaMonk, the first is a very memory-lite instance that simply gets new transactions in "almost-real-time" about a 5 second window loop, until you kill it. To run this, you simply issue `python3 seamonk.py get_transactions` and it will watch for transactions and log them, ready to then be analyzed by the next instance of the main running app.

### Main Running App
The second instance is the main app, which matches new transactions to the whitelist or other criteria and when a payment is detected as a match, records them and performs the required SmartContract transaction(s). To run the main SeaMonk loop, issue `python3 seamonk.py` and it will loop until you kill it.

For the last 2 instances which would both run simultaneously, you can run them in the background by putting `setsid` in front of the commands, for example: `setsid python3 seamonk.py` will start SeaMonk in the background. There's a run.log file generated each time you run which will capture most outputs and errors, but is still a WIP.

## Settings Available and What They Mean
- Blockfrost API ID = This is the ID provided to you from Blockfrost.io which should match the network type (mainnet or testnet)
- Watched Wallet Address = The address you are watching/expecting payment to.
- Collateral Lovelace Amount = Collateral is required with a SmartContract transaction and ensures that if a smart contract failed for some very unlikely reason, you are guaranteeing this amount of collateral against that happening. Usually 2 ADA is enough (2000000 lovelace would be entered)
- Check for Transactions Between Payment... = This would be set to true if you were not also running `seamonk.py get_transactions` - which loops faster and is more reliable in snapshotting all transactions coming in. If this is set to True, between payments processed the main payment function will also check for transactions before the next payment processed or check for payment made.
- Use a Whitelist = Running SeaMonk with a whitelist is the recommended approach if you are watching for payments only from certain addresses or an address. If the sender address is irrelevant you should answer False and leave the recipient setting empty. Otherwise it's recommended to create a `whitelist.txt` file and enter each whitelisted address on a separate line in the file.
- Remove from Whitelist After 1 Payment... = If set to True, once the first payment matching the criteria for a given sender is discovered and recorded, the sender is removed from the whitelist.  Otherwise they can continue to be matched for future matching payments if set to False (it won't remove them from whitelist)
- Watched Wallet skey File Path = This is the .skey file often within a wallets folder and with a .vkey file
- SmartContract File Path = This is the path to the .plutus file, which would have been compiled from the .hs source file for the SmartContract
- Token Policy ID = The long ID string from a given token/nft for example in `23285254a914351bf6efdb3fcb52e1bd76c8a89931b15c2fc108ca82.TheBeginning` the string beginning with '232' and ending before the dot in 'ca82' is the Token Policy ID
- Token Name = The name following the Token Policy ID and dot, for example in `23285254a914351bf6efdb3fcb52e1bd76c8a89931b15c2fc108ca82.TheBeginning` the name is just simple: TheBeginning
- Lovelace amount to expect and watch for = This is the "watch amount" which triggers a matching transaction based on the amount being sent and possibly paired with the sender's address. It is not required and if left blank transactions of ANY amount will be considered a possible match...and if no whitelist or single recipient is set, this would equate to all transactions being a match.  Or if this is left empty and a whitelist or address is being watched for, any amount from the watched-for sender will be considered a match.  And of course, an amount will match only if the transaction matches this amount set in this setting (and the address matching if set).  So if this is setup for expecing payment in a large sum for a single sale of say an NFT, this field would need the expected full amount set as well as the next setting of Price would also need the same lovelace amount set.  This way it is watching for the sale amount and will also send that to the owner of the watch address.
- Lovelace price = This is the price of a sale if any, can be left empty but it can also be used as a means of protecting a SmartContract in some scenarios by providing a certain transaction amount to be performed from the watched wallet back to itself. The SmartContract would then be set to validate that transaction, making it harder for someone directly interacting with the smart contract to change criteria..such as a whitelist..by making it impractical for them.  For example, in the giveaway we ran for JUDE tokens on CypherMonks.com, we configured our SmartContract to only allow the JUDE tokens to be taken from the SmartContract in any quantity if a large payment of 20 ADA was sent to our watched/owned address. So someone could cheat the system, bypass the whitelist, and directly engage our smart contract, but they would have had to study our transactions to understand this particular criteria and then paid us 20 ADA to take any amount of JUDE left in the contract. So the price field can be used in this way and in cooperation with the settings in your validator in your SmartContract. And of course if you are selling something for a set price, this is where you put that in Lovelace...and make sure you also put the same amount in the previous field of the amount to expect and watch for.
- Token quantity amount.. = This is the amount of token to give in each transaction. For example, in our JUDE giveaway this was set to 290. The change is then returned to the SmartContract with the correct Datum, etc, after each trade transaction goes through. If this is setup for an NFT, this would be set to 1 of course or unless you have a more complext SmartContract to engage.
- Lovelace amount to return with the transfer.. = This is a required amount and needs to be at least what the protocol parameters need according to the type of native token. For most NFT or simple native token transfers, given this is setup to do a single policy at this time, is going to be around 1.5 ADA is usually safe, so you would enter 1500000 here
- Single Recipient Wallet Address = If you are watching for payments only from a single address or wallet of addresses, even if so I recommend using the whitelist.

## A few dev notes
SeaMonk is meant to work with a SmartContract but can be slightly altered to work with just wallets. It's a matter of minor edits to the lines of code where the tx-outs are being formatted and ordered. The order of those lines matters when dealing with SmartContracts. The transaction going out to the SmartContract must always be the final tx-out right before the Datum is called, otherwise it will not get the Datum and the transaction will be locked in the SmartContract forever.

## Mentions
I stumbled upon a really fantastic repository from [logicalmechanism](https://github.com/logicalmechanism/Token-Sale-Plutus-Contract) recently, which inspired me to develop SeaMonk, drawing on some of the functionality laid out within their Python scripts, and we adapted the version of our plutus smart contract we used from one of the smart contracts we found in that same repository. So a huge shout out to them!
