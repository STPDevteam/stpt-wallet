<template>
  <CustomizedModal v-model="updateCandidateParams.show" @close="closeModal">
    <template #modal-title>
      <span class="d-block font-weight-bold text-capitalize">update candidate</span>
    </template>
    <template #modal-body>
      <Divider />
      <div v-if="computedupdateCandidateLoading" class="d-flex justify-content-center py-5">
        <div class="spinner-border" role="status">
          <span class="sr-only">Loading...</span>
        </div>
      </div>
      <div v-else class="candidate-modal-body">
        <b-form @submit.prevent="onSubmit">
          <!-- candidate address -->
          <b-form-group label="Candidate address:" label-for="candidateaddress">
            <b-form-input
              id="candidateaddress"
              v-model="formData.address"
              placeholder="Enter address"
              disabled
            ></b-form-input>
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
        <b-button v-if="updateCandidateHash" @click="goMeterScan" class="w-100" type="button" variant="primary"
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
import { regExpList } from '@/constants'

export default {
  name: 'CandidateUpdateModal',
  props: {
    updateCandidateParams: {
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
      formData: {
        address: '',
        name: '',
        description: '',
        ip: '',
        port: 8670,
        commisstionRate: '10',
        publicKey: '',
        autoBid: true,
      },
    }
  },
  watch: {
    'updateCandidateParams.data'(v) {
      if (Object.keys(v).length > 0) {
        const { address, name, description, ipAddr, commission, pubKey } = this.updateCandidateParams.data
        this.formData.address = address
        this.formData.name = name
        this.formData.description = description
        this.formData.ip = ipAddr
        this.formData.commisstionRate = commission / 1e7
        this.formData.publicKey = pubKey
      }
    },
    updateCandidateHash(newVal, oldVal) {
      if (newVal === '' && oldVal.includes('0x')) {
        this.closeModal()
      }
    },
  },
  computed: {
    ...mapState('candidate', ['updateCandidateLoading']),
    ...mapState('wallet', ['account', 'chainId']),
    computedupdateCandidateLoading() {
      const hash = this.updateCandidateLoading[this.formData.name]
      if (hash) {
        if (hash === 'start' || String(hash).includes('0x')) {
          return true
        }
      }
      return false
    },
    updateCandidateHash() {
      const hash = this.updateCandidateLoading[this.formData.name]
      if (hash && String(hash).includes('0x')) {
        return hash
      }
      return ''
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
      updateCandidate: 'candidate/updateCandidate',
    }),
    closeModal() {
      this.$emit('close')
    },
    async onSubmit() {
      let dataBuffer = ScriptEngine.getCandidateUpdateData(
        this.account,
        this.formData.name,
        this.formData.description,
        this.formData.publicKey,
        this.formData.ip,
        parseInt(this.formData.port),
        Math.floor(this.formData.commisstionRate * 100),
        undefined,
        undefined,
        this.formData.autoBid ? 100 : 0,
      )

      this.updateCandidate({ name: this.formData.name, data: '0x' + dataBuffer.toString('hex') })
    },
    goMeterScan() {
      const url = getMeterScanUrl(this.chainId)
      window.open(`${url}/tx/${this.updateCandidateHash}`, '_blank')
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
