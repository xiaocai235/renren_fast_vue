<template>
  <div class="mod-config">
    <el-form :inline="true" :model="dataForm" @keyup.enter.native="getDataList()">
      <el-form-item>
        <el-input v-model="dataForm.fileName" placeholder="文件名称" clearable></el-input>
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
        label="账户姓名">
      </el-table-column>
      <el-table-column
        prop="fileName"
        header-align="center"
        align="center"
        label="文件名称">
      </el-table-column>
      <el-table-column
        header-align="center"
        align="center"
        label="文件URL" min-width="300px;">
        <template slot-scope="scope">
          <a v-for="(itemUrl,index) in scope.row.urls" @click="showPic(itemUrl)" style="margin-right: 10px;">图片{{index}}</a>
        </template>
      </el-table-column>
      <el-table-column
        prop="preAudit"
        header-align="center"
        align="center"
        :formatter="formatPreAudit"
        label="预先审核" >
      </el-table-column>
      <el-table-column
        prop="audit"
        header-align="center"
        align="center"
        :formatter="formatAudit"
        label="审核状态" >
      </el-table-column>
      <el-table-column
        prop="clientType"
        header-align="center"
        align="center"
        :formatter="formatClient"
        label="客户端类型">
      </el-table-column>
      <el-table-column
        prop="fileState"
        header-align="center"
        align="center"
        :formatter="formatFileState"
        label="文件状态">
      </el-table-column>
      <el-table-column
        prop="extensionUrl"
        header-align="center"
        align="center"
        label="推广链接">
        <template slot-scope="scope">
          <a :href="'http://'+scope.row.extensionUrl"
            target="_blank"
            class="buttonText">{{scope.row.extensionUrl}}</a>
        </template>
      </el-table-column>
      <el-table-column
        prop="remark"
        header-align="center"
        align="center"
        label="备注">
      </el-table-column>
      <el-table-column
        prop="createTime"
        header-align="center"
        align="center"
        label="创建时间">
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
          fileName: ''
        },
        fit: ['contain'],
        dataList: [],
        preAudits: [],
        audits: [],
        fileStates: [],
        clients: [],
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
    mounted () {
      this.getDictType()
    },
    methods: {
      // 获取数据列表
      getDataList () {
        this.dataListLoading = true
        this.$http({
          url: this.$http.adornUrl('/generator/smaterial/list'),
          method: 'get',
          params: this.$http.adornParams({
            'page': this.pageIndex,
            'limit': this.pageSize,
            'fileName': this.dataForm.fileName,
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
      getDictType () {
        // 预先审核状态
        this.$http({
          url: this.$http.adornUrl(`/generator/sysdictitem/name`),
          method: 'get',
          params: this.$http.adornParams({
            'dictName': 'PRE_AUDIT'
          })
        }).then(({data}) => {
          if (data && data.code === 0) {
           this.preAudits = data.list;
          } else {
            this.dataList = []
          }
        })
        // 审核状态
        this.$http({
          url: this.$http.adornUrl(`/generator/sysdictitem/name`),
          method: 'get',
          params: this.$http.adornParams({
            'dictName': 'AUDIT',
          })
        }).then(({data}) => {
          if (data && data.code === 0) {
           this.audits = data.list;
          } else {
            this.dataList = []
          }
        })
        // 文件状态
        this.$http({
          url: this.$http.adornUrl(`/generator/sysdictitem/name`),
          method: 'get',
          params: this.$http.adornParams({
            'dictName': 'FILE_STATE',
          })
        }).then(({data}) => {
          if (data && data.code === 0) {
           this.fileStates = data.list;
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
      formatAudit (row,column)
      {
        var result = '';
        this.audits.forEach(item=>{
          if(item.dictItemValue == row.audit)
          {
            result = item.dictItemKey;
          }
        })
        return result;
      },
      formatPreAudit (row, column)
      {
        var result = '';
        this.preAudits.forEach(item=>{
          if(item.dictItemValue == row.preAudit)
          {
            result = item.dictItemKey;
          }
        })
        return result;
      },
      formatFileState (row, column)
      {
        var result = '';
        this.fileStates.forEach(item=>{
          if(item.dictItemValue == row.fileState)
          {
            result = item.dictItemKey;
          }
        })
        return result;
      },
      showPic (url)
      {
        const image = this.$createElement // 创建元素
        this.$msgbox({
              title: '图片预览',
              dangerouslyUseHTMLString: true,
              message: "<image style='width:100%;' src="+url+" />",
              showCancelButton: true,  
              showConfirmButton: true,
              confirmButtonText: '下载',
              closeOnPressEscape: true
            }).then(action => { 
               this.downloadIamge(url,"pic");
            }).catch(() => {
             // 点击取消后执行的方法
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
      downloadIamge (imgsrc, name) { // 下载图片地址和图片名
        let image = new Image();
        // 解决跨域 Canvas 污染问题
        image.setAttribute("crossOrigin", "anonymous")
        image.onload = function() {
          let canvas = document.createElement("canvas")
          canvas.width = image.width
          canvas.height = image.height
          let context = canvas.getContext("2d")
          context.drawImage(image, 0, 0, image.width, image.height)
          let url = canvas.toDataURL("image/png") //得到图片的base64编码数据
          let a = document.createElement("a") // 生成一个a元素
          let event = new MouseEvent("click") // 创建一个单击事件
          a.download = name || "photo" // 设置图片名称
          a.href = url // 将生成的URL设置为a.href属性
          a.dispatchEvent(event) // 触发a的单击事件
        }
        image.src = imgsrc
      }
    }
  }
</script>
