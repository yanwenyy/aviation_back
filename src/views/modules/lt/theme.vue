<template>
  <div class="mod-user">
    <el-form :inline="true" :model="dataForm" @keyup.enter.native="getDataList()">
      <el-form-item>
        <el-button type="primary" @click="addOrUpdateHandle()">新增</el-button>
        <el-button type="primary" @click="deleteHandle()"  :disabled="dataListSelections.length <= 0">批量删除</el-button>
        <el-button type="primary" @click="childModelId='',modelId='',openDialog('','新增板块')"  :disabled="dataListSelections.length <= 0">批量转移</el-button>
      </el-form-item>
      <el-form-item>
        <el-input v-model="dataForm.id" placeholder="ID" clearable></el-input>
      </el-form-item>
      <el-form-item>
        <el-input v-model="dataForm.pname" placeholder="板块名称" clearable></el-input>
      </el-form-item>
      <el-form-item>
        <el-input v-model="dataForm.childName" placeholder="子版块名称" clearable></el-input>
      </el-form-item>
      <el-form-item>
        <el-input v-model="dataForm.title" placeholder="标题" clearable></el-input>
      </el-form-item>
      <el-form-item>
        <el-input v-model="dataForm.byUser" placeholder="发布人" clearable></el-input>
      </el-form-item>
      <el-form-item label="属性设置:">
        <el-select clearable  v-model="dataForm.measure" placeholder="请选择">
          <el-option
            v-for="item in sxList"
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
          placeholder="添加时间（起止日期）">
        </el-date-picker>
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
        label="ID">
      </el-table-column>
      <el-table-column
        prop="name"
        align="center"
        label="板块名称">
      </el-table-column>
      <el-table-column
        prop="childName"
        align="center"
        label="子版块名称">
      </el-table-column>
      <el-table-column
        prop="title"
        align="center"
        label="标题">
      </el-table-column>
      <el-table-column
        prop="byUser"
        align="center"
        label="发布人">
      </el-table-column>
      <el-table-column
        prop="pname"
        align="center"
        label="属性设置">
        <template slot-scope="scope">
          {{ scope.row.ifHigh=='1'?'高亮':''}}
          {{ scope.row.ifLocking=='1'?'锁定':''}}
          {{ scope.row.ifTop=='1'?'置顶':''}}
          <el-button @click="attributeVisible=true,openAttr(scope.row)">设置</el-button>
        </template>
      </el-table-column>
      <el-table-column
        prop="lookNum"
        align="center"
        label="查看数">
      </el-table-column>
      <el-table-column
        prop="replyNum"
        align="center"
        label="回复数">
      </el-table-column>
      <el-table-column
        prop="supportNum"
        align="center"
        label="支持">
      </el-table-column>
      <el-table-column
        prop="opposeNum"
        align="center"
        label="反对">
      </el-table-column>
      <el-table-column
        prop="status"
        align="center"
        label="状态">
        <template slot-scope="scope">
          {{ scope.row.status==0?'在线':'隐藏'}}
        </template>
      </el-table-column>
      <el-table-column
        prop="createDate"
        header-align="center"
        align="center"
        label="发布时间">
      </el-table-column>
      <el-table-column
        fixed="right"
        header-align="center"
        align="center"
        width="150"
        label="操作">
        <template slot-scope="scope">
          <el-button type="text" size="small" @click="ifOnLine(scope.row.id,scope.row.status)">{{ scope.row.status==0?'隐藏':'在线'}}</el-button>
          <el-button v-if="" type="text" size="small" @click="addOrUpdateHandle(scope.row.id,'look')">查看</el-button>
          <el-button v-if="" type="text" size="small" @click="addOrUpdateHandle(scope.row.id)">修改</el-button>
          <el-button v-if="" type="text" size="small" @click="deleteHandle(scope.row.id)">删除</el-button>
          <el-button type="text" size="small"  @click="$router.push({ name: 'reply-view',query:{id:scope.row.id} })">查看回复</el-button>
          <el-button type="text" size="small"  @click="$router.push({ name: 'vote-view',query:{id:scope.row.id} })">查看投票</el-button>
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
    <el-dialog title="批量转移" :visible.sync="dialogFormVisible">
      <el-form>
        <el-form-item label="一级板块">
          <el-select clearable @change="getTwo"  v-model="dataForm.modelId" placeholder="请选择">
            <el-option
              v-for="(item,index) in typeList"
              :key="index"
              :label="item.name"
              :value="item.id">
            </el-option>
          </el-select>
        </el-form-item>
        <el-form-item label="子板块">
          <el-select clearable  @change="getTwoName"  v-model="dataForm.childModelId" placeholder="请选择">
            <el-option
              v-for="(item,index) in typeTwoList"
              :key="index"
              :label="item.childName"
              :value="item.id">
            </el-option>
          </el-select>
        </el-form-item>
      </el-form>
      <div slot="footer" class="dialog-footer">
        <el-button @click="dialogFormVisible = false">关闭</el-button>
        <el-button @click="transfer()" type="primary">确定</el-button>
      </div>
    </el-dialog>
    <el-dialog title="设置属性" :visible.sync="attributeVisible">
      <el-checkbox-group v-model="checkList">
        <el-checkbox label="置顶"></el-checkbox>
        <el-checkbox label="锁定"></el-checkbox>
        <el-checkbox label="高亮"></el-checkbox>
      </el-checkbox-group>
      <div slot="footer" class="dialog-footer">
        <el-button @click="attributeVisible = false">关闭</el-button>
        <el-button @click="setAttribute()" type="primary">确定</el-button>
      </div>
    </el-dialog>
  </div>
