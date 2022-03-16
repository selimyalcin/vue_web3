<template>
  <div class="container">
    <div class="row">
      <div class="col-12">
        <button
          v-if="currentAccount == null"
          class="btn btn-primary"
          @click="connectWallet"
        >
          Connect Wallet
        </button>
        <div class="text-light" v-else>
          {{ currentAccount }}<br />
          <button class="btn btn-info" @click="requestChain">
            Change Chain
          </button>
          <br />
          <button class="btn btn-info mt-2" @click="addChain">Add Chain</button>
        </div>
      </div>
      <div class="col-12 text-center p-3">Account : {{ currentAccount }}</div>
      <div class="col-12">
        <div><button class="btn btn-warning" @click="contractFunction">Call Contract function</button></div>
        <div class="mt-3 p-2">
          {{ contractResponse }}
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import abi from "@/abis/worm.json";
import { ethers } from "ethers";

export default {
  name: "HomeView",
  data() {
    return {
      isOpen: false,
      currentAccount: null,
      currentContract: null,
      contractAddress: "0x036f5ae55da2837392663bb7ae4fdb654642e5a3",
      contractResponse: null,
    };
  },
  methods: {
    async contractFunction() {
      const provider = new ethers.providers.Web3Provider(window.ethereum);

      await provider.send("eth_requestAccounts", []);
      const signer = provider.getSigner();
     
      
      let contract = new ethers.Contract(this.contractAddress, abi, signer);
      let contractSigner = contract.connect(signer);

      //worm contract 'uri' func
      const t = await contractSigner.uri(
        "0x930a5614c727f5C4e01cB06cf9E3538706CEA3a9",
        0
      );
      //worm contract abi func
      this.contractResponse = t;
    },
    addChain() {
      window.ethereum.request({
        method: "wallet_addEthereumChain",
        params: [
          {
            chainId: "0x89",
            rpcUrls: ["https://rpc-mainnet.matic.network/"],
            chainName: "Matic Mainnet",
            nativeCurrency: {
              name: "MATIC",
              symbol: "MATIC",
              decimals: 18,
            },
            blockExplorerUrls: ["https://polygonscan.com/"],
          },
        ],
      });
    },
    requestChain() {
      window.ethereum.request({
        method: "wallet_switchEthereumChain",
        params: [
          {
            chainId: "0xA86A",
          },
        ],
      });
    },
    checkConnectedWalletExist: async function () {
      try {
        const { ethereum } = window;
        if (!ethereum) {
          alert("Make sure you have metamask!");
          return false;
        } else {
          console.log("We have the ethereum object", ethereum);
          const provider = new ethers.providers.Web3Provider(window.ethereum);
        }
        const accounts = await ethereum.request({ method: "eth_accounts" });
        if (accounts.length !== 0) {
          const account = accounts[0];
          console.log("Found an authorized account:", account);
          this.currentAccount = account;
          return true;
        } else {
          console.log("No authorized account found");
          return false;
        }
      } catch (error) {
        console.log(error);
        return false;
      }
    },
    connectWallet: async function () {
      try {
        const { ethereum } = window;
        if (!ethereum) {
          alert("Get MetaMask!");
          return;
        }
        const accounts = await ethereum.request({
          method: "eth_requestAccounts",
        });

        this.currentAccount = accounts[0];
      } catch (error) {
        console.log(error);
      }
    },
  },
  mounted() {
    this.checkConnectedWalletExist();
  },
};
</script>
