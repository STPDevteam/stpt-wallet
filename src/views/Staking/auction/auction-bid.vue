<template>
  <CustomizedModal v-model="auctionBidParams.show" @close="closeModal">
    <template #modal-title>
      <span class="d-block text-capitalize">auction bid</span>
    </template>
    <template #modal-body>
      <Divider />
      <div v-if="computedAuctionBidLoading" class="d-flex justify-content-center py-5">
        <div class="spinner-border" role="status">
          <span class="sr-only">Loading...</span>
        </div>
      </div>
      <div v-else class="modal-body pt-3">
        <b-form @submit.prevent="onSubmit">
          <div class="section">
            <div class="name">Auction ID</div>
            <div class="content">{{ auctionBidParams.data.auctionID }}</div>
          </div>
          <div class="section">
            <div class="name">MTRG on Auction</div>
            <div class="content">{{ auctionBidParams.data._releasedMTRG }}</div>
          </div>
          <div class="section">
            <div class="name">Received MTR</div>
            <div class="content">{{ auctionBidParams.data._receivedMTR }}</div>
          </div>
          <div class="section">
            <div class="name">Expected Price</div>
            <div class="content">{{ expectedPrice }}</div>
          </div>
          <div class="section">
            <div class="name">Amount</div>
            <div class="content">
              <b-form-group label-for="amount">
                <b-input-group :append="currentNetwork.nativeTokenSymbol || ''">
                  <b-form-input
                    id="amount"
                    v-model="formData.amount"
                    placeholder="Enter amount"
                    required
                    :state="amountValidation"
                  ></b-form-input>
                  <b-form-invalid-feedback :state="amountValidation" tooltip>
                    {{ amountValidationMsg }}
                  </b-form-invalid-feedback>
                </b-input-group>
              </b-form-group>
            </div>
          </div>
          <div class="section">
            <div class="name">Estimated result MTRG with current price</div>
            <div class="content">{{ estimateMTRG }}</div>
          </div>
          <b-button class="w-100 mt-1" type="submit" variant="primary">Submit</b-button>
        </b-form>
      </div>
    </template>
    <template #modal-footer>
      <div class="modal-footer w-100 py-4">
        <b-button v-if="auctionBidHash" @click="goMeterScan" class="w-100" type="button" variant="primary"
          >Verse Scan</b-button
        >
      </div>
    </template>
  </CustomizedModal>
</template>

<script>
import { mapActions, mapState } from 'vuex'
import { ScriptEngine } from '@meterio/devkit'

import { getMeterScanUrl } from '@/api'
import BigNumber from 'bignumber.js'

export default {
  name: 'AuctionBidModal',
  props: {
    auctionBidParams: {
      type: Object,
      default() {
        return {
          show: false,
          data: {},
        }
      },
    },
  },
  data() {
    return {
      amountValidationMsg: '',
      formData: {
        amount: '',
      },
    }
  },
  watch: {
    auctionBidHash(newVal, oldVal) {
      if (newVal === '' && oldVal.includes('0x')) {
        this.closeModal()
      }
    },
  },
  computed: {
    ...mapState('wallet', ['account', 'chainId']),
    ...mapState('auction', ['bidLoading']),
    ...mapState('token', ['currentNetwork', 'balances']),
    computedAuctionBidLoading() {
      const hash = this.bidLoading[this.auctionBidParams.data.name]
      if (hash) {
        if (hash === 'start' || String(hash).includes('0x')) {
          return true
        }
      }
      return false
    },
    auctionBidHash() {
      const hash = this.bidLoading[this.auctionBidParams.data.name]
      if (hash && String(hash).includes('0x')) {
        return hash
      }
      return ''
    },
    expectedPrice() {
      let price = new BigNumber(this.auctionBidParams.data.receivedMTR).dividedBy(
        this.auctionBidParams.data.releasedMTRG,
      )
      return price.toFixed()
    },
    amountValidation() {
      if (this.formData.amount == '') {
        return
      }
      const amount = new BigNumber(this.formData.amount)
      if (amount.lte(0)) {
        this.amountValidationMsg = 'Amount should >= 0.'
        return false
      }
      if (amount.gt(this.balances.native)) {
        this.amountValidationMsg = 'Your balance is insufficient.'
        return false
      }
      return true
    },
    estimateMTRG() {
      if (Number.isNaN(this.expectedPrice)) {
        return NaN
      }
      return new BigNumber(this.formData.amount).dividedBy(this.expectedPrice).toFixed()
    },
  },
  methods: {
    ...mapActions({
      auctionBid: 'auction/auctionBid',
    }),
    closeModal() {
      this.$emit('close')
    },
    onSubmit() {
      const fromAddr = this.account
      const value = new BigNumber('1' + '0'.repeat(18)).times(this.formData.amount).toFixed()
      const dataBuffer = ScriptEngine.getBidData(fromAddr, value)
      const scriptData = '0x' + dataBuffer.toString('hex')
      this.auctionBid({ name: this.auctionBidParams.data.name, data: scriptData })
    },
    goMeterScan() {
      const url = getMeterScanUrl(this.chainId)
      window.open(`${url}/tx/${this.auctionBidHash}`, '_blank')
    },
  },
}
</script>

<style lang="scss" scoped>
.modal-body {
  padding: 0 32px;
}
.modal-footer {
  padding: 0 32px;
  overflow-y: auto;
}
</style>
