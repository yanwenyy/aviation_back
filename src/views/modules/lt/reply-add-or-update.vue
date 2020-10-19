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
            <el-input disabled v-model="dataForm.tagName" placeholder="ID"></el-input>
          </el-form-item>
          <el-form-item label="子板块"  v-show="dataForm.id">
            <el-input disabled v-model="dataForm.tagName" placeholder="ID"></el-input>
          </el-form-item>
          <el-form-item label="主题ID"  v-show="dataForm.id">
            <el-input disabled v-model="dataForm.tagName" placeholder="ID"></el-input>
          </el-form-item>
        </div>
      </div>
      <div class="city-view-group">
        <div class="city-view-title">回复信息</div>
        <div class="city-view-box">
          <el-form-item label="回复人"  v-show="dataForm.id">
            <el-input disabled v-model="dataForm.tagName" placeholder="回复人"></el-input>
          </el-form-item>
          <el-form-item label="回复类型"  v-show="dataForm.id">
            <el-input disabled v-model="dataForm.tagName" placeholder="回复人"></el-input>
          </el-form-item>
          <el-form-item label="发布日期"  v-show="dataForm.id">
            <el-input disabled v-model="dataForm.tagName" placeholder="回复人"></el-input>
          </el-form-item>
          <el-form-item label="回复内容"  v-show="dataForm.id">
            <el-input class="el-input" disabled type="textarea" v-model="dataForm.tagName" placeholder="回复人"></el-input>
          </el-form-item>
          <el-form-item label="回复图片"  v-show="dataForm.id">
            <img class="hf-img" src="" alt="">
          </el-form-item>
          <el-form-item label="回复附件"  v-show="dataForm.id">
            <div class="inline-block">无人机适航审定首次审查会顺利召开</div> <span class="downFj inline-block">下载</span>
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
      var validateInteger = (rule, value, callback) => {
        if(value===''){
          callback(new Error('不能为空'))
        }else if (!isInteger(value)) {
          callback(new Error('格式不正确'))
        } else {
          callback()
        }
      };
      var validateMoney = (rule, value, callback) => {
        if(!value){
          callback(new Error('不能为空'))
        }else if (!/(^[1-9]([0-9]+)?(\.[0-9]{1,2})?$)|(^(0){1}$)|(^[0-9]\.[0-9]([0-9])?$)/.test(value)) {
          callback(new Error('金额格式不正确'))
        } else {
          callback()
        }
      };
      return {
        look:'',
        visible: false,
        dialogImageUrl: '',
        dialogVisible: false,
        fileList:[],
        checkList: ['选中且禁用','复选框 A'],
        typeList:[
          {
            label:'行业动态',
            value:'1'
          },
          {
            label:'资料中心',
            value:'2'
          },
          {
            label:'通知公告',
            value:'3'
          }
        ],
        token:'',
        imgUrlfront:'',
        dataForm: {
          id: 0,
          tagName: '',
          type: '',
          fj:'',
        },
        value: '',
        dataRule: {
          dataTime: [
            { required: true, message: '数据时间不能为空', trigger: 'blur' }
          ],
          dataAmount: [
            { required: true,validator: validateInteger, trigger: 'blur' }
          ],
          effectiveData: [
            { required: true, validator: validateInteger,trigger: 'blur' }
          ],
          todayConsumeMoney: [
            { required: true, validator: validateMoney, trigger: 'blur' }
          ],
        }
      }
    },
    created(){
      this.dataForm.type = "";
      this.dataForm.tagName = "";
    },
    methods: {
      //获取富文本内容
      editorContent(modelname,index,content){
        console.log(modelname)
        this.dataForm.content=content
      },
      init (id,look) {
        console.log(id)
        this.dataForm.id = id||0;
        this.look=look;
        this.token=this.$cookie.get('token');
        this.imgUrlfront=this.$http.adornUrl('/jinding/showImg/');
        this.visible = true;
        this.$nextTick(() => {
          this.$refs['dataForm'].resetFields();
          if (this.dataForm.id) {
            this.$http({
              url: this.$http.adornUrl(`/biz/tag/info/${this.dataForm.id}`),
              method: 'get',
              params: this.$http.adornParams()
            }).then(({data}) => {
              if (data && data.code === 10000) {
                var datas=data.data;
                this.dataForm.tagName =datas.tagName;
                this.dataForm.type = datas.type;
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

