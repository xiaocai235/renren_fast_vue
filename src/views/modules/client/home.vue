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
          曝光次数：
          &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
          {{ bg }}
        </el-card>
      </el-col>
      <el-col :span="6">
        <el-card shadow="always">
          点击次数：
           &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
          {{ dj }}
        </el-card>
      </el-col>
      <el-col :span="6">
        <el-card shadow="always">
          今日花费：
           &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
          {{ jrxf }}
        </el-card>
      </el-col>
    </el-row>

    <el-row :gutter="20" style="margin-top: 20px;">
      <el-col :span="12">
        <el-card style="width:100%;">
          <div id="J_chartLineBox" class="chart-box"></div>
        </el-card>
      </el-col>
      <el-col :span="12">
        <el-card style="width:100%;">
          <div id="J_chartLineBox_Y" class="chart-box"></div>
        </el-card>
      </el-col>
    </el-row>
    <el-row :gutter="20" style="margin-top: 20px;">
      <el-col :span="24">
        <el-card>
          <div id="J_chartLineBox_M" class="chart-box"></div>
        </el-card>
      </el-col>
    </el-row>

    <el-row :gutter="20" style="margin-top: 20px;">
      <el-col :span="24">
        <el-card>
          <el-table
            :data="dataList"
            style="width: 100%">
            <el-table-column
              prop="username"
              label="用户名"
              width="180">
            </el-table-column>
            <el-table-column
              prop="email"
              label="邮箱"
              width="180">
            </el-table-column>
            <el-table-column
              prop="mobile"
              label="手机号">
            </el-table-column>
            <el-table-column
              prop="companyName"
              label="公司名称">
            </el-table-column>
            <el-table-column
              prop="companyPerson"
              label="联系人">
            </el-table-column>
            <el-table-column
              prop="remark"
              label="备注">
            </el-table-column>
            <el-table-column
              prop="createTime"
              label="创建时间">
            </el-table-column>
          </el-table>
        </el-card>
      </el-col>
    </el-row>

  </div>
</template>




