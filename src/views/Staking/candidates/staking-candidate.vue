<template>
  <CustomizedModal v-model="stakingCandidateParams.show" @close="closeModal">
    <template #modal-title>
      <span class="d-block font-weight-bold text-capitalize">Become a candidate</span>
    </template>
    <template #modal-body>
      <Divider />
      <div v-if="computedStakingCandidateLoading" class="d-flex justify-content-center py-5">
        <div class="spinner-border" role="status">
          <span class="sr-only">Loading...</span>
        </div>
      </div>
      <div v-else class="candidate-modal-body">
        <b-alert show class="mt-3">List {{ account }} as candidate</b-alert>
        <b-form @submit.prevent="onSubmit">
          <!-- amount -->
          <b-form-group label="Amount:" label-for="amount">
            <b-input-group :append="currentNetwork.governanceTokenSymbol || ''">
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
          <!-- name -->
          <b-form-group label="Name:" label-for="name">
            <b-form-input id="name" v-model="formData.name" placeholder="Enter name" required></b-form-input>
          </b-form-group>
          <!-- description -->
          <b-form-group label="Description:" label-for="description">
            <b-form-input
              id="description"
              v-model="formData.description"
              placeholder="Enter description"
              required
            ></b-form-input>
          </b-form-group>
          <!-- IP -->
          <b-form-group label="IP:" label-for="ip">
            <b-input-group>
              <b-form-input
                id="ip"
                v-model="formData.ip"
                placeholder="Enter ip"
                required
                :state="ipValidation"
              ></b-form-input>
              <b-form-invalid-feedback :state="ipValidation" tooltip>
                {{ ipValidationMsg }}
              </b-form-invalid-feedback>
            </b-input-group>
          </b-form-group>
          <!-- port -->
          <b-form-group label="Port:" label-for="port">
            <b-form-input id="port" v-model="formData.port" placeholder="Enter port" required disabled></b-form-input>
          </b-form-group>
          <!-- commission rate -->
          <b-form-group label="Commission Rate:" label-for="commisstionRate">
            <b-input-group append="%">
              <b-form-input
                id="commisstionRate"
                v-model="formData.commisstionRate"
                placeholder="Enter commisstionRate"
                required
              ></b-form-input>
            </b-input-group>
          </b-form-group>
          <!-- public key -->
          <b-form-group label="Public Key:" label-for="publicKey">
            <b-form-textarea
              id="publicKey"
              v-model="formData.publicKey"
              placeholder="Enter publicKey"
              required
            ></b-form-textarea>
          </b-form-group>
          <!-- enable auto bid -->
          <b-form-group>
            <b-form-checkbox v-model="formData.autoBid">Enable auto-bid</b-form-checkbox>
          </b-form-group>
          <b-button class="w-100" type="submit" variant="primary">Submit</b-button>
        </b-form>
      </div>
    </template>
    <template #modal-footer>
      <div class="candidate-modal-footer w-100 py-4">
        <b-button v-if="stakingCandidateHash" @click="goMeterScan" class="w-100" type="button" variant="primary"
          >Verse Scan</b-button
        >
      </div>
    </template>
  </CustomizedModal>
</template>

<script>
import BigNumber from 'bignumber.js'
import { mapActions, mapState } from 'vuex'
import axios from 'axios'
import { ScriptEngine } from '@meterio/devkit'

import { getMeterScanUrl } from '@/api'
import { regExpList } from '@/constants'

export default {
  name: 'StakingCandidateModal',
  props: {
    stakingCandidateParams: {
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
      ipValidationMsg: '',
      amountValidationMsg: '',
      formData: {
        amount: '',
        name: '',
        description: '',
        ip: '',
        port: 8670,
        commisstionRate: '10',
        textarea: '',
        publicKey: '',
        autoBid: true,
      },
    }
  },
  watch: {
    stakingCandidateHash(newVal, oldVal) {
      if (newVal === '' && oldVal.includes('0x')) {
        this.closeModal()
      }
    },
  },
  computed: {
    ...mapState('candidate', ['stakingCandidateLoading']),
    ...mapState('token', ['balances', 'currentNetwork']),
    ...mapState('wallet', ['account', 'chainId']),
    computedStakingCandidateLoading() {
      const hash = this.stakingCandidateLoading[this.formData.name]
      if (hash) {
        if (hash === 'start' || String(hash).includes('0x')) {
          return true
        }
      }
      return false
    },
    stakingCandidateHash() {
      const hash = this.stakingCandidateLoading[this.formData.name]
      if (hash && String(hash).includes('0x')) {
        return hash
      }
      return ''
    },
    amountValidation() {
      if (this.formData.amount == '') {
        return
      }
      const amount = new BigNumber(this.formData.amount)
      if (amount.lt(100)) {
        this.amountValidationMsg = 'Amount should >= 100.'
        return false
      }
      if (amount.gt(this.balances.energy)) {
        this.amountValidationMsg = 'Your balance is insufficient.'
        return false
      }
      return true
    },
    ipValidation() {
      if (this.formData.ip == '') {
        return
      }
      console.log(this.formData.ip)
      console.log(regExpList.ip)
      console.log(new RegExp(regExpList.ip).test(this.formData.ip))
      if (!new RegExp(regExpList.ip).test(this.formData.ip)) {
        this.ipValidationMsg = 'Invalid IP.'
        return false
      }
      return true
    },
  },
  methods: {
    ...mapActions({
      stakingCandidate: 'candidate/stakingCandidate',
    }),
    closeModal() {
      this.$emit('close')
    },
    async checkWithProbe() {
      const url = `http://${this.formData.ip}:${this.formData.port}/probe`
      let data = { pubkey: '' }
      try {
        const res = await axios.get(url, { timeout: 2500 })
        data = res.data
      } catch (e) {
        throw new Error(`port ${this.formData.port} is not open`)
      }
      if (!data.hasOwnProperty('bestQC') || !data.hasOwnProperty('bestBlock') || !data.hasOwnProperty('pubkey')) {
        throw `verse is not correctly running on node ${this.formData.ip}`
      }
      if (data.pubkey != this.formData.publicKey) {
        throw new Error(
          `pubkey mismatch with node config, please compare your pubkey with http://${this.formData.ip}:${this.formData.port}/probe`,
        )
      }
      return true
    },
    async onSubmit() {
      try {
        await this.checkWithProbe()
      } catch (e) {
        // this.errMsg = e.message;
        console.log(e.message)
        return
      }

      const value = new BigNumber('1' + '0'.repeat(18)).times(this.formData.amount).integerValue().toString(10)
      let fromAddr = this.account
      let dataBuffer = ScriptEngine.getCandidateData(
        fromAddr,
        this.formData.name,
        this.formData.description,
        this.formData.publicKey,
        this.formData.ip,
        parseInt(this.formData.port),
        value,
        Math.floor(this.formData.commisstionRate * 100),
        undefined,
        undefined,
        this.formData.autoBid ? 100 : 0,
      )

      this.stakingCandidate({ name: this.formData.name, data: '0x' + dataBuffer.toString('hex') })
    },
    goMeterScan() {
      const url = getMeterScanUrl(this.chainId)
      window.open(`${url}/tx/${this.stakingCandidateHash}`, '_blank')
    },
  },
}
</script>

<style lang="scss" scoped>
.candidate-modal-body {
  padding: 0 32px;
}
.candidate-modal-footer {
  padding: 0 32px;
  overflow-y: auto;
}
</style>
