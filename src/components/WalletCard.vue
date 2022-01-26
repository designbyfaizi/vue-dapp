<template>
    <div class="flex flex-col justify-center text-center p-5 gap-3">
        <h1 class="my-1 text-slate-600 font-semibold text-4xl text-yellow-600">
            <span class="underline">METAMASK</span>
            <span v-if="defaultAccount !== null" class="text-green-500"> CONNECTED</span>
            <span v-else class="text-red-500"> NOT CONNECTED</span>
        </h1>
        <h4 class="my-1">
            Connection to metamask using window.ethereum methods
        </h4>
        <button
             v-if="defaultAccount === null" 
            class="my-1 mx-auto p-3 bg-blue-500 text-white hover:bg-indigo-500 rounded-lg transform hover:scale-110 transition-all duration-150"
            v-on:click="connectionWalletHandler()"
        >
            {{ connectButtontext }}
        </button>
        <button
             v-if="defaultAccount !== null" 
            class="my-1 mx-auto p-3 bg-red-500 text-white hover:bg-yellow-500 rounded-lg transform hover:scale-110 transition-all duration-150"
            v-on:click="console.log('Disconnect')"
        >
            Disconnect Wallet
        </button>
        <div v-if="defaultAccount !== null"  class="accountDisplay my-1">
            <h3>
                Address:
                <span class="text text-indigo-600 py-2 px-4 bg-indigo-100 rounded-lg w-max mx-auto">{{ defaultAccount }}</span>
            </h3>
        </div>
        <div v-if="defaultAccount !== null" class="balanceDisplay my-1">
            <h3>
                Balance:
                <span class="text text-green-600 py-2 px-4 bg-green-100 rounded-lg">{{ userBalance }} ETH</span>
            </h3>
        </div>
        <div v-if="defaultAccount !== null" class="balanceDisplay my-1">
            <h3>
                Balance in USD:
                <span class="text text-yellow-600 py-2 px-4 bg-yellow-100 rounded-lg">{{ usdBalance }} USD</span>
            </h3>
        </div>
        {{ errorMessage }}

        <div class="paymentForm mx-auto min-w-[36rem] max-w-2xl bg-gray-900 min-h-[fit-content] shadow-2xl rounded-lg p-5 text-white flex flex-col gap-3">
            <h1 class="text text-xl">Send ETH</h1>
            <input
                class="bg-gray-800 p-2 rounded-md outline-none border-2 border-gray-700 focus:border-blue-400 transition-all duration-150 focus:placeholder-blue-400 text-sm"
                type="text"
                placeholder="Recipient Address"
                v-model="transaction.address"
            />
            <input
                class="bg-gray-800 p-2 rounded-md outline-none border-2 border-gray-700 focus:border-blue-400 transition-all duration-150 focus:placeholder-blue-400 text-sm"
                type="number"
                placeholder="Amount in ETH"
                v-model="transaction.eth"
            />

            <button
                v-on:click="startPayment"
                class="p-2 bg-indigo-100 text-gray-700 rounded-md transform hover:scale-105 hover:bg-gray-700 hover:text-gray-100 transition-all duration-150 w-20 mx-auto"
            >
                SEND
            </button>
            <div v-if="error.message.length === 0" class="">
            </div>
            <div v-else class="err text-red-500 bg-red-50 py-2 rounded-md">
                <p>{{error.message}}</p>
            </div>
        </div>
    </div>
</template>

<script>
import { ethers } from "ethers";
import axios from "axios";
import Web3 from "web3";

var web3 = new Web3(
    Web3.givenProvider || "ws://some.local-or-remote.node:8546"
);
export default {
    name: "WalletCard",
    // components: {
    //     VueMetamask,
    // },
    data() {
        return {
            connectButtontext: "Connect Wallet",
            defaultAccount: null,
            userBalance: null,
            usdBalance: null,
            errorMessage: null,
            msg: "This is demo net work",
            transaction: {
                address: "",
                eth: null,
            },
            provider: null,
            address: null,
            error:{
                message: ""
            },
            metamaskAvailable: true
        };
    },
    methods: {
        print(data) {
            console.log(data);
        },
        async detectProvider() {
            let provider;
            if (window.ethereum) {
                provider = window.ethereum;
                const result = await window.ethereum.request({
                    method: "eth_requestAccounts",
                });
                this.accountChangeHandler(result[0]);
            } else if (web3) {
                provider = web3.currentProvider;
            } else {
                alert("No Ethereum Browser detected! Check out MetaMask :D");
            }
            
            return provider;
        },
        async connectionWalletHandler() {
            let provider = await this.detectProvider();
            this.provider = provider;
            if (provider) {
                this.metamaskAvailable = true;
                return this.getAccounts();
            }
            this.metamaskAvailable = false;
            alert("Can't conect to wallet. No provider Found!");
        },
        async getAccounts() {
            try{
                const accounts = await web3.eth.getAccounts();
                if (accounts.length === 0) {
                    this.error.message = "Please connect to MetaMask!"
                    return console.log("Please connect to MetaMask!");
                }
                //console.log("Account: ", accounts[0])
                this.accountChangeHandler(accounts[0]);
            }
            catch(err){
                console.log(err);
                this.error.message = err.message;
            }
        },
        async accountChangeHandler(newAccount) {
            this.defaultAccount = newAccount;
            this.getUserBalance(newAccount.toString());
        },
        async getUserBalance(address) {
            const balance = await window.ethereum.request({
                method: "eth_getBalance",
                params: [address, "latest"],
            });
            this.userBalance = ethers.utils.formatEther(balance);
            console.log(this.userBalance);
            this.getBalanceInUSD(this.userBalance);
        },
        async chainChangedHandler() {
            // window.location.reload();
            this.connectionWalletHandler();
        },
        async getBalanceInUSD(balance) {
            const result = await axios.get(
                "https://min-api.cryptocompare.com/data/price?fsym=ETH&tsyms=BTC,USD,EUR"
            );
            const usdValue = result.data.USD;
            this.usdBalance = balance * usdValue;
        },
        async startPayment(){
            try{
                if(!window.ethereum){
                // throw new Error("No crypto wallet found.")
                this.error.message = "No crypto wallet found.";
            }
            else{
                await this.getAccounts();
            }
            }
            catch(err){
                console.log(err);
                this.error.message = err.message;
            }
        }
    },
    mounted() {
        this.getAccounts();
        window.ethereum.on("accountsChanged", this.accountChangeHandler);
        window.ethereum.on("chainChanged", this.chainChangedHandler);
    },
    watch: {
        userBalance: function () {
            this.getBalanceInUSD(this.userBalance);
        },
    },
};
</script>

<style lang="scss" scoped></style>
