<template>
  <div class="tinymce-editor">
    <editor v-model="myValue" :init="init" :disabled="disabled" @onClick="onClick"></editor>
  </div>
</template>
<script>
import tinymce from 'tinymce/tinymce'
import Editor from '@tinymce/tinymce-vue'
import 'tinymce/icons/default/icons.min.js'
import 'tinymce/themes/silver'
// 编辑器插件plugins
// 更多插件参考：https://www.tiny.cloud/docs/plugins/
import 'tinymce/plugins/image' // 插入上传图片插件
import 'tinymce/plugins/media' // 插入视频插件
import 'tinymce/plugins/table' // 插入表格插件
import 'tinymce/plugins/lists' // 列表插件
import 'tinymce/plugins/wordcount' // 字数统计插件
import 'tinymce/plugins/preview' // 预览
import 'tinymce/plugins/autolink' // 自动连接
import 'tinymce/plugins/fullscreen' // 全屏
import 'tinymce/plugins/link' // 连接

export default {
  components: {
    Editor
  },
  props: {
    value: {
      type: String,
      default: ''
    },
    disabled: {
      type: Boolean,
      default: false
    },
    plugins: {
      type: [String, Array],
      default:
        'lists image media table wordcount preview autolink fullscreen link'
    },
    toolbar: {
      type: [String, Array],
      default:
        'undo redo |  formatselect | bold italic forecolor backcolor | alignleft aligncenter alignright alignjustify | bullist numlist outdent indent | lists image media table | removeformat preview fullscreen link'
    }
  },
  data() {
    return {
      files: [],
      init: {
        language_url: '/tinymce/zh_CN.js', //public目录下
        language: 'zh_CN',
        skin_url: '/tinymce/skins/ui/oxide', //public目录下
        height: 300,
        elementpath: false,
        plugins: this.plugins, // 父组件传入 或者 填写个默认的插件 要选用什么插件都可以 去官网可以查到
        toolbar: this.toolbar, // 工具栏 我用到的也就是lists image media table wordcount 这些 根据需求而定
        images_upload_url:
          'https://api-daily.3songshu.com/ops/ops/sys/file/uploadFile', //上传路径
        // 此处为图片上传处理函数，这个直接用了base64的图片形式上传图片，
        // 如需ajax上传可参考https://www.tiny.cloud/docs/configure/file-image-upload/#images_upload_handler

        // 官网抄的图片上传 项目如果用了vue-resource可以用$http 因为比较懒就没改
        file_picker_types: 'media',
        media_live_embeds: true,
        file_picker_callback: function(cb, value, meta) {
          //当点击meidia图标上传时,判断meta.filetype == 'media'有必要，因为file_picker_callback是media(媒体)、image(图片)、file(文件)的共同入口
          if (meta.filetype == 'media') {
            //创建一个隐藏的type=file的文件选择input
            let input = document.createElement('input')
            input.setAttribute('type', 'file')
            input.setAttribute('accept', 'video/mp4')
            // 文件改变判断
            input.onchange = function() {
              let file = this.files[0]
              /*
               * 上传前的校验 例如
               * 1.文件大小不能超过5M
               **/
              var isGT5M = file.size / 1024 / 1024 > 50
              if (isGT5M) {
                console.log('对不起您上传的文件大于50M 请重新上传！')
                // this.$message({
                //   message: '对不起您上传的文件大于50M 请重新上传！',
                //   type: 'warning'
                // })
                return false
              }
              uploadFile(file)
            }
            //触发点击
            input.click()
            // 上传文件
            var uploadFile = function(file) {
              var xhr, formData
              xhr = new XMLHttpRequest()
              xhr.open(
                'POST',
                'https://api-daily.3songshu.com/ops/ops/sys/file/uploadFile'
              )
              xhr.withCredentials = self.credentials
              xhr.upload.onprogress = function(e) {
                // 进度(e.loaded / e.total * 100)
                // 进度(e.loaded / e.total * 100)
                let percent = (e.loaded / e.total) * 100
                if (percent < 100) {
                  tinymce.activeEditor.setProgressState(true) //是否显示loading状态：1（显示）0（隐藏）
                } else {
                  tinymce.activeEditor.setProgressState(false)
                }
              }
              xhr.onerror = function() {
                console.log(xhr.status)
                tinymce.activeEditor.setProgressState(false)
                return
              }
              xhr.onerror = function() {
                //根据自己的需要添加代码
                console.log(xhr.status)
                return
              }
              xhr.onload = function() {
                let json
                if (xhr.status < 200 || xhr.status >= 300) {
                  console.log('HTTP 错误: ' + xhr.status)
                  return
                }
                json = JSON.parse(xhr.responseText)
                //假设接口返回JSON数据为{status: 0, msg: "上传成功", data: {location: "/localImgs/1546434503854.mp4"}}
                if (json.success) {
                  //接口返回的文件保存地址
                  let mediaLocation = json.data.url
                  //cb()回调函数，将mediaLocation显示在弹框输入框中
                  cb(mediaLocation, { title: file.name })
                } else {
                  console.log(json.msg)
                  return
                }
              }
              formData = new FormData()
              //假设接口接收参数为file,值为选中的文件
              formData.append('file', file)
              //正式使用将下面被注释的内容恢复
              xhr.send(formData)
            }
          }
        },
        images_upload_handler: (blobInfo, success, failure) => {
          var xhr, formData
          xhr = new XMLHttpRequest()
          xhr.withCredentials = false
          xhr.open(
            'POST',
            'https://api-daily.3songshu.com/ops/ops/sys/file/uploadFile'
          )
          xhr.onload = function() {
            var json
            if (xhr.status != 200) {
              failure('HTTP Error: ' + xhr.status)
              return
            }
            json = JSON.parse(xhr.responseText)
            success(json.data.url)
          }
          formData = new FormData()
          formData.append('file', blobInfo.blob(), blobInfo.filename())
          xhr.send(formData)
        }
      },
      myValue: this.value
    }
  },
  watch: {
    value(newValue) {
      this.myValue = newValue
    },
    myValue(newValue) {
      this.$emit('input', newValue)
    }
  },
  mounted() {
    tinymce.init({})
  },
  methods: {
    onClick(e) {
      this.$emit('onClick', e, tinymce)
    }
  }
}
</script>