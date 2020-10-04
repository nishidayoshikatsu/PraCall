<template>
  <div class="page-header">
    <nuxt-link to="/">
      <img src="~/static/solar-system.svg" width="60" height="60" />
      <h3>PraCall</h3>
    </nuxt-link>
  </div>
</template>

<script></script>

<style lang="scss" scoped>
.page-header {
  width: 100vw;
  height: 80px;
  background-color: #3c78d8ff;
  display: flex;
  align-self: center;
  a {
    display: flex;
    align-self: center;
    flex-direction: row;
    margin: 15px;
    h3 {
      color: red;
      display: flex;
      align-self: center;
      margin-left: 15px;
      font-size: 35px;
      text-decoration: none;
    }
  }
}
</style>
