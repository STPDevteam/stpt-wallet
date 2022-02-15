<template>
  <div id="app">
    <Login v-if="isSelecting" />
    <router-view v-else-if="!renderLoading" />
    <div v-else class="renderLoading">
      <b-icon icon="circle-fill" animation="throb" font-scale="4"></b-icon>
    </div>
    <WalletBoard
      :dappId="dappId"
      :walletSelect="walletSelect"
      :walletCheck="walletCheck"
      @isSelecting="getIsSelecting"
      @walletState="getWalletState"
    />
  </div>
</template>

<script>
import { mapActions, mapState } from 'vuex'
import Login from '@/views/Login'
import WalletBoard from '@/WalletBoard'
export default {
  name: 'App',
  components: {
    Login,
    WalletBoard,
  },
  data() {
    const { VUE_APP_BLOCKNATIVE_DAPP_ID, VUE_APP_INFURA_KEY } = process.env
    const dappId = VUE_APP_BLOCKNATIVE_DAPP_ID
    const infuraKey = VUE_APP_INFURA_KEY
    console.log(dappId, infuraKey)
    return {
      dappId,
      isSelecting: false,
      walletSelect: {
        description: 'Please select a wallet to connect to verse passport wallet',
        wallets: [
          { walletName: 'metamask', preferred: true },
          { walletName: 'imToken', preferred: true, infuraKey },
          { walletName: 'walletConnect', preferred: true, infuraKey },
        ],
      },
      walletCheck: [{ checkName: 'accounts' }, { checkName: 'connect' }],
    }
  },
  computed: {
    ...mapState('token', ['renderLoading']),
    ...mapState('wallet', ['chainId']),
  },
  methods: {
    ...mapActions({
      actionWalletInfo: 'wallet/actionWalletInfo',
    }),
    getIsSelecting(isSelecting) {
      this.isSelecting = isSelecting
    },

    getWalletState(walletState) {
      const { address, network, wallet, balance } = walletState

      this.actionWalletInfo({ account: address, chainId: network, wallet: wallet, balance: balance })
    },
  },
}
</script>

<style lang="scss">
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  color: #2c3e50;
  height: 100%;
  overflow: auto;
}
.renderLoading {
  position: absolute;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
}
</style>
