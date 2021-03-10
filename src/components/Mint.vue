<template>
  <div>
    <div v-if="account">
      <b>Welcome back</b><br><i style="font-size:12px">{{ account }}</i>
      <hr />
      <div style="padding: 0 20px">
        <b-field label="IPFS Hash">
          <b-input v-model="toMint"></b-input>
        </b-field>
        <b-field label="Name">
          <b-input v-model="name"></b-input>
        </b-field>
        <b-field label="Description">
          <b-input
            maxlength="200"
            v-model="description"
            type="textarea"
          ></b-input>
        </b-field>
        <b-button type="is-primary" expanded v-on:click="mint">MINT!</b-button>
      </div>
    </div>
    <div v-if="!account">
      Please connect your Metamask wallet first,<br>window should be open
      automatically or click below button.<br /><br>
      <b-button type="is-primary" v-on:click="connect"
        >CONNECT METAMASK</b-button
      >
    </div>
  </div>
</template>

<script>
var Web3 = require("web3");
const ABI = require("../util/abi.json");
const axios = require("axios");

export default {
  name: "Mint",
  data() {
    return {
      web3: new Web3(process.env.VUE_APP_NETWORK),
      contractAddress: process.env.VUE_APP_SMART_CONTRACT_ADDRESS,
      account: "",
      contract: {},
      name: "",
      description: "",
      toMint: "",
      axios: axios,
    };
  },
  mounted() {
    this.connect()
  },
  methods: {
    async connect(){
      window.ethereum.enable();
      let accounts = await this.web3.eth.getAccounts();
      let contract = await new this.web3.eth.Contract(ABI, this.contractAddress, {
        gasLimit: "5000000",
      });
      this.contract = contract;
      this.account = accounts[0];
    },
    async mint() {
      let minted = await this.contract.methods
        .mintLoop(this.toMint)
        .send({ from: this.account });
      alert("Successfully minted at: " + minted.transactionHash);
      console.log(minted);
      this.toMint = "";
    },
  },
};
</script>