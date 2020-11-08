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
      <el-form-item label="行业动态标签">
        <el-checkbox-group v-model="dataForm.tagEntities">
          <el-checkbox v-for="item in checkList" :disabled="look=='look'" :value="item.tagId" :label="item.tagId" :key="item.tagId">{{item.tagName}}</el-checkbox>
        </el-checkbox-group>
      </el-form-item>
      <el-form-item label="封面图">
        <div class="inline-block box-img" v-if="dataForm.coverImg&&dataForm.coverImg!=''">
          <el-image class="look-img" title="点击查看大图"
                    :src="dataForm.coverImg.indexOf('http')!=-1?dataForm.coverImg:coverImgfront+dataForm.coverImg" >
          </el-image>
          <i v-show="look!='look'" class="el-icon-error box-img-del" @click="dataForm.coverImg='',dataForm.ifRecom=''"></i>
        </div>
        <div class="inline-block box-img"  v-if="dataForm.coverImg==''||!dataForm.coverImg">
          <el-upload :disabled="look=='look'"
                     :show-file-list="!dataForm.id&& dataForm.coverImg==''"
                     :headers="{'token':token}"
                     :action="this.$http.adornUrl('/jinding/file/upload')"
                     :on-success="handleChange2"
                     :on-error="handleChange2"
                     list-type="picture-card"
                     :on-remove="handleRemove2">
            <i class="el-icon-plus"></i>
          </el-upload>
        </div>
      </el-form-item>
      <el-form-item label="首页图片位推荐">
        <el-radio  :disabled="look=='look'||dataForm.coverImg==''" v-model="dataForm.ifRecom" label="1">推荐</el-radio>
        <el-radio  :disabled="look=='look'" v-model="dataForm.ifRecom" label="0">不推荐</el-radio>
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
      <el-form-item label="内容">
        <UEditor :key="key"  v-if="look!='look'"  class="editor inline-block" :contentUrl='"/biz/trendmaterial/info/"'  :id='"editor_tr_original"' :index="0" :econtent="dataForm.content"  :val="dataForm.id" :modelname="'tr_original'" @func="editorContent" ></UEditor>
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
  import { isInteger} from '@/utils/validate'
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
        checkList: [],
        token:'',
        coverImgfront:'',
        dataForm: {
          id: '',
          title: '',
          source: '',
          coverImg: '',
          ifRecom: '',
          status: '',
          createDate: '',
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
    mounted (){
      this.dataForm={
        id: '',
        title: '',
        source: '',
        tagEntities:[],
        coverImg: '',
        ifRecom: '',
        status: '',
        createDate: '',
        content: '',
      };
      this.fileList=[];
      // 标签列表
      this.$http({
        url: this.$http.adornUrl('/biz/tag/select/list'),
        method: 'GET',
        params: this.$http.adornParams({
          'type': '1'
        })
      }).then(({data}) => {
        if (data && data.code === 10000) {
          this.checkList=data.data;
        }
      })
    },
    methods: {
      //下载附件
      down (name,realName){
        var url='/jinding/download/'+name;
        window.open(this.$http.adornUrl(url));
        // let elink = document.createElement("a");
        // elink.download = decodeURI(realName);  ;
        // elink.style.display = "none";
        // elink.href = this.$http.adornUrl(url);
        // document.body.appendChild(elink);
        // elink.click();
        // console.log(elink)
        // URL.revokeObjectURL(elink.href); // 释放URL 对象
        // document.body.removeChild(elink);
      },
      //获取富文本内容
      editorContent(modelname,index,content){
        this.dataForm.content=content
      },
      init (id,look) {
        this.key=this.key+1;
        this.dataForm={
          id: '',
          title: '',
          source: '',
          tagEntities:[],
          coverImg: '',
          ifRecom: '',
          status: '',
          createDate: '',
          content: '',
        };
        this.fileList=[];
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
                for(var j=0;j<datas.tagEntities.length;j++){
                  this.dataForm.tagEntities.push(datas.tagEntities[j].tagId)
                }
                this.dataForm.coverImg = datas.coverImg;
                this.dataForm.ifRecom =String(datas.ifRecom);
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
              url: this.$http.adornUrl(`/biz/trendmaterial/${!this.dataForm.id ? 'save' : 'update'}`),
              method: 'post',
              data: this.$http.adornData({
                'type':0,
                'id': this.dataForm.id || undefined,
                'title': this.dataForm.title,
                'source': this.dataForm.source,
                'tagId': this.dataForm.tagEntities.join(","),
                'coverImg': this.dataForm.coverImg,
                'ifRecom': this.dataForm.coverImg!=''?this.dataForm.ifRecom:'0',
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
      handleRemove2(file, fileList) {
        this.dataForm.coverImg='';
        this.dataForm.ifRecom='0';
      },
      //上传图片
      handleChange2(response, file, fileList){
        if (response && response.code === 10000) {
          this.$message({
            message: '上传成功',
            type: 'success',
            duration: 1500,
            onClose: () => {
              this.dataForm.coverImg=response.data;
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

