<template>
  <div class="url-info-box">
    <img src="../assets/logo.svg" class="logo" alt="" style="width: 80px;">
    <p class="description">
     <el-form ref="form" :model="form" >
        <el-form-item >
          <el-input v-model="APP_URL"></el-input>
        </el-form-item>
        <el-form-item>
          <el-button type="primary" @click="onSubmit">确定</el-button>
        </el-form-item>
      </el-form>
    </p>
  </div>
</template>
<script>
export default {
  data () {
    return {
      APP_URL: this.$config.APP_URL || ''
    }
  },
  methods: {
    onSubmit () {
      this.$emit('urlOk', this.APP_URL)
    }
  }
}
</script>

<style type="text/css">
  .url-info-box{
    height: 100%;
    width: 100%;
    min-width: 400px;
    background: #fff;
    text-align: center;
  }

  .app-info-box .description{
    font-size: 1rem;
    line-height: 35px;
  }

</style>
