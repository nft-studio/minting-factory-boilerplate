<template>
  <div>
    <div v-if="account">
      <b>Welcome back</b><br /><i style="font-size: 12px">{{ account }}</i>
      <hr />
      <div style="padding: 0 20px">
        <b-field v-if="!fileToMint.name">
          <b-upload v-model="fileToMint" expanded drag-drop>
            <section class="section">
              <div class="content has-text-centered">
                <p>Drop your files here or click to upload</p>
              </div>
            </section>
          </b-upload>
        </b-field>
        <b-field name="License" style="width:100%">
          <b-select placeholder="Select a license" v-model="license" style="width:100%">
            <option style="width:100%" v-for="option in licenses" :value="option" :key="option">
              {{ option }}
            </option>
          </b-select>
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
        <b-button
          type="is-primary"
          v-if="ipfsFile && ipfsMetadata && !isMinting"
          expanded
          v-on:click="mint"
          >MINT!</b-button
        >
        <b-button
          type="is-primary"
          v-if="!ipfsFile && !ipfsMetadata && !isUploadingIPFS"
          expanded
          v-on:click="uploadFile"
          >Upload file to IPFS</b-button
        >
        <b-button
          type="is-primary"
          v-if="ipfsFile && !ipfsMetadata && !isUploadingMetadata"
          expanded
          v-on:click="createJson"
          >Upload metadata to IPFS</b-button
        ><br />
        <div v-if="fileToMint.name">
          Selected file: <b>{{ fileToMint.name }}</b>
        </div>
        <div v-if="ipfsFile">
          File IPFS hash is:
          <b
            ><a :href="'https://ipfs.io/ipfs/' + ipfsFile" target="_blank">{{
              ipfsFile
            }}</a></b
          >
        </div>
        <div v-if="ipfsMetadata">
          Metadata IPFS hash is:
          <b
            ><a
              :href="'https://ipfs.io/ipfs/' + ipfsMetadata"
              target="_blank"
              >{{ ipfsMetadata }}</a
            ></b
          >
        </div>
        <div v-if="isMinting">Minting NFT, please wait...</div>
        <div v-if="isUploadingIPFS">Uploading file to IPFS, please wait...</div>
        <div v-if="isUploadingMetadata">
          Uploading metadata to IPFS, please wait...
        </div>
      </div>
    </div>
    <div v-if="!account">
      Please connect your Metamask wallet first,<br />window should be open
      automatically or click below button.<br /><br />
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
const FormData = require("form-data");

export default {
  name: "Mint",
  data() {
    return {
      web3: new Web3(window.ethereum),
      contractAddress: process.env.VUE_APP_SMART_CONTRACT_ADDRESS,
      account: "",
      contract: {},
      license: "",
      licenses: [
        "CC BY",
        "CC BY-SA",
        "CC BY-NC",
        "CC BY-NC-SA",
        "CC BY-ND",
        "CC BY-NC-ND",
        "CC0",
      ],
      fileToMint: {},
      isUploadingIPFS: false,
      isUploadingMetadata: false,
      isMinting: false,
      name: "",
      ipfsFile: "",
      ipfsMetadata: "",
      description: "",
      axios: axios,
      infuraURL: "https://ipfs.infura.io:5001/api/v0/add",
    };
  },
  mounted() {
    this.connect();
  },
  methods: {
    async connect() {
      window.ethereum.enable();
      let accounts = await this.web3.eth.getAccounts();
      let contract = await new this.web3.eth.Contract(
        ABI,
        this.contractAddress,
        {
          gasLimit: "5000000",
        }
      );
      this.contract = contract;
      this.account = accounts[0];
    },
    async uploadFile() {
      const app = this;
      if (app.fileToMint.name && !app.isUploadingIPFS) {
        app.isUploadingIPFS = true;
        const formData = new FormData();
        formData.append("file", app.fileToMint);
        axios({
          method: "post",
          url: app.infuraURL,
          data: formData,
          headers: {
            "Content-Type": "multipart/form-data",
          },
        }).then(function (response) {
          app.ipfsFile = response.data.Hash;
          app.isUploadingIPFS = false;
        });
      } else {
        alert("Select a file first!");
      }
    },
    async createJson() {
      const app = this;
      if (
        app.name !== "" &&
        app.description !== "" &&
        !app.isUploadingMetadata
      ) {
        app.isUploadingMetadata = true;
        const formData = new FormData();
        formData.append(
          "metadata",
          JSON.stringify({
            name: app.name,
            description: app.description,
            image: "https://ipfs.io/ipfs/" + app.ipfsFile,
            external_url: "https://nftstudi.io/#/opera/" + app.ipfsFile,
            attributes: [
              {
                trait_type: "License",
                value: app.license,
              },
            ],
          })
        );
        axios({
          method: "post",
          url: app.infuraURL,
          data: formData,
          headers: {
            "Content-Type": "multipart/form-data",
          },
        }).then(function (response) {
          app.ipfsMetadata = response.data.Hash;
          app.isUploadingMetadata = false;
        });
      } else {
        alert("Please add name and description!");
      }
    },
    async mint() {
      const app = this;
      if (!app.isMinting) {
        app.isMinting = true;
        try {
          let minted = await app.contract.methods
            .mintOpera(app.ipfsMetadata)
            .send({ from: this.account });
          alert("Successfully minted at: " + minted.transactionHash);
          app.isMinting = false;
        } catch (e) {
          alert(e);
        }
      }
    },
  },
};
</script>