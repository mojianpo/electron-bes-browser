<template>
  <div class="app-info-box">
    <img src="../assets/logo.svg" class="logo" alt="">
    <h1 class="title">{{title}}</h1>
    <p class="description">
      基于Electron开发的应用浏览器，让WEB应用看起来像原生应用一样COOL<br/>
      浏览器主题可定制，配置简单，实现了简单的网页浏览器功能<br/>
      定制化修改根目录下配置文件：.env.js
    </p>
    <ul class="info">
      <li><strong>Electron: </strong><span>{{electron}}</span></li>
      <li><strong>Chrome: </strong><span>{{chrome}}</span></li>
      <li><strong>Node: </strong><span>{{node}}</span></li>
      <li><strong>v8: </strong><span>{{v8}}</span></li>
      <li><strong>Version: </strong><span>{{version}}</span></li>
    </ul>

    <p class="copyright"><a href="#" @click="open('https://www.github.com/mojianpo')">@mojianpo</a></p>
  </div>
</template>
<script>
export default {
  data () {
    return {
      APP_URL: this.$config.APP_URL,
      title: this.$config.APP_TITLE,
      webview: null,
      electron: process.versions['atom-shell'],
      chrome: process.versions.chrome,
      node: process.versions.node,
      v8: process.versions.v8,
      // platform: require('os').platform(),
      version: this.$config.APP_VERSION
    }
  },
  methods: {
    openAppUrl () {
      let url = this.APP_URL
      if (url) {
        this.open(url)
      }
    },
    open (link) {
      this.$electron.shell.openExternal(link)
    }
  }
}
</script>

<style type="text/css">
  .app-info-box{
    height: 100%;
    width: 100%;
    background: #fff;
    text-align: center;
  }
  .app-info-box .logo{
    max-width: 100px;
  }
  .app-info-box .title{
    font-size: 2rem;
    font-weight: 100;
    margin: 0 auto;
  }
  .app-info-box .description{
    font-size: 1rem;
    line-height: 35px;
    text-align: center;
  }
  .app-info-box .info{
    margin-top: 20px;
    display: block;
  }
  .app-info-box .info:after,
  .app-info-box .info:before{
    content: "";
    display: table;
    clear: both;
  }
  .app-info-box .info li{
    display: inline-block;
    list-style: none;
    margin-right: 5px;
  }
  .app-info-box .copyright{
    margin-top: 15px;
    color: #666;
    font-size: 14px;
  }
 
</style>
