# Smart property issuance scripts

## Dependencies: 

You'll need python2.7.6 and a synced bitcoind node (Unlike the rest of mastercoin-tools which uses obelisk).

## Setup:

1. Make sure bitcoind is set up right.

   ```
cmlacy@blockchain-2:~$ bitcoind getinfo
   ```
   
2. Create ``~/.bitcoin/bitcoin.conf`` if it complains.

   ```
server=1
daemon=1
rpcuser=<username>
rpcpassword=<password>
rpcallowip=<local IP>
   ```
   
 Alternatively if you have a remote bitcoind instance you can use that as well. On the machine where you will run the scripts create ``~/.bitcoin/bitcoin.conf`` with the following lines:

   ```
rpcuser=<remote username>
rpcpassword=<remote password>
rpcconnect=<Remote Server IP>
rpcport=<Remote Port>
   ```
   
 Dont' forget to add a whitelist entry (rpcallowip) to your remote bitcoind (or bitcoin-qt) for the ip address of the machine the scripts will run from. 

3. Install libraries

   ```
$ sudo apt-get install python-pip
$ sudo pip install bitcoin-python pybitcointools
   ```
   
4. Get these scripts

   ```
$ git clone https://github.com/hyperfocus/mastercoin-tools.git
$ cd mastercoin-tools/scripts
   ```

## Creating a fixed-issuance smart property:
   
1. Create .json file which describes the currency.  Note: 
   ecosystem: 1=MSC, 2=TMSC
   property_type: 1=Indivisible, 2=Divisible
   previous_property_id: 0 if not amending an existing property.
   previous_property_id: 0 if making a new property.

   ```
{
  "transaction_type": 50,
  "ecosystem": 2,
  "property_type": 2,
  "previous_property_id": 0,
  "property_category": "Testing",
  "property_subcategory": "Smart Property Test Sequence 1",
  "property_name": "Doubloons",
  "property_url": "https://docs.google.com/a/engine.co/spreadsheet/ccc?key=0Al4FhV693WqWdGNFRDhEMTFtaWNmcVNObFlVQmNOa1E&usp=drive_web#gid=0",
  "property_data": "Test Issuing a new Currency",
  "number_properties": 1000,
  "transaction_from": "1GoTC332Lr9ycKerCP9n7Pnri8e3KPeDLR",
  "from_private_key": "5K34DNVDzUQHFwGMdgF6tS8TE8poHXoYbNA9JT7feVdQGL4u67zS"
}
   ```
   
2. Run the script with your .json file as input.

   ```
$ cat doubloons.json | python generateTX50_SP.py 
   ```
3. It may return an error:

   ```
{"status": "NOT OK", "fix": "bitcoind importprivkey 5K34DNVDzUQHFwGMdgF6tS8TE8poHXoYbN9JT7feVdQGL4u67zS imported_1397503463", "error": "Couldn't find address in wallet, please run 'fix' on the machine"}
   ```
   In that case, run the command listed in “fix” to correct it, with “false” added at the end to suppress rescanning the blockchain:
   ```
$ bitcoind importprivkey 5K34DNVDzUQHFwGMdgF6tS8TE8poHXoYbN9JT7feVdQGL4u67zS imported_1397503463 false
   ```
   
4. And run again:

   ```
$ cat doubloons.json | python generateTX50_SP.py
   ```

