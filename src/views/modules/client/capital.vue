<template>
  <div class="mod-demo-echarts">
     <el-alert
      title="您好！  欢迎登陆！"
      type="info"
      :closable="true">
    </el-alert>

    <el-row :gutter="12" style="margin-top: 20px;">
      <el-col :span="6">
        <el-card shadow="always">
          账户余额：
          &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
          {{ sy }}
        </el-card>
      </el-col>
      <el-col :span="6">
        <el-card shadow="always">
          累计充值：
          &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
          {{ cz }}
        </el-card>
      </el-col>
      <el-col :span="6">
        <el-card shadow="always">
          累计消费：
          &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
          {{ hf }}
        </el-card>
      </el-col>
      <el-col :span="6">
        <el-card shadow="always">
          今日消费：
          &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
          {{ jrxf }}
        </el-card>
      </el-col>
    </el-row>

    <el-row :gutter="20" style="margin-top: 20px;">
      <el-col :span="24">
        <el-card>
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
                </el-form-item>
              </el-form>
              
              <el-table
                :data="dataList"
                border
                v-loading="dataListLoading"
                style="width: 100%;">
                <el-table-column
                  type="selection"
                  header-align="center"
                  align="center"
                  width="50" v-if="1==2">
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
        </el-card>
      </el-col>
    </el-row>

  </div>
</template>

<script>
  export default {
    data () {
      return {
        dataForm: {
          key: ''
        },
        timer: null,
        userId: "",
        cz: 0.000,
        hf: 0.000,
        sy: 0.000,
        jrxf:0.000,
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
    created () {
      this.getDictType()
    },
    mounted () {
      this.getDataList()      
    },
    beforeDestroy () {
      this.socket.send('close');
      this.timer = null;
    },
    methods: {
      // 获取数据列表
      getDataList () {
        this.dataListLoading = true
         //获取数据汇总信息
        this.$http({
           url: this.$http.adornUrl('/generator/scapital/summary'),
           method: 'get',
           params: {}
        }).then(({data}) => {
           if (data && data.code === 0) {   
                //充值
                this.cz = data.summary.chongzhi;
                //花费
                this.hf = data.summary.xiaofei;
                //剩余
                this.sy = data.summary.shengyu;
                //今日消费
                this.jrxf = data.summary.jinrixiaofei;
           }
        })
        this.$http({
          url: this.$http.adornUrl('/generator/scapital/info'),
          method: 'get',
          params: this.$http.adornParams({
            'page': this.pageIndex,
            'limit': this.pageSize,
            'capitalType': this.dataForm.capitalType,
            'clientType': this.dataForm.clientType,
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
        })
        this.dataListLoading = false
        this.$http({
          url: this.$http.adornUrl('/sys/user/info'),
          method: 'get',
          params: {}
        }).then(({data}) => {
          if (data && data.code === 0) {
            this.userId= data.user.userId;
            this.webSocket();
          } else {
            this.$message.error("get user message error");
          }
        })
      },
      webSocket() {
        const that = this;
        window.clearInterval(that.timer);
        if (typeof (WebSocket) == 'undefined') {
          this.$notify({
            title: '提示',
            message: '当前浏览器无法接收实时报警信息，请使用谷歌浏览器！',
            type: 'warning',
            duration: 0
          });
        } else {
          var url = this.$http.adornUrl(`/websocket/account_basic_websocket/`+this.userId);
          url = url.replace("http:","ws:")
          this.socket = new WebSocket(url);
          var thatSocket = this.socket;
          this.socket.onopen = function() {
            console.log('web socket open');
            that.timer = setInterval(function(){
              thatSocket.send("get account message");
            }, 1000*30);
          };

          this.socket.onmessage = function(msg) {
            // 转换为json对象
 			if(msg.isTrusted)
            {
            	const data = msg.data;
		        if(data != "连接成功")
		        {

		          data = JSON.parse(data);
		          //充值
		          that.cz = data.chongzhi;
		            
		          //花费
		          that.hf = data.xiaofei;

		          //剩余
		          that.sy = data.shengyu;
		            //今日消费
		          that.jrxf = data.jinrixiaofei;
		        }
	    	}
          };
          this.socket.onerror = function() {
            that.$message.error("web socket error");
          };
          this.socket.onclose = function() {
            console.log('web socket close');
          }

        }
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
      }
    }
  }
</script>