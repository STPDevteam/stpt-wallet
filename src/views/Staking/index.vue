<template>
  <div class="wallet">
    <TopContent />
    <!-- token table -->
    <BodyContainer v-if="isSupportNetwork" />
    <CustomizedModal v-model="unSupportModal">
      <template #modal-title>
        <span class="d-block font-weight-bold text-capitalize">Notice</span>
      </template>
      <template #modal-body>
        <div class="my-alert alert alert-danger mt-2 text-center font-weight-bold" role="alert">
          Please connect to a supported network in the dropdown menu.
        </div>
      </template>
      <template #modal-footer>
        <div></div>
      </template>
    </CustomizedModal>
    <!-- <div v-else class="alert alert-danger mt-2 text-center font-weight-bold" role="alert">
      Please connect to a supported network in the dropdown menu.
    </div> -->
  </div>
</template>

<script>
import TopContent from './top.vue'
import BodyContainer from './body-container.vue'
import { mapActions, mapState } from 'vuex'
export default {
  name: 'Wallet',
  components: {
    TopContent,
    BodyContainer,
  },
  data() {
    return {
      timer: null,
      unSupportModal: false,
    }
  },
  watch: {
    isSupportNetwork(_new) {
      if (_new) this.unSupportModal = false
      else this.unSupportModal = true
    },
  },
  computed: {
    ...mapState('token', ['isSupportNetwork']),
  },
  created() {
    this.timer = setInterval(() => {
      this.getCandidatesNoLoading()
      this.getBucketsNoLoading()
    }, 1000 * 30)
  },
  methods: {
    ...mapActions({
      getCandidatesNoLoading: 'candidate/getCandidatesNoLoading',
      getBucketsNoLoading: 'bucket/getBucketsNoLoading',
    }),
  },
  beforeDestroy() {
    clearInterval(this.timer)
  },
}
</script>

<style lang="scss" scoped>
.my-alert {
  margin: 0 24px 20px;
}
</style>
