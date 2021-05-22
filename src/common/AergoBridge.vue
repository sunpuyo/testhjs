<script>
export const aergoBridgeAbi = {
  "language": "lua",
  "version": "0.2",
  "functions": [
    {
      "name": "constructor",
      "arguments": [
        {
          "name": "aergoErc20"
        },
        {
          "name": "tAnchor"
        },
        {
          "name": "tFinal"
        },
        {
          "name": "unfreeze_fee"
        }
      ],
      "view": false,
      "payable": false,
      "feeDelegation": false
    },
    {
      "name": "freeze",
      "arguments": [
        {
          "name": "receiver"
        },
        {
          "name": "amount"
        }
      ],
      "view": false,
      "payable": true,
      "feeDelegation": false
    },
    {
      "name": "onARC2Received",
      "arguments": [
        {
          "name": "operator"
        },
        {
          "name": "from"
        },
        {
          "name": "tokenId"
        },
        {
          "name": "receiver"
        }
      ],
      "view": false,
      "payable": false,
      "feeDelegation": false
    },
    {
      "name": "default",
      "arguments": [],
      "view": false,
      "payable": true,
      "feeDelegation": false
    },
    {
      "name": "mintARC2",
      "arguments": [
        {
          "name": "receiver"
        },
        {
          "name": "tokenId"
        },
        {
          "name": "lockERC721BlockNum"
        },
        {
          "name": "tokenOrigin"
        },
        {
          "name": "merkleProof"
        }
      ],
      "view": false,
      "payable": false,
      "feeDelegation": false
    },
    {
      "name": "unfreeze",
      "arguments": [
        {
          "name": "receiver"
        },
        {
          "name": "balance"
        },
        {
          "name": "merkleProof"
        }
      ],
      "view": false,
      "payable": false,
      "feeDelegation": false
    },
    {
      "name": "verifyDepositProof",
      "arguments": [
        {
          "name": "mapKey"
        },
        {
          "name": "mapPosition"
        },
        {
          "name": "value"
        },
        {
          "name": "merkleProof"
        }
      ],
      "view": false,
      "payable": false,
      "feeDelegation": false
    },
    {
      "name": "burn",
      "arguments": [
        {
          "name": "receiver"
        },
        {
          "name": "amount"
        },
        {
          "name": "mintAddress"
        }
      ],
      "view": false,
      "payable": false,
      "feeDelegation": false
    },
    {
      "name": "unlock",
      "arguments": [
        {
          "name": "receiver"
        },
        {
          "name": "balance"
        },
        {
          "name": "tokenAddress"
        },
        {
          "name": "merkleProof"
        }
      ],
      "view": false,
      "payable": false,
      "feeDelegation": false
    },
    {
      "name": "mint",
      "arguments": [
        {
          "name": "receiver"
        },
        {
          "name": "balance"
        },
        {
          "name": "tokenOrigin"
        },
        {
          "name": "merkleProof"
        }
      ],
      "view": false,
      "payable": false,
      "feeDelegation": false
    },
    {
      "name": "tokensReceived",
      "arguments": [
        {
          "name": "operator"
        },
        {
          "name": "from"
        },
        {
          "name": "value"
        },
        {
          "name": "receiver"
        }
      ],
      "view": false,
      "payable": false,
      "feeDelegation": false
    },
    {
      "name": "unfreezeFeeUpdate",
      "arguments": [
        {
          "name": "fee"
        }
      ],
      "view": false,
      "payable": false,
      "feeDelegation": false
    },
    {
      "name": "newAnchor",
      "arguments": [
        {
          "name": "root"
        },
        {
          "name": "height"
        }
      ],
      "view": false,
      "payable": false,
      "feeDelegation": false
    },
    {
      "name": "oracleUpdate",
      "arguments": [
        {
          "name": "newOracle"
        }
      ],
      "view": false,
      "payable": false,
      "feeDelegation": false
    },
    {
      "name": "tAnchorUpdate",
      "arguments": [
        {
          "name": "tAnchor"
        }
      ],
      "view": false,
      "payable": false,
      "feeDelegation": false
    },
    {
      "name": "tFinalUpdate",
      "arguments": [
        {
          "name": "tFinal"
        }
      ],
      "view": false,
      "payable": false,
      "feeDelegation": false
    }
  ],
  "state_variables": [
    {
      "name": "_mintedNFTs",
      "type": "map",
      "len": 0
    },
    {
      "name": "_unfreezes",
      "type": "map",
      "len": 0
    },
    {
      "name": "_unlocks",
      "type": "map",
      "len": 0
    },
    {
      "name": "_tFinal",
      "type": "value",
      "len": 0
    },
    {
      "name": "_locks",
      "type": "map",
      "len": 0
    },
    {
      "name": "_mints",
      "type": "map",
      "len": 0
    },
    {
      "name": "_bridgeTokens",
      "type": "map",
      "len": 0
    },
    {
      "name": "_mintsARC2",
      "type": "map",
      "len": 0
    },
    {
      "name": "_tAnchor",
      "type": "value",
      "len": 0
    },
    {
      "name": "_unfreezeFee",
      "type": "value",
      "len": 0
    },
    {
      "name": "_burns",
      "type": "map",
      "len": 0
    },
    {
      "name": "_burnsARC2",
      "type": "map",
      "len": 0
    },
    {
      "name": "_anchorRoot",
      "type": "value",
      "len": 0
    },
    {
      "name": "_anchorHeight",
      "type": "value",
      "len": 0
    },
    {
      "name": "_oracle",
      "type": "value",
      "len": 0
    },
    {
      "name": "_aergoErc20Bytes",
      "type": "value",
      "len": 0
    },
    {
      "name": "_bridgeNFTs",
      "type": "map",
      "len": 0
    },
    {
      "name": "_mintedTokens",
      "type": "map",
      "len": 0
    }
  ]
};

export default {}
</script>