5. This outputs the actual raw transaction in hex:

   ```
{"rawtransaction": {"hex": "0100000001e604d2bbdec6071b4ca4f160629799f63bd198310e133920ec10cadc0aa5473d000000008c493046022100fedcf3465b9f348d869c886d86b8daf89cf78712c149ece5475c2be11d5d423c022100b404b1b29267e80c371bf34f70dd330234844f6cc54410d86fdc1434b9326409014104e1d72d21642d38252d08305a9f299a317d5f16a7ddf5b8d7aa686ca2d3476740dccd4d8348bf78bdded9998d037bb9fe1418edc5baccd902b76313d5f4b90d1cffffffff067c150000000000001976a914946cb2e08075bcbaf157e47bcb67eb2b2339d24288ac44480000000000001976a914ad52e22ed4d2680504ed9da83de49194c79a02d188ac744000000000000089514104e1d72d21642d38252d08305a9f299a317d5f16a7ddf5b8d7aa686ca2d3476740dccd4d8348bf78bdded9998d037bb9fe1418edc5baccd902b76313d5f4b90d1c2102b7ffeca977e95e14e2eb032797719d0ebba2c07fbbe8298c4f138cf3fbfe611c2102cbe73cdd70750603842c54d64fdd926ace126d688954a9e33adc0119cdf9500a53ae744000000000000089514104e1d72d21642d38252d08305a9f299a317d5f16a7ddf5b8d7aa686ca2d3476740dccd4d8348bf78bdded9998d037bb9fe1418edc5baccd902b76313d5f4b90d1c2102b9f9fcd9a4703c6f70355a487d986fe2b003d1fe48d5cae2213b767b9f0ccb54210218fc5ccfdf8372472fb178423cd809b2826266209a911f88a4912973805a179753ae744000000000000089514104e1d72d21642d38252d08305a9f299a317d5f16a7ddf5b8d7aa686ca2d3476740dccd4d8348bf78bdded9998d037bb9fe1418edc5baccd902b76313d5f4b90d1c210297f9bb89d9503791766407733edd01fd53915f4cf9877bf2b8bf60dddc84c77d2102ce468c241932479ca7f330909dfc5fccf2ef2b9a2cf2be7f8621250f03dbcc8553ae744000000000000089514104e1d72d21642d38252d08305a9f299a317d5f16a7ddf5b8d7aa686ca2d3476740dccd4d8348bf78bdded9998d037bb9fe1418edc5baccd902b76313d5f4b90d1c2102c40a53b580dd3aff657f2b4d8229f36367f004d68adfa375c8e439f0a7a7e8092102010e97a73c8f178099968488006e69e78908b00bf23a016fda17702f2e39b90b53ae00000000", "complete": true}}
   ```

6. Send the transaction by copying and pasting that hex string (without its quotes) as an argument to ``bitcoind sendrawtransaction``

   ```
bitcoind sendrawtransaction 0100000001e604d2bbdec6071b4ca4f160629799f63bd198310e133920ec10cadc0aa5473d000000008c493046022100fedcf3465b9f348d869c886d86b8daf89cf78712c149ece5475c2be11d5d423c022100b404b1b29267e80c371bf34f70dd330234844f6cc54410d86fdc1434b9326409014104e1d72d21642d38252d08305a9f299a317d5f16a7ddf5b8d7aa686ca2d3476740dccd4d8348bf78bdded9998d037bb9fe1418edc5baccd902b76313d5f4b90d1cffffffff067c150000000000001976a914946cb2e08075bcbaf157e47bcb67eb2b2339d24288ac44480000000000001976a914ad52e22ed4d2680504ed9da83de49194c79a02d188ac744000000000000089514104e1d72d21642d38252d08305a9f299a317d5f16a7ddf5b8d7aa686ca2d3476740dccd4d8348bf78bdded9998d037bb9fe1418edc5baccd902b76313d5f4b90d1c2102b7ffeca977e95e14e2eb032797719d0ebba2c07fbbe8298c4f138cf3fbfe611c2102cbe73cdd70750603842c54d64fdd926ace126d688954a9e33adc0119cdf9500a53ae744000000000000089514104e1d72d21642d38252d08305a9f299a317d5f16a7ddf5b8d7aa686ca2d3476740dccd4d8348bf78bdded9998d037bb9fe1418edc5baccd902b76313d5f4b90d1c2102b9f9fcd9a4703c6f70355a487d986fe2b003d1fe48d5cae2213b767b9f0ccb54210218fc5ccfdf8372472fb178423cd809b2826266209a911f88a4912973805a179753ae744000000000000089514104e1d72d21642d38252d08305a9f299a317d5f16a7ddf5b8d7aa686ca2d3476740dccd4d8348bf78bdded9998d037bb9fe1418edc5baccd902b76313d5f4b90d1c210297f9bb89d9503791766407733edd01fd53915f4cf9877bf2b8bf60dddc84c77d2102ce468c241932479ca7f330909dfc5fccf2ef2b9a2cf2be7f8621250f03dbcc8553ae744000000000000089514104e1d72d21642d38252d08305a9f299a317d5f16a7ddf5b8d7aa686ca2d3476740dccd4d8348bf78bdded9998d037bb9fe1418edc5baccd902b76313d5f4b90d1c2102c40a53b580dd3aff657f2b4d8229f36367f004d68adfa375c8e439f0a7a7e8092102010e97a73c8f178099968488006e69e78908b00bf23a016fda17702f2e39b90b53ae00000000
6e5c37b1d408faba6a028ed1ef4e9a753c3720ee2551fa93c6f87bd7e1afa3ac
   ```

