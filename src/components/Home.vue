<template>
  <div>
    <el-button type="info" @click="logout">注销</el-button>
  </div>
</template>

<script>
export default {
  methods: {
    logout() {
      window.sessionStorage.clear()
      this.$message.info('成功注销')
      this.$router.push('/login')
    }
  }
}
</script>

<style>
</style>