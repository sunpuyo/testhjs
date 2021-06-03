<template>
  <center>
  <div id="app">
    <h2>Ethereum to Aergo</h2>
    <h3>Lock ERC-721 NFT to Eth Merkle Bridge</h3>
    Send ERC-721 token to Eth Merkle Bridge (ADDRESS) with Calldata
    Example...


    <h3>Mint ARC-2 from Aergo Merkle Bridge</h3>
    <form id="mintForm" @submit.prevent="sendMintOnAergoReq">
      <div class="labeled-text">
        <label>ERC-721 NFT Contract Address</label> <input type="text" v-model="eth_lock_nft_addr"><br>
      </div>
      <div class="labeled-text">
        <label>ERC-721 NFT ID to Transfer</label> <input type="text" v-model="eth_lock_tokenId"><br>
      </div>
      <div class="labeled-text">
        <label>Receiver Aergo Address</label> <input type="text" v-model="aergo_receiver"><br>
      </div>
      <button>MINT ARC-2</button>
    </form>
    
    <h2>Aergo to Ethereum</h2>
    <h3>Burn ARC-2 on Aergo Merkle Bridge</h3>
    Send ARC-2 NFT to Aergo Merkle Bridge (ADDRESS) with a receiver address on ethereum at Calldata 
    <br/> ethereum address format on calldata must be without 0x and lowercase.

    <h3>Unlock ERC-721 from Ether Merkle Bridge</h3>
    <form id="unlockForm" @submit.prevent="sendUnlockOnEthReq">
      <div class="labeled-text">
        <label>ERC-721 NFT Contract Address</label> <input type="text" v-model="eth_lock_nft_addr"><br>
      </div>
      <div class="labeled-text">
        <label>ERC-721 NFT ID to Transfer</label> <input type="text" v-model="eth_lock_tokenId"><br>
      </div>
      <div class="labeled-text">
        <label>Receiver Ethereum Address</label> <input type="text" v-model="eth_receiver"><br>
      </div>
      <button>Unlock ERC-721</button>
    </form>


  </div>
  </center>
</template>

<script>
import { AergoClient, GrpcWebProvider } from "@herajs/client";
import { aergoToEth, ethToAergo } from "eth-merkle-bridge-js"; //aergoToEth
import { aergoBridgeAbi } from "./common/AergoBridge";
import { etherBridgeAbi } from "./common/EtherBridge";
import { web3, updateDefaultAccount } from "./common/Web3Loader";
import { sendTxToAergoConnect } from "./common/util";

const bridgeAergoAddr = "AmhAMtqsrf4akxMy89fhiNuyw7u7ooY9TV6Ke5FFctDYpFVAB44V";
const aergoURL = "http://localhost:7845";
const bridgeEthAddr = "0x89eD1D1C145F6bF3A7e62d2B8eB0e1Bf15Cb2374";


export default {
  name: 'App',
  components: {
  },
  data: () => ({
    eth_lock_tokenId: '1',
    eth_lock_nft_addr: '0x6Bba61e44686f336f4b5A546f4475be1449e7Ee6',
    aergo_receiver: 'AmMMvPuthQZPybWr5amX7rd14Nf6mfmgG7XQxTbA1mFFaKHB5Bx1',
    eth_receiver: '0x035d4303f9508ddcab6d074cbc5ed82cd0b436ad',

    etheraccount: null,
    aergoaccount: null
  }),
   methods: {
    sendMintOnAergoReq: async function () {
      try {
        if(! this.aergoaccount) {
          // login to aergo connect
          this.aergoaccount = await this.getAergoActiveAccount()
        }
        let hera = new AergoClient({}, new GrpcWebProvider({ url: aergoURL }));
      
        await ethToAergo.validateARC2Mintable(
          web3,
          hera,
          bridgeEthAddr,
          bridgeAergoAddr,
          this.aergo_receiver,
          this.eth_lock_tokenId,
          this.eth_lock_nft_addr,
       );
        
        const builtTx = await ethToAergo.buildUnlockERC721Tx(
          web3,
          hera,
          this.aergoaccount.address,
          bridgeEthAddr,
          bridgeAergoAddr, 
          aergoBridgeAbi, //최신화
          this.aergo_receiver,
          this.eth_lock_tokenId,
          this.eth_lock_nft_addr
        );

        console.log(await sendTxToAergoConnect(hera, bridgeAergoAddr, builtTx));

      } catch (e) {
        console.error(e);
        alert(e); 
      }
    },
    sendUnlockOnEthReq: async function() {
      try {
        let hera = new AergoClient({}, new GrpcWebProvider({ url: aergoURL }));
        
        await this.connectMetamask();
        await aergoToEth.validateERC721Unlockable(
          web3,
          hera,
          bridgeEthAddr,
          bridgeAergoAddr,
          this.eth_receiver,
          this.eth_lock_tokenId, 
          this.eth_lock_nft_addr, 
        );
  
        const ret = await aergoToEth.unlockERC721(
          web3,
          hera, 
          bridgeEthAddr,
          etherBridgeAbi,
          bridgeAergoAddr,
          this.eth_receiver,
          this.eth_lock_tokenId, 
          this.eth_lock_nft_addr
        );

        console.log(ret);

      } catch (e) {
        console.error(e);
        if(typeof(e) == 'string') {
          alert(e); 
        } else {
          alert(JSON.stringify(e)); 
        }
      }
    },
    getAergoActiveAccount: async function () {
      return new Promise(function(resolve, reject) {
        window.addEventListener("AERGO_ACTIVE_ACCOUNT", event => {
            if(event.detail.error) {
              console.error(event.detail.error);
              reject(event.detail.error);
            }
            console.log('login to aergo: ', event.detail.account);
            
            resolve(event.detail.account);
          });
          window.postMessage({
            type: "AERGO_REQUEST",
            action: "ACTIVE_ACCOUNT"
          });
      });
    },
    connectMetamask: async function () {
      await window.ethereum.enable();
      this.etheraccount = window.ethereum.selectedAddress;
      updateDefaultAccount(window.ethereum.selectedAddress);
    }
  },
  created() {
    // set ethereum account change listener
    if (typeof window.ethereum == 'undefined') {
      alert('Please install Metamask first');
    }
  }
}

</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: left;
  color: #2c3e50;
  margin-top: 60px;
  width: 800px;
}
.labeled-text {
  margin-bottom: 50px;
}
.labeled-text label {
  width: 30%;
  display: inline-block;
}
.labeled-text input {
  width: 65%;
}
</style>