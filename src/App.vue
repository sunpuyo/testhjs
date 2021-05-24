<template>
  <center>
  <div id="app">
    <h2>Ethereum to Aergo</h2>
    <h3>Lock ERC-721 NFT to Eth Merkle Bridge</h3>
    Send ERC-721 token to Eth Merkle Bridge (ADDRESS) with Calldata
    Example...


    <h3>Mint ARC-2 from Aergo Merkle Bridge</h3>
    <form id="myForm" @submit.prevent="getAergoActiveAccount">
      <button>Connect to Aergo</button>
    </form>
    <form id="myForm" @submit.prevent="sendMintOnAergoReq">
      <v-content class="labeled-text">
        <label>ERC-721 NFT Contract Address</label> <input type="text" v-model="eth_lock_nft_addr"><br>
      </v-content>
      <v-content class="labeled-text">
        <label>ERC-721 NFT ID to Transfer</label> <input type="text" v-model="eth_lock_tokenId"><br>
      </v-content>
      <v-content class="labeled-text">
        <label>Receiver Aergo Address</label> <input type="text" v-model="aergo_receiver"><br>
      </v-content>
      <button>MINT ARC-2</button>
    </form>
    
    <h2>Aergo to Ethereum</h2>
    <h3>Burn ARC-2 on Aergo Merkle Bridge</h3>
    Send ARC-2 NFT to Aergo Merkle Bridge (ADDRESS) with a receiver address on ethereum at Calldata 
    <br/> ethereum address format on calldata must be without 0x and lowercase.

    <h3>Unlock ERC-721 from Ether Merkle Bridge</h3>
    <form id="myForm" @submit.prevent="sendUnlockOnEthReq">
      <v-content class="labeled-text">
        <label>ERC-721 NFT Contract Address</label> <input type="text" v-model="eth_lock_nft_addr"><br>
      </v-content>
      <v-content class="labeled-text">
        <label>ERC-721 NFT ID to Transfer</label> <input type="text" v-model="eth_lock_tokenId"><br>
      </v-content>
      <v-content class="labeled-text">
        <label>Receiver Ethereum Address</label> <input type="text" v-model="eth_receiver"><br>
      </v-content>
      <button>Unlock ERC-721</button>
    </form>


  </div>
  </center>
</template>

<script>
import { AergoClient, GrpcWebProvider, Contract } from "@herajs/client";
import { aergoToEth, ethToAergo } from "eth-merkle-bridge-js"; //aergoToEth
import { aergoBridgeAbi } from "./common/AergoBridge";
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
  
        // TODO create unlock tx

      } catch (e) {
        alert(e); 
      }
    },
    sendReserved: async function() {
      this.connectMetamask();
    },
    getAergoActiveAccount() {
      window.addEventListener("AERGO_ACTIVE_ACCOUNT", event => {
        if(event.detail.error) {
          console.log(event.detail.error)
          return;
        }
        console.log('login to aergo: ', event.detail.account);
        this.aergoaccount = event.detail.account;
      });
      window.postMessage({
        type: "AERGO_REQUEST",
        action: "ACTIVE_ACCOUNT"
      });
    },
    connectMetamask() {
      window.ethereum.enable();
    }
  },
  created() {
    // set ethereum account change listener
    web3.currentProvider.connection.publicConfigStore.on("update", account => {
      this.etheraccount = account;
      updateDefaultAccount(account.selectedAddress);
    });
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