7. The output will be the transaction hash ID.  Check http://blockchain.info to see the status of the transation.  You can also find the transaction via blockchain.info's page for the issuing address.

## Creating a variable-issuance crowdsale property:
   
1. Create .json file which describes the currency.  Note: 
   ecosystem: 1=MSC, 2=TMSC
   previous_property_id: 0 if not amending an existing property.
   currency_identifier desired: what you want these to be bought with.  1=MSC, 2=TMSC
   number_properties: How many of this property buyers should get for each <currency_identifier> spent.
   deadline: UNIX timestamp (UTC timezone) of the crowdsale end.  See http://www.onlineconversion.com/unix_time.htm
   ```
{
  "transaction_type": 51,
  "ecosystem": 2,
  "property_type": 2,
  "previous_property_id": 0,
  "property_category": "Testing",
  "property_subcategory": "Smart Property Test Sequence 1",
  "property_name": "Ducats",
  "property_url": "https://docs.google.com/a/engine.co/spreadsheet/ccc?key=0Al4FhV693WqWdDZ2R2tfakJKOGF4VThNSmNOTjR5YlE&usp=drive_web#gid=0",
  "property_data": "Test issuing a new crowdsale",
  "currency_identifier_desired": 2,
  "number_properties": 10,
  "deadline": 1397869200,
  "earlybird_bonus": 0,
  "percentage_for_issuer": 0,
  "transaction_from": "1LBB97UAhyQjtJJwk71LvzijnZw7o29XRm",
  "from_private_key": "5J6TFQ2ULGHErgFX23qeBQVeragqqjTwdnLcFpCLpKp5HFagWEZ"
}
   ```
   
2. Run the script with your .json file as input.

   ```
$ cat ducats.json | python generateTX51_SP.py 
   ```
3. It may return an error:

   ```
{"status": "NOT OK", "fix": "bitcoind importprivkey 5J6TFQ2ULGHErgFX23qeBQVeragqqjTwdnLcFpCLpKp5HFagWEZ imported_1397611891", "error": "Couldn't find address in wallet, please run 'fix' on the machine"}
   ```
   In that case, run the command listed in “fix” to correct it.
   ```
$ bitcoind importprivkey 5J6TFQ2ULGHErgFX23qeBQVeragqqjTwdnLcFpCLpKp5HFagWEZ imported_1397611891
   ```
   
4. And run again:

   ```
$ cat ducats.json | python generateTX51_SP.py 
   ```