</template>

<script>
  import AddOrUpdate from './theme-add-or-update'
  export default {
    components: {
      AddOrUpdate
    },
    data () {
      return {
        path:window.SITE_CONFIG.cdnUrl,
        dataForm: {
          id: '',
          pname: '',
          childName: '',
          title: '',
          byUser: '',
          measure: '',
          status: '',
          createTime: '',
          modelId:'',
          childModelId:''
        },
        oneName:'',
        twoName:"",
        checkList:[],
        dialogTitle:'',
        dialogFormVisible:false,
        attributeVisible:false,
        attrForm:'',
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
        sxList:[
          {
            label:'置顶',
            value:'1'
          },
          {
            label:'锁定',
            value:'2'
          },
          {
            label:'高亮',
            value:'3'
          }
        ],
        typeList:[],
        typeTwoList:[],
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
      //板块下拉列表
      this.$http({
        url: this.$http.adornUrl('/biz/jobmodel/select/list'),
        method: 'GET',
        params: this.$http.adornParams({
          'status': 1,
        })
      }).then(({data}) => {
        this.typeList = data.data
      });

    },
    methods: {
      //打开属性设置
      openAttr(form){
        this.checkList=[];
        this.attrForm=form;
        if(form.ifHigh==1){
          this.checkList.push('高亮')
        }
        if(form.ifLocking==1){
          this.checkList.push('锁定')
        }
        if(form.ifTop==1){
          this.checkList.push('置顶')
        }
      },
      //属性设置
      setAttribute(){
        this.$http({
          url: this.$http.adornUrl(`/biz/theme/attribute`),
          method: 'post',
          data: this.$http.adornData({
            'id': this.attrForm.id,
            'ifHigh': this.checkList.indexOf('高亮')!=-1?'1':'0',
            'ifLocking':this.checkList.indexOf('锁定')!=-1?'1':'0',
            'ifTop': this.checkList.indexOf('置顶')!=-1?'1':'0',
          })
        }).then(({data}) => {
          if (data && data.code == 10000) {
            this.$message({
              message: '操作成功',
              type: 'success',
              duration: 1500,
              onClose: () => {
                this.attributeVisible = false;
                this.getDataList()
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
          url: this.$http.adornUrl(`/biz/theme/info/${id}/${status==0?'1':'0'}`),
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
      //一级版块选择时
      getTwo(val){
        if(this.dataForm.modelId!=''){
          this.typeTwoList =[];
          this.dataForm.childModelId='';
          this.oneName = val ? this.typeList.find(ele => ele.id === val).name : '';
          //子板块下拉列表
          this.$http({
            url: this.$http.adornUrl('/biz/jobchildmodel/select/list'),
            method: 'GET',
            params: this.$http.adornParams({
              'id': this.dataForm.modelId,
              'status': 1,
            })
          }).then(({data}) => {
            this.typeTwoList = data.data
          });
        }
      },
      //获取二级版块名字
      getTwoName(val){
        this.twoName = val ? this.typeTwoList.find(ele => ele.id === val).childName : '';
      },
      //批量转移
      transfer(){
        // var ids =this.dataListSelections.map(item => {
        //   return item.id
        // });
        var ids=[],list=this.dataListSelections,i=0,len=list.length;
        for(;i<len;i++){
          ids.push(list[i].id)
        }
        console.log(ids)
        this.$confirm(`您确定要将选中的${ids.length}条数据转移至“${this.oneName} — ${this.twoName}”板块下吗？删除后数据将无法恢复！`, '提示', {
          confirmButtonText: '确定',
          cancelButtonText: '取消',
          type: 'warning'
        }).then(() => {
          this.$http({
            url: this.$http.adornUrl('/biz/theme/bach/udpate'),
            method: 'POST',
            data: this.$http.adornData({
              'childModelId': this.dataForm.childModelId,
              'modelId': this.dataForm.modelId,
              'id':ids,
            },false)
          }).then(({data}) => {
            if (data && data.code === 10000) {
              this.$message({
                message: '操作成功',
                type: 'success',
                duration: 1500,
                onClose: () => {
                  this.dialogFormVisible = false
                  this.getDataList()
                }
              })
            } else {
              this.$message.error(data.msg)
            }
          })
        }).catch(() => {})
      },
      //打开弹窗
      openDialog(type,title){
        this.dialogFormVisible=true;
        this.dialogTitle=title;
      },
      //重置搜索条件
      resetForm(){
        this.dataForm={
          id: '',
          pname: '',
          childName: '',
          title: '',
          byUser: '',
          measure: '',
          status: '',
          createTime: '',
        }
        this.getDataList()
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
          url: this.$http.adornUrl('/biz/theme/list'),
          method: 'get',
          params: this.$http.adornParams({
            'pageNum': this.pageIndex,
            'pageSize': this.pageSize,
            'id': this.dataForm.id,
            'pname': this.dataForm.pname,
            'childName': this.dataForm.childName,
            'title': this.dataForm.title,
            'byUser': this.dataForm.byUser,
            'measure': this.dataForm.measure,
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
            url: this.$http.adornUrl('/biz/theme/delete'),
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
