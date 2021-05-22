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
    <form id="myForm" @submit.prevent="sendPost">
      <v-content class="labeled-text">
        <label>ERC-721 NFT Contract Address</label> <input type="text" v-model="eth_lock_nft_addr"><br>
      </v-content>
      <v-content class="labeled-text">
        <label>ERC-721 NFT ID to Transfer</label> <input type="text" v-model="eth_lock_tokenId"><br>
      </v-content>
      <v-content class="labeled-text">
        <label>Receiver Aergo Address</label> <input type="text" v-model="receiverAergoAddr"><br>
      </v-content>
      <button>MINT ARC-2</button>
    </form>
    
    <h2>Aergo to Ethereum</h2>
    <h3>Burn ARC-2 on Aergo Merkle Bridge</h3>
    Send ARC-2 NFT to Aergo Merkle Bridge (ADDRESS) with Calldata
    Example...

    <h3>Unlock ERC-721 from Eth Merkle Bridge</h3>
    <div class="input-group">
      <input v-model="aergo_burn_tokenId" type="text" class="form-text" placeholder="ERC-721 NFT ID to Transfer (Same with ERC-721)">
      <input v-model="eth_receiver" type="text" class="form-text" placeholder="Receiver Ethereum Address"><br/>
      <span class="input-group-btn">
        <button class="btn btn-default" type="button">Unlock ERC-721</button>
      </span>
    </div>


  </div>
  </center>
</template>

<script>
import { AergoClient, GrpcWebProvider, Contract } from "@herajs/client";
import { ethToAergo } from "eth-merkle-bridge-js"; //aergoToEth
import { aergoBridgeAbi } from "./common/AergoBridge";
import { web3, updateDefaultAccount } from "./common/Web3Loader";
import { sendTxToAergoConnect } from "./common/util";

const bridgeAergoAddr = "AmhAMtqsrf4akxMy89fhiNuyw7u7ooY9TV6Ke5FFctDYpFVAB44V";
const aergoURL = "http://localhost:7845";
const bridgeEthAddr = "0x89eD1D1C145F6bF3A7e62d2B8eB0e1Bf15Cb2374";
//const ethURL = "http://localhost:8545";



(async () => {
    try {
        // fetch a previous block hash
        const fromBridgeAergoAddr = "Amgru14RQMrwRM8TnnLrT3aa3TePJr2F4yfbqN81KffkXKXbw4oz";

        let fromHerajs = new AergoClient(
            {},
            new GrpcWebProvider({ url: 'https://alpha-api-http.aergo.io'})
          );
        let toHerajs = new AergoClient(
            {},
            new GrpcWebProvider({ url: 'https://testnet-api-http.aergo.io'})
          );

        const toContract = Contract.fromAbi(aergoBridgeAbi).setAddress(fromBridgeAergoAddr);
        let query = toContract.queryState("_sv__anchorHeight");
        const lastMergedHeight = await toHerajs.queryContractState(query); // get last anchoring height
        console.log('Get last merged height: ', lastMergedHeight);

        // get last anchoring block
        const mergeBlockHeader = await fromHerajs.getBlockHeaders(lastMergedHeight, 1)
        const rootFrom = Buffer.from(mergeBlockHeader[0].header.blocksroothash, 'base64')

        // fetch freeze states from contract 
        const freezesKey = Buffer.from('_sv__freezes-'.concat("AmPXCFudjn399bVXr6Fcwmvb7gVmUjjK7KpoAy9bnmEryJ1aBNkT"), 'utf-8');

        const fromContract = Contract.fromAbi(aergoBridgeAbi).setAddress(fromBridgeAergoAddr);

        query = fromContract.queryState(freezesKey); // query current state

        const working_proof = await fromHerajs.queryContractStateProof(query); // this works well
        console.log('This works well', working_proof);

        query = fromContract.queryState(freezesKey, false, rootFrom); // query past state with rootFrom
        // FIXME this raises timeout exception. Maybe it is not work
        const timeout_proof = await fromHerajs.queryContractStateProof(query); 

        console.log("This raise timeout exception", timeout_proof);

    } catch (e) {
        console.error(e);
    }
})();


export default {
  name: 'App',
  components: {
  },
  data: () => ({
    eth_lock_tokenId: '1',
    eth_lock_nft_addr: '0x6Bba61e44686f336f4b5A546f4475be1449e7Ee6',
    receiverAergoAddr: 'AmMMvPuthQZPybWr5amX7rd14Nf6mfmgG7XQxTbA1mFFaKHB5Bx1',
    aergo_burn_tokenId: '',
    eth_receiver: '',

    etheraccount: null,
    aergoaccount: null
  }),
   methods: {
    sendPost: async function () {
      try {
        let hera = new AergoClient({}, new GrpcWebProvider({ url: aergoURL }));
      
        await ethToAergo.validateARC2mintable(
          web3,
          hera,
          bridgeEthAddr,
          bridgeAergoAddr,
          this.receiverAergoAddr,
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
          this.receiverAergoAddr,
          this.eth_lock_tokenId,
          this.eth_lock_nft_addr
        );

        console.log(await sendTxToAergoConnect(hera, bridgeAergoAddr, builtTx));

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