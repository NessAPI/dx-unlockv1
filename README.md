# dx-unlockv1
Unlocks (Liquidity Pair LP) Tokens locked using Dxlock v1, before the unlock time.\
\
__I found out that [Decurity](https://blog.decurity.io/dx-protocol-vulnerability-disclosure-bddff88aeb1d) already found this vulnerability and got awarded a $500 bounty. This repo provides an easy PoC for this vulnerability.__


## Usage
Add the dxlock.abi file on [Remix](https://remix.ethereum.org/) \
Interact with the dxlock smart contract (V1 Contract: 0xeb3a9c56d963b971d320f889be2fb8b59853e449) using the .abi\
Call the unlockToken function with parameter vaultId 0 (0 for default, if you have more than 1 locker with your wallet use the right vaultid)\

## Unbreak liquidity pair
To "unbreak" LP's after unlock, use rugdocs [lp breaker](https://rugdoc.io/lp-breaker/)




## Research
While dxlock did not verify their v1 locker contract [0xeb3a9c56d963b971d320f889be2fb8b59853e449](https://bscscan.com/address/0xeb3a9c56d963b971d320f889be2fb8b59853e449), decompiling their bytecode still gives us an interesting look in their locking mechanism.

```py
# Palkeoramix decompiler. 

def unlockToken(uint256 _tokenId): # not payable
  require calldata.size - 4 >= 32
  if not unknownae4f4df1[caller][_tokenId].field_0:
      revert with 0x8c379a000000000000000000000000000000000000000000000000000000000, 
                  32,
                  41,
                  0x216572723a204c6f636b446570202d207573657220646f65736e7420686176652061206c6f636b6572,
                  mem[205 len 23]
  if not unknownae4f4df1[caller][_tokenId].field_8:
      revert with 0x8c379a000000000000000000000000000000000000000000000000000000000, 
                  32,
                  44,
                  0x216572723a204c6f636b446570202d2075736572277320746f6b656e7320617265206e6f74206c6f636b6564,
                  mem[208 len 20]
  if not unknownae4f4df1[caller][_tokenId].field_512:
      revert with 0x8c379a000000000000000000000000000000000000000000000000000000000, 
                  32,
                  49,
                  0x216572723a204c6f636b446570202d206d75737420686176652061746c656173742031207061796f757420766573746564,
                  mem[213 len 15]
  if block.timestamp > unknownae4f4df1[caller][_tokenId].field_768:
      unknownae4f4df1[caller][_tokenId].field_8 = 0
  require ext_code.size(unknownae4f4df1[caller][_tokenId].field_1280)
  static call unknownae4f4df1[caller][_tokenId].field_1280.balanceOf(address tokenOwner) with:
          gas gas_remaining wei
         args this.address
  if not ext_call.success:
      revert with ext_call.return_data[0 len return_data.size]
  require return_data.size >= 32
  if ext_call.return_data < unknownae4f4df1[caller][_tokenId].field_512:
      revert with 0x8c379a000000000000000000000000000000000000000000000000000000000, 
                  32,
                  43,
                  0x656572723a204c6f636b6572202d206e6f206d6f726520746f6b656e73206c65667420746f20726566756e,
                  mem[207 len 21]
  require ext_code.size(unknownae4f4df1[caller][_tokenId].field_1280)
  call unknownae4f4df1[caller][_tokenId].field_1280.transfer(address to, uint256 tokens) with:
       gas gas_remaining wei
      args caller, unknownae4f4df1[caller][_tokenId].field_512
  if not ext_call.success:
      revert with ext_call.return_data[0 len return_data.size]
  require return_data.size >= 32
  if not ext_call.return_data[0]:
      revert with 0x8c379a000000000000000000000000000000000000000000000000000000000, 
                  32,
                  45,
                  0x216572723a204c6f636b6572202d20546f6b656e20726566756e6420746f2063726561746f72206661696c6564,
                  mem[209 len 19]
  log 0x70b37289: caller, unknownae4f4df1[caller][_tokenId].field_1280, unknownae4f4df1[caller][_tokenId].field_512, block.timestamp

```


Using this decompiled code, we can now write the ABI for the dxlock contract
```json
    {
        "constant": false,
        "inputs": [
            {

                "name": "vaultId",
                "type": "uint256"
            }
        ],
        "name": "unlockToken",
        "outputs": [],
        "payable": false,
        "stateMutability": "nonpayable",
        "type": "function"
    }
```

## Security
The ABI is a json type file used to interact with smart contracts on the blockchain. There are no risks using this .abi on your own.
Since the hardcode of a smart contract can not be changed after deploying to the blockchain, this is a vulnerability regarding all LP's inside the Dxsale v1 locker. Be careful for tokens with LP locked inside the V1, since it can be taken out. 


## Thanks to
https://github.com/eveem-org/panoramix \
https://rugdoc.io/lp-breaker/ \
https://blog.decurity.io/dx-protocol-vulnerability-disclosure-bddff88aeb1d