<script>
  import echarts from 'echarts'
  export default {
    data () {
      return {
         chartLine: null,
         dataList: [],
         sy: 0.000,
         jrxf: 0.000,
         bg: 0.000,
         dj: 0.000,
         userId: '',
         timer: null
      }
    },
    mounted () {
      this.initChartLine()
      this.initChartLineM()
      this.initChartLineY()
      this.init()
    },
    activated () {
      // 由于给echart添加了resize事件, 在组件激活时需要重新resize绘画一次, 否则出现空白bug
      if (this.chartLine) {
        this.chartLine.resize()
      }
      this.getDataList()
    },
    beforeDestroy () {
      this.socket.send('close');
      this.timer = null;
    },
    methods: {
      init () {
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

        this.$http({
          url: this.$http.adornUrl('/generator/scapital/summary2'),
          method: 'get',
          params: {}
        }).then(({data}) => {
           if (data && data.code === 0) {   
              //充值
              this.bg = data.summary.exposures;
              //花费
              this.dj = data.summary.clicks;
              //剩余
              this.sy = data.summary.shengyu;
              //今日消费
              this.jrxf = data.summary.jinrixiaofei;
           }
        })
      },
      // 折线图
      initChartLine () {
        var option = {
          'title': {
            'text': '本周数据汇总'
          },
          'tooltip': {
            'trigger': 'axis'
          },
          'legend': {
            'data': [ '点击量','曝光量', '到达量']
          },
          'grid': {
            'left': '3%',
            'right': '4%',
            'bottom': '3%',
            'containLabel': true
          },
          'toolbox': {
            'feature': {
              'saveAsImage': { }
            }
          },
          'xAxis': {
            'type': 'category',
            'boundaryGap': false,
            'data':[]
          },
          'yAxis': {
            'type': 'value'
          },
          'series':[]
        }
        for(var i=0;i<7;i++)
        {
          var item = this.getWeek(i)
          option.xAxis.data.push(item);
        };
        this.$http({
          url: this.$http.adornUrl('/generator/srealdata/chartLine'),
          method: 'get',
          params: {}
        }).then(({data}) => {
            option.series = data;

            this.chartLine = echarts.init(document.getElementById('J_chartLineBox'))
            this.chartLine.setOption(option)
            window.addEventListener('resize', () => {
              this.chartLine.resize()
            })
        })
      },
      initChartLineM() {
        var option = {
          'title': {
            'text': '本月数据汇总'
          },
          'tooltip': {
            'trigger': 'axis'
          },
          'legend': {
            'data': [ '点击量','曝光量', '到达量']
          },
          'grid': {
            'left': '3%',
            'right': '4%',
            'bottom': '3%',
            'containLabel': true
          },
          'toolbox': {
            'feature': {
              'saveAsImage': { }
            }
          },
          'xAxis': {
            'type': 'category',
            'boundaryGap': false,
            'data':[]
          },
          'yAxis': {
            'type': 'value'
          },
          'series':[]
        }
        var current = new Date();
        var year = current.getFullYear();
        var month = current.getMonth()+1;
        for(var i=1;i<=this.getCountDays();i++)
        {
            option.xAxis.data.push(year+"年"+month+"月"+i+"日");
        }

        this.$http({
          url: this.$http.adornUrl('/generator/srealdata/chartLineM'),
          method: 'get',
          params: {}
        }).then(({data}) => {
            option.series = data;

            this.chartLine = echarts.init(document.getElementById('J_chartLineBox_M'))
            this.chartLine.setOption(option)
            window.addEventListener('resize', () => {
              this.chartLine.resize()
            })
        })
      },
      initChartLineY() {
        var option = {
          'title': {
            'text': '本年度数据汇总'
          },
          'tooltip': {
            'trigger': 'axis'
          },
          'legend': {
            'data': [ '点击量','曝光量', '到达量']
          },
          'grid': {
            'left': '3%',
            'right': '4%',
            'bottom': '3%',
            'containLabel': true
          },
          'toolbox': {
            'feature': {
              'saveAsImage': { }
            }
          },
          'xAxis': {
            'type': 'category',
            'boundaryGap': false,
            'data':[]
          },
          'yAxis': {
            'type': 'value'
          },
          'series':[]
        }
        var current = new Date();
        var year = current.getFullYear();
        for(var i=1;i<=12;i++)
        {
            option.xAxis.data.push(year+"年"+i+"月");
        }
        this.$http({
          url: this.$http.adornUrl('/generator/srealdata/chartLineY'),
          method: 'get',
          params: {}
        }).then(({data}) => {
            option.series = data;
            this.chartLine = echarts.init(document.getElementById('J_chartLineBox_Y'))
            this.chartLine.setOption(option)
            window.addEventListener('resize', () => {
              this.chartLine.resize()
            })
        })
      },
      getCountDays() {
        var curDate = new Date();
        var curMonth = curDate.getMonth();
        curDate.setMonth(curMonth + 1);
        curDate.setDate(0);
        return curDate.getDate();
      },
      getWeek(i) {
        var now = new Date();
        var firstDay=new Date(now - (now .getDay() - 1 ) * 86400000);
        firstDay.setDate(firstDay.getDate() + i);
        var mon = Number(firstDay.getMonth()) + 1;
        return now.getFullYear() + "年" + mon + "月" + firstDay.getDate()+"日";
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
          var url = this.$http.adornUrl(`/websocket/home_basic_websocket/`+this.userId);
          url = url.replace("http:","ws:")
          this.socket = new WebSocket(url);
          var thatSocket = this.socket;
          this.socket.onopen = function() {
            console.log('web socket open');
            
            that.timer = setInterval(function(){
              thatSocket.send("get home message");
            }, 1000*30);
          };

          this.socket.onmessage = function(msg) {
            // 转换为json对象
            if(msg.isTrusted)
            {
              const data = msg.data;
              if(data !=  "连接成功")
              {
                data = JSON.parse(data);

                //剩余
                that.sy = data.shengyu;
                //今日消费
                that.jrxf = data.jinrixiaofei;
                // 曝光总量
                that.bg = data.exposures;
                // 点击总量
                that.dj = data.clicks;
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
      getDataList () {
        this.dataListLoading = true
        this.$http({
          url: this.$http.adornUrl('/sys/user/info'),
          method: 'get',
          params: {}
        }).then(({data}) => {
          if (data && data.code === 0) {
            this.dataList = [];
            this.dataList.push(data.user);
          } else {
            this.dataList = []
          }
          this.dataListLoading = false
        })
      }
    }
  }
</script>

<style lang="scss">
  .mod-demo-echarts {
    > .el-alert {
      margin-bottom: 10px;
    }
    > .el-row {
      margin-top: -10px;
      margin-bottom: -10px;
      .el-col {
        padding-top: 10px;
        padding-bottom: 10px;
      }
    }
    .chart-box {
      min-height: 400px;
    }
  }
</style>