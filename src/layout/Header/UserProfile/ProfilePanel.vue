<template>
  <div class="profile-panel position-absolute right-lg-4 right-3 right-sm-0 container">
    <div class="profile-panel-top d-flex justify-content-between">
      <img class="profile-img" src="/img/header/profile.svg" alt="" />
      <div class="account-info w-100">
        <div class="account-name">
          <span class="account-name-left">Account {{ account.substr(2, 4) }}</span>
          <span class="account-name-right">Verse</span>
        </div>
        <CopyAddress :account="account" :short="headerUserOperationActive" ref="copyAddress" />
      </div>
    </div>
    <Divider />
    <div class="profile-option-list">
      <ProfileOptionListItem
        v-for="item of optionList"
        :key="item.id"
        :option="item"
        @click.native="clickProfileOptionItem(item.id)"
      />
    </div>
  </div>
</template>

<script>
import CopyAddress from '@/components/CopyAddress'
import ProfileOptionListItem from '@/components/ProfileOptionListItem'
import { mapMutations, mapState } from 'vuex'
export default {
  name: 'ProfilePanel',
  components: {
    CopyAddress,
    ProfileOptionListItem,
  },
  data() {
    return {
      optionList: [
        {
          id: 'qrcode',
          svg: 'qrcode',
          name: 'Show QR Code',
        },
        {
          id: 'logout',
          svg: 'logout',
          name: 'Logout',
        },
      ],
    }
  },
  computed: {
    ...mapState('modal', ['headerUserOperationActive']),
    ...mapState('wallet', ['account']),
  },
  methods: {
    ...mapMutations({
      setShowQRCodeModal: 'modal/setShowQRCodeModal',
      setShowLogoutModal: 'modal/setShowLogoutModal',
    }),
    clickProfileOptionItem(id) {
      if (id === 'qrcode') {
        this.setShowQRCodeModal()
      }
      if (id === 'logout') {
        this.setShowLogoutModal()
      }
    },
  },
}
</script>

<style lang="scss" scoped>
.profile-panel {
  border: 1px solid #e9ecef;
  border-radius: 3px;
  background-color: #fff;
  top: 80px;
  max-width: 480px;
  z-index: 1;
  .profile-panel-top {
    padding: 32px;
    .profile-img {
      width: 64px;
      height: 64px;
    }
    .account-info {
      display: flex;
      flex-direction: column;
      justify-content: space-between;
      padding-left: 20px;
      .account-name {
        display: flex;
        flex-direction: row;
        justify-content: space-between;
        .account-name-left {
          font-weight: 600;
          font-size: 14px;
        }
        .account-name-right {
          color: #6e798f;
          font-size: 12px;
        }
      }
    }
  }

  .profile-option-list {
    padding: 12px;
  }
}
</style>
