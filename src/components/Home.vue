<template>
  <div>
    <el-button type="info" @click="logOut">退出</el-button>
  </div>
</template>

<script>
export default {
  methods:{
    logOut(){
      window.sessionStorage.clear()
      this.$router.push('/login')
    }
  }
}
</script lang="less" scoped>

<style>

</style>