<template>
  <div class="mod-policy">
    <h2 style="border-bottom: 1px solid #ccc;padding-bottom: 20px;margin-bottom: 50px">查看回复</h2>
    <el-form label-position="left" label-width="120px" :model="dataForm" :rules="dataRule" ref="dataForm">
      <div class="city-view-group">
        <div class="city-view-title">全部回复({{list.length}})</div>
        <div class="city-view-box">
         <div v-for="item in list"  class="reply-list">
           <div class="reply-name">{{item.type=='1'?'管理员':item.userName}} <span v-show="item.author=='0'" class="lz">楼主</span></div>
           <div class="reply-date">{{showtime(item.careateDate)}}</div>
           <div class="reply-reply-msg box-sizing"><span class="reply-name">@{{item.ptype=='1'?'管理员':item.replyUserName}}:</span>{{item.pcontent}}</div>
           <div class="reply-msg">
             {{item.content}}
           </div>
           <div class="reply-btn">
             <span class="pointer" @click="dialogFormVisible=true,dataForm.replyId=item.id">回复</span>
             <span class="pointer" @click="del(item.id)">删除</span>
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
          <el-input class="el-input" type="textarea" v-model="dataForm.content" placeholder="请输入您的回复"></el-input>
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
          <div class="inline-block box-img"  v-if="dataForm.imgList==''||!dataForm.imgList">
            <el-upload :disabled="look=='look'"
                       :limit="3"
                       :headers="{'token':token}"
                       :action="this.$http.adornUrl('/jinding/file/upload')"
                       :on-success="handleChange2"
                       :on-error="handleChange2"
                       list-type="picture-card"
                       :file-list="imgList"
                       :on-remove="handleRemove2">
              <i class="el-icon-plus"></i>
              <div slot="tip" class="el-upload__tip">最多上传3张，且不超过500kb</div>
            </el-upload>
          </div>
        </el-form-item>

      </el-form>
      <div slot="footer" class="dialog-footer">
        <el-button @click="dialogFormVisible = false">关闭</el-button>
        <el-button type="primary" @click="subReply()">确定</el-button>
      </div>
    </el-dialog>
  </div>
</template>
<script>
  import { isInteger } from '@/utils/validate'
  export default {
    inject:['removeTabHandle'],
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
        imgList:[],
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
        list:[],
        dataForm: {
          id: '',
          replyId:'',
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
      this.init(this.$route.query.id)
    },
    methods: {
      //删除回复
      del(id){
        this.$confirm(`确认删除该条数据吗?删除后数据不可恢复`, '提示', {
          confirmButtonText: '确定',
          cancelButtonText: '取消',
          type: 'warning'
        }).then(() => {
          this.$http({
            url: this.$http.adornUrl(`/biz/discuss/del/${id}`),
            method: 'GET',
          }).then(({data}) => {
            if (data && data.code === 10000) {
              this.$message({
                message: '操作成功',
                type: 'success',
                duration: 1500,
                onClose: () => {
                  this.init(this.dataForm.id)
                }
              })
            } else {
              this.$message.error(data.msg)
            }
          })
        }).catch(() => {})
        // '<img src='+  window.SITE_CONFIG['baseUrl']+'/jinding/showImg/'+ data.data+'" title="'+data.title+'" alt="'+data.original+'" width="100%">'
      },
      //提交回复
      subReply(){
        var tbAnnexActions=this.fileList!=''?[]:'',i=0,list=this.fileList,len=list.length;
        if(list!=''){
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
        }
        var img='',imgList=this.imgList,j=0,jen=imgList.length;
        if(imgList!=''){
          for(;j<jen;j++){
            img+=imgList[j].response.data;
          }
        }
        this.$http({
          url: this.$http.adornUrl(`/biz/discuss/save`),
          method: 'post',
          data: this.$http.adornData({
            'id': this.dataForm.replyId || undefined,
            'content': this.dataForm.content,
            'img': img,
            'tbAnnexActions': tbAnnexActions,
          })
        }).then(({data}) => {
          if (data && data.code == 10000) {
            this.$message({
              message: '操作成功',
              type: 'success',
              duration: 1500,
              onClose: () => {
                this.dialogFormVisible = false;
                this.init(this.dataForm.id)
              }
            })
          } else {
            this.$message.error(data.msg)
          }
        })
      },
      //获取富文本内容
      editorContent(modelname,index,content){
        console.log(modelname)
        this.dataForm.content=content
      },
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
              url: this.$http.adornUrl(`/biz/discuss/discuss/list`),
              method: 'get',
              params: this.$http.adornParams({
                id:this.dataForm.id
              })
            }).then(({data}) => {
              if (data && data.code === 10000) {
                var datas=data.data;
                this.list=datas;
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
        this.imgList=fileList;
      },
      //上传图片
      handleChange2(response, file, fileList){
        if (response && response.code === 10000) {
          this.$message({
            message: '上传成功',
            type: 'success',
            duration: 1500,
            onClose: () => {
              this.imgList=fileList;
            }
          })
        } else {
          this.$message.error(response.msg)
        }
      },
      showtime(time) {
        let date =
          typeof time === "number"
            ? new Date(time)
            : new Date((time || "").replace(/-/g, "/"));
        let diff = (new Date().getTime() - date.getTime()) / 1000;
        let dayDiff = Math.floor(diff / 86400);

        let isValidDate =
          Object.prototype.toString.call(date) === "[object Date]" &&
          !isNaN(date.getTime());

        if (!isValidDate) {
          window.console.error("不是有效日期格式");
        }
        const formatDate = function(date) {
          let today = new Date(date);
          let year = today.getFullYear();
          let month = ("0" + (today.getMonth() + 1)).slice(-2);
          let day = ("0" + today.getDate()).slice(-2);
          let hour = today.getHours();
          let minute = today.getMinutes();
          let second = today.getSeconds();
          return `${year}-${month}-${day} ${hour}:${minute}:${second}`;
        };
        //小于0或者大于等于31显示原时间
        if (isNaN(dayDiff) || dayDiff < 0 || dayDiff >= 31) {
          return formatDate(date);
        }
        return (
          (dayDiff === 0 &&
            ((diff < 60 && "刚刚") ||
              (diff < 120 && "1分钟前") ||
              (diff < 3600 && Math.floor(diff / 60) + "分钟前") ||
              (diff < 7200 && "1小时前") ||
              (diff < 86400 && Math.floor(diff / 3600) + "小时前"))) ||
          (dayDiff === 1 && "昨天") ||
          (dayDiff < 7 && dayDiff + "天前") ||
          (dayDiff < 31 && Math.ceil(dayDiff / 7) + "周前")
        );
      },
      closePage:function () {
        this.removeTabHandle(this.$store.state.common.mainTabsActiveName)
      },
    }
  }
</script>
<style>
  .reply-list{
    margin-bottom: 20px;
  }
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
