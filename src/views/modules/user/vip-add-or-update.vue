<template>
  <el-dialog
    :title="look=='look'?'查看':!dataForm.id ? '新增' : '修改'"
    :close-on-click-modal="false"
    :visible.sync="visible">
    <el-form :model="dataForm" :rules="dataRule" ref="dataForm" @keyup.enter.native="dataFormSubmit()" label-width="80px">
      <div class="city-view-title">会员登录设置</div>
      <el-form-item label="ID"  v-show="dataForm.id">
        <el-input disabled v-model="dataForm.id" placeholder="ID"></el-input>
      </el-form-item>
      <el-form-item label="用户名">
        <el-input :disabled="look=='look'" v-model="dataForm.userName" placeholder="用户名"></el-input>
      </el-form-item>
      <el-form-item label="单位/个人真实名称">
        <el-input :disabled="look=='look'" v-model="dataForm.nameDes" placeholder="单位/个人真实名称"></el-input>
      </el-form-item>
      <el-form-item label="联系人">
        <el-input :disabled="look=='look'" v-model="dataForm.contact" placeholder="联系人"></el-input>
      </el-form-item>
      <el-form-item label="电话">
        <el-input :disabled="look=='look'" v-model="dataForm.phone" placeholder="电话"></el-input>
      </el-form-item>
      <el-form-item label="用户属性:">
        <el-select  :disabled="look=='look'" clearable  v-model="dataForm.userRole" placeholder="请选择">
          <el-option
            v-for="item in sxList"
            :key="item.value"
            :label="item.label"
            :value="item.value">
          </el-option>
        </el-select>
      </el-form-item>
      <el-form-item label="密码" prop="password">
        <el-input type="password" :disabled="look=='look'" v-model="dataForm.password" placeholder="密码"></el-input>
      </el-form-item>
      <el-form-item label="确认密码" prop="passwordConfirm">
        <el-input type="password" :disabled="look=='look'" v-model="dataForm.passwordConfirm" placeholder="确认密码"></el-input>
      </el-form-item>
      <el-form-item label="登陆状态" v-show="dataForm.id">
        <el-input  :disabled="look=='look'||dataForm.id!=0" v-model="dataForm.state==1?'正常':'冻结'" placeholder="登陆状态"></el-input>
      </el-form-item>
      <el-form-item label="会员展示状态"  v-show="dataForm.id">
        <el-input :disabled="look=='look'||dataForm.id!=0" v-model="dataForm.ifShow==1?'展示':'隐藏'" placeholder="会员展示状态"></el-input>
      </el-form-item>
      <el-form-item label="添加时间"  v-show="dataForm.id">
        <el-input :disabled="look=='look'||dataForm.id!=0" v-model="dataForm.insertTime" placeholder="添加时间"></el-input>
      </el-form-item>
      <div class="city-view-title">会员信息</div>
      <el-form-item label="会员logo">
        <div class="inline-block box-img" v-if="dataForm.logo&&dataForm.logo!=''">
          <el-image class="look-img" title="点击查看大图"
                    :src="dataForm.logo.indexOf('http')!=-1?dataForm.logo:logofront+dataForm.logo" :preview-src-list="srcList" >
          </el-image>
          <i class="el-icon-error box-img-del" @click="dataForm.logo=''"></i>
        </div>
        <div class="inline-block box-img"  v-if="dataForm.logo==''||!dataForm.logo">
          <el-upload :disabled="look=='look'"
                     :show-file-list="!dataForm.id&& dataForm.logo==''"
                     :headers="{'token':token}"
                     :action="this.$http.adornUrl('/jinding/file/upload')"
                     :on-success="handleChange"
                     :on-error="handleChange"
                     list-type="picture-card"
                     :on-remove="handleRemove">
            <i class="el-icon-plus"></i>
          </el-upload>
        </div>
      </el-form-item>
      <el-form-item label="会员简介">
        <UEditor  v-if="look!='look'"  class="editor inline-block" :contentUrl='"/biz/trendmaterial/info/"'  :id='"editor_tr_original"' :index="0" :econtent="dataForm.content"  :val="dataForm.id" :modelname="'tr_original'" @func="editorContent" ></UEditor>
        <div class="inline-block html-div" v-html="dataForm.content"  v-if="look=='look'"></div>
      </el-form-item>
    </el-form>
    <span slot="footer" class="dialog-footer">
      <el-button @click="visible = false">取消</el-button>
      <el-button v-show="look!='look'" type="primary" @click="dataFormSubmit()">确定</el-button>
    </span>
  </el-dialog>
</template>

