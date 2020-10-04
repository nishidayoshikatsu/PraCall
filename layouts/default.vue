<template>
  <div>
    <page-header />
    <nuxt />
  </div>
</template>

<script>
import PageHeader from '~/components/PageHeader'

export default {
  components: {
    PageHeader,
  },
}
</script>

<style lang="scss" scoped>
* {
  background-color: #322f2f;
}
</style>
