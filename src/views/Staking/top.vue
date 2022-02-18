<template>
  <div class="top-content bg-myprimary-color">
    <b-container>
      <div class="amount-content mt-4">
        <b-row>
          <b-col md="6" class="d-flex flex-wrap justify-content-between">
            <div v-for="(item, index) in computedBalance" :key="index">
              <div class="symbol-label text-right">{{ item.symbol || '-' }}</div>
              <div class="balance-label">{{ item.balance || '-' }}</div>
            </div>
          </b-col>
          <b-col md="6" class="d-flex flex-wrap justify-content-md-end my-3">
            <b-button :class="{ 'btn-color': status === 'candidate' }" variant="mylight" @click="candidate">
              <b-icon icon="people-fill"></b-icon>
              <span class="ml-1 font-weight-bold">Candidates</span>
            </b-button>
            <div class="mx-md-2"></div>
            <b-button :class="{ 'btn-color': status === 'vote' }" variant="mylight" @click="bucket">
              <b-icon icon="lock-fill"></b-icon>
              <span class="ml-1 font-weight-bold">Votes</span>
            </b-button>
            <div class="mx-md-2"></div>
            <b-button :class="{ 'btn-color': status === 'auction' }" variant="mylight" @click="auction">
              <b-icon icon="hourglass-split"></b-icon>
              <span class="ml-1 font-weight-bold">Auction</span>
            </b-button>
          </b-col>
        </b-row>
      </div>
    </b-container>
  </div>
</template>
<script>
import { mapMutations, mapState } from 'vuex'
import { BigNumber } from 'bignumber.js'
export default {
  name: 'WalletTop',
  computed: {
    ...mapState('token', ['balances', 'currentNetwork']),
    ...mapState('modal', ['status']),
    computedBalance() {
      return [
        {
          balance: this.balances.native ? new BigNumber(this.balances.native).toFormat(2) : '--',
          symbol: this.currentNetwork.nativeTokenSymbol || 'STPT',
        },
        {
          balance: this.balances.energy ? new BigNumber(this.balances.energy).toFormat(2) : '--',
          symbol: this.currentNetwork.governanceTokenSymbol || 'STPD',
        },
        {
          balance: this.balances.bound ? new BigNumber(this.balances.bound).toFormat(2) : '--',
          symbol: 'Locked ' + (this.currentNetwork.governanceTokenSymbol || 'STPD'),
        },
      ]
    },
  },
  methods: {
    ...mapMutations({
      setStatus: 'modal/setStatus',
    }),
    candidate() {
      this.setStatus('candidate')
    },
    bucket() {
      this.setStatus('vote')
    },
    auction() {
      this.setStatus('auction')
    },
  },
}
</script>
<style lang="scss" scoped>
.top-content {
  margin-top: 10px;
  margin-bottom: 10px;
  padding: 80px 0px;
  .current-network-name {
    color: hsla(0, 0%, 100%, 0.6);
  }
  .symbol-label {
    color: hsla(0, 0%, 100%, 0.6);
  }
  .balance-label {
    font-size: 30px;
    font-weight: bold;
    color: #fff;
  }
  .amount-content {
    .amount {
      font-size: 30px;
      color: #fff;
      .token-symbol {
        color: lightgray;
        font-size: 20px;
        font-weight: normal;
      }
    }
    ::v-deep .btn {
      &.btn-color {
        color: #003cb2;
        background-color: white;
      }
    }
  }
}
</style>