<script>
  import UEditor from '@/components/ueditor/ueditor.vue'
  import { isEmail } from '@/utils/validate'
  export default {
    components: {
      UEditor
    },
    data () {
      var validatePassword = (rule, value, callback) => {
        if (!this.dataForm.id && !/\S/.test(value)) {
          callback(new Error('密码不能为空'))
        } else {
          callback()
        }
      }
      var validateComfirmPassword = (rule, value, callback) => {
        if (!this.dataForm.id && !/\S/.test(value)) {
          callback(new Error('确认密码不能为空'))
        } else if (this.dataForm.password !== value) {
          callback(new Error('确认密码与密码输入不一致'))
        } else {
          callback()
        }
      }
      return {
        look:'',
        visible: false,
        dialogImageUrl: '',
        dialogVisible: false,
        sxList:[
          {
            label:'管理员',
            value:'1'
          },
          {
            label:'会员',
            value:'2'
          }
        ],
        token:'',
        logofront:'',
        dataForm: {
          id: '',
          userName:'',
          nameDes: '',
          contact: '',
          phone: '',
          userRole: '',
          password: '',
          passwordConfirm: '',
          state: '',
          ifShow: '',
          insertTime: '',
          logo: '',
          content:''
        },
        value: '',
        dataRule: {
          password: [
            { validator: validatePassword, trigger: 'blur' }
          ],
          passwordConfirm: [
            { validator: validateComfirmPassword, trigger: 'blur' }
          ],
        }
      }
    },
    mounted(){
      this.dataForm={
        id: '',
        userName:'',
        nameDes: '',
        contact: '',
        phone: '',
        userRole: '',
        password: '',
        passwordConfirm: '',
        state: '',
        ifShow: '',
        insertTime: '',
        logo: '',
        content: ''
      }
    },
    methods: {
      //获取富文本内容
      editorContent(modelname,index,content){
        console.log(modelname)
        this.dataForm.content=content
      },
      init (id,look) {
        this.dataForm={
          id: '',
          userName:'',
          nameDes: '',
          contact: '',
          phone: '',
          userRole: '',
          password: '',
          passwordConfirm: '',
          state: '',
          ifShow: '',
          insertTime: '',
          logo: '',
          content: ''
          };
        this.dataForm.id = id||0;
        this.look=look||'';
        this.token=this.$cookie.get('token');
        this.logofront=this.$http.adornUrl('/jinding/showImg/');
        this.visible = true;
        this.$nextTick(() => {
          this.$refs['dataForm'].resetFields();
          if (this.dataForm.id) {
            this.$http({
              url: this.$http.adornUrl(`/biz/user/info/${this.dataForm.id}`),
              method: 'get',
              params: this.$http.adornParams()
            }).then(({data}) => {
              if (data && data.code === 10000) {
                var datas=data.data;
                this.dataForm.userName =datas.userName;
                this.dataForm.nameDes =datas.nameDes;
                this.dataForm.contact = datas.contact;
                this.dataForm.phone =datas.phone;
                this.dataForm.userRole = datas.userRole;
                this.dataForm.state = datas.state;
                this.dataForm.ifShow =datas.ifShow;
                this.dataForm.insertTime = datas.insertTime;
                this.dataForm.logo =datas.logo;
                this.dataForm.content = datas.content;
              }
            })
          }
        })
      },
      // 表单提交
      dataFormSubmit () {
        if(this.dataForm.password!=this.dataForm.passwordConfirm){
          this.$message.error('两次密码不一致')
        }else{
          this.$refs['dataForm'].validate((valid) => {
            if (valid) {
              this.$http({
                url: this.$http.adornUrl(`/biz/user/${!this.dataForm.id ? 'save' : 'update'}`),
                method: 'post',
                data: this.$http.adornData({
                  'id': this.dataForm.id || undefined,
                  'userName': this.dataForm.userName,
                  'nameDes': this.dataForm.nameDes,
                  'contact': this.dataForm.contact,
                  'phone': this.dataForm.phone,
                  'userRole': this.dataForm.userRole,
                  'password': this.dataForm.password,
                  'logo': this.dataForm.logo,
                  'content': this.dataForm.content,
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
        }
      },
      handleRemove(file, fileList) {
        this.dataForm.logo='';
      },
      //上传图片
      handleChange(response, file, fileList){
        if (response && response.code === 10000) {
          this.$message({
            message: '上传成功',
            type: 'success',
            duration: 1500,
            onClose: () => {
              this.dataForm.logo=response.data;
            }
          })
        } else {
          this.$message.error(response.msg)
        }
      },
      handlePictureCardPreview(file) {
        this.dialogImageUrl = file.url;
        this.dialogVisible = true;
      }
    }
  }
</script>
<style scoped>
  .look-img{
    width: 150px;
    height: 150px;
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
  .html-div{
    width: 90%;
    border: 1px solid #eee;
    padding:10px;
    box-sizing: border-box;
  }
  >>>.html-div img{
    width: 100%!important;
    height: auto;
  }
</style>

