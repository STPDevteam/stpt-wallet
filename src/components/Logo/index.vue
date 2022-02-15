<template>
  <div class="logo d-flex justify-content-between align-items-center">
    <img class="logo-img" :src="logo.src" alt="" />
    <h1 class="logo-name text-nowrap font-xlarge-size font-weight-bold ml-2 my-0">{{ logo.name }} Staking</h1>
  </div>
</template>

<script>
export default {
  name: 'Logo',
  props: {
    logo: {
      type: Object,
      default() {
        return {
          src: '/logo.png',
          name: 'Verse',
        }
      },
    },
  },
}
</script>

<style lang="scss" scoped>
.logo {
  .logo-img {
    width: 40px;
    height: 40px;
  }
}
</style>
