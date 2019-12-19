<template>
  <div class="mod-demo-echarts">
    <el-form :inline="true" :model="dataForm" @keyup.enter.native="getDataList()">
      <el-form-item>
        <el-input v-model="dataForm.jobName" placeholder="活动名称" clearable></el-input>
      </el-form-item>
      <el-form-item>
        <el-button @click="getDataList()">查询</el-button>
        <el-button v-if="isAuth('generator:srealdata:save')" type="primary" @click="addOrUpdateHandle()">新增</el-button>
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
        width="50" v-if="false">
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
        prop="jobName"
        header-align="center"
        align="center"
        label="活动名称">
      </el-table-column>
      <el-table-column
        prop="startTime"
        header-align="center"
        align="center"
        label="开始时间">
      </el-table-column>
      <el-table-column
        prop="endTime"
        header-align="center"
        align="center"
        label="结束时间">
      </el-table-column>
      <el-table-column
        prop="cpc"
        header-align="center"
        align="center"
        label="CPC">
      </el-table-column>
      <el-table-column
        prop="cpm"
        header-align="center"
        align="center"
        label="CPM">
      </el-table-column>
      <el-table-column
        prop="statu"
        header-align="center"
        align="center"
        label="状态">
        <template slot-scope="scope">
            <el-tag v-if="scope.row.statu == 1">未开始</el-tag>
            <el-tag type="success" v-if="scope.row.statu == 2">正在运行</el-tag>
            <el-tag type="info" v-if="scope.row.statu == 3">已暂停</el-tag>
            <el-tag type="warning" v-if="scope.row.statu == 4">已结束</el-tag>
        </template>
      </el-table-column>
      <el-table-column
        prop="consumeMoney"
        header-align="center"
        align="center"
        label="消耗金额">
      </el-table-column>
      <el-table-column
        prop="clicks"
        header-align="center"
        align="center"
        label="点击次数">
      </el-table-column>
      <el-table-column
        prop="clicksRate"
        header-align="center"
        align="center"
        label="点击率">
      </el-table-column>
      <el-table-column
        prop="exposures"
        header-align="center"
        align="center"
        label="曝光次数">
      </el-table-column>
      <el-table-column
        prop="arrives"
        header-align="center"
        align="center"
        label="到达次数">
      </el-table-column>
      <el-table-column
        prop="arrivesRate"
        header-align="center"
        align="center"
        label="到达率">
      </el-table-column>
      <el-table-column
        prop="createName"
        header-align="center"
        align="center"
        label="创建人名称">
      </el-table-column>
      <el-table-column
        prop="createTime"
        header-align="center"
        align="center"
        label="创建时间">
      </el-table-column>
      <el-table-column
        fixed="right"
        header-align="center"
        align="center"
        width="150"
        label="操作">
        <template slot-scope="scope">
            <el-button type="text" size="small" v-if="scope.row.statu==4" @click="areaInfo(scope.row.id)">地区</el-button>
            <el-button type="text" size="small" v-if="scope.row.statu==2" @click="stop(scope.row.id)">暂停</el-button>
            <el-button type="text" size="small" v-if="scope.row.statu==3" @click="recovery(scope.row.id)">恢复</el-button>
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
  import AddOrUpdate from './srealdata-add-or-update'
  export default {
    data () {
      return {
        dataForm: {
          jobName: ''
        },
        userId: '',
        userName: '',
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
      this.getParam()
      this.getDataList()
    },
    inject: ['refresh'],
    methods: {
      getParam()
      {
        if(this.$route.params.userId) {
          this.userId = this.$route.params.userId;
        }
        if(this.$route.params.userName)
        {
          this.userName = this.$route.params.userName;
        }
      },
      // 获取数据列表
      getDataList () {
        this.dataListLoading = true
        this.$http({
          url: this.$http.adornUrl('/generator/srealdata/list'),
          method: 'get',
          params: this.$http.adornParams({
            'page': this.pageIndex,
            'limit': this.pageSize,
            'userId': this.userId,
            'jobName': this.dataForm.jobName
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
          this.$refs.addOrUpdate.init(id,this.userId,this.userName)
        })
      },
      areaInfo (id) {
        this.$nextTick(() => {
          this.$router.push({ name: 'srealdata-area',params: {realDataId: id}});
        })
      },
      stop (id) {
        this.$http({
            url: this.$http.adornUrl('/generator/srealdata/stop'),
            method: 'post',
            data: this.$http.adornData(id, false)
          }).then(({data}) => {
            if (data && data.code === 0) {
              this.$message({
                message: '操作成功',
                type: 'success',
                duration: 1500,
              });
              this.refresh();
            }
            else
            {
              this.$message({
                message: '操作失败',
                type: 'error',
                duration: 1500,
              });
            }
          });
      },
      recovery (id) {
        this.$http({
            url: this.$http.adornUrl('/generator/srealdata/recovery'),
            method: 'post',
            data: this.$http.adornData(id, false)
          }).then(({data}) => {
            if (data && data.code === 0) {
              this.$message({
                message: '操作成功',
                type: 'success',
                duration: 1500,
              });
              this.refresh();
            }
            else
            {
              this.$message({
                message: data.msg,
                type: 'error',
                duration: 1500,
              });
            }
          });
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
            url: this.$http.adornUrl('/generator/srealdata/delete'),
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
