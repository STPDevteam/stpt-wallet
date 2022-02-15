<template>
  <CustomizedModal v-model="unboundParams.show" @close="closeModal">
    <template #modal-title>
      <span class="d-block text-capitalize">unbound</span>
    </template>
    <template #modal-body>
      <Divider />
      <div v-if="computedUnboundLoading" class="d-flex justify-content-center py-5">
        <div class="spinner-border" role="status">
          <span class="sr-only">Loading...</span>
        </div>
      </div>
      <div v-else class="modal-body">
        <b-form @submit.prevent="onSubmit">
          <div class="alert alert-info my-3" role="alert">
            This action will allow user to withdraw MTRG locked in staking after a lockdown period of 7 days (maturity
            time) <br />
            <b>Caution:</b> <br />
            <ul>
              <li>
                Once successfully <b>Unbound</b>, the <i>State</i> column on the bucket should start with a countdown –
                <i>Maturing in 7 days</i>
              </li>
              <li>
                After the end of 7-day countdown, the MTRG are locked in staking until the end of epoch (maximum of 1
                hr.)
              </li>
              <li>Unbounding without undelegating will earn you staking rewards during the maturity time</li>
            </ul>
          </div>
          <div class="section">
            <div class="name">Vote ID</div>
            <div class="content text-break">{{ unboundParams.data.id }}</div>
          </div>
          <div class="section">
            <div class="name">Vote Owner</div>
            <div class="content">
              <AddressLable :address="unboundParams.data.owner" />
            </div>
          </div>
          <div class="section">
            <div class="name">Vote Amount</div>
            <div class="content">{{ unboundParams.data.votes }}</div>
          </div>
          <b-button class="w-100 my-3" type="submit" variant="primary">Confirm</b-button>
        </b-form>
      </div>
    </template>
    <template #modal-footer>
      <div class="modal-footer w-100 py-4">
        <b-button v-if="unboundHash" @click="goMeterScan" class="w-100" type="button" variant="primary"
          >Verse Scan</b-button
        >
      </div>
    </template>
  </CustomizedModal>
</template>

<script>
import { mapActions, mapState } from 'vuex'
import { ScriptEngine } from '@meterio/devkit'
import BigNumber from 'bignumber.js'

import { getMeterScanUrl } from '@/api'

export default {
  name: 'UnboundModal',
  props: {
    unboundParams: {
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
    return {}
  },
  computed: {
    ...mapState('wallet', ['account', 'chainId']),
    ...mapState('bucket', ['unboundLoading']),
    computedUnboundLoading() {
      const hash = this.unboundLoading[this.unboundParams.data.candidateName]
      if (hash) {
        if (hash === 'start' || String(hash).includes('0x')) {
          return true
        }
      }
      return false
    },
    unboundHash() {
      const hash = this.unboundLoading[this.unboundParams.data.candidateName]
      if (hash && String(hash).includes('0x')) {
        return hash
      }
      return ''
    },
  },
  watch: {
    unboundHash(newVal, oldVal) {
      if (newVal === '' && oldVal.includes('0x')) {
        this.closeModal()
      }
    },
  },
  methods: {
    ...mapActions({
      unboundAction: 'bucket/unbound',
    }),
    closeModal() {
      this.$emit('close')
    },
    onSubmit() {
      const value = new BigNumber(this.unboundParams.data.value).toFixed()
      let holderAddr = this.account
      const dataBuffer = ScriptEngine.getUnboundData(holderAddr, this.unboundParams.data.id, value)
      const scriptData = '0x' + dataBuffer.toString('hex')
      this.unboundAction({ name: this.unboundParams.data.candidateName, data: scriptData })
    },
    goMeterScan() {
      const url = getMeterScanUrl(this.chainId)
      window.open(`${url}/tx/${this.unboundHash}`, '_blank')
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
