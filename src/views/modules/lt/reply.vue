<template>
  <div class="mod-user">
    <el-form :inline="true" :model="dataForm" @keyup.enter.native="getDataList()">
      <el-form-item>
        <el-button type="primary" @click="deleteHandle()"  :disabled="dataListSelections.length <= 0">批量删除</el-button>
      </el-form-item>
      <el-form-item>
        <el-input v-model="dataForm.id" placeholder="主题ID" clearable></el-input>
      </el-form-item>
      <el-form-item>
        <el-input v-model="dataForm.userName" placeholder="回复人" clearable></el-input>
      </el-form-item>
      <el-form-item>
        <el-input v-model="dataForm.content" placeholder="回复内容" clearable></el-input>
      </el-form-item>
      <el-form-item label="回复类型:">
        <el-select clearable  v-model="dataForm.type" placeholder="请选择">
          <el-option
            v-for="item in typeList"
            :key="item.value"
            :label="item.label"
            :value="item.value">
          </el-option>
        </el-select>
      </el-form-item>
      <el-form-item>
        <el-date-picker
          v-model="dataForm.createTime"
          type="date"
          placeholder="回复时间">
        </el-date-picker>
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
        type="selection"
        header-align="center"
        align="center"
        width="50">
      </el-table-column>
      <el-table-column
       type="index"
        header-align="center"
        align="center"
        width="80"
        label="序号">
      </el-table-column>
      <el-table-column
        prop="id"
        align="center"
        label="主题ID">
      </el-table-column>
      <el-table-column
        prop="userName"
        header-align="center"
        align="center"
        label="回复人">
      </el-table-column>
      <el-table-column
        prop="content"
        header-align="center"
        align="center"
        label="回复内容">
      </el-table-column>
      <el-table-column
        prop="level"
        header-align="center"
        align="center"
        label="回复类型">
        <template slot-scope="scope">
          {{ scope.row.level==1?'留言':'回复'}}
        </template>
      </el-table-column>
      <el-table-column
        prop="careateDate"
        header-align="center"
        align="center"
        label="回复时间">
      </el-table-column>
      <el-table-column
        fixed="right"
        header-align="center"
        align="center"
        width="150"
        label="操作">
        <template slot-scope="scope">
          <el-button v-if="" type="text" size="small" @click="addOrUpdateHandle(scope.row.id,'look')">查看</el-button>
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
    <add-or-update v-if="addOrUpdateVisible" ref="addOrUpdate" @refreshDataList="getDataList"></add-or-update>
  </div>
</template>

<script>
  import AddOrUpdate from './reply-add-or-update'
  export default {
    data () {
      return {
        path:window.SITE_CONFIG.cdnUrl,
        dataForm: {
          id: '',
          type: '',
          content:'',
          userName: '',
          createTime:'',
        },
        typeList:[
          {
            label:'留言',
            value:'1'
          },
          {
            label:'回复',
            value:'2'
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
    components: {
      AddOrUpdate,
    },
    activated () {
      this.getDataList();
    },
    methods: {
      //重置搜索条件
      resetForm(){
        this.dataForm={
          id: '',
          type: '',
          content:'',
          userName: '',
          createTime:'',
        }
        this.$refs.child.reset()
      },
      // 获取数据列表
      getDataList () {
        var createTimeStart='',createTimeEnd=''
        if(this.dataForm.createTime!=undefined&&this.dataForm.createTime!=""&&this.dataForm.createTime!=null&&this.dataForm.createTime.length!=0){
          createTimeStart=this.dataForm.createTime[0]
          createTimeEnd=this.dataForm.createTime[1]
        }
        this.dataListLoading = true
        this.$http({
          url: this.$http.adornUrl('/biz/discuss/list'),
          method: 'get',
          params: this.$http.adornParams({
            'pageNum': this.pageIndex,
            'pageSize': this.pageSize,
            'id': this.dataForm.id,
            'type': this.dataForm.type,
            'content': this.dataForm.content,
            'userName': this.dataForm.userName,
            'startTime': createTimeStart,
            'endTime': createTimeEnd,
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
        })
        this.$confirm(`确认删除该条数据吗?删除后数据不可恢复`, '提示', {
          confirmButtonText: '确定',
          cancelButtonText: '取消',
          type: 'warning'
        }).then(() => {
          this.$http({
            url: this.$http.adornUrl('/biz/discuss/delete'),
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
