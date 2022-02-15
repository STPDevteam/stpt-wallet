<template>
  <CustomizedModal v-model="bucketParams.show" @close="closeModal">
    <template #modal-title>
      <span class="d-block font-weight-bold text-capitalize">delegate vote</span>
    </template>
    <template #modal-body>
      <Divider />
      <div v-if="computedDelegateLoading" class="d-flex justify-content-center py-5">
        <div class="spinner-border" role="status">
          <span class="sr-only">Loading...</span>
        </div>
      </div>
      <div v-else class="modal-body pt-3">
        <b-form @submit.prevent="onSubmit">
          <b-alert show>This action uses an existing bucket to vote</b-alert>
          <div class="section">
            <div class="name">Vote ID</div>
            <div class="content">{{ formData.id }}</div>
          </div>

          <div class="section">
            <div class="name">From</div>
            <div class="content">{{ formData.owner }}</div>
          </div>
          <div class="section">
            <div class="name">Votes</div>
            <div class="content">{{ formData.currentAmount }} {{ currentNetwork.governanceTokenSymbol }}</div>
          </div>
          <!-- new candidate -->
          <b-form-group label="New Candidate:" label-for="newcandidate">
            <!-- <b-form-select
              id="newcandidate"
              v-model="formData.newCandidate"
              :options="candidateOptions"
              autofocus="true"
              required
            ></b-form-select> -->
            <div class="v-select-container">
              <v-select
                :class="{ selectError: selectValid }"
                v-model="formData.newCandidate"
                :options="candidateOptions"
                :reduce="(c) => c.value"
              ></v-select>
              <span v-if="selectValid" class="selectStatus">{{ selectValidMsg }}</span>
            </div>
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
      <div class="modal-footer w-100 py-4">
        <b-button v-if="delegateHash" @click="goMeterScan" class="w-100" type="button" variant="primary"
          >Verse Scan</b-button
        >
      </div>
    </template>
  </CustomizedModal>
</template>

<script>
import { mapActions, mapState } from 'vuex'
import { BigNumber } from 'bignumber.js'
import { ScriptEngine } from '@meterio/devkit'

import { getMeterScanUrl } from '@/api'

export default {
  name: 'DelegateModal',
  props: {
    bucketParams: {
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
      selectValidMsg: '',
      formData: {
        id: '',
        owner: '',
        candidate: '',
        newCandidate: '',
        currentAmount: '',
        autoBid: true,
      },
    }
  },
  watch: {
    'bucketParams.data'(v) {
      if (Object.keys(v).length > 0) {
        console.log(this.bucketParams.data)
        const { id, owner, value, candidate } = this.bucketParams.data
        this.formData.id = id
        this.formData.owner = owner
        this.formData.candidate = candidate
        this.formData.currentAmount = new BigNumber(value).div(1e18).toFormat()
      }
    },
    delegateHash(newVal, oldVal) {
      if (newVal === '' && oldVal.includes('0x')) {
        this.closeModal()
      }
    },
  },
  computed: {
    ...mapState('candidate', ['candidates']),
    ...mapState('bucket', ['delegateLoading']),
    ...mapState('wallet', ['account', 'chainId']),
    ...mapState('token', ['currentNetwork']),
    computedDelegateLoading() {
      const hash = this.delegateLoading[this.bucketParams.data.candidateName]
      if (hash) {
        if (hash === 'start' || String(hash).includes('0x')) {
          return true
        }
      }
      return false
    },
    delegateHash() {
      const hash = this.delegateLoading[this.bucketParams.data.candidateName]
      if (hash && String(hash).includes('0x')) {
        return hash
      }
      return ''
    },
    candidateOptions() {
      const formatCandidate = this.candidates
        .map((c) => {
          return {
            label: c.name + '   (' + c.address.substr(0, 8) + '...' + c.address.substr(c.address.length - 6) + ')',
            value: c.address,
          }
        })
        .sort((a, b) => {
          return Math.random() > 0.5 ? 1 : -1
        })
      return [
        {
          label: 'Choose new candidate',
          value: '',
        },
        ...formatCandidate,
      ]
    },
    selectValid() {
      if (!this.formData.newCandidate) {
        this.selectValidMsg = 'Choose candidate please.'
        return true
      }
    },
  },
  methods: {
    ...mapActions({
      delegate: 'bucket/delegate',
    }),
    closeModal() {
      this.$emit('close')
    },
    async onSubmit() {
      const value = new BigNumber(this.bucketParams.data.value).toFixed()
      let holderAddr = this.account
      const dataBuffer = ScriptEngine.getDelegateData(
        holderAddr,
        this.formData.newCandidate,
        this.formData.id,
        value,
        undefined,
        undefined,
        this.formData.autoBid ? 100 : 0,
      )
      const scriptData = '0x' + dataBuffer.toString('hex')
      this.delegate({ name: this.bucketParams.data.candidateName, data: scriptData })
    },
    goMeterScan() {
      const url = getMeterScanUrl(this.chainId)
      window.open(`${url}/tx/${this.delegateHash}`, '_blank')
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
