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
        <el-input v-model="dataForm.advertisingSpace" placeholder="广告位" clearable></el-input>
      </el-form-item>
      <el-form-item>
        <el-select  v-model="dataForm.clientType" placeholder="客户端" style="width:100%;">
          <el-option v-for="item in clients" :key="item.dictItemValue" :label="item.dictItemKey" :value="item.dictItemValue"></el-option>
        </el-select>
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
        <el-button v-if="isAuth('generator:smedia:save')" type="primary" @click="addOrUpdateHandle()">新增</el-button>
        <el-button v-if="isAuth('generator:smedia:delete')" type="danger" @click="deleteHandle()" :disabled="dataListSelections.length <= 0">批量删除</el-button>
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
      <el-table-column
        prop="createTime"
        header-align="center"
        align="center"
        label="创建时间">
      </el-table-column>
      <el-table-column
        prop="isComment"
        label="范围"
        filter-placement="bottom-end">
        <template slot-scope="scope">
          <el-tag
            :type="scope.row.isComment == 1 ? 'primary' : 'success'"
            close-transition>{{scope.row.isComment == 1 ? '通用' : '私有'}}</el-tag>
        </template>
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
  import AddOrUpdate from './smedia-add-or-update'
  export default {
    data () {
      return {
        dataForm: {
          advertisingPlatform: '',
          mediaType: '',
          mediaName: '',
          advertisingSpace: '',

          clientType: '',
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
        clients: [],
        sizes: [],
        dataList: [],
        traffics: [],
        spaces: [],
        redirects: []
      }
    },
    components: {
      AddOrUpdate
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
            'redirectType': this.dataForm.redirectType
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
          url: this.$http.adornUrl(`/generator/sysdictitem/list`),
          method: 'get',
          params: this.$http.adornParams({
            'dictId': 3,
          })
        }).then(({data}) => {
          if (data && data.code === 0) {
           this.clients = data.page.list;
          } else {
            this.clients = []
          }
        })
        // 尺寸
        this.$http({
          url: this.$http.adornUrl(`/generator/sysdictitem/list`),
          method: 'get',
          params: this.$http.adornParams({
            'dictId': 9,
            'ispage': 0
          })
        }).then(({data}) => {
          if (data && data.code === 0) {
           this.sizes = data.page.list;
          } else {
            this.sizes = []
          }
        })

        // 流量类型
        this.$http({
          url: this.$http.adornUrl(`/generator/sysdictitem/list`),
          method: 'get',
          params: this.$http.adornParams({
            'dictId': 10,
          })
        }).then(({data}) => {
          if (data && data.code === 0) {
           this.traffics = data.page.list;
          } else {
            this.traffics = []
          }
        })
        // 广告位类型
        this.$http({
          url: this.$http.adornUrl(`/generator/sysdictitem/list`),
          method: 'get',
          params: this.$http.adornParams({
            'dictId': 11,
          })
        }).then(({data}) => {
          if (data && data.code === 0) {
           this.spaces = data.page.list;
          } else {
            this.spaces = []
          }
        })

        // 是否可以重定向
        this.$http({
          url: this.$http.adornUrl(`/generator/sysdictitem/list`),
          method: 'get',
          params: this.$http.adornParams({
            'dictId': 12,
          })
        }).then(({data}) => {
          if (data && data.code === 0) {
           this.redirects = data.page.list;
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
            url: this.$http.adornUrl('/generator/smedia/delete'),
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
