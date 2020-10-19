<template>
  <el-dialog
    :title="look=='look'?'查看':!dataForm.id ? '新增' : '修改'"
    :close-on-click-modal="false"
    :visible.sync="visible">
    <el-form :model="dataForm" :rules="dataRule" ref="dataForm" @keyup.enter.native="dataFormSubmit()" label-width="80px">
      <el-form-item label="标题">
        <el-input :disabled="look=='look'" v-model="dataForm.name" placeholder="标题"></el-input>
      </el-form-item>
      <el-form-item label="排序">
        <el-input :disabled="look=='look'" v-model="dataForm.sort" placeholder="排序"></el-input>
        <p class="input-msg">请输入数字，数字越大越靠前</p>
      </el-form-item>
      <el-form-item label="展示位置">
        <el-radio :disabled="look=='look'" v-model="dataForm.showPlace" label="1">首页</el-radio>
      </el-form-item>
      <el-form-item label="跳转地址">
        <el-input :disabled="look=='look'" v-model="dataForm.jumpUrl" placeholder="跳转地址"></el-input>
      </el-form-item>
      <el-form-item label="图片">
        <div class="inline-block box-img" v-if="dataForm.imgUrl&&dataForm.imgUrl!=''">
          <el-image class="look-img" title="点击查看大图"
                    :src="dataForm.imgUrl.indexOf('http')!=-1?dataForm.imgUrl:imgUrlfront+dataForm.imgUrl" :preview-src-list="srcList" >
          </el-image>
          <i class="el-icon-error box-img-del" @click="dataForm.imgUrl=''"></i>
        </div>
        <div class="inline-block box-img"  v-if="dataForm.imgUrl==''||!dataForm.imgUrl">
          <el-upload :disabled="look=='look'"
            :show-file-list="!dataForm.id&& dataForm.imgUrl==''"
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
    </el-form>
    <span slot="footer" class="dialog-footer">
      <el-button @click="visible = false">取消</el-button>
      <el-button v-show="look!='look'" type="primary" @click="dataFormSubmit()">确定</el-button>
    </span>
  </el-dialog>
</template>

<script>
  import { isInteger } from '@/utils/validate'
  export default {
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
        token:'',
        imgUrlfront:'',
        srcList: [],
        dataForm: {
          id: 0,
          name: '',
          sort: '',
          showPlace: '1',
          jumpUrl: '',
          imgUrl:''
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
      this.dataForm.name = "";
      this.dataForm.sort = "";
      this.dataForm.jumpUrl = "";
      this.dataForm.imgUrl = "";
      },
    methods: {
      init (id,look) {
        this.dataForm.id = id||0;
        this.look=look;
        this.token=this.$cookie.get('token');
        this.imgUrlfront=this.$http.adornUrl('/jinding/showImg/');
        this.visible = true;
        this.$nextTick(() => {
          this.$refs['dataForm'].resetFields();
          if (this.dataForm.id) {
            this.$http({
              url: this.$http.adornUrl(`/biz/banner/info/${this.dataForm.id}`),
              method: 'get',
              params: this.$http.adornParams()
            }).then(({data}) => {
              if (data && data.code === 10000) {
                var datas=data.data;
                this.dataForm.name =datas.name;
                this.dataForm.sort = datas.sort;
                this.dataForm.jumpUrl = datas.jumpUrl;
                this.dataForm.imgUrl = datas.imgUrl;
              }
            })
          }else{
            this.dataForm.name = "";
            this.dataForm.sort = "";
            this.dataForm.jumpUrl = "";
            this.dataForm.imgUrl = "";
          }
        })
      },
      // 表单提交
      dataFormSubmit () {
        this.$refs['dataForm'].validate((valid) => {
          if (valid) {
            this.$http({
              url: this.$http.adornUrl(`/biz/banner/${!this.dataForm.id ? 'save' : 'update'}`),
              method: 'post',
              data: this.$http.adornData({
                'id': this.dataForm.id || undefined,
                'name': this.dataForm.name,
                'sort': this.dataForm.sort,
                'showPlace': this.dataForm.showPlace,
                'jumpUrl': this.dataForm.jumpUrl,
                'imgUrl': this.dataForm.imgUrl
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
        this.dataForm.imgUrl='';
      },
      //上传图片
      handleChange(response, file, fileList){
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
      handlePictureCardPreview(file) {
        this.dialogImageUrl = file.url;
        this.dialogVisible = true;
      }
    }
  }
</script>
<style scoped>
  >>> .el-input{
    width: 90%;
  }
  .input-msg{
    font-size: 12px;
    color:#999;
    margin-top: 0;
  }
</style>

