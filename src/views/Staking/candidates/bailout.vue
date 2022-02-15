<template>
  <CustomizedModal v-model="bailOutParams.show" @close="closeModal">
    <template #modal-title>
      <span class="d-block text-capitalize">bail out</span>
    </template>
    <template #modal-body>
      <Divider />
      <div v-if="computedBailOutLoading" class="d-flex justify-content-center py-5">
        <div class="spinner-border" role="status">
          <span class="sr-only">Loading...</span>
        </div>
      </div>
      <div v-else class="modal-body">
        <div class="section">
          <div class="name">Name</div>
          <div class="content">{{ bailOutParams.data.name }}</div>
        </div>
        <div class="section">
          <div class="name">Public Key</div>
          <div class="content">{{ bailOutParams.data.pubKey }}</div>
        </div>
        <div class="section">
          <div class="name">Total Points</div>
          <div class="content">{{ bailOutParams.data.totalPoints }}</div>
        </div>
        <div class="section">
          <div class="name">Bail Amount</div>
          <div class="content">{{ bailOutParams.data._bailAmount }}</div>
        </div>
        <div class="section">
          <div class="name">Jailed Time</div>
          <div class="content">{{ bailOutParams.data._jailedTime }}</div>
        </div>

        <b-button class="w-100" variant="primary" @click="bailOut">Submit</b-button>
      </div>
    </template>
    <template #modal-footer>
      <div class="modal-footer w-100 py-4">
        <b-button v-if="bailOutHash" @click="goMeterScan" class="w-100" type="button" variant="primary"
          >Verse Scan</b-button
        >
      </div>
    </template>
  </CustomizedModal>
</template>

<script>
import { mapActions, mapState } from 'vuex'

import { getMeterScanUrl } from '@/api'

import { ScriptEngine } from '@meterio/devkit'

export default {
  name: 'BailOutModal',
  props: {
    bailOutParams: {
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
  watch: {
    bailOutHash(newVal, oldVal) {
      if (newVal === '' && oldVal.includes('0x')) {
        this.closeModal()
      }
    },
  },
  computed: {
    ...mapState('wallet', ['account', 'chainId']),
    ...mapState('bailout', ['bailOutLoading']),
    computedBailOutLoading() {
      const hash = this.bailOutLoading[this.bailOutParams.data.name]
      if (hash) {
        if (hash === 'start' || String(hash).includes('0x')) {
          return true
        }
      }
      return false
    },
    bailOutHash() {
      const hash = this.bailOutLoading[this.bailOutParams.data.name]
      if (hash && String(hash).includes('0x')) {
        return hash
      }
      return ''
    },
  },
  methods: {
    ...mapActions({
      bailOutAction: 'bailout/bailOut',
    }),
    bailOut() {
      const dataBuffer = ScriptEngine.getBailOutData(this.account)
      const scriptData = '0x' + dataBuffer.toString('hex')
      this.bailOutAction({ name: this.bailOutParams.data.name, data: scriptData })
    },
    closeModal() {
      this.$emit('close')
    },
    goMeterScan() {
      const url = getMeterScanUrl(this.chainId)
      window.open(`${url}/tx/${this.bailOutHash}`, '_blank')
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
