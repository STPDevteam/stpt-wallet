<template>
  <CustomizedModal v-model="voteParams.show" @close="closeModal">
    <template #modal-title>
      <span class="d-block font-weight-bold text-capitalize">vote to candidate</span>
    </template>
    <template #modal-body>
      <Divider />
      <div v-if="computedStakingVoteLoading" class="d-flex justify-content-center py-5">
        <div class="spinner-border" role="status">
          <span class="sr-only">Loading...</span>
        </div>
      </div>
      <div v-else class="passport-modal-body mt-3">
        <b-form @submit.prevent="onSubmit">
          <!-- lock period -->
          <div class="section">
            <div class="name">Lock Period</div>
            <div class="content">One week</div>
          </div>
          <!-- source -->
          <b-form-group label="Source:" label-for="source">
            <b-form-select id="source" v-model="formData.source" :options="sourceOptions" required></b-form-select>
          </b-form-group>
          <!-- candidate -->
          <b-form-group label="Candidate:" label-for="candidate">
            <!-- <b-form-select
              id="candidate"
              v-model="formData.candidate"
              :options="candidateOptions"
              required
            ></b-form-select> -->
            <div class="v-select-container">
              <v-select
                :class="{ selectError: selectValid }"
                v-model="formData.candidate"
                :options="candidateOptions"
                :reduce="(c) => c.value"
              ></v-select>
              <span v-if="selectValid" class="selectStatus">{{ selectValidMsg }}</span>
            </div>
          </b-form-group>
          <!-- amount -->
          <b-form-group v-if="formData.source === 'bound'" label="Amount:" label-for="amount">
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
          <!-- bucket -->
          <b-form-group v-if="formData.source === 'delegate'" label="Existing Vote:" label-for="bucket">
            <b-form-select id="bucket" v-model="formData.bucketID" :options="bucketOptions" required></b-form-select>
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
      <div class="passport-modal-footer w-100 py-4">
        <b-button v-if="stakingVoteHash" @click="goMeterScan" class="w-100" type="button" variant="primary"
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
  name: 'StakingVoteModal',
  props: {
    voteParams: {
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
      selectValidMsg: '',
      formData: {
        lockPeriod: 1,
        source: 'bound',
        candidate: '',
        amount: '',
        autoBid: true,
        bucketID: '',
      },
      lockPeriodOptions: [
        { text: 'Lock for one week', value: 1 },
        { text: 'Lock for two weeks', value: 2 },
        { text: 'Lock for three weekds', value: 3 },
        { text: 'Lock for four weeks', value: 4 },
      ],
      sourceOptions: [
        { text: 'Governance balance', value: 'bound' },
        { text: 'Existing vote', value: 'delegate' },
      ],
    }
  },
  watch: {
    'voteParams.data'() {
      this.formData.candidate = this.voteParams.data.address
    },
    stakingVoteHash(newVal, oldVal) {
      if (newVal === '' && oldVal.includes('0x')) {
        this.closeModal()
      }
    },
  },
  computed: {
    ...mapState('candidate', ['candidates', 'stakingVoteLoading']),
    ...mapState('bucket', ['buckets']),
    ...mapState('wallet', ['account', 'chainId']),
    ...mapState('token', ['balances', 'currentNetwork']),
    computedStakingVoteLoading() {
      const hash = this.stakingVoteLoading[this.voteParams.data.name]
      if (hash) {
        if (hash === 'start' || String(hash).includes('0x')) {
          return true
        }
      }
      return false
    },
    stakingVoteHash() {
      const hash = this.stakingVoteLoading[this.voteParams.data.name]
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
    bucketOptions() {
      return this.buckets
        .filter((b) => {
          return String(b.owner) == this.account
        })
        .map((b) => {
          return {
            text:
              b.id.substr(0, 8) +
                '...' +
                b.id.substr(b.id.length - 6) +
                ' (' +
                new BigNumber(b.totalVotes).minus(b.bonusVotes).dividedBy(1e18).toFixed(2, 1) +
                ') ' +
                this.currentNetwork.governanceTokenSymbol || '' + ')',
            value: b.id,
          }
        })
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
    selectValid() {
      if (!this.formData.candidate) {
        this.selectValidMsg = 'Choose candidate please.'
        return true
      }
    },
  },
  methods: {
    ...mapActions({
      stakingVote: 'candidate/stakingVote',
    }),
    closeModal() {
      this.$emit('close')
    },
    async onSubmit() {
      const holderAddr = this.account
      let dataBuffer = ''
      if (this.formData.source === 'bound') {
        const value = new BigNumber('1' + '0'.repeat(18)).times(this.formData.amount).integerValue().toString(10)
        dataBuffer = ScriptEngine.getBoundData(
          this.formData.lockPeriod,
          holderAddr,
          this.formData.candidate,
          value,
          undefined,
          undefined,
          this.formData.autoBid ? 100 : 0,
        )
      } else if (this.formData.source === 'delegate') {
        let bucket
        for (var i in this.buckets) {
          const b = this.buckets[i]
          if (b.id == this.formData.bucketID) {
            bucket = b
          }
        }
        if (!bucket) {
          this.errMsg = 'could not find bucket with the given ID'
          return
        }

        dataBuffer = ScriptEngine.getDelegateData(
          holderAddr,
          this.formData.candidate,
          this.formData.bucketID,
          bucket.votes,
          undefined,
          undefined,
          this.formData.autoBid ? 100 : 0,
        )
      }
      const scriptData = '0x' + dataBuffer.toString('hex')
      this.stakingVote({ name: this.voteParams.data.name, data: scriptData })
    },
    goMeterScan() {
      const url = getMeterScanUrl(this.chainId)
      window.open(`${url}/tx/${this.stakingVoteHash}`, '_blank')
    },
  },
}
</script>

<style lang="scss" scoped>
.passport-modal-body {
  padding: 0 32px;
}
.passport-modal-footer {
  padding: 0 32px;
  overflow-y: auto;
}
</style>
