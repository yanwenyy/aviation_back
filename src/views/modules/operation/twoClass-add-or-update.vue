<template>
  <el-dialog
    :title="look=='look'?'查看':!dataForm.id ? '新增' : '修改'"
    :close-on-click-modal="false"
    :visible.sync="visible">
    <el-form :model="dataForm" :rules="dataRule" ref="dataForm" @keyup.enter.native="dataFormSubmit()" label-width="80px">
      <el-form-item label="请选择一级:">
        <el-select clearable  v-model="dataForm.pid" placeholder="请选择">
          <el-option
            v-for="item in typeList"
            :key="item.id"
            :label="item.name"
            :value="item.id">
          </el-option>
        </el-select>
      </el-form-item>
      <el-form-item label="请输入分类名称" prop="name">
        <el-input :disabled="look=='look'" v-model="dataForm.name" placeholder="请输入分类名称"></el-input>
      </el-form-item>
      <el-form-item label="请输入排序">
        <el-input :disabled="look=='look'" v-model="dataForm.sort" placeholder="请输入排序"></el-input>
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
      return {
        look:'',
        visible: false,
        typeList:[],
        dataForm: {
          id: 0,
          pid:'',
          name: '',
          sort: '',
        },
        value: '',
        dataRule: {
          name: [
            { required: true, message: '标签名称不能为空', trigger: 'blur' }
          ],
        }
      }
    },
    created(){
      //一级分类列表
      this.$http({
        url: this.$http.adornUrl('/biz/classlevel/select/list'),
        method: 'GET',
        params: this.$http.adornParams({ 'level': 1 })
      }).then(({data}) => {
        this.typeList = data.data
      });
      this.dataForm.pid = "";
      this.dataForm.name = "";
      this.dataForm.sort = "";
    },
    methods: {
      init (id) {
        this.dataForm.id = id||0;
        this.visible = true;
        this.$nextTick(() => {
          this.$refs['dataForm'].resetFields();
          if (this.dataForm.id) {
            this.$http({
              url: this.$http.adornUrl(`/biz/classlevel/info/${this.dataForm.id}`),
              method: 'get',
              params: this.$http.adornParams()
            }).then(({data}) => {
              if (data && data.code === 10000) {
                var datas=data.data;
                this.dataForm.pid =datas.pid;
                this.dataForm.name =datas.name;
                this.dataForm.sort = datas.sort;
              }
            })
          }else{
            this.dataForm.pid ="";
            this.dataForm.name = "";
            this.dataForm.sort = "";
          }
        })
      },
      // 表单提交
      dataFormSubmit () {
        this.$refs['dataForm'].validate((valid) => {
          if (valid) {
            this.$http({
              url: this.$http.adornUrl(`/biz/classlevel/second/${!this.dataForm.id ? 'save' : 'update'}`),
              method: 'post',
              data: this.$http.adornData({
                'id': this.dataForm.id || undefined,
                'pid': this.dataForm.pid,
                'name': this.dataForm.name,
                'sort': this.dataForm.sort,
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
    width: 70%;
  }
  .input-msg{
    font-size: 12px;
    color:#999;
    margin-top: 0;
  }
</style>

