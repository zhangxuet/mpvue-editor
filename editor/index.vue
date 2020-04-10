<template>
  <div class="edit">
    <view class="container" :style="{height: editorHeight + 'px'}">
      <editor id="editor" class="ql-container" :placeholder="placeholder" @change="onStatusChange">
      </editor>
    </view>

    <view class="toolbar" @click="format" :hidden="keyboardHeight > 0 ? false : true" :style="{bottom: isIOS ? keyboardHeight : 0 + 'px'}">
      <i class="iconfont icon-charutupian" @click="insertImage"></i>
      <i class="iconfont icon-format-header-2" :class="formats.header === 2 ? 'ql-active' : ''" data-name="header" data-value="2"></i>
      <i class="iconfont icon-format-header-3" :class="formats.header === 3 ? 'ql-active' : ''" data-name="header" data-value="3"></i>
      <i class="iconfont icon-zitijiacu" :class="formats.bold ? 'ql-active' : ''" data-name="bold"></i>
      <i class="iconfont icon-zitixieti" :class="formats.italic ? 'ql-active' : ''" data-name="italic"></i>
      <i class="iconfont icon-zitixiahuaxian" :class="formats.underline ? 'ql-active' : ''" data-name="underline"></i>
      <i class="iconfont icon--checklist" data-name="list" data-value="check"></i>
      <i class="iconfont icon-youxupailie" :class="formats.list === 'ordered' ? 'ql-active' : ''" data-name="list" data-value="ordered"></i>
      <i class="iconfont icon-wuxupailie" :class="formats.list === 'bullet' ? 'ql-active' : ''" data-name="list" data-value="bullet"></i>
    </view>
  </div>
</template>
<script>
export default {
  data () {
    return {
      formats: {},
      readOnly: false,
      placeholder: '开始输入...',
      editorHeight: 300,
      keyboardHeight: 0,
      isIOS: false
    }
  },
  readOnlyChange () {
    this.readOnly = !this.data.readOnly
  },
  onLoad () {
    const platform = wx.getSystemInfoSync().platform
    const isIOS = platform === 'ios'
    // this.setData({ isIOS})
    this.isIOS = isIOS
    const that = this
    this.updatePosition(0)
    let keyboardHeight = 0
    wx.onKeyboardHeightChange(res => {
      if (res.height === keyboardHeight) return
      const duration = res.height > 0 ? res.duration * 1000 : 0
      keyboardHeight = res.height
      setTimeout(() => {
        wx.pageScrollTo({
          scrollTop: 0,
          success () {
            that.updatePosition(keyboardHeight)
            that.editorCtx.scrollIntoView()
          }
        })
      }, duration)
    })
  },
  calNavigationBarAndStatusBar () {
    const systemInfo = wx.getSystemInfoSync()
    const { statusBarHeight, platform } = systemInfo
    const isIOS = platform === 'ios'
    const navigationBarHeight = isIOS ? 44 : 48
    return statusBarHeight + navigationBarHeight
  },
  blur () {
    this.editorCtx.blur()
  },
  insertDivider () {
    this.editorCtx.insertDivider({
      success: function () {
        console.log('insert divider success')
      }
    })
  },
  clear () {
    this.editorCtx.clear({
      success: function (res) {
        console.log('clear success')
      }
    })
  },
  removeFormat () {
    this.editorCtx.removeFormat()
  },
  insertDate () {
    const date = new Date()
    const formatDate = `${date.getFullYear()}/${date.getMonth() + 1}/${date.getDate()}`
    this.editorCtx.insertText({
      text: formatDate
    })
  },
  methods: {
    onStatusChange (e) {
      this.formats = e.detail
    },
    insertImage () {
      const that = this
      wx.chooseImage({
        count: 1,
        success: function (res) {
          that.editorCtx.insertImage({
            src: res.tempFilePaths[0],
            data: {
              id: 'abcd',
              role: 'god'
            },
            width: '80%',
            success: function () {
              console.log('insert image success')
            }
          })
        }
      })
    },
    format (e) {
      let { name, value } = e.target.dataset
      if (!name) return
      // console.log('format', name, value)
      this.editorCtx.format(name, value)
    },
    updatePosition (keyboardHeight) {
      const toolbarHeight = 50
      const { windowHeight } = wx.getSystemInfoSync()
      let editorHeight = keyboardHeight > 0 ? (windowHeight - keyboardHeight - toolbarHeight) : windowHeight
      // this.setData({ editorHeight, keyboardHeight })
      this.editorHeight = editorHeight
      this.keyboardHeight = keyboardHeight
    },
    onEditorReady () {
      const that = this
      wx.createSelectorQuery().select('#editor').context(function (res) {
        that.editorCtx = res.context
      }).exec()
    }
  },
  mounted () {
    this.onEditorReady()
  }
}
</script>
<style scoped lang="less">
  @import './main.less';
</style>