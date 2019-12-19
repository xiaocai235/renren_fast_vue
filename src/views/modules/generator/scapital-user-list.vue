<template>
  <div class="mod-config">
    <el-form :inline="true" :model="dataForm" @keyup.enter.native="getDataList()">
      <el-form-item>
        <el-select v-model="dataForm.capitalType" placeholder="资金类型">
          <el-option v-for="item in capitals" :key="item.dictItemValue" :label="item.dictItemKey" :value="item.dictItemValue"></el-option>
        </el-select>
      </el-form-item>
    
      <el-form-item>
        <el-select v-model="dataForm.clientType" placeholder="客户端类型">
          <el-option v-for="item in clients" :key="item.dictItemValue" :label="item.dictItemKey" :value="item.dictItemValue"></el-option>
        </el-select>
      </el-form-item>

      <el-form-item>
        <el-button @click="getDataList()">查询</el-button>
        <el-button v-if="isAuth('generator:scapital:delete')" type="danger" @click="deleteHandle()" :disabled="dataListSelections.length <= 0">批量删除</el-button>
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
        prop="id"
        header-align="center"
        align="center"
        label="ID">
      </el-table-column>
      <el-table-column
        prop="userName"
        header-align="center"
        align="center"
        label="用户名">
      </el-table-column>
      <el-table-column
        prop="money"
        header-align="center"
        align="center"
        label="金额">
      </el-table-column>
      <el-table-column
        prop="capitalType"
        header-align="center"
        align="center"
        :formatter="formatCapital"
        label="资金类型">
      </el-table-column>
      <el-table-column
        prop="clientType"
        header-align="center"
        align="center"
        :formatter="formatClient"
        label="客户端类型">
      </el-table-column>
      <el-table-column
        prop="remark"
        header-align="center"
        align="center"
        label="备注">
      </el-table-column>
      <el-table-column
        prop="createName"
        header-align="center"
        align="center"
        label="操作人">
      </el-table-column>
      <el-table-column
        prop="createTime"
        header-align="center"
        align="center"
        label="操作时间">
      </el-table-column>
       <el-table-column
        fixed="right"
        header-align="center"
        align="center"
        width="150"
        label="操作">
        <template slot-scope="scope">
          <el-button type="text" size="small" @click="addOrUpdateHandle(scope.row.id)">修改</el-button>
          <el-button type="text" size="small" @click="deleteHandle(scope.row.id)">删除</el-button>
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
  import AddOrUpdate from './scapital-add-or-update'
  export default {
    data () {
      return {
        dataForm: {
          userId: '',
          userName: '',
          capitalType: '',
          clientType: ''
        },
        capitals: [],
        clients: [],
        dataList: [],
        pageIndex: 1,
        pageSize: 10,
        totalPage: 0,
        dataListLoading: false,
        dataListSelections: [],
        addOrUpdateVisible: false
      }
    },
    components: {
      AddOrUpdate
    },
    activated () {
        if(this.$route.params.userId && this.$route.params.userName) {
          if(this.$route.params.userId){
            this.userId = this.$route.params.userId;
          }
          if(this.$route.params.userName)
          {
            this.userName = this.$route.params.userName;
          }
          this.getDataList()
        }
        else
        {
          this.$router.push({ name: 'home'});
        }

      
    },
    mounted () {
      this.getDictType()
    },
    methods: {
      // 获取数据列表
      getDataList () {
        this.dataListLoading = true
        this.$http({
          url: this.$http.adornUrl('/generator/scapital/list'),
          method: 'get',
          params: this.$http.adornParams({
            'page': this.pageIndex,
            'limit': this.pageSize,
            'userId': this.userId,
            'userName': this.userName,
            'capitalType': this.dataForm.capitalType,
            'clientType': this.dataForm.clientType
          })
        }).then(({data}) => {
          if (data && data.code === 0) {
            this.dataList = data.page.list
            this.totalPage = data.page.totalCount
          } else {
            this.dataList = []
            this.totalPage = 0
          }
          this.dataListLoading = false
        })
      },

      getDictType () {
        // 资金类型
        this.$http({
          url: this.$http.adornUrl(`/generator/sysdictitem/name`),
          method: 'get',
          params: this.$http.adornParams({
            'dictName': 'CAPITAL_TYPE',
          })
        }).then(({data}) => {
          if (data && data.code === 0) {
           this.capitals = data.list;
          } else {
            this.dataList = []
          }
        })
        // 客户端类型
        this.$http({
          url: this.$http.adornUrl(`/generator/sysdictitem/name`),
          method: 'get',
          params: this.$http.adornParams({
            'dictName': 'CLIENT_TYPE',
          })
        }).then(({data}) => {
          if (data && data.code === 0) {
            this.clients = data.list;
          } else {
            this.dataList = []
          }
        })

      },
      formatCapital (row, column) {
        var result = '';
        this.capitals.forEach(item=>{
          if(item.dictItemValue == row.capitalType)
          {
            result = item.dictItemKey;
          }
        })
        return result;
      },
      formatClient (row, column)
      {
        var result = '';
        this.clients.forEach(item=>{
          if(item.dictItemValue == row.clientType)
          {
            result = item.dictItemKey;
          }
        })
        return result;
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
      addOrUpdateHandle (id) {
        this.addOrUpdateVisible = true
        this.$nextTick(() => {
          this.$refs.addOrUpdate.init(id,null,null)
        })
      },
      // 删除
      deleteHandle (id) {
        var ids = id ? [id] : this.dataListSelections.map(item => {
          return item.id
        })
        this.$confirm(`确定对[id=${ids.join(',')}]进行[${id ? '删除' : '批量删除'}]操作?`, '提示', {
          confirmButtonText: '确定',
          cancelButtonText: '取消',
          type: 'warning'
        }).then(() => {
          this.$http({
            url: this.$http.adornUrl('/generator/scapital/delete'),
            method: 'post',
            data: this.$http.adornData(ids, false)
          }).then(({data}) => {
            if (data && data.code === 0) {
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
        })
      }
    }
  }
</script>
