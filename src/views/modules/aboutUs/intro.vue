<template>
    <div>
      <el-form>
        <div class="city-view-title">介绍内容</div>
        <el-form-item label="会员简介">
          <UEditor  class="editor" :val="0" :contentUrl='"/biz/introduction/info/id?"'   :id='"editor_tr_original"' :index="0" :econtent="dataForm.content" :modelname="'tr_original'" @func="editorContent" ></UEditor>
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
      inject:['removeTabHandle'],
      components: {
        UEditor
      },
      data () {
        return {
          dataForm: {
            content:''
          }
        }
      },
      activated(){
        // 信息
        this.$http({
          url: this.$http.adornUrl('/biz/introduction/info/id'),
          method: 'GET',
        }).then(({data}) => {
          if (data && data.code === 10000) {
            this.dataForm.content=data.data.content;
          }
        })
      },
      methods:{
        //获取富文本内容
        editorContent(modelname,index,content){
          // console.log(content)
          this.dataForm.content=content
        },
        //提交信息
        dataFormSubmit(){
          this.$http({
            url: this.$http.adornUrl(`/biz/introduction/update`),
            method: 'post',
            data: this.$http.adornData(this.dataForm)
          }).then(({data}) => {
            if (data && data.code == 10000) {
              this.$message({
                message: '操作成功',
                type: 'success',
                duration: 1500,
                onClose: () => {
                  this.removeTabHandle(this.$store.state.common.mainTabsActiveName)
                }
              })
            } else {
              this.$message.error(data.msg)
            }
          })
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
