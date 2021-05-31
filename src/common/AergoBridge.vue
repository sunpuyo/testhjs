<script>
export const aergoBridgeAbi = {
  version: "0.2",
  language: "lua",
  functions: [
    {
      name: "constructor",
      arguments: [
        { name: "aergoErc20" },
        { name: "tAnchor" },
        { name: "tFinal" },
        { name: "unfreeze_fee" },
      ],
    },
    {
      name: "freeze",
      payable: true,
      arguments: [{ name: "receiver" }, { name: "amount" }],
    },
    {
      name: "onARC2Received",
      arguments: [
        { name: "operator" },
        { name: "from" },
        { name: "tokenId" },
        { name: "receiver" },
      ],
    },
    { name: "default", payable: true, arguments: [] },
    {
      name: "mintARC2",
      arguments: [
        { name: "receiver" },
        { name: "tokenId" },
        { name: "lockERC721BlockNum" },
        { name: "tokenOrigin" },
        { name: "merkleProof" },
      ],
    },
    {
      name: "unfreeze",
      arguments: [
        { name: "receiver" },
        { name: "balance" },
        { name: "merkleProof" },
      ],
    },
    {
      name: "verifyDepositProof",
      arguments: [
        { name: "mapKey" },
        { name: "mapPosition" },
        { name: "value" },
        { name: "merkleProof" },
      ],
    },
    {
      name: "burn",
      arguments: [
        { name: "receiver" },
        { name: "amount" },
        { name: "mintAddress" },
      ],
    },
    {
      name: "unlock",
      arguments: [
        { name: "receiver" },
        { name: "balance" },
        { name: "tokenAddress" },
        { name: "merkleProof" },
      ],
    },
    {
      name: "mint",
      arguments: [
        { name: "receiver" },
        { name: "balance" },
        { name: "tokenOrigin" },
        { name: "merkleProof" },
      ],
    },
    {
      name: "tokensReceived",
      arguments: [
        { name: "operator" },
        { name: "from" },
        { name: "value" },
        { name: "receiver" },
      ],
    },
    { name: "unfreezeFeeUpdate", arguments: [{ name: "fee" }] },
    { name: "newAnchor", arguments: [{ name: "root" }, { name: "height" }] },
    { name: "oracleUpdate", arguments: [{ name: "newOracle" }] },
    { name: "tAnchorUpdate", arguments: [{ name: "tAnchor" }] },
    { name: "tFinalUpdate", arguments: [{ name: "tFinal" }] },
  ],
  state_variables: [
    { name: "_mintedNFTs", type: "map" },
    { name: "_unfreezes", type: "map" },
    { name: "_unlocks", type: "map" },
    { name: "_tFinal", type: "value" },
    { name: "_locks", type: "map" },
    { name: "_mints", type: "map" },
    { name: "_bridgeTokens", type: "map" },
    { name: "_mintsARC2", type: "map" },
    { name: "_tAnchor", type: "value" },
    { name: "_unfreezeFee", type: "value" },
    { name: "_burns", type: "map" },
    { name: "_burnsARC2", type: "map" },
    { name: "_anchorRoot", type: "value" },
    { name: "_anchorHeight", type: "value" },
    { name: "_oracle", type: "value" },
    { name: "_aergoErc20Bytes", type: "value" },
    { name: "_bridgeNFTs", type: "map" },
    { name: "_mintedTokens", type: "map" },
  ],
};

export default {};
</script>