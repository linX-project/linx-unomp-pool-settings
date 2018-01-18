# linx-unomp-pool-settings
**Linx specific settings for mining pools**

From Block `#315001` a 5% dev fee is effective.
This does not reduce the standard block reward of 50 LINX for miners, it's a separate TX.
Mining pools will need to make a change to their code to allow for the new block template and avoid receiving rejected blocks.

**!! IMPORTANT !!**

DO NOT try to implement this code BEFORE block `#315001` as it will fail. This will only work from
block `#315001` onwards.

**Reconfigure your pool for Linx**

There are only a couple of things that need to be changed :

1) The first `rewardRecipient` in your pool config file must pay 2.5 to the Dev Fee address. 
   If it is not the first payout or the address or amount is changed, blocks will be rejected.
   
   ```
   "rewardRecipients": {
   
        // Dev fee MUST come first  
        // Do not change this line
        "XF7kCcs4woQD9WWnCHuN6SWPeUNK2fBspr": 2.5,
        
        // Now add your pool fee here as normal    
        "YOURPOOLFEEWALLETADDRESSHERE": 1.0
    },
    ```

2) You also need to replace, or update, the node `pool.js` and `transactions.js` files to handle the new block template.
   Here are the functions that have been changed if you want to apply the modifications yourself.
   
   in `pool.js`
   
   ```
   function SetupRecipients(){
        var recipients = [];
        options.feePercent = 0;
        options.rewardRecipients = options.rewardRecipients || {};
        for (var r in options.rewardRecipients){
            var percent = options.rewardRecipients[r];
            var rObj = {
                percent: percent
            };
            try {
                if (r.length === 40)
                    rObj.script = util.miningKeyToScript(r);
                else
                    rObj.script = util.addressToScript(r);
                recipients.push(rObj);
                options.feePercent += percent;
            }
            catch(e){
                emitErrorLog('Error generating transaction output script for ' + r + ' in rewardRecipients');
            }
        }
        if (recipients.length === 0){
            emitErrorLog('No rewardRecipients have been setup which means no fees will be taken');
        }
        options.recipients = recipients;
   }
   ```
   
   in `transactions.js`
   ```
   for (var i = 0; i < recipients.length; i++){
        var recipientReward = Math.floor(recipients[i].percent * 100000000);
        rewardToPool -= recipientReward;

        txOutputBuffers.push(Buffer.concat([
            util.packInt64LE(recipientReward),
            util.varIntBuffer(recipients[i].script.length),
            recipients[i].script
        ]));
   }
   ```
   
   Once implemeted blocks will resume as normal.
   
   Acknowledgements: Huge thanks to [ocminer](https://github.com/ocminer) from [suprnova](https://suprnova.cc) for his help with this.
