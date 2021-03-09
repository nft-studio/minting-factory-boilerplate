<template>
  <div>
    Minting from {{ account }}
    <hr />
    <div v-if="toMint">You're minting {{ toMint }}</div>
    <b-field>
      <b-upload v-model="dropFile" v-if="!toMint" drag-drop>
        <section class="section">
          <div class="content has-text-centered">
            <p>Drop your file here or click to upload</p>
          </div>
        </section>
      </b-upload>
    </b-field>
    <button v-if="toMint" v-on:click="mint">MINT!</button>
  </div>
</template>

<script>
var Web3 = require("web3");
const ABI = require("../util/abi.json");
const crypto = require("crypto");

export default {
  name: "Mint",
  data() {
    return {
      web3: new Web3("ws://localhost:7545"),
      contractAddress: "0xfF2868445fb4360704173B1c5b7ae785eF5dA47B",
      account: "",
      contract: {},
      dropFile: {},
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
      console.log(minted);
      this.toMint = "";
      this.dropFile = {};
    },
  },
  watch: {
    dropFile() {
      const app = this;
      var reader = new FileReader();
      reader.onload = function (event) {
        var readed = event.target.result;
        let hash = crypto
          .createHash("sha256")
          .update(new Uint8Array(readed))
          .digest("hex");
        app.toMint = "0x" + hash;
      };
      reader.readAsArrayBuffer(app.dropFile);
    },
  },
};
</script>