<template>
  <div>
    <h2 style="border-bottom: 1px solid #ccc;padding-bottom: 20px;margin-bottom: 50px">查看投票</h2>
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
        prop="nameDes"
        align="center"
        label="投票人">
      </el-table-column>
      <el-table-column
        prop="type"
        align="center"
        label="投票属性">
        <template slot-scope="scope">
          {{ scope.row.type==0?'反对':'支持'}}
        </template>
      </el-table-column>
      <el-table-column
        prop="careateDate"
        header-align="center"
        align="center"
        label="投票时间">
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
  </div>
</template>

<script>
  export default {
    data () {
      return {
        path:window.SITE_CONFIG.cdnUrl,
        dataForm: {
          type: '',
          tagName: '',
        },
        typeList:[],
        twoTypeList:[],
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
      // 获取数据列表
      getDataList () {
        this.dataForm.id = this.$route.query.id||'';
        this.dataListLoading = true
        this.$http({
          url: this.$http.adornUrl('/biz/votes/list'),
          method: 'get',
          params: this.$http.adornParams({
            'pageNum': this.pageIndex,
            'pageSize': this.pageSize,
            'id': this.dataForm.id
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
    }
  }
</script>

<style scoped>

</style>