5. This outputs the actual raw transaction in hex:

   ```
{"rawtransaction": {"hex": "0100000001e604d2bbdec6071b4ca4f160629799f63bd198310e133920ec10cadc0aa5473d000000008c493046022100fedcf3465b9f348d869c886d86b8daf89cf78712c149ece5475c2be11d5d423c022100b404b1b29267e80c371bf34f70dd330234844f6cc54410d86fdc1434b9326409014104e1d72d21642d38252d08305a9f299a317d5f16a7ddf5b8d7aa686ca2d3476740dccd4d8348bf78bdded9998d037bb9fe1418edc5baccd902b76313d5f4b90d1cffffffff067c150000000000001976a914946cb2e08075bcbaf157e47bcb67eb2b2339d24288ac44480000000000001976a914ad52e22ed4d2680504ed9da83de49194c79a02d188ac744000000000000089514104e1d72d21642d38252d08305a9f299a317d5f16a7ddf5b8d7aa686ca2d3476740dccd4d8348bf78bdded9998d037bb9fe1418edc5baccd902b76313d5f4b90d1c2102b7ffeca977e95e14e2eb032797719d0ebba2c07fbbe8298c4f138cf3fbfe611c2102cbe73cdd70750603842c54d64fdd926ace126d688954a9e33adc0119cdf9500a53ae744000000000000089514104e1d72d21642d38252d08305a9f299a317d5f16a7ddf5b8d7aa686ca2d3476740dccd4d8348bf78bdded9998d037bb9fe1418edc5baccd902b76313d5f4b90d1c2102b9f9fcd9a4703c6f70355a487d986fe2b003d1fe48d5cae2213b767b9f0ccb54210218fc5ccfdf8372472fb178423cd809b2826266209a911f88a4912973805a179753ae744000000000000089514104e1d72d21642d38252d08305a9f299a317d5f16a7ddf5b8d7aa686ca2d3476740dccd4d8348bf78bdded9998d037bb9fe1418edc5baccd902b76313d5f4b90d1c210297f9bb89d9503791766407733edd01fd53915f4cf9877bf2b8bf60dddc84c77d2102ce468c241932479ca7f330909dfc5fccf2ef2b9a2cf2be7f8621250f03dbcc8553ae744000000000000089514104e1d72d21642d38252d08305a9f299a317d5f16a7ddf5b8d7aa686ca2d3476740dccd4d8348bf78bdded9998d037bb9fe1418edc5baccd902b76313d5f4b90d1c2102c40a53b580dd3aff657f2b4d8229f36367f004d68adfa375c8e439f0a7a7e8092102010e97a73c8f178099968488006e69e78908b00bf23a016fda17702f2e39b90b53ae00000000", "complete": true}}
   ```

6. Send the transaction by copying and pasting that hex string (without its quotes) as an argument to ``bitcoind sendrawtransaction``

   ```
bitcoind sendrawtransaction 0100000001e604d2bbdec6071b4ca4f160629799f63bd198310e133920ec10cadc0aa5473d000000008c493046022100fedcf3465b9f348d869c886d86b8daf89cf78712c149ece5475c2be11d5d423c022100b404b1b29267e80c371bf34f70dd330234844f6cc54410d86fdc1434b9326409014104e1d72d21642d38252d08305a9f299a317d5f16a7ddf5b8d7aa686ca2d3476740dccd4d8348bf78bdded9998d037bb9fe1418edc5baccd902b76313d5f4b90d1cffffffff067c150000000000001976a914946cb2e08075bcbaf157e47bcb67eb2b2339d24288ac44480000000000001976a914ad52e22ed4d2680504ed9da83de49194c79a02d188ac744000000000000089514104e1d72d21642d38252d08305a9f299a317d5f16a7ddf5b8d7aa686ca2d3476740dccd4d8348bf78bdded9998d037bb9fe1418edc5baccd902b76313d5f4b90d1c2102b7ffeca977e95e14e2eb032797719d0ebba2c07fbbe8298c4f138cf3fbfe611c2102cbe73cdd70750603842c54d64fdd926ace126d688954a9e33adc0119cdf9500a53ae744000000000000089514104e1d72d21642d38252d08305a9f299a317d5f16a7ddf5b8d7aa686ca2d3476740dccd4d8348bf78bdded9998d037bb9fe1418edc5baccd902b76313d5f4b90d1c2102b9f9fcd9a4703c6f70355a487d986fe2b003d1fe48d5cae2213b767b9f0ccb54210218fc5ccfdf8372472fb178423cd809b2826266209a911f88a4912973805a179753ae744000000000000089514104e1d72d21642d38252d08305a9f299a317d5f16a7ddf5b8d7aa686ca2d3476740dccd4d8348bf78bdded9998d037bb9fe1418edc5baccd902b76313d5f4b90d1c210297f9bb89d9503791766407733edd01fd53915f4cf9877bf2b8bf60dddc84c77d2102ce468c241932479ca7f330909dfc5fccf2ef2b9a2cf2be7f8621250f03dbcc8553ae744000000000000089514104e1d72d21642d38252d08305a9f299a317d5f16a7ddf5b8d7aa686ca2d3476740dccd4d8348bf78bdded9998d037bb9fe1418edc5baccd902b76313d5f4b90d1c2102c40a53b580dd3aff657f2b4d8229f36367f004d68adfa375c8e439f0a7a7e8092102010e97a73c8f178099968488006e69e78908b00bf23a016fda17702f2e39b90b53ae00000000
6e5c37b1d408faba6a028ed1ef4e9a753c3720ee2551fa93c6f87bd7e1afa3ac
   ```

