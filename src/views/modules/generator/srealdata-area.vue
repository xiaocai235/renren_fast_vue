<template>
    <div class="mod-demo-echarts" v-loading="dataListLoading">
        <el-row :gutter="20">
          <el-col :span="12">
            <el-card>
              <div id="J_click_chartMapBox" class="chart-box" style="height: 350px;"></div>
            </el-card>
          </el-col>
          <el-col :span="12">
            <el-card>
              <div id="J_expores_chartMapBox" class="chart-box" style="height: 350px;"></div>
            </el-card>
          </el-col>
          <el-col :span="24" style="margin-top:20px;">
            <el-card>
              <div id="J_arrivers_chartMapBox" class="chart-box" style="height: 350px;"></div>
            </el-card>
          </el-col>
        </el-row>
    </div>
</template>
<script>

export default {
    data() {
        return {
            realDataId: '',
            clicksList:[],
            arriversList: [],
            exposuresList: [],
            clickChartMap: null,
            dataListLoading: false,
            exposureChartMap: null,
            arriverChartMap: null
        };
    },
    mounted () {
        this.getParam()
        this.initChartMap()
    },
    activated () {
      this.initChartMap()
      if (this.clickChartMap) {
        this.clickChartMap.resize()
      }
      if(this.exposureChartMap)
      {
        this.exposureChartMap.resize()
      }
      if(this.arriverChartMap)
      {
        this.arriverChartMap.resize()
      }
      
    },
    methods:{
        getParam()
        {
            if(this.$route.params.realDataId) {
               this.realDataId = this.$route.params.realDataId;
            }
            else
            {
                this.$message.error("无法从路由中获取realDataId");
                return false;
            }
        },
        initChartMap () {
            this.dataListLoading = true
            var option = {
                title: {
                    text: ''
                },
                tooltip: {
                    trigger: 'item'
                },
                visualMap: {//颜色的设置  dataRange
                    x: 'left',
                    y: 'center',
                    splitList: [],
                    text:['高','低']
                },
                toolbox: {//工具栏
                    show: true,
                    orient : 'vertical',
                    x: 'right',
                    y: 'center',
                    feature : {
                        mark : {show: true},
                        dataView : {show: true, readOnly: false},
                        restore : {show: true},
                        saveAsImage : {show: true}
                    }
                },
                roamController: {
                    show: true,
                    x: 'right',
                    mapTypeControl: {
                        'china': true
                    }
                },
                series: [
                   {
                        name: '',
                        type: 'map',
                        mapType: 'china',
                        roam: true,  
                        itemStyle:{
                            normal:{
                                label:{
                                    show:true,
                                    textStyle: {
                                        color: "rgb(0, 0, 0)"
                                    }
                                }
                            },
                            emphasis:{
                                label:{show:true}
                            }
                        },
                        top:"1%",
                        data:[]
                    }
                ]
            }
            this.$http({
              url: this.$http.adornUrl('/generator/srealdata/area'),
              method: 'get',
              params: this.$http.adornParams({
                'realDataId': this.realDataId
              })
            }).then(({data}) => {
              if (data && data.code === 0) {
                var jsonArea = JSON.parse(data.area);
                //区域点击数量统计
                var clickColor = [
                        {end: 1000, color: '#E0FFFF'},
                        {start: 1000, end: 3000, color: '#BBE7F9'},
                        {start: 3000, end: 6000, color: '#95CFF4'},
                        {start: 6000, end: 10000, color: '#70B7EE'},
                        {start: 10000, end: 13000, color: '#4B9EE8'},
                        {start: 13000, end: 20000, color: '#2586E3'},
                        {start: 20000, color: '#0048DE'}
                    ];

                option.title.text = '区域点击数量统计';
                option.series[0].data = jsonArea.clicks;
                option.series[0].name = "点击数量";
                option.visualMap.splitList = clickColor;
                this.clickChartMap = echarts.init(document.getElementById('J_click_chartMapBox'))
                this.clickChartMap.setOption(option)
                this.clickChartMap.resize()

                //区域曝光数量统计
                var exposureColor = [
                        {end: 3000, color: '#E0FFFF'},
                        {start: 3000, end: 6000, color: '#BBE7F9'},
                        {start: 6000, end: 9000, color: '#95CFF4'},
                        {start: 9000, end: 12000, color: '#70B7EE'},
                        {start: 12000, end: 20000, color: '#4B9EE8'},
                        {start: 20000, end: 30000, color: '#2586E3'},
                        {start: 30000, color: '#0048DE'}
                    ];
                option.title.text = '区域曝光数量统计';
                option.series[0].name = "曝光数量";
                option.series[0].data = jsonArea.exposures;
                option.visualMap.splitList = exposureColor;
                this.exposureChartMap = echarts.init(document.getElementById('J_expores_chartMapBox'))
                this.exposureChartMap.setOption(option)
                this.exposureChartMap.resize()

                //区域到达数量统计
                var arriverColor = [
                        {end: 1000, color: '#E0FFFF'},
                        {start: 1000, end: 3000, color: '#BBE7F9'},
                        {start: 3000, end: 6000, color: '#95CFF4'},
                        {start: 6000, end: 10000, color: '#70B7EE'},
                        {start: 10000, end: 13000, color: '#4B9EE8'},
                        {start: 13000, end: 20000, color: '#2586E3'},
                        {start: 20000, color: '#0048DE'}
                    ];
                option.title.text = '区域到达数量统计';
                option.series[0].name = "到达数量";
                option.series[0].data = jsonArea.arrivers;
                option.visualMap.splitList = arriverColor;
                this.arriverChartMap = echarts.init(document.getElementById('J_arrivers_chartMapBox'))
                this.arriverChartMap.setOption(option)
                this.arriverChartMap.resize()
              } else {
                this.$message.error("获取数据信息失败，请联系管理员");
              }
              this.dataListLoading = false
            })
        }
    }
}
</script>
<style lang="scss" scoped>
    .mapBox {
        width: 100%;
        height: 100%;   
    }
    .map-title{
        position: absolute;
        padding-left:10px;
        padding-top:5px;
    }
</style>