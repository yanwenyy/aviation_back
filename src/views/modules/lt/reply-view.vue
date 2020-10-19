<template>
  <div class="mod-policy">
    <h2 style="border-bottom: 1px solid #ccc;padding-bottom: 20px;margin-bottom: 50px">查看回复</h2>
    <el-form label-position="left" label-width="120px" :model="dataForm" :rules="dataRule" ref="dataForm">
      <div class="city-view-group">
        <div class="city-view-title">全部回复(3)</div>
        <div class="city-view-box">
         <div  class="reply-list">
           <div class="reply-name">刘德华 <span class="lz">楼主</span></div>
           <div class="reply-date">8小时前</div>
           <div class="reply-reply-msg box-sizing"><span class="reply-name">@刘德华:</span>最早发布的内容刘德华最早发布的内容刘德华最早发布的内容刘德华最早发布的内容刘德华最早发布的内容的内容刘德华最早发布的内容刘德华最早发布的内容的内容刘德华最早发布的内容刘德华最早发</div>
           <div class="reply-msg">
             回复的内容回复的内容回复的内容回复的内容回复的内容回复的内容回复的内容回复的内容回复的内容回复的内容回复的内容回复的内容回复的内容回复的内容回复的内容回复的内容回复的内容回复的内容回复的内容回复的内容回复的内容回复的内容回复的内容回复的内容回复的内容回复的内容回复的内容回复的内容回复的内容回复的内容回复的内容回复的内容回复的内容回复的内容回复的内容回复的内容回复的内容回复的内容
           </div>
           <div class="reply-btn">
             <span class="pointer" @click="dialogFormVisible=true">回复</span>
             <span class="pointer">删除</span>
           </div>
         </div>
        </div>
      </div>
      <el-form-item class="close-btn">
        <el-button type="info" @click="closePage()">关闭</el-button>
      </el-form-item>
    </el-form>
    <el-dialog title="管理员回复" :visible.sync="dialogFormVisible">
      <el-form>
        <el-form-item label="回复">
          <el-input class="el-input" type="textarea" v-model="dataForm.tagName" placeholder="请输入您的回复"></el-input>
        </el-form-item>
        <el-form-item label="上传附件" >
          <el-upload
            class="inline-block"
            :headers="{'token':token}"
            :action="this.$http.adornUrl('/jinding/file/upload')"
            :on-success="handleChange"
            :on-error="handleChange"
            :on-remove="handleRemove"
            :file-list="fileList"
          >
            <el-button v-show="look!='look'" type="warning">选择文件</el-button>
          </el-upload>
        </el-form-item>
        <el-form-item label="上传图片">
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
                       :on-success="handleChange2"
                       :on-error="handleChange2"
                       list-type="picture-card"
                       :on-remove="handleRemove2">
              <i class="el-icon-plus"></i>
              <div slot="tip" class="el-upload__tip">只能上传jpg/png文件，且不超过500kb</div>
            </el-upload>
          </div>
        </el-form-item>

      </el-form>
      <div slot="footer" class="dialog-footer">
        <el-button @click="dialogFormVisible = false">关闭</el-button>
      </div>
    </el-dialog>
  </div>
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
        dialogFormVisible: false,
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
<style>
  .reply-btn{
    margin-top: 20px;
    color:#00a0e9;
  }
  .reply-btn>span{
    display: inline-block;
    margin-right: 20px;
  }
  .reply-msg{
    line-height: 20px;
  }
  .reply-reply-msg{
    padding:10px;
    background: #eee;
    border-radius: 4px;
    margin-bottom: 20px;
    line-height: 20px;
  }
  .reply-date{
    font-size: 14px;
    color:#999;
    margin: 10px 0;
  }
  .reply-name{
    font-size: 16px;
    font-weight: bold;
  }
  .pointer{
    cursor: pointer;
  }
  .lz{
    color:orange;
    font-size: 14px;
    font-weight: normal;
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
  .city-input{
    width: 80%;
  }
  .input-msg{
    font-size: 12px;
    color:#999;
    margin-top: 0;
  }
  .small-input{
    width: 200px;
  }
  .close-btn{
    text-align: center;
    margin-top: 20px;
  }
</style>
