<template>
  <div class="mod-config">
    <el-form :inline="true" :model="dataForm" @keyup.enter.native="getDataList()">
      <el-form-item>
        <el-input v-model="dataForm.jobName" placeholder="活动名称" clearable></el-input>
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
      show-summary
      :summary-method="getSummary"
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
        prop="jobName"
        header-align="center"
        align="center"
        label="活动名称">
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
        fixed="right"
        header-align="center"
        align="center"
        width="150"
        label="操作">
        <template slot-scope="scope">
            <el-button type="text" size="small" v-if="scope.row.statu==4 && isAuth('generator:service:area')" @click="areaInfo(scope.row.id)">地区分布</el-button>
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
  </div>
</template>

<script>
  export default {
    data () {
      return {
        dataForm: {
          jobName: ''
        },
        dataList: [],
        pageIndex: 1,
        pageSize: 10,
        totalPage: 0,
        dataListLoading: false,
        dataListSelections: [],
        addOrUpdateVisible: false
      }
    },
    activated () {
      this.getDataList()
    },
    methods: {
      // 获取数据列表
      getDataList () {
        this.dataListLoading = true
        this.$http({
          url: this.$http.adornUrl('/generator/srealdata/list'),
          method: 'get',
          params: this.$http.adornParams({
            'page': this.pageIndex,
            'limit': this.pageSize,
            'jobName': this.dataForm.jobName,
            'type': 'client'
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
      getSummary(param)
      {
        const { columns, data } = param;
        const sums = [];
        columns.forEach((column, index) => {
          if(index == 0)
          {
            sums[index] = '合计';
            return
          }
          const values = data.map(item => Number(item[column.property]));
          if(column.property == 'cpc')
          { 
            var by = this.dataList.length;        
            sums[index] = (values.reduce((prev,curr) => {
              const value = Number(curr);
              if (!isNaN(value)) {
                return prev + curr;
              } else {
                return prev;
              }
            },0)/by).toFixed(2);
          }
          if(column.property == 'cpm')
          {
            var by = this.dataList.length;        
            sums[index] = (values.reduce((prev,curr) => {
              const value = Number(curr);
              if (!isNaN(value)) {
                return prev + curr;
              } else {
                return prev;
              }
            },0)/by).toFixed(2);
          }
          if(column.property == 'clicksRate')
          {
            var by = this.dataList.length;        
            sums[index] = (values.reduce((prev,curr) => {
              const value = Number(curr);
              if (!isNaN(value)) {
                return prev + curr;
              } else {
                return prev;
              }
            },0)/by).toFixed(2);
          }
          if(column.property == 'arrivesRate')
          {
            var by = this.dataList.length;        
            sums[index] = (values.reduce((prev,curr) => {
              const value = Number(curr);
              if (!isNaN(value)) {
                return prev + curr;
              } else {
                return prev;
              }
            },0)/by).toFixed(2);
          }
          if(column.property == 'consumeMoney')
          {
            var by = this.dataList.length;        
            sums[index] = values.reduce((prev,curr) => {
              const value = Number(curr);
              if (!isNaN(value)) {
                return prev + curr;
              } else {
                return prev;
              }
            },0);
          }
          if(column.property == 'clicks')
          {
            var by = this.dataList.length;        
            sums[index] = values.reduce((prev,curr) => {
              const value = Number(curr);
              if (!isNaN(value)) {
                return prev + curr;
              } else {
                return prev;
              }
            },0);
          }
          if(column.property == 'exposures')
          {
            var by = this.dataList.length;        
            sums[index] = values.reduce((prev,curr) => {
              const value = Number(curr);
              if (!isNaN(value)) {
                return prev + curr;
              } else {
                return prev;
              }
            },0);
          }
          if(column.property == 'arrives')
          {
            var by = this.dataList.length;        
            sums[index] = values.reduce((prev,curr) => {
              const value = Number(curr);
              if (!isNaN(value)) {
                return prev + curr;
              } else {
                return prev;
              }
            },0);
          }
        });
        return sums;
      },
      areaInfo (id) {
        this.$nextTick(() => {
          this.$router.push({ name: 'srealdata-area',params: {realDataId: id}});
        })
      },
      getWeek(i) {
        var now = new Date();
        var firstDay=new Date(now - (now .getDay() - 1 ) * 86400000);
        firstDay.setDate(firstDay.getDate() + i);
        mon = Number(firstDay.getMonth()) + 1;
        return now.getFullYear() + "/" + mon + "/" + firstDay.getDate();
      },
      getSummaries (param) {
        console.log(param);
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
      }
    }
  }
</script>