7. The output will be the transaction hash ID.  Check http://blockchain.info to see the status of the transation.  You can also find the transaction via blockchain.info's page for the issuing address.

## Manually closing a crowdsale
   
1. Create .json file which describes the currency.  Note: 
   property_id: the property ID of the property to cancel.
   ```
{
  "transaction_type": 53,
  "property_id": 2147483658,
  "transaction_from": "1LBB97UAhyQjtJJwk71LvzijnZw7o29XRm",
  "from_private_key": "5J6TFQ2ULGHErgFX23qeBQVeragqqjTwdnLcFpCLpKp5HFagWEZ"
}
   ```
   
2. Run the script with your .json file as input.

   ```
$ cat cancel-ducats.json | python generateTX53_SP.py 
   ```
3. It may return an error:

   ```
{"status": "NOT OK", "fix": "bitcoind importprivkey 5J6TFQ2ULGHErgFX23qeBQVeragqqjTwdnLcFpCLpKp5HFagWEZ imported_1397611891", "error": "Couldn't find address in wallet, please run 'fix' on the machine"}
   ```
   In that case, run the command listed in “fix” to correct it.
   ```
$ bitcoind importprivkey 5J6TFQ2ULGHErgFX23qeBQVeragqqjTwdnLcFpCLpKp5HFagWEZ imported_1397611891
   ```
   
4. And run again:

   ```
$ cat cancel-ducats.json | python generateTX53_SP.py 
   ```

5. This outputs the actual raw transaction in hex:

   ```
False False True
{"rawtransaction": {"hex": "010000000153eeb76c1e21677c1e683a18755a8b567da626a0870dee8c7b5d18e70f0dc9ec010000008b483045022100f23b95a045a93429e602d02de2cccae010ad1eb1ed0639c7a9a162c10bcddd33022002d09cd7963a741c7d6636d0afa25bad47650d5857a6cffe28e8f3e80f1ba69d0141047905461a24edc485b6e4fd98a97cc4d12ea7c2e25c187c5f13e461d31e37e6c118a74add7cc146cadb0f3f1cf25cb503347706823f044b7682cd535035e8cc7affffffff037c150000000000001976a914946cb2e08075bcbaf157e47bcb67eb2b2339d24288ac683c0100000000001976a914d257038a962bd29a40268f939c91b69523ed17c888acf82a000000000000675141047905461a24edc485b6e4fd98a97cc4d12ea7c2e25c187c5f13e461d31e37e6c118a74add7cc146cadb0f3f1cf25cb503347706823f044b7682cd535035e8cc7a210281c10f942911b31414f489f86d5fd4d5cd44038e6776d8c4163b10a721acacdd52ae00000000", "complete": true}}
   ```

6. Send the transaction by copying and pasting that hex string (without its quotes) as an argument to ``bitcoind sendrawtransaction``

   ```
$ bitcoind sendrawtransaction 010000000153eeb76c1e21677c1e683a18755a8b567da626a0870dee8c7b5d18e70f0dc9ec010000008b483045022100f23b95a045a93429e602d02de2cccae010ad1eb1ed0639c7a9a162c10bcddd33022002d09cd7963a741c7d6636d0afa25bad47650d5857a6cffe28e8f3e80f1ba69d0141047905461a24edc485b6e4fd98a97cc4d12ea7c2e25c187c5f13e461d31e37e6c118a74add7cc146cadb0f3f1cf25cb503347706823f044b7682cd535035e8cc7affffffff037c150000000000001976a914946cb2e08075bcbaf157e47bcb67eb2b2339d24288ac683c0100000000001976a914d257038a962bd29a40268f939c91b69523ed17c888acf82a000000000000675141047905461a24edc485b6e4fd98a97cc4d12ea7c2e25c187c5f13e461d31e37e6c118a74add7cc146cadb0f3f1cf25cb503347706823f044b7682cd535035e8cc7a210281c10f942911b31414f489f86d5fd4d5cd44038e6776d8c4163b10a721acacdd52ae00000000
   ```

