<template>
  <el-dialog
    :title="look=='look'?'查看':!dataForm.id ? '新增' : '修改'"
    :close-on-click-modal="false"
    :visible.sync="visible">
    <el-form :model="dataForm" :rules="dataRule" ref="dataForm" @keyup.enter.native="dataFormSubmit()" label-width="80px">
      <el-form-item label="申请类型">
        <el-input :disabled="look=='look'" v-model="dataForm.type==1?'个人':'企业'" placeholder="申请类型"></el-input>
      </el-form-item>
      <el-form-item label="企业名称/个人姓名">
        <el-input :disabled="look=='look'" v-model="dataForm.nameDes" placeholder="企业名称/个人姓名"></el-input>
      </el-form-item>
      <el-form-item label="专业领域">
        <el-input :disabled="look=='look'" v-model="dataForm.professional" placeholder="专业领域"></el-input>
      </el-form-item>
      <el-form-item label="申请人姓名">
        <el-input :disabled="look=='look'" v-model="dataForm.applyName" placeholder="申请人姓名"></el-input>
      </el-form-item>
      <el-form-item label="联系电话">
        <el-input :disabled="look=='look'" v-model="dataForm.phone" placeholder="联系电话"></el-input>
      </el-form-item>
      <el-form-item label="联系邮箱">
        <el-input :disabled="look=='look'" v-model="dataForm.email" placeholder="联系邮箱"></el-input>
      </el-form-item>
      <el-form-item label="申请时间">
        <el-input :disabled="look=='look'" v-model="dataForm.insertTime" placeholder="申请时间"></el-input>
      </el-form-item>
      <el-form-item label="基本情况介绍">
        <el-input class="el-input" type="textarea" v-show="look=='look'" :disabled="look=='look'" v-model="dataForm.introduction" placeholder="会员简介"></el-input>
      </el-form-item>
      <el-form-item label="入会后简要工作设想">
        <el-input class="el-input" type="textarea" v-show="look=='look'" :disabled="look=='look'" v-model="dataForm.workIdea" placeholder="会员简介"></el-input>
      </el-form-item>
    </el-form>
    <span slot="footer" class="dialog-footer">
      <el-button @click="visible = false">关闭</el-button>
    </span>
  </el-dialog>
</template>

<script>
  export default {
    data () {
      return {
        look:'',
        visible: false,
        dialogImageUrl: '',
        dialogVisible: false,
        dataForm: {
          id: '',
          nameDes: '',
          professional: '',
          applyName: '',
          phone: '',
          email: '',
          insertTime: '',
          introduction: '',
          workIdea: ''
        },
        value: '',
        dataRule: {
        }
      }
    },
    activated(){
      this.dataForm={
        id: '',
        nameDes: '',
        professional: '',
        applyName: '',
        phone: '',
        email: '',
        insertTime: '',
        introduction: '',
        workIdea: ''
      }
    },
    methods: {
      init (id,look) {
        console.log(id)
        this.dataForm.id = id||0;
        this.look=look;
        this.visible = true;
        this.$nextTick(() => {
          this.$refs['dataForm'].resetFields();
          if (this.dataForm.id) {
            this.$http({
              url: this.$http.adornUrl(`/biz/application/info/${this.dataForm.id}`),
              method: 'get',
              params: this.$http.adornParams()
            }).then(({data}) => {
              if (data && data.code === 10000) {
                var datas=data.data;
                this.dataForm.nameDes =datas.nameDes;
                this.dataForm.professional = datas.professional;
                this.dataForm.applyName =datas.applyName;
                this.dataForm.phone =datas.phone;
                this.dataForm.email = datas.email;
                this.dataForm.insertTime =datas.insertTime;
                this.dataForm.introduction = datas.introduction;
                this.dataForm.workIdea = datas.workIdea;
              }
            })
          }
        })
      },
    }
  }
</script>
<style scoped>
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
</style>

