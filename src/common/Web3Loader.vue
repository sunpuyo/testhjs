<script>
import Web3 from "web3";

function loadWeb3() {
  // load ethereum web3
  // Modern dapp browsers...
  if (window.ethereum) {
    return new Web3(window.ethereum);
  }
  // Legacy dapp browsers...
  else if (window.web3) {
    return new Web3(window.web3.currentProvider);
  }
  // Non-dapp browsers...
  else {
    /* eslint-disable no-console */
    console.error(
      "Non-Ethereum browser detected. You should consider trying MetaMask!"
    );
    return new Web3(new Web3.providers.HttpProvider("http://localhost:8545"));
  }
}

export function updateDefaultAccount(address) {
  web3.eth.defaultAccount = address;
}
export const web3 = loadWeb3(); 

export default {};
</script>