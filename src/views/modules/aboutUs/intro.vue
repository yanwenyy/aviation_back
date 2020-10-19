<template>
    <div>
      <el-form>
        <div class="city-view-title">介绍内容</div>
        <el-form-item label="会员简介">
          <UEditor  class="editor" :contentUrl='"/biz/trpolicyoriginal/info/"'  :id='"editor_tr_original"' :index="0" :econtent="dataForm.content" :modelname="'tr_original'" @func="editorContent" ></UEditor>
        </el-form-item>
      </el-form>
      <div class="dialog-footer">
        <el-button type="primary" @click="dataFormSubmit()">确定</el-button>
      </div>
    </div>
</template>

<script>
  import UEditor from '@/components/ueditor/ueditor.vue'
    export default {
      components: {
        UEditor
      },
      data () {
        return {
          dataForm: {
            content: ''
          }
        }
      },
      methods:{
        //获取富文本内容
        editorContent(modelname,index,content){
          console.log(modelname)
          this.dataForm.content=content
        },
        //提交信息
        dataFormSubmit(){
          console.log(this.dataForm.content)
        }
      }
    }
</script>

<style scoped>
  .editor{
    width: 90%;
  }
  .dialog-footer{
    text-align: center;
  }
  .city-view-title{
    font-size: 16px;
    font-weight: bold;
    margin-bottom: 20px;
  }
</style>
