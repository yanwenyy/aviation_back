<template>
  <el-dialog
    :title="look=='look'?'查看':!dataForm.id ? '新增' : '修改'"
    :close-on-click-modal="false"
    :visible.sync="visible">
    <el-form :model="dataForm" :rules="dataRule" ref="dataForm" @keyup.enter.native="dataFormSubmit()" label-width="80px">
      <div class="city-view-group">
        <div class="city-view-title">主题信息</div>
        <div class="city-view-box">
          <el-form-item label="板块"  v-show="dataForm.id">
            <el-input disabled v-model="dataForm.name" placeholder="ID"></el-input>
          </el-form-item>
          <el-form-item label="子板块"  v-show="dataForm.id">
            <el-input disabled v-model="dataForm.childName" placeholder="ID"></el-input>
          </el-form-item>
          <el-form-item label="主题ID"  v-show="dataForm.id">
            <el-input disabled v-model="dataForm.id" placeholder="ID"></el-input>
          </el-form-item>
        </div>
      </div>
      <div class="city-view-group">
        <div class="city-view-title">回复信息</div>
        <div class="city-view-box">
          <el-form-item label="回复人"  v-show="dataForm.id">
            <el-input disabled v-model="dataForm.userName" placeholder="回复人"></el-input>
          </el-form-item>
          <el-form-item label="回复类型"  v-show="dataForm.id">
            <el-input disabled v-model="dataForm.level=='1'?'留言':'回复'" placeholder="回复类型"></el-input>
          </el-form-item>
          <el-form-item label="发布日期"  v-show="dataForm.id">
            <el-input disabled v-model="dataForm.careateDate" placeholder="发布日期"></el-input>
          </el-form-item>
          <el-form-item label="回复内容"  v-show="dataForm.id">
            <el-input class="el-input" disabled type="textarea" v-model="dataForm.content" placeholder="回复人"></el-input>
          </el-form-item>
          <el-form-item label="回复图片"  v-show="dataForm.id">
            <div class="inline-block box-img" v-if="imgList&&imgList!=''">
              <el-image v-for="(item,index ) in imgList"  :key="index" class="look-img" title="点击查看大图"
                        :src="item.indexOf('http')!=-1?item:imgUrlfront+item">
              </el-image>
            </div>
          </el-form-item>
          <el-form-item label="回复附件"  v-show="dataForm.id">
            <div v-for="item in fileList">{{item.name}} <el-button type="warning" @click="down(item.data,item.name)">下载附件</el-button></div>
          </el-form-item>
        </div>
      </div>
    </el-form>
    <span slot="footer" class="dialog-footer">
      <el-button @click="visible = false">取消</el-button>
      <el-button type="primary" @click="dataFormSubmit()">删除</el-button>
    </span>
  </el-dialog>
</template>

<script>
  import UEditor from '@/components/ueditor/ueditor.vue'
  import { isInteger } from '@/utils/validate'
  export default {
    components: {
      UEditor
    },
    data () {
      return {
        look:'',
        visible: false,
        imgUrlfront:'',
        dialogImageUrl: '',
        dialogVisible: false,
        fileList:[],
        dataForm: {
          id: '',
        },
        imgList:[],
        value: '',
        dataRule: {
        }
      }
    },
    created(){
      this.dataForm.type = "";
      this.dataForm.tagName = "";
    },
    methods: {
      //下载附件
      down (name,realName){
        var url='/jinding/download/'+name+'?fileName='+realName;
        window.open(this.$http.adornUrl(url));
      },
      //获取富文本内容
      editorContent(modelname,index,content){
        console.log(modelname)
        this.dataForm.content=content
      },
      init (id,look) {
        console.log(id)
        this.dataForm.id = id||0;
        this.look=look;
        this.imgUrlfront=this.$http.adornUrl('/jinding/showImg/');
        this.visible = true;
        this.$nextTick(() => {
          this.$refs['dataForm'].resetFields();
          if (this.dataForm.id) {
            this.$http({
              url: this.$http.adornUrl(`/biz/discuss/info/${this.dataForm.id}`),
              method: 'get',
              params: this.$http.adornParams()
            }).then(({data}) => {
              if (data && data.code === 10000) {
                var datas=data.data;
                this.dataForm =datas;
                this.imgList=datas.img.split(",")||[];
                console.log(this.imgList)
                var list=data.data.tbAnnexActions,i=0,len=list.length;
                this.fileList=[];
                if (len!= 0) {
                  for (;i<len;i++) {
                    this.fileList.push({
                      data: list[i].fileRealName,
                      name: list[i].fileOriginalName,
                      id: list[i].id
                    })
                  }
                }
              }
            })
          }else{
            this.dataForm.tagName = "";
            this.dataForm.type = "";
          }
        })
      },
      // 表单提交
      dataFormSubmit () {
        this.$refs['dataForm'].validate((valid) => {
          if (valid) {
            this.$http({
              url: this.$http.adornUrl(`/biz/tag/${!this.dataForm.id ? 'save' : 'update'}`),
              method: 'post',
              data: this.$http.adornData({
                'tagId': this.dataForm.id || undefined,
                'tagName': this.dataForm.tagName,
                'type': this.dataForm.type,
              })
            }).then(({data}) => {
              if (data && data.code == 10000) {
                this.$message({
                  message: '操作成功',
                  type: 'success',
                  duration: 1500,
                  onClose: () => {
                    this.visible = false
                    this.$emit('refreshDataList')
                  }
                })
              } else {
                this.$message.error(data.msg)
              }
            })
          }
        })
      },
      handleRemove(file, fileList) {
        this.dataForm.fj='';
      },
      //上传图片
      handleChange(response, file, fileList){
        if (response && response.code === 10000) {
          console.log(response  )
          this.$message({
            message: '上传成功',
            type: 'success',
            duration: 1500,
            onClose: () => {
              this.dataForm.fj=response.data;
            }
          })
        } else {
          this.$message.error(response.msg)
        }
      },
      handleRemove2(file, fileList) {
        this.dataForm.imgUrl='';
      },
      //上传图片
      handleChange2(response, file, fileList){
        if (response && response.code === 10000) {
          this.$message({
            message: '上传成功',
            type: 'success',
            duration: 1500,
            onClose: () => {
              this.dataForm.imgUrl=response.data;
            }
          })
        } else {
          this.$message.error(response.msg)
        }
      },
    }
  }
</script>
<style scoped>
  .hf-img{
    width: 50%;
    height: auto;
  }
  .downFj{
    font-size: 14px;
    color:green;
    margin-left: 20px;
  }
  >>> .el-form-item__label{
    width: 150px!important;
  }
  >>> .el-input{
    width: 90%;
  }
  >>> .el-dialog{
    width: 80%!important;
  }
  .input-msg{
    font-size: 12px;
    color:#999;
    margin-top: 0;
  }
  .city-view-title{
    font-size: 16px;
    font-weight: bold;
    margin-bottom: 20px;
  }
  >>> #edui1{
    width: 100%!important;
  }
  .editor{
    width: 90%!important;
  }
  .city-view-title{
    font-size: 16px;
    font-weight: bold;
    margin-bottom: 20px;
  }
  .city-view-box{
    border:1px solid #ccc;
    border-radius: 4px;
    padding:20px;
    margin-bottom:30px;
    position: relative
  }
</style>

