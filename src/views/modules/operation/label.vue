<template>
  <div class="mod-user">
    <el-form :inline="true" :model="dataForm" @keyup.enter.native="getDataList()">
      <el-form-item>
        <el-button v-if="" type="primary" @click="addOrUpdateHandle()">新增</el-button>
      </el-form-item>
      <el-form-item label="所属分类:">
        <el-select clearable  v-model="dataForm.type" placeholder="请选择">
          <el-option
            v-for="item in typeList"
            :key="item.value"
            :label="item.label"
            :value="item.value">
          </el-option>
        </el-select>
      </el-form-item>
      <el-form-item label="标签名称:">
        <el-input v-model="dataForm.tagName" placeholder="标签名称" clearable></el-input>
      </el-form-item>
      <el-form-item>
        <el-button @click="getDataList()">查询</el-button>
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
        prop="type"
        align="center"
        label="所属分类">
        <template slot-scope="scope">
          {{ scope.row.type==1?'行业动态':scope.row.type==2?'资料中心':'通知公告'}}
        </template>
      </el-table-column>
      <el-table-column
        prop="tagName"
        header-align="center"
        align="center"
        label="标签名称">
      </el-table-column>
      <el-table-column
        prop="insertTime"
        header-align="center"
        align="center"
        label="创建日期">
      </el-table-column>
      <el-table-column
        fixed="right"
        header-align="center"
        align="center"
        width="150"
        label="操作">
        <template slot-scope="scope">
          <el-button v-if="" type="text" size="small" @click="addOrUpdateHandle(scope.row.tagId,'look')">查看</el-button>
          <el-button v-if="" type="text" size="small" @click="addOrUpdateHandle(scope.row.tagId)">修改</el-button>
          <el-button v-if="" type="text" size="small" @click="deleteHandle(scope.row.tagId)">删除</el-button>
          <el-button v-if="" type="text" size="small" @click="ifOnLine(scope.row.tagId,scope.row.status)">{{scope.row.status==0?'下线':'上线'}}</el-button>
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
  import AddOrUpdate from './label-add-or-update'
  export default {
    data () {
      return {
        path:window.SITE_CONFIG.cdnUrl,
        dataForm: {
          type: '',
          tagName: '',
        },
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
      // 获取数据列表
      getDataList () {
        this.dataListLoading = true
        this.$http({
          url: this.$http.adornUrl('/biz/tag/list'),
          method: 'get',
          params: this.$http.adornParams({
            'pageNum': this.pageIndex,
            'pageSize': this.pageSize,
            'type': this.dataForm.type,
            'tagName': this.dataForm.tagName,
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
        var userIds = id ? [id] : this.dataListSelections.map(item => {
          return item.id
        })
        this.$confirm(`确认删除该条数据吗?删除后数据不可恢复`, '提示', {
          confirmButtonText: '确定',
          cancelButtonText: '取消',
          type: 'warning'
        }).then(() => {
          this.$http({
            url: this.$http.adornUrl('/biz/banner/delete'),
            method: 'post',
            data: this.$http.adornData(userIds, false)
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
      //上线下线
      ifOnLine (id,status) {
        this.$http({
          url: this.$http.adornUrl(`/biz/tag/info/${status==0?'1':'0'}/${id}`),
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
      }
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
