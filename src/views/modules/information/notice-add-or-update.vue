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
      <el-form-item label="标签:">
        <!--<el-select :disabled="look=='look'" clearable  v-model="dataForm.tagId" placeholder="请选择">-->
          <!--<el-option-->
            <!--v-for="item in bqList"-->
            <!--:key="item.tagId"-->
            <!--:label="item.tagName"-->
            <!--:value="item.tagId">-->
          <!--</el-option>-->
        <!--</el-select>-->
        <el-checkbox-group v-model="dataForm.tagEntities">
          <el-checkbox v-for="item in checkList" :disabled="look=='look'" :value="item.tagId" :label="item.tagId" :key="item.tagId">{{item.tagName}}</el-checkbox>
        </el-checkbox-group>
      </el-form-item>
      <el-form-item label="状态" v-show="dataForm.id">
        <el-input disabled v-model="dataForm.status==0?'在线':'隐藏'" placeholder="状态"></el-input>
      </el-form-item>
      <el-form-item label="添加日期"  v-show="dataForm.id">
        <el-input disabled v-model="dataForm.insertTime" placeholder="ID"></el-input>
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
        <div v-for="item in fileList"><span class="inline-block fj-name">{{item.name}} </span><el-button type="warning" @click="down(item.data,item.name)">下载附件</el-button></div>
      </el-form-item>
      <el-form-item label="内容">
        <UEditor :key="key"  v-if="look!='look'"  class="editor inline-block" :contentUrl='"/biz/notice/info/"'  :id='"editor_tr_original"' :index="0" :econtent="dataForm.content"  :val="dataForm.id" :modelname="'tr_original'" @func="editorContent" ></UEditor>
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
        fileList:[],
        bqList:[],
        checkList: [],
        token:'',
        dataForm: {
          id: '',
          title: '',
          tagId: '',
          status:'',
          insertTime: '',
          content: '',
          tagEntities:[],
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
        title: '',
        tagId: '',
        status:'',
        insertTime: '',
        content: '',
        tagEntities:[],
      };
      this.fileList=[];

    },
    methods: {
      //下载附件
      down (name,realName){
        var url='/jinding/download/'+name+'?fileName='+realName;
        window.open(this.$http.adornUrl(url));
      },
      //获取富文本内容
      editorContent(modelname,index,content){
        this.dataForm.content=content
      },
      init (id,look) {
        // 标签列表
        this.$http({
          url: this.$http.adornUrl('/biz/tag/select/list'),
          method: 'GET',
          params: this.$http.adornParams({
            'type': '3'
          })
        }).then(({data}) => {
          if (data && data.code === 10000) {
            this.bqList=data.data;
            this.checkList=data.data;
          }
        })
        this.dataForm={
          id: '',
          title: '',
          tagId: '',
          status:'',
          insertTime: '',
          content: '',
          tagEntities:[],
        };
        this.fileList=[];
        this.key=this.key+1;
        this.dataForm.id = id||0;
        this.look=look;
        this.token=this.$cookie.get('token');
        this.imgUrlfront=this.$http.adornUrl('/jinding/showImg/');
        this.visible = true;
        this.$nextTick(() => {
          this.$refs['dataForm'].resetFields();
          if (this.dataForm.id) {
            this.$http({
              url: this.$http.adornUrl(`/biz/notice/info/${this.dataForm.id}`),
              method: 'get',
              params: this.$http.adornParams()
            }).then(({data}) => {
              if (data && data.code === 10000) {
                var datas=data.data;
                this.dataForm.title =datas.title;
                this.dataForm.content =datas.content;
                this.dataForm.tagId =String(datas.tagId);
                for(var j=0;j<datas.tagEntities.length;j++){
                  this.dataForm.tagEntities.push(datas.tagEntities[j].tagId)
                }
                // if(datas.tagEntities){
                //   for(var j=0;j<datas.tagEntities.length;j++){
                //     this.dataForm.tagEntities.push(datas.tagEntities[j].tagId)
                //   }
                // }
                this.dataForm.status = datas.status;
                this.dataForm.createDate =datas.insertTime;
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
              url: this.$http.adornUrl(`/biz/notice/${!this.dataForm.id ? 'save' : 'update'}`),
              method: 'post',
              data: this.$http.adornData({
                'id': this.dataForm.id || undefined,
                'title': this.dataForm.title,
                // 'tagId': this.dataForm.tagId,
                'tagId': this.dataForm.tagEntities.join(","),
                'tbAnnexActions': tbAnnexActions,
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
  .fj-name{
    width: 60%;
    word-break: break-all;
    margin-bottom: 20px;
  }
  >>> .el-form-item__label{
    width: 150px!important;
  }
  >>> .el-form-item__content{
    margin-left: 150px!important;
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

