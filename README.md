# SeaMonk
A gateway application which facilitates a simple->complex transaction incorporating a Smart Contract on the Cardano blockchain.

SeaMonk makes it possible for a user to send any amount of ADA which is either expected as an exact amount, any amount, or an amount over a minimum...and perform a complex transaction as a result, such as a Token Swap in either a sale, an exchange 1-sided dex, a simple OTC trade, or a giveaway or similar...in all cases utilizing a smart contract but without the need for the user to input datum, redeemer, or include the plutus script in a tx.  

SeaMonk monitors all incoming tx's for a given wallet and can match and act accordingly on amount and address of the sender. It can run in transaction logging mode only as well, to simply log all wallet transactions, written to a file. It can also be run to simply mint an NFT manually. Future feature will incorporate this capability via an automatic minting feature, which will mint an NFT with preset data on-the-fly when a particular type of payment is received. When NFT autominting-on-payment-type mode is running, an important feature is allowing to pass dynamic data to the NFT's metadata at mint time. This allows adding something custom like the buyer's address, the tx hash, or even (with some modification of the source code) data from a matching list of options associated with the whitelist...for example a buyer's name or other info they want minted on it if they win.

Payments received that are outside constraints on any mode in which SeaMonk runs, can also be set to automatically refund the sender. All transactions, whether fulfilling a swap or auction event or NFT minting or refunds, are handled in an optimized fashion, reducing fees. For example, if a smart contract is low on balance for swaps and auto-replenish is enabled, SeaMonk won't replenish it until absolutely necessary (like when a swap request exceeds the avail SC balance) and will do so in the same transaction as fulfilling the swap request. This involves what would normally be 3 transactions. 

SeaMonk can also be used without a smart contract with some minor adjustments, acting as an active gateway application to "watch for" specified amounts or sending addresses and performing tasks or creating transactions according to criteria. However it is developed with the incorporation of smart contracts in mind, as a way to allow for anyone with any Cardano wallet, to interact with arguably any Smart Contract, according to the criteria setup within SeaMonk and the Smart Contract.

## How To Use
After initial setup and config, SeaMonk runs continuously (until process killed) performing two primary functions: Getting and logging new transactions into watched wallet; Performing swaps on any incoming transactions which match criteria setup in the running profile.

SeaMonk has a simple swapping Cardano smart contract source built in, which will attempt to compile if no other smartcontract is provided during setup. This simple smart contract validates based on a tx-out being present when redeeming which possesses the watched wallet address and an amount set in the settings of SeaMonk for "price". This can be the actual price of an NFT deposited for sale and available via either direct interaction with the SC or using SeaMonk as a simple gateway to the SC...or else is used as a validation security layer for protecting an SC when many tokens are available for trade to many via SeaMonk gateway.  

For example, if you have native tokens you want to make available for swap/dex at a set price per token, you can utilize the same smartcontract and protect the total sum against direct SC interaction by simply setting the price to equal what the total sum would be in a swap/dex trade. Say you have 100,000 native tokens to trade at a rate of 100 per 1 ADA. You can enforce a required tx equal to 1,000 ADA paid to yourself which must be included in any trade to withdraw any amount of tokens from the SC..thereby ensuring trading is fair and protecting against someone emptying the SC at a much lower rate otherwise intended...and giving a simple smartcontract more flexibility and complexity via SeaMonk. If you don't have enough ADA to do such a price requirement setting in SeaMonk, there is a built-in feature which allows for a smaller amount and to utilize the SC in "batches" of smaller amounts which are replenished when the SC is low on token balance. You can, for example, set the tokens to be deposited to 10,000 in our example rather than the full 100,000, and set the Recurring setting to True and your price to 100 ADA, which protects your 10K at your rate of 100/1. When the balance is too low on the SC to fulfill a swap request, SeaMonk will replenish the SC and perform the requested swap (if within params and criteria) in a single tx, saving on fees by replenishing the balance only when a swap request triggers it, so no additional high fees are paid by you and the difference is very slight.

### Some of SeaMonk Features
1. Simultaneous tx logging and processing
It's important to keep an accurate log of incoming transactions and not cause unnecessary delays while processing swaps or other transactions. To do this SeaMonk runs a seperate thread of getting transactions, every 2 seconds, regardless of other functions being performed. 

2. TX status tracking
When a transaction is performed by SeaMonk, no further functions will execute (other than the tx logger thread) until the processed TX shows up in the watched wallet on the blockchain in real time.

3. Whitelisting
SeaMonk can be run with or without a whitelist. When running with a whitelist only added addresses are considered. Without a whitelist, all transactions are checked (besides self: watched address or SC address). Matching whitelisted addresses is only 1 criteria and a transaction may also be matched against the amount sent by the address. If setup to do so, SeaMonk will also remove addresses from the whitelist after a single swap is performed...or they can remain indefinitely per your setup. In the case where the whitelist is setup to only allow 1 swap, if an address tries to swap and fails due to trying to swap too much or too little, they will not be removed from the whitelist. Only a valid swap will trigger a removal when set to 1 swap.