7. The output will be the transaction hash ID.  Check http://blockchain.info to see the status of the transaction.  You can also find the transaction via blockchain.info's page for the issuing address.



## Redeeming your escrow funds:
The master protocol works by encoding its data on the blockchain using escrow funds, so each transaction can take a little bit of bitcoin and lock it away in escrow until you redeem it.  In most cases, this will be negligible, but people who make many trades, or are testing, may find an excessive amount of their bitcoin inaccessible over time.  Use the ``redeemMultiSig.py`` script to unlock this money and get it back.
   
1. Create .json file whichdefines the address to redeem, and where to send the money.
   ```
{ 
  "transaction_to": "1LBB97UAhyQjtJJwk71LvzijnZw7o29XRm",
  "transaction_from": "1LBB97UAhyQjtJJwk71LvzijnZw7o29XRm",
  "from_private_key": "5J6TFQ2ULGHErgFX23qeBQVeragqqjTwdnLcFpCLpKp5HFagWEZ",
  "fee": 0.0001
}
   ```
   
2. Run the script with your .json file as input.

   ```
$ cat redeem-escrows.json | python redeemMultiSig.py
   ```
3. It may return an error:

   ```
{"status": "NOT OK", "fix": "bitcoind importprivkey 5J6TFQ2ULGHErgFX23qeBQVeragqqjTwdnLcFpCLpKp5HFagWEZ imported_1397611891", "error": "Couldn't find address in wallet, please run 'fix' on the machine"}
   ```
   In that case, run the command listed in “fix” to correct it.
   ```
$ bitcoind importprivkey 5J6TFQ2ULGHErgFX23qeBQVeragqqjTwdnLcFpCLpKp5HFagWEZ imported_1397611891
   ```
   
4. And run again:

   ```
cmlacy@blockchain-2:~/mastercoin-tools/scripts$ cat redeem-escrows.json | python redeemMultiSig.py
   ```

5. This outputs the actual raw transaction in hex:

   ```
{"rawtransaction": {"hex": "0100000001e604d2bbdec6071b4ca4f160629799f63bd198310e133920ec10cadc0aa5473d000000008c493046022100fedcf3465b9f348d869c886d86b8daf89cf78712c149ece5475c2be11d5d423c022100b404b1b29267e80c371bf34f70dd330234844f6cc54410d86fdc1434b9326409014104e1d72d21642d38252d08305a9f299a317d5f16a7ddf5b8d7aa686ca2d3476740dccd4d8348bf78bdded9998d037bb9fe1418edc5baccd902b76313d5f4b90d1cffffffff067c150000000000001976a914946cb2e08075bcbaf157e47bcb67eb2b2339d24288ac44480000000000001976a914ad52e22ed4d2680504ed9da83de49194c79a02d188ac744000000000000089514104e1d72d21642d38252d08305a9f299a317d5f16a7ddf5b8d7aa686ca2d3476740dccd4d8348bf78bdded9998d037bb9fe1418edc5baccd902b76313d5f4b90d1c2102b7ffeca977e95e14e2eb032797719d0ebba2c07fbbe8298c4f138cf3fbfe611c2102cbe73cdd70750603842c54d64fdd926ace126d688954a9e33adc0119cdf9500a53ae744000000000000089514104e1d72d21642d38252d08305a9f299a317d5f16a7ddf5b8d7aa686ca2d3476740dccd4d8348bf78bdded9998d037bb9fe1418edc5baccd902b76313d5f4b90d1c2102b9f9fcd9a4703c6f70355a487d986fe2b003d1fe48d5cae2213b767b9f0ccb54210218fc5ccfdf8372472fb178423cd809b2826266209a911f88a4912973805a179753ae744000000000000089514104e1d72d21642d38252d08305a9f299a317d5f16a7ddf5b8d7aa686ca2d3476740dccd4d8348bf78bdded9998d037bb9fe1418edc5baccd902b76313d5f4b90d1c210297f9bb89d9503791766407733edd01fd53915f4cf9877bf2b8bf60dddc84c77d2102ce468c241932479ca7f330909dfc5fccf2ef2b9a2cf2be7f8621250f03dbcc8553ae744000000000000089514104e1d72d21642d38252d08305a9f299a317d5f16a7ddf5b8d7aa686ca2d3476740dccd4d8348bf78bdded9998d037bb9fe1418edc5baccd902b76313d5f4b90d1c2102c40a53b580dd3aff657f2b4d8229f36367f004d68adfa375c8e439f0a7a7e8092102010e97a73c8f178099968488006e69e78908b00bf23a016fda17702f2e39b90b53ae00000000", "complete": true}}
   ```

