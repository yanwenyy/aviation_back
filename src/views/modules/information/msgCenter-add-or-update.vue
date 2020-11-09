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
      <el-form-item label="来源">
        <el-input :disabled="look=='look'" v-model="dataForm.source" placeholder="来源"></el-input>
      </el-form-item>
      <el-form-item label="资料中心标签">
        <el-checkbox-group v-model="dataForm.tagEntities">
          <el-checkbox v-for="item in checkList" :disabled="look=='look'" :value="item.tagId" :label="item.tagId" :key="item.tagId">{{item.tagName}}</el-checkbox>
        </el-checkbox-group>
      </el-form-item>
      <el-form-item label="关联分类">
        <div  class="inline-block">
          <div v-for="item in relationCheck">
            <span class="inline-block">{{item.name}}:</span>
            <div class="inline-block">
              <el-checkbox-group v-model="dataForm.levelTwoClass">
                <el-checkbox v-for="i in item.childClassLevel	" :disabled="look=='look'" :value="i.id" :label="i.id" :key="i.id">{{i.name}}</el-checkbox>
              </el-checkbox-group>
            </div>
          </div>
        </div>
      </el-form-item>
      <el-form-item label="状态"  v-show="dataForm.id">
        <el-input disabled v-model="dataForm.status==0?'在线':'隐藏'" placeholder="状态"></el-input>
      </el-form-item>
      <el-form-item label="添加日期"  v-show="dataForm.id">
        <el-input disabled v-model="dataForm.createDate" placeholder="添加日期"></el-input>
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
      <el-form-item label="引文">
        <el-input type="textarea" maxlength="300" show-word-limit :disabled="look=='look'" v-model="dataForm.preface" placeholder="引文"></el-input>
      </el-form-item>
      <el-form-item label="内容">
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
        checkList: [],
        relationList:[],
        relationCheck:[],
        token:'',
        coverImgfront:'',
        dataForm: {
          id: '',
          title: '',
          source: '',
          status: '',
          createDate: '',
          content: '',
          tagEntities:[],
          levelTwoClass:[],
          preface:''
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
    mounted (){
      this.dataForm={
        id: '',
        title: '',
        source: '',
        tagEntities:[],
        status: '',
        createDate: '',
        content: '',
        levelTwoClass:[],
        preface:''
      };
      this.fileList=[];
      // 标签列表
      this.$http({
        url: this.$http.adornUrl('/biz/tag/select/list'),
        method: 'GET',
        params: this.$http.adornParams({
          'type': '2'
        })
      }).then(({data}) => {
        if (data && data.code === 10000) {
          this.checkList=data.data;
        }
      })
      //关联列表
      this.$http({
        url: this.$http.adornUrl('/aviation/select/list'),
        method: 'GET',
      }).then(({data}) => {
        if (data && data.code === 10000) {
          this.relationCheck=data.data;
        }
      })
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
        this.dataForm.id = id||0;
        this.look=look;
        this.token=this.$cookie.get('token');
        this.coverImgfront=this.$http.adornUrl('/jinding/showImg/');
        this.visible = true;
        this.$nextTick(() => {
          this.$refs['dataForm'].resetFields();
          if (this.dataForm.id) {
            this.$http({
              url: this.$http.adornUrl(`/biz/trendmaterial/info/${this.dataForm.id}`),
              method: 'get',
              params: this.$http.adornParams()
            }).then(({data}) => {
              if (data && data.code === 10000) {
                var datas=data.data;
                this.dataForm.title =datas.title;
                this.dataForm.source = datas.source;
                this.dataForm.content =datas.content;
                this.dataForm.tagEntities=[];
                for(var j=0;j<datas.tagEntities.length;j++){
                  this.dataForm.tagEntities.push(datas.tagEntities[j].tagId)
                }
                this.dataForm.levelTwoClass=datas.levelTwoClass&&datas.levelTwoClass.split(",")||[];
                this.dataForm.status = datas.status;
                this.dataForm.preface = datas.preface;
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
        // console.log(this.dataForm.tagEntities)
        this.$refs['dataForm'].validate((valid) => {
          if (valid) {
            this.$http({
              url: this.$http.adornUrl(`/biz/trendmaterial/${!this.dataForm.id ? 'save' : 'update'}`),
              method: 'post',
              data: this.$http.adornData({
                'type':1,
                'id': this.dataForm.id || undefined,
                'title': this.dataForm.title,
                'source': this.dataForm.source,
                'tagId': this.dataForm.tagEntities.join(","),
                'levelTwoClass': this.dataForm.levelTwoClass.join(','),
                'tbAnnexActions': tbAnnexActions,
                'content': this.dataForm.content,
                'preface': this.dataForm.preface
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
  .look-img{
    width: 200px;
    height: 200px;
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

