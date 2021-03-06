<template>
<div id="app" :style="style">
<el-container class="browser">
  <el-header class="browser-header" :height="theme.titleBarHeight"
    :style="{
      'line-height': theme.titleBarHeight,
      'background-color': theme.titleBarBgColor,
      'color': theme.titleBarTextColor,
      'margin-top': hideTitleBar ? ('-' + theme.titleBarHeight) : ''
    }"
    >
    <div class="browser-title">
      <img src="./assets/logo.svg" alt="" class="app-icon" @click="handleShowMenu">
      <span class="app-name">{{name}}</span>
      <span class="app-title" v-if="title">&nbsp;-&nbsp;{{title}}</span>
    </div>
    <div class="header-right">
      <!-- <button class="ctrl-icon fullscreen" title="全屏切换" @click="handleFullScreen"></button> -->
      <button class="ctrl-icon minimize" title="最小化窗口" @click="handleMinimize"></button>
      <button class="ctrl-icon maximize" title="最大化窗口" @click="handleMaximize"></button>
      <button class="ctrl-icon close" title="关闭窗口" @click="handleClose"></button>
    </div>
  </el-header>
  <el-container>
    <!-- <el-aside class="browser-aside" :width="sideWidth">Aside</el-aside> -->
    <el-main class="browser-body">
      <router-view></router-view>
      <el-dialog
        title="关于"
        :visible.sync="showAppInfo"
        :modal-append-to-body="true"
        >
        <app-info></app-info>
      </el-dialog>
       <el-dialog
        title="切换URL"
        :visible.sync="showUrlInfo"
        :modal-append-to-body="true"
        width="500px"
        >
        <url-info @urlOk="urlOk"></url-info>
      </el-dialog>
    </el-main>
  </el-container>
</el-container>
</div>
</template>

