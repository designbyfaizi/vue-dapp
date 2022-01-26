<template>
    <img class="metamask" src="@/assets/metamask.svg" alt="MetaMask" />
    <wallet-card />
</template>

<script>
import WalletCard from "./components/WalletCard.vue";

export default {
    name: "App",
    components: {
        WalletCard,
    },
};
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

.metamask{
  max-width: 200px;
  margin: auto;
}
</style>
