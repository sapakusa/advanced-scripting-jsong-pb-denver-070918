
# Advanced Scripting

### Pay To Script Hash

So far, we've been doing single-key transactions, ones where only a single private key has to sign in order to disperse the funds. But what if we wanted something a little more complicated? A company that does $100 million in Bitcoin transactions might not want all of the funds in a single public key as that key can be stolen by an employee and all funds lost. What can we do?

The solution is multi-sig, or multiple signatures. This was built into Bitcoin from the beginning, but was a bit clunky at first and so it wasn't used. In fact, as we'll discover, it turns out Satoshi never really tested OP_CHECKMULTISIG as it has a very obvious off-by-one error. The bug has had to stay in the protocol as fixing it would require a hard fork.

#### Exercise: Find the hash160 of the RedeemScript

```
5221022626e955ea6ea6d98850c994f9107b036b1334f18ca8830bfff1295d21cfdb702103b287eaf122eea69030a0e9feed096bed8045c8b98bec453e1ffac7fbdbd4bb7152ae
```


```python
from helper import hash160

hex_redeem_script = '5221022626e955ea6ea6d98850c994f9107b036b1334f18ca8830bfff1295d21cfdb702103b287eaf122eea69030a0e9feed096bed8045c8b98bec453e1ffac7fbdbd4bb7152ae'

# bytes.fromhex script
# hash160 result
# hex() to display
```

### Pay to Script Hash [TODO: INSERT VIDEOS]

Pay to Script Hash is a very general solution to the long address problem. It's possible to have a more complicated script than a multisig and there's no real way to compress them into addresses, either. To make this work, we have to be able to take the hash of a bunch of script elements and then somehow reveal the pre-image script elements later. This is at the heart of the design around pay-to-script-hash.

Pay to script hash was introduced in 2011 to a lot of controversy. There were multiple proposals, but as we'll see, the reasoning behind how pay-to-script-hash works was kludgy, but sound.

Essentially, pay-to-script-hash executes a very special rule only when the script goes in this pattern:

`<redeemScript> OP_HASH160 <hash> OP_EQUAL`

If this exact sequence ends up being true, then whatever is in the redeemScript is then interpreted as script again and then put on the stack. Again, this is a very special pattern and the Bitcoin codebase makes sure to check for this particular sequence and only interprets the redeemScript as additional script elements if this sequence is encountered.

If this sounds hacky, it is. But before we get to that, let's look a little closer at exactly how this plays out.

What we need to do for p2sh is to take a hash of this and store the original script. We put the hash of the multisig as the scriptPubKey with everything but the original script like so:

`OP_HASH160 <hash> OP_EQUAL`

The multisig pubkey that is hashed is known as the redeemScript and that is what we reveal as part of the redemption.

Redeeming a p2sh script involves not only revealing the redeemScript, but also solving the redeemScript. At this point, you might wonder, where is the redeemScript stored? The redeemScript is not on the blockchain until actual redemption, so it must be stored locally by the creator of the pay-to-script-hash address.

Redemption for the 1-of-2 multisig looks like this:

`OP_0 <signature> <redeemScript>`

Note that the OP_0 needs to be there because of the OP_CHECKMULTISIG bug.

The key here is that upon execution of the exact sequence

`<redeemScript> OP_HASH160 <hash> OP_EQUAL`

the redeemScript is immediately put on the stack if the result is true. In other words, if we reveal a script that hashes to the hash in the scriptPubKey, that redeemScript acts like the scriptPubKey instead. We are essentially hashing the script that locks the funds and putting that into the blockchain instead of the script itself.

This is a bit hacky and there's a lot of special-cased code in Bitcoin to handle this. Why didn't the core devs do something a lot less hacky and more intuitive? Well, it turns out that there was indeed another proposal BIPXX which used something called OP_EVAL, which would have been a lot more elegant. A script like this would have sufficed:

`OP_DUP OP_HASH160 <hash> OP_EQUAL OP_EVAL`

`OP_EVAL` would be taking the top element of the script and expanded it.

Unfortunately, this much more elegant solution comes with an unwanted side-effect, namely Turing-completeness. Turing completeness is undesirable as it makes the security of a smart contract much harder to guarantee. Thus, the more hacky, but less vulnerable option of special-casing was chosen as part of BIP16. This was implemented in 2011 and continues to be a part of the network today.


```python
# P2SH address construction example

from helper import encode_base58_checksum

print(encode_base58_checksum(b'\x05'+bytes.fromhex('74d691da1574e6b3c192ecfb52cc8984ee7b6c56')))
```

### Addresses

P2SH addresses have a very similar structure to P2PKH addresses. Namely, there's 20 bytes that are being encoded with a particular prefix and a checksum that helps identify if any of the characters are wrong encoded in Base58.

Specifically, P2SH uses the 5 byte one mainnet which translates to addresses that start with a 3 in base58.

### Test Driven Exercise

Write methods to translate h160 to p2sh and p2pkh addresses.


```python
def h160_to_p2pkh_address(h160, testnet=False):
    '''Takes a byte sequence hash160 and returns a p2pkh address string'''
    # p2pkh has a prefix of b'\x00' for mainnet, b'\x6f' for testnet
    pass


def h160_to_p2sh_address(h160, testnet=False):
    '''Takes a byte sequence hash160 and returns a p2sh address string'''
    # p2sh has a prefix of b'\x05' for mainnet, b'\xc4 for testnet
    pass
```
