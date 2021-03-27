<template>
  <div id="app">
    <HelloWorld msg="Welcome to Your Vue.js App"/>
    check console log
  </div>
</template>

<script>
import HelloWorld from './components/HelloWorld.vue';
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
    HelloWorld
  }
}

</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
</style>