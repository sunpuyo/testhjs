<script>

export function sendTxToAergoConnect(herajs, contractID, builtTx) {
  //let herajs = new AergoClient({}, new GrpcWebProvider({ url: endpoint }));

  builtTx.to = contractID;
  builtTx.payload_json = JSON.parse(builtTx.payload);
  delete builtTx.payload;

  return new Promise((resolve, reject) => {
    try {
      // register event handler
      window.addEventListener("AERGO_SEND_TX_RESULT", (event) => {
        setTimeout(async () => {
          try {
            if (event.detail.error) {
              reject(event.detail.error);
            } else {
              const receipt = await herajs.getTransactionReceipt(
                event.detail.hash
              );
              resolve(receipt);
            }
          } catch (err) {
            reject(err);
          }
        }, 2000);
      }, {
        once: true
      });
      
      // send build tx request
      window.postMessage({
        type: "AERGO_REQUEST",
        action: "SEND_TX",
        data: builtTx
      });
    } catch (err) {
      reject(err);
    }
  });
}

export default {};
</script>