6. Send the transaction by copying and pasting that hex string (without its quotes) as an argument to ``bitcoind sendrawtransaction``

   ```
bitcoind sendrawtransaction 0100000001e604d2bbdec6071b4ca4f160629799f63bd198310e133920ec10cadc0aa5473d000000008c493046022100fedcf3465b9f348d869c886d86b8daf89cf78712c149ece5475c2be11d5d423c022100b404b1b29267e80c371bf34f70dd330234844f6cc54410d86fdc1434b9326409014104e1d72d21642d38252d08305a9f299a317d5f16a7ddf5b8d7aa686ca2d3476740dccd4d8348bf78bdded9998d037bb9fe1418edc5baccd902b76313d5f4b90d1cffffffff067c150000000000001976a914946cb2e08075bcbaf157e47bcb67eb2b2339d24288ac44480000000000001976a914ad52e22ed4d2680504ed9da83de49194c79a02d188ac744000000000000089514104e1d72d21642d38252d08305a9f299a317d5f16a7ddf5b8d7aa686ca2d3476740dccd4d8348bf78bdded9998d037bb9fe1418edc5baccd902b76313d5f4b90d1c2102b7ffeca977e95e14e2eb032797719d0ebba2c07fbbe8298c4f138cf3fbfe611c2102cbe73cdd70750603842c54d64fdd926ace126d688954a9e33adc0119cdf9500a53ae744000000000000089514104e1d72d21642d38252d08305a9f299a317d5f16a7ddf5b8d7aa686ca2d3476740dccd4d8348bf78bdded9998d037bb9fe1418edc5baccd902b76313d5f4b90d1c2102b9f9fcd9a4703c6f70355a487d986fe2b003d1fe48d5cae2213b767b9f0ccb54210218fc5ccfdf8372472fb178423cd809b2826266209a911f88a4912973805a179753ae744000000000000089514104e1d72d21642d38252d08305a9f299a317d5f16a7ddf5b8d7aa686ca2d3476740dccd4d8348bf78bdded9998d037bb9fe1418edc5baccd902b76313d5f4b90d1c210297f9bb89d9503791766407733edd01fd53915f4cf9877bf2b8bf60dddc84c77d2102ce468c241932479ca7f330909dfc5fccf2ef2b9a2cf2be7f8621250f03dbcc8553ae744000000000000089514104e1d72d21642d38252d08305a9f299a317d5f16a7ddf5b8d7aa686ca2d3476740dccd4d8348bf78bdded9998d037bb9fe1418edc5baccd902b76313d5f4b90d1c2102c40a53b580dd3aff657f2b4d8229f36367f004d68adfa375c8e439f0a7a7e8092102010e97a73c8f178099968488006e69e78908b00bf23a016fda17702f2e39b90b53ae00000000
6e5c37b1d408faba6a028ed1ef4e9a753c3720ee2551fa93c6f87bd7e1afa3ac
   ```

7. The output will be the transaction hash ID.  Check http://blockchain.info to see the status of the transation.  You can also find the transaction via blockchain.info's page for the issuing address.



