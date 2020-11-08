<template>
  <div>
    <script :id="id" type="text/plain" ></script>
  </div>
</template>

<script>

  export default {
    name: "UEditor",
    props: {
      id: {
        type: String
      },
      index: {
        type: Number
      },
      econtent:{
        type: String
      },
      modelname:{
        type: String
      },
      val:{
        type: Number
      },
      url:{
        type: String
      },
      contentUrl:{
        type: String
      }
    },
    data() {
      return {
        editor: null
      }
    },
    mounted() {
      //初始化UE
      const _this = this;

      this.editor = UE.delEditor(this.id);
      this.editor = UE.getEditor(this.id,{
        serverUrl: window.SITE_CONFIG['baseUrl']+"/ueditor/exec?token="+_this.$cookie.get('token'),
        zIndex:99999999
      });
      UE.Editor.prototype._bkGetActionUrl=UE.Editor.prototype.getActionUrl;
      UE.Editor.prototype.getActionUrl=function(action){
        if (action == 'uploadimage'){
          return 'jinding/file/upload';    /* 这里填上你自己的上传图片的action */
        }else{
          return this._bkGetActionUrl.call(this, action);
        }
      };
      this.editor.ready(function() {
        if(_this.val!=null&&_this.val!=undefined){
          // _this.$emit('ready', _this.editor);
          _this.$http({
            url: _this.$http.adornUrl(_this.contentUrl+_this.val),
            method: 'get',
            params: _this.$http.adornParams()
          }).then(({data}) => {
            if((data.code==200||data.code==10000)&&data.data!=null){
              _this.editor.setContent( data.data.content);
            }
          });
        }
        if(_this.econtent!=null&&_this.econtent!=undefined&&_this.econtent!=""){_this.editor.setContent( _this.econtent)}

      });
      this.editor.addListener("contentChange",function(){
        var html=_this.editor.getContent()
        _this.$emit('func',_this.modelname,_this.index,html)
      })
    },
    destoryed() {
      this.editor.destory();
    },
    methods:{
      getUEContent: function(){
        return this.editor.getContent();
      },
      getContentTxt: function(){
        return this.editor.getContentTxt();
      }
    }
  }
</script>
