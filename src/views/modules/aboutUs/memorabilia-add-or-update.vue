<template>
  <el-dialog
    :title="look=='look'?'查看':!dataForm.id ? '新增' : '修改'"
    :close-on-click-modal="false"
    :visible.sync="visible">
    <el-form :model="dataForm" :rules="dataRule" ref="dataForm" @keyup.enter.native="dataFormSubmit()" label-width="80px">
      <el-form-item label="ID"  v-show="dataForm.id">
        <el-input disabled v-model="dataForm.id" placeholder="ID"></el-input>
      </el-form-item>
      <el-form-item label="标题">
        <el-input :disabled="look=='look'" v-model="dataForm.title" placeholder="标题"></el-input>
      </el-form-item>
      <el-form-item label="时间">
        <el-date-picker
          v-model="dataForm.releaseDate"
          type="date"
          value-format="yyyy-MM-dd"
          placeholder="请选择时间">
        </el-date-picker>
      </el-form-item>
      <el-form-item label="添加时间"  v-show="dataForm.id">
        <el-date-picker
          v-model="dataForm.insertTime"
          type="date"
          value-format="yyyy-MM-dd"
          placeholder="请选择时间">
        </el-date-picker>
      </el-form-item>
      <el-form-item label="状态"  v-show="dataForm.id">
        <el-select clearable  v-model="dataForm.status" placeholder="请选择">
          <el-option
            v-for="item in ztList"
            :key="item.value"
            :label="item.label"
            :value="item.value">
          </el-option>
        </el-select>
      </el-form-item>
      <el-form-item label="内容">
        <UEditor :key="key"  v-if="look!='look'"  class="editor inline-block" :contentUrl='"/biz/bigevent/info/"'  :id='"editor_tr_original"' :index="0" :econtent="dataForm.content"  :val="dataForm.id" :modelname="'tr_original'" @func="editorContent" ></UEditor>
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
        key: 0,
        look:'',
        visible: false,
        dialogImageUrl: '',
        dialogVisible: false,
        ztList:[
          {
            label:'在线',
            value:'0'
          },
          {
            label:'隐藏',
            value:'1'
          }
        ],
        token:'',
        imgUrlfront:'',
        dataForm: {
          id: '',
          title: '',
          releaseDate: '',
          insertTime: '',
          status: '',
          content:'',
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
    activated (){

    },
    methods: {
      //获取富文本内容
      editorContent(modelname,index,content){
        this.dataForm.content=content
      },
      init (id,look) {
        this.dataForm={
          id:'',
          title: '',
          releaseDate: '',
          insertTime: '',
          status: '',
          content:'',
        }
        this.key=this.key+1;
        this.dataForm.id = id||0;
        this.look=look;
        this.token=this.$cookie.get('token');
        this.visible = true;
        this.$nextTick(() => {
          this.$refs['dataForm'].resetFields();
          if (this.dataForm.id) {
            this.$http({
              url: this.$http.adornUrl(`/biz/bigevent/info/${this.dataForm.id}`),
              method: 'get',
              params: this.$http.adornParams()
            }).then(({data}) => {
              if (data && data.code === 10000) {
                var datas=data.data;
                this.dataForm.title =datas.title;
                this.dataForm.releaseDate = datas.releaseDate;
                this.dataForm.insertTime =datas.insertTime;
                this.dataForm.status = datas.status;
                this.dataForm.content=datas.content;
              }
            })
          }
        })
      },
      // 表单提交
      dataFormSubmit () {
        this.$refs['dataForm'].validate((valid) => {
          if (valid) {
            this.$http({
              url: this.$http.adornUrl(`/biz/bigevent/${!this.dataForm.id ? 'save' : 'update'}`),
              method: 'post',
              data: this.$http.adornData({
                'id': this.dataForm.id || undefined,
                'title': this.dataForm.title,
                'releaseDate': this.dataForm.releaseDate+" 00:00:00",
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