<script>
import AppInfo from '@/components/app-info'
import UrlInfo from '@/components/url-info'
export default {
  name: 'bes-electron-chrome',
  components: {
    AppInfo,
    UrlInfo
  },
  data () {
    return {
      theme: {
        titleBarHeight: (this.$config.APP_TITLE_BAR_HEIGHT || 30) + 'px',
        titleBarBgColor: this.$config.APP_TITLE_BAR_BGCOLOR,
        titleBarTextColor: this.$config.APP_TITLE_BAR_TEXTCOLOR,
        footerBarHeight: (this.$config.APP_FOOTER_BAR_HEIGHT || 30) + 'px',
        windowBorderColor: this.$config.APP_WINDOWN_BORDER_COLOR,
        windowBorder: this.$config.APP_WINDOWN_BORDER
      },
      hideTitleBar: this.$config.APP_TITLE_BAR_HIDE === 'true',
      sideWidth: '150px',
      name: (this.$config.APP_NAME || '') + ' ' + (this.$config.APP_VERSION || ''),
      title: this.$config.APP_TITLE || '',
      menu: null,
      showAppInfo: false,
      showUrlInfo: false
    }
  },
  computed: {
    style () {
      let style = {}
      if (this.theme) {
        style['border'] = this.theme.windowBorder
        style['border-color'] = this.theme.windowBorderColor
      }
      return style
    }
  },
  methods: {
    urlOk (url) {
      this.showUrlInfo = false
      if (url && this.$root) {
        this.$root.webview && this.$root.webview.loadURL(url)
      }
    },
    currentWindow () {
      return this.$electron.remote.getCurrentWindow()
    },
    handleFullScreen () {
      let win = this.currentWindow()
      if (win.isFullScreen()) {
        win.setFullScreen(false)
      } else {
        win.setFullScreen(true)
      }
    },
    handleMinimize () {
      let win = this.currentWindow()
      win.minimize()
    },
    handleMaximize () {
      let win = this.currentWindow()
      if (win.isMaximized()) {
        win.unmaximize()
      } else {
        win.maximize()
      }
    },
    handleClose () {
      let win = this.currentWindow()
      if (confirm('你确定要关闭应用吗？')) {
        win.close()
      }
    },
    handleShowMenu () {
      this.menu.popup(this.currentWindow(), 0, 31)
    },
    initMenu () {
      let that = this
      let Menu = this.$electron.remote.Menu
      let MenuItem = this.$electron.remote.MenuItem
      let win = this.currentWindow()
      // this.menu = Menu.setApplicationMenu(Menu.buildFromTemplate(MenuTemplate))
      let skipTaskbar = this.$config.skipTaskbar
      let contextMenuItem = [
        {
          label: '返回',
          click: () => {
            // console.log(this)
            that.$root.webview && that.$root.webview.goBack()
          }
        },
        {
          label: '前进',
          click: () => {
            // console.log(contextMenuItem[0].enabled = false)
            that.$root.webview && that.$root.webview.goForward()
          }
        },
        {
          label: '重新加载页面',
          click () {
            that.$root.webview && that.$root.webview.reload()
          }
        },
        {type: 'separator'},
        {label: '切换全屏', role: 'togglefullscreen'},
        {
          label: '显示/隐藏标题栏',
          click () {
            that.hideTitleBar = !that.hideTitleBar
          }
        },
        {
          label: '显示/隐藏任务栏图标',
          click () {
            win.setSkipTaskbar(!skipTaskbar)
            skipTaskbar = !skipTaskbar
          }
        },
        {type: 'separator'},
        {
          label: '强制重新加载框架',
          click () {
            that.$root.webview && that.$root.webview.reloadIgnoringCache()
          }
        },
        {label: '重新加载应用', role: 'reload'},
        {label: '强制加载应用', role: 'forcereload'},
        {type: 'separator'},
        {label: '应用控制台', role: 'toggledevtools'},
        {
          label: '审查元素',
          click () {
            if (that.$root.webview) {
              if (that.$root.webview.isDevToolsOpened()) {
                that.$root.webview.closeDevTools()
              } else {
                that.$root.webview.openDevTools()
              }
            }
          }
        }
      ]
      let contextMenu = new Menu()
      for (let i in contextMenuItem) {
        contextMenu.append(new MenuItem(contextMenuItem[i]))
      }
      if (that.$config.contextmenu) {
        document.addEventListener('contextmenu', (e) => {
          contextMenu.popup(that.currentWindow(), e.x, e.y)
        })
      }
      let menu = new Menu()
      menu.append(new MenuItem({
        label: '主页',
        click () {
          if (that.$config.APP_URL) {
            that.$root.webview && that.$root.webview.loadURL(that.$config.APP_URL)
          }
        }
      }))
      menu.append(new MenuItem({
        label: '切换',
        click () {
          that.showUrlInfo = true
        }
      }))
      menu.append(new MenuItem({label: '调试', submenu: contextMenuItem}))
      menu.append(new MenuItem({label: '切换全屏', role: 'togglefullscreen'}))
      menu.append(new MenuItem({label: '关闭退出', role: 'quit'}))
      menu.append(new MenuItem({
        label: '重启应用',
        click () {
          that.$electron.ipcRenderer.send('relaunch')
        }
      }))
      menu.append(new MenuItem({
        label: '关于',
        click () {
          that.showAppInfo = true
        }
      }))

      this.menu = menu
    }
  },
  created () {
    this.initMenu()
  }
}
</script>
<style type="text/css">
  #app{
    margin: 0 auto;
    position: absolute;
    width: 100%;
    height: 100%;
    border: solid 1px #181A1F;
  }
  .browser{
    height: 100%;
  }
  .browser-aside{
    background: #21252B;
    border-right: solid 1px #181A1F;
    padding: 5px;
    color: #F0F0F2;
  }
  .browser-header{
    background: #21252B;
    color: #F0F0F2;
    line-height: 30px;
    padding: 0 8px;
    font-size: 13px;
    /*border-bottom: solid 1px #181A1F;*/
    box-sizing: border-box;
    -webkit-app-region: drag;
    user-select:none;
  }
  .browser-footer{
    background: #21252B;
    color: #F0F0F2;
    line-height: 30px;
  }
  .app-icon{
    width: 20px;
    height: 20px;
    vertical-align: middle;
    margin-top: -5px;
    margin-right: 5px;
    -webkit-app-region: no-drag;
  }
  .header-left{
    float: left;
  }
  .browser-title{
    float: left;
    white-space: nowrap;
    text-overflow: ellipsis;
    overflow: hidden;
  }
  .header-right{
    float: right;
  }
  .browser-body{
    padding: 0;
  }
  .ctrl-icon{
    display: inline-block;
    outline: none;
    border: none;
    width: 12px;
    height: 12px;
    margin-right: 5px;
    border-radius: 50%;
    background: #BFBFBF;
    -webkit-app-region: no-drag;
  }
  .ctrl-icon:active{
    opacity: 0.7;
  }
  .ctrl-icon:last-child{
    margin-right: 0;
  }
  .ctrl-icon.minimize{
    background: #35CB4B;
  }
  .ctrl-icon.maximize{
    background: #FDBD41;
  }
  .ctrl-icon.close{
    background: #FC625D;
  }
</style>