4. Criteria
SeaMonk can be setup with various criteria: whitelist, specific amount sent, min-amount sent, price sent. Specific amount matches an exact amount of ADA sent. This can be the price, as in an NFT sale, and in such a case you would enter the same amount for both settings in setup, Price and the Watch for Amount would match to only match on transactions which match the price. If the matching criteria is a specific amount which may not be a price, in a case where you are doing a native token giveaway for example, you can set the matching amount to for example 1.5 ADA and the price to a higher amount such as 100 ADA for example, to protect your "lump sum" of native tokens from being all taken rather than doing a giveaway with a whitelist.  You may also want to do a native token swap/dex setup, wherein you are selling tokens at a rate, like 100 per 1 ada for example...in which case you would leave the watch-amount blank in settings and it would prompt you for a min-amount which is the minimum someone needs to buy in the swap setup. For example, you can set this min to 5 ADA, meaning only payments over 5 ADA will be considered for matching.

5. Handling Refunds
In the case where someone sends more ADA than can be fulfilled in a swap, this is automatically refunded to the sender, minus a fee you set in setup. For example, you may have your SC setup for recurring/replenishing as you have only 10K tokens in it at any time, so you can cover them with a 100 ADA price script validation requirement as previously discussed...in this case the maximum tokens a single swap can handle is also 10K tokens. If someone sends 200 ADA for 20K tokens, it will be refunded in this situation, minus the fee you determine.  At this point below-min txs are not refunded automatically by SeaMonk. 

6. Logging
Logging is still a work in progress, however a run.log file is generated and written to for most important info and errors for each profile run. In addition transactions.log and payments.log are created and appended to as transactions and payments are found and processed.  Transactions log is a log of ALL transactions for the watched wallet address, from the start of run of SeaMonk. This way as the balance and UTxOs change with transactions by the watched wallet, either initiated by SeaMonk or you, there should be no gaps in transaction history and kept in the profiles directory for that profile.  Payments log is updated with completed swaps and any refund which exceeds the token trade capacity.

7. Automatic SC Replenishing
As mentioned, SeaMonk will always attempt to replenish a low SmartContract token balance when setup to do so. It will only do this if needed to perform a valid swap transaction, and will do so in the same transaction to keep fees to the minimum.

TODO: Other features will be outlined soon or can be seen in the source code.


## Running SeaMonk
### Prereq
You'll need to have Cardano node and cli installed/synced on the server/system where you'll run SeaMonk. You'll also want your watched wallet to be on this same system, with its .skey and .vkey files accessible. The smartcontract you'll be using will also need to be on this system.

For now SeaMonk has only been tested on Debian and Ubuntu, but should work flawlessly on any system where you can run Python. The file paths for settings config do need to be full paths, so if your wallet skey files are in a wallet folder in the same folder with SeaMonk, you'll still want to enter the full path to that skey file for example.  The whitelist needs to be in the same folder as seamonk.py, named simply whitelist.txt.

Before you run or configure, you'll need to adjust the variables in the top of the cardanotx.py file, to match whether you are using mainnet or testnet and which blockfrost also, mainnet or testnet.

### First-run
On first run, just simply `python3 seamonk.py`, SeaMonk will walk you through configuration and generate a profile.json file with your settings. To change these settings, you can run `python3 seamonk.py reconfigure`

### Deposit
After it's setup, you will need to fund the SmartContract. Running `python3 seamonk.py deposit` will walk you through this process.

### Get Transactions
It's recommended to run SeaMonk and configure the "Check" option to true during setup, to allow getting transactions in a threaded way, with monitoring and acting on transactions. This logs all transactions to a transactions.log file for the profile and is what the process checks payments against. However it's also possible to run SeaMonk only to gather and log all transactions. To run this, you simply issue `python3 seamonk.py --option get_transactions` and it will watch for transactions and log them and that is all.

### Main Running App
The main program of SeaMonk both logs and matches new transactions to the whitelist or other criteria and when a payment is detected as a match, records them and performs the required SmartContract transaction(s), either swapping, refunding, or replenishing the SC if needed (while performing the swap that triggered it to run too low). To run the main SeaMonk loop, issue simply `python3 seamonk.py` and it will loop until you kill the process.

You can run SeaMonk in the background by putting `setsid` in front of the command in linux, for example: `setsid python3 seamonk.py` will start SeaMonk in the background. There's a run.log file generated each time you run which will capture most outputs and errors, but is still a WIP.

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

TODO: This is an incomplete list, will update soon

## A few dev notes
SeaMonk is meant to work with a SmartContract but can be slightly altered to work with just wallets. It's a matter of minor edits to the lines of code where the tx-outs are being formatted and ordered. The order of those lines matters when dealing with SmartContracts. The transaction going out to the SmartContract must always be the final tx-out right before the Datum is called, otherwise it will not get the Datum and the transaction will be locked in the SmartContract forever.

For NFT Minting do these steps in this order:
1. prepare the minting wallet, fund it, then fund self so latest txs are from self
2. setup seamonk in its own folder. have all wallet skey and vkey file paths ready, blockfrost ID, and know how locking height will work.
3. setup images and locate file paths for each, any alterations to the html template if changing the format etc.
4. run seamonk first time setup, notate the block height resultant (make sure it will work) and the json file to customize
5. customize json file and save
6. run seamonk and go live

## Mentions
I stumbled upon a really fantastic repository from [logicalmechanism](https://github.com/logicalmechanism/Token-Sale-Plutus-Contract) recently, which inspired me to develop SeaMonk, drawing on some of the functionality laid out within their Python scripts, and we adapted the version of our plutus smart contract we used from one of the smart contracts we found in that same repository. So a huge shout out to them!
