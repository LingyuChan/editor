<template>
  <div class="editor">
    VueEditor
    <quill-editor v-model="content"
                ref="myQuillEditor"
                :options="editorOption">
    </quill-editor>
  </div>
</template>

<script>
import {quillEditor, Quill} from 'vue-quill-editor'
import 'quill/dist/quill.core.css'
import 'quill/dist/quill.snow.css'
import 'quill/dist/quill.bubble.css'
// 图片点击按钮上传
import {ImageExtend, QuillWatch} from 'quill-image-extend-module'  // QuillWatch
import axios from 'axios'
Quill.register('modules/ImageExtend', ImageExtend)

export default {
  name: 'VueEditor',
  data(){
    return {
      content:'<h1>dgfdgdfgdfs</h1>', // ***富文本内容
      editorOption:{
        modules: {
          toolbar: {
            container:[
              ["bold", "italic", "underline", "strike"], // 加粗 斜体 下划线 删除线
              ["blockquote", "code-block"], // 引用  代码块
              [{ header: 1 }, { header: 2 }], // 1、2 级标题
              [{ list: "ordered" }, { list: "bullet" }], // 有序、无序列表
              [{ script: "sub" }, { script: "super" }], // 上标/下标
              [{ indent: "-1" }, { indent: "+1" }], // 缩进
              // [{'direction': 'rtl'}],                         // 文本方向
              [{ size: ["small", false, "large", "huge"] }], // 字体大小
              [{ header: [1, 2, 3, 4, 5, 6, false] }], // 标题
              [{ color: [] }, { background: [] }], // 字体颜色、字体背景颜色
              [{ font: [] }], // 字体种类
              [{ align: [] }], // 对齐方式
              ["clean"], // 清除文本格式
              ["link", "image", "video"] // 链接、图片、视频
            ],

            handlers: {
              // 图片处理绑定到ImageExtend
              'image': function () {
                QuillWatch.emit(this.quill.id)
              }
            }
          }, //工具菜单栏配置
          ImageExtend: {
            loading: true,
            name: 'XXX1', // 接口action
            action: "XXX2", // 接口url
            response: function(res){
              return res.url; // 返回上传后的文件链接
            }
          },
          // 粘贴事件拦截
          clipboard: {
            matchers: [
              ['img', this.handlePasteImg]
            ],
          }
        },
      }
    }
  },
  props: {
    msg: String
  },
  components:{
    quillEditor
  },
  created() {
    console.log('this is current quill instance object', this.editor)
  },
  methods: {
    // 处理复制img到富文本框
    handlePasteImg: function(node, delta){
      // 调整为上传，给回路径
      // POSt url
      // form Data： upfile: (binary) 
      //  return res.url
      let _self = this;
      let base64code = delta && delta.ops && delta.ops[0] && delta.ops[0]['insert'] && delta.ops[0]['insert']['image'];
      if(base64code && base64code.indexOf("http://")!==0){
        let file = _self.dataURLtoFile(base64code, 'paste.png');
        var formData = new FormData();
        formData.append("XXX1", file);
        _self.gUpload(formData, function(res) {
          // 2、处理插入
          
          // 获取富文本对象
          let quill = _self.$refs['myQuillEditor'].quill
          // 获取编辑器光标位置
          let length = quill.selection.savedRange.index;
          // 插入图片至光标位置，imgUrl为图片地址
          quill.insertEmbed(length, 'image', res.url);
          // 移动光标至图片后面
          quill.setSelection(length + 1);
        });
        // 1、等待请求返回做不到同步，先返回空，在请求回调中再添加
        delta.ops = [];
        return delta;
      }else{
        return delta;
      }
    },

    // 图片的base64编码转为图片文件
    dataURLtoFile: function(dataurl, name) {
      var arr = dataurl.split(','), mime = arr[0].match(/:(.*?);/)[1],
        bstr = atob(arr[1]), n = bstr.length, u8arr = new Uint8Array(n);
      while(n--){
        u8arr[n] = bstr.charCodeAt(n);
      }
      return new File([u8arr], name, {type:mime});
    },

    // 网络处理：上传操作
    gUpload: function(params, callBack) {
      let url = "XXX2"; // 接口url
      axios.post(url, params, {
        withCredentials: false,
      }).then(function(response) {
        let data = response.data
        if(data.state!=="SUCCESS"){
          console.log("error: ", data.state);
        }else{
          callBack(data);
        }
      })
      .catch(function() {
        console.log("截图上传失败");
      });
    },
  },
}
</script>

<style scoped>
h3 {
  margin: 40px 0 0;
}
ul {
  list-style-type: none;
  padding: 0;
}
li {
  display: inline-block;
  margin: 0 10px;
}
a {
  color: #42b983;
}

.quill-editor{
  width: 60%;
  height: 480px;
  margin: 0 auto;
}
</style>
