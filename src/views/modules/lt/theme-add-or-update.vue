<template>
  <el-dialog
    :title="look=='look'?'查看':!dataForm.id ? '新增' : '修改'"
    :close-on-click-modal="false"
    :visible.sync="visible">
    <el-form :model="dataForm" :rules="dataRule" ref="dataForm" @keyup.enter.native="dataFormSubmit()" label-width="80px">
      <el-form-item label="选择板块">
        <el-select  @change="getChild()"  :disabled="look=='look'" clearable  v-model="dataForm.jobModelId" placeholder="请选择">
          <el-option
            v-for="item in typeList"
            :key="item.id"
            :label="item.name"
            :value="item.id">
          </el-option>
        </el-select>
      </el-form-item>
      <el-form-item label="选择子板块">
        <el-select  :disabled="look=='look'" clearable  v-model="dataForm.childModelId" placeholder="请选择">
          <el-option
            v-for="item in typeTwoList"
            :key="item.id"
            :label="item.childName"
            :value="item.id">
          </el-option>
        </el-select>
      </el-form-item>
      <el-form-item label="状态"  v-show="dataForm.id">
        <el-input disabled v-model="dataForm.status" placeholder="ID"></el-input>
      </el-form-item>
      <el-form-item label="添加日期"  v-show="dataForm.id">
        <el-input disabled v-model="dataForm.createDate" placeholder="ID"></el-input>
      </el-form-item>
      <el-form-item label="ID"  v-show="dataForm.id">
        <el-input disabled v-model="dataForm.id" placeholder="ID"></el-input>
      </el-form-item>
      <el-form-item label="标题">
        <el-input :disabled="look=='look'" v-model="dataForm.title" placeholder="标题"></el-input>
      </el-form-item>
      <el-form-item label="内容">
        <UEditor  v-if="look!='look'"  class="editor inline-block" :contentUrl='"/biz/theme/info/"'  :id='"editor_tr_original"' :index="0" :econtent="dataForm.content"  :val="dataForm.id" :modelname="'tr_original'" @func="editorContent" ></UEditor>
        <div class="inline-block html-div" v-html="dataForm.content"  v-if="look=='look'"></div>
      </el-form-item>
      <el-form-item label="附件"  v-if="look!='look'">
        <el-upload
          class="inline-block"
          :headers="{'token':token}"
          :action="this.$http.adornUrl('/jinding/file/upload')"
          :on-success="handleChange"
          :on-error="handleChange"
          :on-remove="handleRemove"
          :file-list="fileList"
        >
          <el-button type="warning">选择文件</el-button>
        </el-upload>
      </el-form-item>
      <el-form-item label="附件"  v-if="look=='look'">
        <div v-for="item in fileList">{{item.name}} <el-button type="warning" @click="down(item.id)">下载附件</el-button></div>
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
        look:'',
        visible: false,
        dialogImageUrl: '',
        dialogVisible: false,
        fileList:[],
        checkList: ['选中且禁用','复选框 A'],
        typeList:[],
        typeTwoList:[],
        token:'',
        imgUrlfront:'',
        dataForm: {
          id: '',
          jobModelId: '',
          childModelId: '',
          status:'',
          createDate: '',
          title: '',
          content:''
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
      this.dataForm={
        id: '',
        jobModelId: '',
        childModelId: '',
        status:'',
        createDate: '',
        title: '',
        content:''
      }
      //板块下拉列表
      this.$http({
        url: this.$http.adornUrl('/biz/jobmodel/select/list'),
        method: 'GET',
      }).then(({data}) => {
        this.typeList = data.data
      });
    },
    methods: {
      //获取子版块下拉列表
      getChild(){
        if(this.dataForm.jobModelId!=''){
          //子板块下拉列表
          this.$http({
            url: this.$http.adornUrl('/biz/jobchildmodel/select/list'),
            method: 'GET',
            params: this.$http.adornParams({
              'id': this.dataForm.jobModelId,
            })
          }).then(({data}) => {
            this.typeTwoList = data.data
          });
        }
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
        this.visible = true;
        this.$nextTick(() => {
          this.$refs['dataForm'].resetFields();
          if (this.dataForm.id) {
            this.$http({
              url: this.$http.adornUrl(`/biz/theme/info/${this.dataForm.id}`),
              method: 'get',
              params: this.$http.adornParams()
            }).then(({data}) => {
              if (data && data.code === 10000) {
                var datas=data.data;
                this.dataForm.jobModelId =datas.jobModelId;
                this.dataForm.childModelId = datas.childModelId;
                this.dataForm.status =datas.status;
                this.dataForm.createDate = datas.createDate;
                this.dataForm.title =datas.title;
                this.dataForm.content = datas.content;
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
          }
        })
      },
      // 表单提交
      dataFormSubmit () {
        var tbAnnexActions=[],i=0,list=this.fileList,len=list.length;
        for(;i<len;i++){
          if(list[i].response){
            tbAnnexActions.push({
              fileOriginalName:list[i].name,
              fileRealName:list[i].response.data
            })
          }else{
            tbAnnexActions.push({
              fileOriginalName:list[i].name,
              fileRealName:list[i].data
            })
          }
        }
        this.$refs['dataForm'].validate((valid) => {
          if (valid) {
            this.$http({
              url: this.$http.adornUrl(`/biz/theme/${!this.dataForm.id ? 'save' : 'update'}`),
              method: 'post',
              data: this.$http.adornData({
                'id': this.dataForm.id || undefined,
                'jobModelId': this.dataForm.jobModelId,
                'childModelId': this.dataForm.childModelId,
                'title': this.dataForm.title,
                'content': this.dataForm.content,
                'tbAnnexActions': tbAnnexActions,
              })
            }).then(({data}) => {
              if (data && data.code == 10000) {
                this.$message({
                  message: '操作成功',
                  type: 'success',
                  duration: 1500,
                  onClose: () => {
                    this.visible = false
                    this.$emit('refreshDataList');
                    this.dataForm={
                      id: '',
                      jobModelId: '',
                      childModelId: '',
                      status:'',
                      createDate: '',
                      title: '',
                      content:''
                    }
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
      handleRemove(file, fileList) {
        this.fileList=fileList;
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
              this.fileList=fileList;
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

