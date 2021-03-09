<template>
  <div>
    Minting from {{ account }}
    <hr />
    <input v-model="toMint" />
    <button v-on:click="mint">MINT!</button>
  </div>
</template>

<script>
var Web3 = require("web3");
const ABI = require("../util/abi.json");
export default {
  name: "Mint",
  data() {
    return {
      web3: new Web3("ws://localhost:7545"),
      contractAddress: "0xfF2868445fb4360704173B1c5b7ae785eF5dA47B",
      account: "",
      contract: {},
      toMint: "",
    };
  },
  async mounted() {
    window.ethereum.enable();
    let accounts = await this.web3.eth.getAccounts();
    let contract = await new this.web3.eth.Contract(ABI, this.contractAddress, {
      gasLimit: "5000000",
    });
    this.contract = contract;
    this.account = accounts[0];
    console.log(accounts);
    console.log(contract);
  },
  methods: {
    async mint() {
      let minted = await this.contract.methods
        .mintLoop(this.toMint)
        .send({ from: this.account });
      alert("Successfully minted at: " + minted.transactionHash);
      console.log(minted)
    },
  },
};
</script>