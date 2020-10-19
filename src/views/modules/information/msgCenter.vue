<template>
  <div class="mod-user">
    <el-form :inline="true" :model="dataForm" @keyup.enter.native="getDataList()">
      <el-form-item>
        <el-button type="primary" @click="addOrUpdateHandle()">新增</el-button>
      </el-form-item>
      <el-form-item>
        <el-input v-model="dataForm.id" placeholder="ID" clearable></el-input>
      </el-form-item>
      <el-form-item>
        <el-input v-model="dataForm.title" placeholder="标题" clearable></el-input>
      </el-form-item>
      <el-form-item>
        <el-input v-model="dataForm.source" placeholder="来源" clearable></el-input>
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
      <el-form-item label="添加时间（起止日期）">
        <el-date-picker
          v-model="dataForm.createTime"
          type="daterange"
          range-separator="一"
          value-format="yyyy-MM-dd"
          start-placeholder="开始日期"
          end-placeholder="结束日期">
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
        prop="id"
        header-align="center"
        align="center"
        width="80"
        label="ID">
      </el-table-column>
      <el-table-column
        prop="title"
        align="center"
        label="标题">
      </el-table-column>
      <el-table-column
        prop="clickNum"
        header-align="center"
        align="center"
        label="点击">
      </el-table-column>
      <el-table-column
        prop="source"
        header-align="center"
        align="center"
        label="来源">
      </el-table-column>
      <el-table-column
        prop="status"
        align="center"
        label="状态">
        <template slot-scope="scope">
          {{ scope.row.status==0?'在线':'隐藏'}}<el-button @click="ifOnLine(scope.row.id,scope.row.status)">{{ scope.row.status==0?'隐藏':'在线'}}</el-button>
        </template>
      </el-table-column>
      <el-table-column
        prop="insertTime"
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
          <el-button v-if="" type="text" size="small" @click="addOrUpdateHandle(scope.row.id,'look')">查看</el-button>
          <el-button v-if="" type="text" size="small" @click="addOrUpdateHandle(scope.row.id)">修改</el-button>
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
  import AddOrUpdate from './msgCenter-add-or-update'
  export default {
    data () {
      return {
        path:window.SITE_CONFIG.cdnUrl,
        dataForm: {
          id: '',
          title: '',
          source: '',
          status: '',
          createTime: '',
        },
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
    components: {
      AddOrUpdate,
    },
    activated () {
      this.getDataList();
    },
    methods: {
      //上线下线
      ifOnLine (id,status) {
        this.$http({
          url: this.$http.adornUrl(`/biz/trendmaterial/info/${status==0?'1':'0'}/${id}`),
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
      //重置搜索条件
      resetForm(){
        this.dataForm={
          id: '',
          title: '',
          source: '',
          status: '',
          createTime: '',
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
          url: this.$http.adornUrl('/biz/trendmaterial/list'),
          method: 'get',
          params: this.$http.adornParams({
            'pageNum': this.pageIndex,
            'pageSize': this.pageSize,
            'type': 1,
            'id': this.dataForm.id,
            'title': this.dataForm.title,
            'source': this.dataForm.source,
            'status': this.dataForm.status,
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
            url: this.$http.adornUrl('/biz/trendmaterial/delete'),
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
