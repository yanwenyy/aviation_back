<template>
  <div>
    <el-dialog
      :title="look=='look'?'查看':!dataForm.id ? '新增' : '修改'"
      :close-on-click-modal="false"
      :visible.sync="visible">
      <el-form :model="dataForm" :rules="dataRule" ref="dataForm" @keyup.enter.native="dataFormSubmit()" label-width="80px">
        <el-form-item label="板块名称">
          <el-select  :disabled="look=='look'" clearable  v-model="dataForm.pid" placeholder="请选择">
            <el-option
              v-for="item in typeList"
              :key="item.id"
              :label="item.name"
              :value="item.id">
            </el-option>
          </el-select>
        </el-form-item>
        <el-form-item label="子板块名称">
          <el-input  :disabled="look=='look'" v-model="dataForm.childName" placeholder="子板块名称"></el-input>
        </el-form-item>
        <el-form-item label="开放权限">
          <el-select  :disabled="look=='look'" clearable  v-model="dataForm.purviewType" placeholder="请选择">
            <el-option
              v-for="item in qxList"
              :key="item.value"
              :label="item.label"
              :value="item.value">
            </el-option>
          </el-select>
        </el-form-item>
        <el-form-item label="选择开放会员" v-show="dataForm.purviewType==1">
          <el-button v-show="look!='look'" type="primary" @click="dialogVisible=true">点击选择</el-button>
          <el-table
            v-show="dataForm.id"
            :data="dataForm.users"
            border
            style="width: 100%;margin: 20px 0">
            <el-table-column
              type="index"
              align="center"
              label="序号">
            </el-table-column>
            <el-table-column
              prop="id"
              align="center"
              label="会员ID">
            </el-table-column>
            <el-table-column
              prop="nameDes"
              header-align="center"
              align="center"
              label="真实姓名">
            </el-table-column>
            <el-table-column
              prop="sort"
              header-align="center"
              align="center"
              label="用户名">
            </el-table-column>
            <el-table-column
              prop="sort"
              header-align="center"
              align="center"
              label="企业名称">
            </el-table-column>
          </el-table>
        </el-form-item>
        <el-form-item label="板块介绍">
          <el-input class="el-input" type="textarea" :disabled="look=='look'" v-model="dataForm.introduction" placeholder="板块介绍"></el-input>
        </el-form-item>
        <el-form-item label="状态"  v-show="dataForm.id">
          <el-input disabled v-model="dataForm.status=='0'?'显示':'隐藏'" placeholder="ID"></el-input>
        </el-form-item>
        <el-form-item label="添加日期"  v-show="dataForm.id">
          <el-input disabled v-model="dataForm.createDate" placeholder="ID"></el-input>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
      <el-button @click="visible = false">取消</el-button>
      <el-button v-show="look!='look'" type="primary" @click="dataFormSubmit()">确定</el-button>
    </span>
    </el-dialog>
    <el-dialog title="选择开放会员" :visible.sync="dialogVisible">
      <el-table
        :data="userList"
        border
        @selection-change="selectionChangeHandle"
        style="width: 100%;">
        <el-table-column
          type="selection"
          header-align="center"
          align="center"
          width="50">
        </el-table-column>
        <el-table-column
          type="index"
          align="center"
          label="序号">
        </el-table-column>
        <el-table-column
          prop="id"
          align="center"
          label="会员ID">
        </el-table-column>
        <el-table-column
          prop="nameDes"
          header-align="center"
          align="center"
          label="真实姓名">
        </el-table-column>
        <el-table-column
          prop="sort"
          header-align="center"
          align="center"
          label="用户名">
        </el-table-column>
        <el-table-column
          prop="sort"
          header-align="center"
          align="center"
          label="企业名称">
        </el-table-column>
      </el-table>
      <div slot="footer" class="dialog-footer">
        <el-button @click="dialogVisible = false">取消</el-button>
        <el-button type="primary" @click="addUsers()">确 定</el-button>
      </div>
    </el-dialog>
  </div>
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
        userList:[],
        qxList:[
          {
            label:'全部会员',
            value:'0'
          },
          {
            label:'指定会员',
            value:'1'
          },
        ],
        typeList:[],
        dataForm: {
          id: '',
          pid: '',
          childName: '',
          purviewType:'',
          introduction: '',
          status: '',
          createDate:'',
          users:[]
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
    mounted(){
      this.dataForm={
        id: '',
        pid: '',
        childName: '',
        purviewType:'',
        introduction: '',
        status: '',
        createDate:'',
        users:[]
      };
      this.userList=[];
      //板块下拉列表
      this.$http({
        url: this.$http.adornUrl('/biz/jobmodel/select/list'),
        method: 'GET',
      }).then(({data}) => {
        this.typeList = data.data
      });
      //选择开放会员列表
      this.$http({
        url: this.$http.adornUrl('/biz/user/select/list'),
        method: 'GET',
      }).then(({data}) => {
        this.userList = data.data
      });
    },
    methods: {
      //选择指定的会员
      addUsers(){
        this.dialogVisible=false;
        this.dataForm.users=this.dataListSelections;
      },
      // 多选
      selectionChangeHandle (val) {
        this.dataListSelections = val
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
              url: this.$http.adornUrl(`/biz/jobchildmodel/info/${this.dataForm.id}`),
              method: 'get',
              params: this.$http.adornParams()
            }).then(({data}) => {
              if (data && data.code === 10000) {
                var datas=data.data;
                this.dataForm.pid =datas.pid;
                this.dataForm.childName = datas.childName;
                this.dataForm.purviewType =datas.purviewType;
                this.dataForm.introduction = datas.introduction;
                this.dataForm.status =String(datas.status);
                this.dataForm.createDate = datas.createDate;
                this.dataForm.users=datas.users;
              }
            })
          }
        })
      },
      // 表单提交
      dataFormSubmit () {
        var ids = this.dataForm.purviewType=='1'&&this.dataListSelections.map(item => {
          return item.id
        });
        this.$refs['dataForm'].validate((valid) => {
          if (valid) {
            this.$http({
              url: this.$http.adornUrl(`/biz/jobchildmodel/${!this.dataForm.id ? 'save' : 'update'}`),
              method: 'post',
              data: this.$http.adornData({
                'id': this.dataForm.id || undefined,
                'pid': this.dataForm.pid,
                'childName': this.dataForm.childName,
                'purviewType': this.dataForm.purviewType,
                'introduction': this.dataForm.introduction,
                'purview': this.dataForm.purviewType=='1'?ids.join(","):null
              })
            }).then(({data}) => {
              if (data && data.code == 10000) {
                this.$message({
                  message: '操作成功',
                  type: 'success',
                  duration: 1500,
                  onClose: () => {
                    this.visible = false;
                    this.dataForm={
                      id: '',
                      pid: '',
                      childName: '',
                      purviewType:'',
                      introduction: '',
                      status: '',
                      createDate:'',
                      users:[]
                    };
                    this.userList=[];
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
</style>

