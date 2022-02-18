<template>
  <div>
    <b-alert style="z-index: 999" id="alert" v-model="alertShow" dismissible>
      Participate in the Verse Testnet node competition, view
      <a
        target="_blank"
        href="https://stp-dao.gitbook.io/verse-network/wallet-setup/interact-with-verse-testnet-using-metamask"
        >tutorials</a
      >.
    </b-alert>

    <div
      class="
        header
        position-fixed
        fixed-top
        d-flex
        flex-column flex-lg-row
        justify-content-lg-between
        align-items-center
        px-lg-4 px-3
      "
      :style="alertShow ? `top: ${alertHeight}px` : '0'"
    >
      <div class="header-left w-100 d-flex justify-content-between">
        <div class="header-logo-content d-flex align-items-center">
          <Logo :logo="logoParams" />
        </div>
        <div class="more-operation d-flex d-lg-none d-block">
          <button ref="moreBtn" class="more-operation-btn font-xlarge-size font-weight-bold">
            <div class="icon-outer-box">
              <b-icon :icon="computedIcon"></b-icon>
            </div>
          </button>
        </div>
      </div>
      <div
        ref="operationList"
        class="operation-list w-100 flex-column flex-lg-row d-lg-flex justify-content-lg-end"
        :class="computedClass"
      >
        <SupportNetworks class="mb-2 mb-lg-0" />
        <UserProfile class="mb-2 mb-lg-0" />
      </div>
    </div>
  </div>
</template>

<script>
import Logo from '@/components/Logo'
import UserProfile from './UserProfile'
import SupportNetworks from './SupportNetworks'
import { mapMutations, mapState } from 'vuex'
export default {
  name: 'Header',
  components: {
    Logo,
    UserProfile,
    SupportNetworks,
  },
  data() {
    return {
      alertShow: true,
      alertHeight: 40,
    }
  },
  computed: {
    ...mapState('modal', ['headerUserOperationActive']),
    ...mapState('token', ['currentNetwork']),
    computedIcon() {
      return this.headerUserOperationActive ? 'x' : 'list'
    },
    computedClass() {
      return this.headerUserOperationActive ? ['d-flex'] : ['d-none']
    },
    logoParams() {
      return {
        src: this.currentNetwork.nativeTokenSymbol
          ? '/img/header/supportnet/' + String(this.currentNetwork.nativeTokenSymbol).toLowerCase() + '.png'
          : '/img/header/supportnet/stpt.png',
        name: this.currentNetwork.name,
      }
    },
  },
  mounted() {
    document.addEventListener('click', this.clickMoreBtnEvent)
    this.alertHeight = document.querySelector('#alert')?.offsetHeight || 40
  },
  methods: {
    ...mapMutations({
      setHeaderUserOperationActive: 'modal/setHeaderUserOperationActive',
    }),
    clickMoreBtnEvent(e) {
      if (this.$refs.moreBtn === e.target) {
        return this.setHeaderUserOperationActive()
      }

      if (this.$refs.operationList && this.$refs.operationList.contains(e.target)) {
        return
      }

      this.setHeaderUserOperationActive(false)
    },
  },
  beforeDestroy() {
    document.removeEventListener('click', this.clickMoreBtnEvent)
  },
}
</script>

<style lang="scss" scoped>
.header {
  min-height: 80px;
  background-color: #fff;
  z-index: 1;
  .header-left {
    height: 80px;
    .header-logo-content {
      .span-wallet-v2 {
        white-space: nowrap;
        font-size: 23px;
        line-height: 80px;
      }
    }
    .more-operation {
      .more-operation-btn {
        background-color: #fff;
      }
      .icon-outer-box {
        pointer-events: none;
      }
    }
  }
}
</style>
