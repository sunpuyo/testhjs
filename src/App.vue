<template>
  <div id="app">
    <h2>Ethereum to Aergo</h2>
    <h3>Lock ERC-721 to Eth Merkle Bridge</h3>
    <div class="input-group">
      <input v-model="eth_lock_tokenId" type="text" class="form-text" placeholder="ERC-721 NFT ID to Transfer">
      <input v-model="aergo_receiver" type="text" class="form-text" placeholder="Receiver Aergo Address">
      <span class="input-group-btn">
        <button class="btn btn-default" type="button">Lock ERC-721</button>
      </span>
    </div>
    <h3>Mint ARC-2 from Aergo Merkle Bridge</h3>
    <div class="input-group">
      <input v-model="eth_lock_tokenId" type="text" class="form-text" placeholder="ERC-721 NFT ID to Transfer">
      <input v-model="aergo_receiver" type="text" class="form-text" placeholder="Receiver Aergo Address"><br/>
      <input v-model="eth_lock_blockheight" type="text" class="form-text" placeholder="Lock Block Height on Ethereum">
      <span class="input-group-btn">
        <button class="btn btn-default" type="button">MINT ARC-2</button>
      </span>
    </div>
    
    <h2>Aergo to Ethereum</h2>
    <h3>Burn ARC-2 on Aergo Merkle Bridge</h3>
    <div class="input-group">
      <input v-model="aergo_burn_tokenId" type="text" class="form-text" placeholder="ARC-2 NFT ID to Transfer (Same with ERC-721)">
      <input v-model="eth_receiver" type="text" class="form-text" placeholder="Receiver Ethereum Address">
      <span class="input-group-btn">
        <button class="btn btn-default" type="button">Burn ARC-2</button>
      </span>
    </div>
    <h3>Unlock ERC-721 from Eth Merkle Bridge</h3>
    <div class="input-group">
      <input v-model="aergo_burn_tokenId" type="text" class="form-text" placeholder="ERC-721 NFT ID to Transfer (Same with ERC-721)">
      <input v-model="eth_receiver" type="text" class="form-text" placeholder="Receiver Ethereum Address"><br/>
      <input v-model="aergo_burn_blockheight" type="text" class="form-text" placeholder="Lock Block Height on Ethereum">
      <span class="input-group-btn">
        <button class="btn btn-default" type="button">Unlock ERC-721</button>
      </span>
    </div>


  </div>

</template>

<script>
import { AergoClient, GrpcWebProvider, Contract } from "@herajs/client";
import { aergoBridgeAbi } from "./AergoBridge";

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
.input-group {
  margin-bottom:50px;
  width: 100%;
}
.form-text {
  width: 100%;
  background-color: "red";
}
</style>