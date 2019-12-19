<template>
  <div class="mod-config">
    <el-form :inline="true" :model="dataForm" @keyup.enter.native="getDataList()">
      <el-form-item>
        <el-input v-model="dataForm.advertisingPlatform" placeholder="广告平台" clearable></el-input>
      </el-form-item>
      <el-form-item>
        <el-input v-model="dataForm.mediaType" placeholder="媒体类型" clearable></el-input>
      </el-form-item>
      <el-form-item>
        <el-input v-model="dataForm.mediaName" placeholder="媒体名称" clearable></el-input>
      </el-form-item>
      <el-form-item>
        <el-select  v-model="dataForm.trafficType" placeholder="流量类型" style="width:100%;">
          <el-option v-for="item in traffics" :key="item.dictItemValue" :label="item.dictItemKey" :value="item.dictItemValue"></el-option>
        </el-select>
      </el-form-item>
      <el-form-item>
        <el-select  v-model="dataForm.advertisingSpaceType" placeholder="广告位类型" style="width:100%;">
          <el-option v-for="item in spaces" :key="item.dictItemValue" :label="item.dictItemKey" :value="item.dictItemValue"></el-option>
        </el-select>
      </el-form-item>
      <el-form-item>
        <el-select  v-model="dataForm.redirectType" placeholder="重定向" style="width:100%;">
          <el-option v-for="item in redirects" :key="item.dictItemValue" :label="item.dictItemKey" :value="item.dictItemValue"></el-option>
        </el-select>
      </el-form-item>
      <el-form-item>
        <el-button @click="getDataList()">查询</el-button>
      </el-form-item>
    </el-form>
    <el-table
      :data="dataList"
      border
      stripe
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
        prop="clientType"
        header-align="center"
        align="center"
        :formatter="formatClient"
        label="客户端类型">
      </el-table-column>
      <el-table-column
        prop="advertisingPlatform"
        header-align="center"
        align="center"
        label="广告平台">
      </el-table-column>
      <el-table-column
        prop="mediaType"
        header-align="center"
        align="center"
        label="媒体类型">
      </el-table-column>
      <el-table-column
        prop="mediaName"
        header-align="center"
        align="center"
        label="媒体名称">
         <template slot-scope="scope">
          <a  v-if="scope.row.redirectType==1?true:false" :href="'http://'+scope.row.url"
            target="_blank"
            class="buttonText">{{scope.row.mediaName}}</a>
          <font v-else>{{scope.row.mediaName}}</font>
        </template>
      </el-table-column>
      <el-table-column
        prop="advertisingSpace"
        header-align="center"
        align="center"
        label="广告位">
         <template slot-scope="scope">
          <a v-if="scope.row.redirectType==1?true:false" :href="'http://'+scope.row.url"
            target="_blank"
            class="buttonText">{{scope.row.advertisingSpace}}</a>
            <font v-else>{{scope.row.advertisingSpace}}</font>
        </template>
      </el-table-column>
      <el-table-column
        prop="advertisingSize"
        header-align="center"
        align="center"
        label="尺寸">
      </el-table-column>
      <el-table-column
        prop="sizeRequirement"
        header-align="center"
        align="center"
        label="尺寸要求">
      </el-table-column>
      <el-table-column
        prop="trafficType"
        header-align="center"
        align="center"
        label="流量类型"
        :formatter="formatTraffic">
      </el-table-column>
      <el-table-column
        prop="advertisingSpaceType"
        header-align="center"
        align="center"
        :formatter="formatSpace"
        label="广告位类型">
      </el-table-column>
      <el-table-column
        prop="volumeOfExhibition"
        header-align="center"
        align="center"
        label="日点击量">
      </el-table-column>
      <el-table-column
        prop="redirectType"
        header-align="center"
        align="center"
        :formatter="formatRedirects"
        label="重定向">
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
  </div>
</template>

<script>
  export default {
    data () {
      return {
        dataForm: {
          advertisingPlatform: '',
          mediaType: '',
          mediaName: '',
          advertisingSpace: '',

          advertisingSize: '',
          trafficType: '',
          advertisingSpaceType: '',
          redirectType: ''
        },
        pageIndex: 1,
        pageSize: 10,
        totalPage: 0,
        dataListLoading: false,
        dataListSelections: [],
        addOrUpdateVisible: false,
        dataList: [],
        traffics: [],
        spaces: [],
        redirects: []
      }
    },
    activated () {
      this.getDataList()
    },
    mounted () {
      this.getDictType()
    },
    methods: {
      // 获取数据列表
      getDataList () {
        this.dataListLoading = true
        this.$http({
          url: this.$http.adornUrl('/generator/smedia/list'),
          method: 'get',
          params: this.$http.adornParams({
            'page': this.pageIndex,
            'limit': this.pageSize,
            'advertisingPlatform': this.dataForm.advertisingPlatform,
            'mediaType': this.dataForm.mediaType,
            'mediaName': this.dataForm.mediaName,
            'advertisingSpace': this.dataForm.advertisingSpace,
            'clientType': this.dataForm.clientType,
            'advertisingSize': this.dataForm.advertisingSize,
            'trafficType': this.dataForm.trafficType,
            'advertisingSpaceType': this.dataForm.advertisingSpaceType,
            'redirectType': this.dataForm.redirectType,
            'clientType': '1'
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
      formatTraffic (row, column)
      {
        var result = '';
        this.traffics.forEach(item=>{
          if(item.dictItemValue == row.trafficType)
          {
            result = item.dictItemKey;
          }
        })
        return result;
      },
      formatSpace (row, column)
      {
        var result = '';
        this.spaces.forEach(item=>{
          if(item.dictItemValue == row.advertisingSpaceType)
          {
            result = item.dictItemKey;
          }
        })
        return result;
      },
      formatRedirects (row,column)
      {
        var result = '';
        this.redirects.forEach(item=>{
          if(item.dictItemValue == row.redirectType)
          {
            result = item.dictItemKey;
          }
        })
        return result;
      },
       getDictType () {
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
            this.clients = []
          }
        })

        // 流量类型
        this.$http({
          url: this.$http.adornUrl(`/generator/sysdictitem/name`),
          method: 'get',
          params: this.$http.adornParams({
            'dictName': 'TRAFFIC_TYPE',
          })
        }).then(({data}) => {
          if (data && data.code === 0) {
           this.traffics = data.list;
          } else {
            this.traffics = []
          }
        })
        // 广告位类型
        this.$http({
          url: this.$http.adornUrl(`/generator/sysdictitem/name`),
          method: 'get',
          params: this.$http.adornParams({
            'dictName': 'ADVERSITING_SPACE_TYPE',
          })
        }).then(({data}) => {
          if (data && data.code === 0) {
           this.spaces = data.list;
          } else {
            this.spaces = []
          }
        })

        // 是否可以重定向
        this.$http({
          url: this.$http.adornUrl(`/generator/sysdictitem/name`),
          method: 'get',
          params: this.$http.adornParams({
            'dictName': 'IS_REDIRECT',
          })
        }).then(({data}) => {
          if (data && data.code === 0) {
           this.redirects = data.list;
          } else {
            this.redirects = []
          }
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
          this.$refs.addOrUpdate.init(id)
        })
      }
    }
  }
</script>
