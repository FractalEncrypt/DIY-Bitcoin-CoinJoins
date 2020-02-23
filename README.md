# DIY-Bitcoin-CoinJoins
Bitcoin Privacy For All - Simple DIY CoinJoins 

This is a work in progress, all input is greatly appreciated!

# Objectives

Objectives
- Document some random thoughts on CoinJoins in hopes of making this accessible to others
- Outline a simple 2-input/2-output self CoinJoin
- Outline a 10-input/10-output self CoinJoin
- Outline Pre-Mix concept
- Outline equal outputs
- Outline unequal outputs
- Outline equal change outputs
- Outline the PSBT workflow for multiple participant CoinJoins
- Outline generational CoinJoins with new incoming unmixed UTXOs (mix of 5, 3 remix, 2 new)
- Automation through code
  - plugin /on top of Bitcoin Core(?)
  - View all UTXOs
  - View UTXO status (Unmixed, Mixed, Remixed)
  - Automation of all UTXO mixing, creation of even-value change outputs
   - Multi-level-even-value-outputs will be required in most cases (EvenAmount1,EvenAmount2,...)
- Outline ideas for post CoinJoin use
  - Cold Storage
  - Obsfuscated Merchant Payments by coinjoining "change outputs"
  - Normal Merchant Payments
  - PayJoins with merchants using PSBT at POS
- Costs and fees?
- if you are okay with waiting (Cold storage), you can do this with very low fees. 
- fees are paid per "byte" 
- "bytes" are a tad mysterious as to how they're calculated to me at this time - looking for community input here please.
 - Bitcoin Core will not allow fees below 1 (or 1.5) sats/byte
 - 500 sats seems sufficient for slow confirmation time fees for the 2-input/2-output CoinJoins
 - 1500 sats for 10-in/10-out CoinJoin transactions
- Testnet ethos- Do not use these methods on mainnet without community review of these ideas.
  - The best way to learn is by doing. 
    - Play on testnet, do a few rounds of these manual CoinJoins to build your skills
    - Practice may not make perfect, but it'll get you more comfortable with the process

# Creating a simple 2-Input/2-Output CoinJoin Transaction

Creating a simple 2-Input/2-Output CoinJoin Transaction
understand that fees are simply the differnce between the input value and the output values - BE SUPER INSANELY CARFEUL. LIKE TRIPLE CHECK THIS SHIT. Always know the exact values of inputs/outputs/fees, and make sure everything matches by using the decode features on your raw transaction hex's.

- listunspent
  - this returns a list of all your unspent outputs (UTXOs)
  - we are going to need the "txid" and "vout" info for each UTXO
- Add the values of the UTXOs to coinjoin
  - subtract an amount for the fees (Fees = F)
  - Divide the remaining value by 2
    - (UTXO1+UTXO2) = X
    - (X-F)/2 = CoinJoin_Output
- getnewaddress
  - we're going to need 2 new receive addresses
- createrawtransaction
  - Here we create our transaction using the txid+vout for each UTXO, the 2 new receive addresses, and the CoinJoin_Output
- This will return a raw hex
- Sign Transaction
  - signrawtransactionwithwallet rawtxhex
  - should return "complete" :true
- Decoderawtransaction to double-check
- Sendrawtransaction
- This returns a tx id
- Paste ID into block explorer

# Creating a 10-Input/10-Output CoinJoin Transaction

Creating a 10-Input/10-Output CoinJoin Transaction
- listunspent
  - this returns a list of all your unspent outputs (UTXOs)
  - we are going to need the "txid" and "vout" info for each UTXO
- Add the values of the UTXOs to coinjoin
  - subtract an amount for the fees (Fees = F)
  - Divide the remaining value by 10
    - (UTXO1+UTXO2+UTXO3+UTXO4+UTXO5+UTXO6+UTXO7+UTXO8+UTXO9+UTXO10+) = X
    - (X-F)/10 = CoinJoin_Output
- getnewaddress
  - we're going to need 2 new receive addresses
- createrawtransaction
  - Here we create our transaction using the txid+vout for each UTXO, the 2 new receive addresses, and the CoinJoin_Output
- This will return a raw hex
- Sign Transaction
  - signrawtransactionwithwallet rawtxhex
  - should return "complete" :true
- Decoderawtransaction to double-check
- Sendrawtransaction
- This returns a tx id
- Paste ID into block explorer

# Equal and Unequal CoinJoin Outputs

Equal and Unequal CoinJoin Outputs

Equal CoinJoin Outputs
- Assume total value of inputs = 0.5 BTC
- Assume individual value outputs = 0.1 BTC each
    - 5 equal coinjoined outputs are created
    
Unequal CoinJoin Outputs
- Assume total value of inputs = 0.5 BTC
- Assume different individual value outputs = 0.1, 0.1, 0.1, 0.05, 0.05, 0.05, 0.025, 0.025
    - 8 coinjoined outputs are created with different values
    - Each value is always created as a pair (or multiples)
 - The more possible combinations between input and output the better for breaking deterministic heuristics
- even amounts of inputs and outputs in multiple rounds would be required for best practices
    
# Equal Change CoinJoin Outputs

Equal Change CoinJoin Outputs

Sometimes (or most times) you will be unable to divide your unspent UTXOs evenly into nice clean amounts, so you may end up with odd-valued change amounts. We have a few options here;
- Assume unspent UTXOs in the amount of 0.58472788
 - User may wish to create 5 outputs of 0.1 and will be left with remaining change amount of 0.08472788
  - This change can be divided into 2 or more equal outputs
  - It can be paid in miners fees if it is insignifigant
  - The user may instead choose to forgo change, and allow for odd-amount outputs
   - This would seem to be better for privacy, but maybe not?
   - 0.58472788 divided into 5 equal outputs of 0.116945576‬ would retain the value in the least amount of outputs
   - creating many outputs of very low value should be avoided to be mindful of future dust limitations.
   
# PSBT workflow for multiple participant CoinJoins

PSBT workflow for multiple participant CoinJoins
 1. Createpsbt
 2. Have individual users send their PSBTs to coordinator
 3. Coordinator uses joinpsbts to join the separate PSBTs into one
 4. Coordinator sends joined PSBT back to users
 5. Each user signs with walletprocesspsbt and sends back to coordinator
 6. Coordinator uses combinepsbt.
 7. Coordinator uses finalizepsbt and then transmits to the network using sendrawtransaction.
 
Use decodepsbt on the combined PSBT to view the fees that you will be paying. TRIPLE CHECK THIS - is it what you expect?
