<template>
  <div class="mod-user">
    <el-form :inline="true" :model="dataForm" @keyup.enter.native="getDataList()">
      <el-form-item>
        <el-button type="primary" @click="openDialog('','新增板块',dataForm.addName='')">新增</el-button>
      </el-form-item>
      <el-form-item>
        <el-input v-model="dataForm.name" placeholder="板块名称" clearable></el-input>
      </el-form-item>
      <el-form-item label="状态:">
        <el-select clearable  v-model="dataForm.status" placeholder="请选择">
          <el-option
            v-for="item in ztList"
            :key="item.value"
            :label="item.label"
            :value="item.value">
          </el-option>
        </el-select>
      </el-form-item>
      <el-form-item>
        <el-button @click="getDataList()">查询</el-button>
        <el-button @click="resetForm()">重置</el-button>
      </el-form-item>
    </el-form>
    <el-table
      :data="dataList"
      border
      v-loading="dataListLoading"
      @selection-change="selectionChangeHandle"
      style="width: 100%;">
      <el-table-column
       type="index"
        header-align="center"
        align="center"
        width="80"
        label="序号">
      </el-table-column>
      <el-table-column
        prop="name"
        align="center"
        label="板块名称">
      </el-table-column>
      <el-table-column
        prop="status"
        align="center"
        label="状态">
        <template slot-scope="scope">
          {{ scope.row.status==0?'显示 ':'隐藏'}}
        </template>
      </el-table-column>
      <el-table-column
        prop="createDate"
        header-align="center"
        align="center"
        label="添加时间">
      </el-table-column>
      <el-table-column
        fixed="right"
        header-align="center"
        align="center"
        width="150"
        label="操作">
        <template slot-scope="scope">
          <el-button type="text" size="small"  @click="ifOnLine(scope.row.id,scope.row.status)">{{ scope.row.status==0?'隐藏':'显示 '}}</el-button>
          <el-button v-if="" type="text" size="small" @click="openDialog(scope.row.id,'编辑板块'),dataForm.addName=scope.row.name">修改</el-button>
          <el-button v-if="" type="text" size="small" @click="deleteHandle(scope.row.id)">删除</el-button>
        </template>
      </el-table-column>
    </el-table>
    <el-pagination
      @size-change="sizeChangeHandle"
      @current-change="currentChangeHandle"
      :current-page="pageIndex"
      :page-sizes="[10, 20, 50, 100]"
      :page-size="pageSize"
      :total="totalPage"
      layout="total, sizes, prev, pager, next, jumper">
    </el-pagination>
    <!-- 弹窗, 新增 / 修改 -->
    <el-dialog :title="dialogTitle" :visible.sync="dialogFormVisible">
      <el-form>
        <el-form-item label="填写名称">
          <el-input v-model="dataForm.addName" autocomplete="off"></el-input>
        </el-form-item>
      </el-form>
      <div slot="footer" class="dialog-footer">
        <el-button @click="dialogFormVisible = false">关闭</el-button>
        <el-button type="primary" @click="addWork()">确 定</el-button>
      </div>
    </el-dialog>
  </div>
</template>

<script>
  export default {
    data () {
      return {
        path:window.SITE_CONFIG.cdnUrl,
        dataForm: {
          status: '',
          name: '',
          addName:'',
          addId:''
        },
        dialogTitle:'',
        dialogFormVisible:false,
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
        dataList: [],
        pageIndex: 1,
        pageSize: 10,
        totalPage: 0,
        dataListLoading: false,
        dataListSelections: [],
        addOrUpdateVisible: false,
        drVisibel:false
      }
    },
    activated () {
      this.getDataList();
    },
    methods: {
      //新增或修改板块
      addWork(){
        this.$http({
          url: this.$http.adornUrl(`/biz/jobmodel/${!this.dataForm.addId ? 'save' : 'update'}`),
          method: 'post',
          data: this.$http.adornData({
            'id': this.dataForm.addId || undefined,
            'name': this.dataForm.addName
          })
        }).then(({data}) => {
          if (data && data.code == 10000) {
            this.$message({
              message: '操作成功',
              type: 'success',
              duration: 1500,
              onClose: () => {
                this.dialogFormVisible=false;
                this.getDataList();
              }
            })
          } else {
            this.$message.error(data.msg)
          }
        })
      },
      //上线下线
      ifOnLine (id,status) {
        this.$http({
          url: this.$http.adornUrl(`/biz/jobmodel/info/${id}/${status==0?'1':'0'}`),
          method: 'GET'
        }).then(({data}) => {
          if (data && data.code === 10000) {
            this.$message({
              message: '操作成功',
              type: 'success',
              duration: 1500,
              onClose: () => {
                this.getDataList()
              }
            })
          } else {
            this.$message.error(data.msg)
          }
        })
      },
      //打开弹窗
      openDialog(type,title){
        this.dialogFormVisible=true;
        this.dialogTitle=title;
        this.dataForm.addId=type;
      },
      //重置搜索条件
      resetForm(){
        this.dataForm={
          status: '',
          name: '',
        }
        this.getDataList();
      },
      // 获取数据列表
      getDataList () {
        this.dataListLoading = true
        this.$http({
          url: this.$http.adornUrl('/biz/jobmodel/list'),
          method: 'get',
          params: this.$http.adornParams({
            'pageNum': this.pageIndex,
            'pageSize': this.pageSize,
            'status': this.dataForm.status,
            'name': this.dataForm.name,
          })
        }).then(({data}) => {
          if (data && data.code === 10000) {
            this.dataList = data.data
            this.totalPage = data.total
          } else {
            this.dataList = []
            this.totalPage = 0
          }
          this.dataListLoading = false
        })
      },
      // 每页数
      sizeChangeHandle (val) {
        this.pageSize = val
        this.pageIndex = 1
        this.getDataList()
      },
      // 当前页
      currentChangeHandle (val) {
        this.pageIndex = val
        this.getDataList()
      },
      // 多选
      selectionChangeHandle (val) {
        this.dataListSelections = val
      },
      // 新增 / 修改
      addOrUpdateHandle (id,look) {
        this.addOrUpdateVisible = true
        this.$nextTick(() => {
          this.$refs.addOrUpdate.init(id,look)
        })
      },
      // 删除
      deleteHandle (id) {
        var id = id ? [id] : this.dataListSelections.map(item => {
          return item.id
        });
        this.$confirm(`确认删除该条数据吗?删除后数据不可恢复`, '提示', {
          confirmButtonText: '确定',
          cancelButtonText: '取消',
          type: 'warning'
        }).then(() => {
          this.$http({
            url: this.$http.adornUrl('/biz/jobmodel/delete/'),
            method: 'POST',
            data: this.$http.adornData(id, false)
          }).then(({data}) => {
            if (data && data.code === 10000) {
              this.$message({
                message: '操作成功',
                type: 'success',
                duration: 1500,
                onClose: () => {
                  this.getDataList()
                }
              })
            } else {
              this.$message.error(data.msg)
            }
          })
        }).catch(() => {})
      },
    }
  }
</script>
<style>
  .inline-block{
    display: inline-block;
  }
  .dr-notice-warn{
    width: 100%;
    box-sizing: border-box;
    padding:10px;
    background: #FFE5E0;
    color: red;
  }
  .dr-notice-body{
    padding:10px;
  }
  .dr-notice-body>div{
    margin-bottom: 20px;
  }
</style>
