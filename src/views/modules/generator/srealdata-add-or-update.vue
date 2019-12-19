<template>
  <el-dialog
    :title="!dataForm.id ? '新增' : '修改'"
    :close-on-click-modal="false"
    :visible.sync="visible">
    <el-form :model="dataForm" :rules="dataRule" ref="dataForm" @keyup.enter.native="dataFormSubmit()" label-width="80px">
    <el-form-item label="账户ID" prop="userId" v-if="false">
      <el-input v-model="dataForm.userId" placeholder="账户ID"></el-input>
    </el-form-item>
    <el-form-item label="账户名称" prop="userName" v-if="false">
      <el-input v-model="dataForm.userName" placeholder="账户名称"></el-input>
    </el-form-item>
    <el-form-item label="活动名称" prop="jobName">
      <el-input v-model="dataForm.jobName" placeholder="活动名称"></el-input>
    </el-form-item>
    <el-form-item label="开始时间" prop="startTime">
      <el-date-picker v-model="dataForm.startTime"
            type="datetime" placeholder="开始时间" style="width: 100%" default-time="00:00:00"></el-date-picker>
    </el-form-item>
    <el-form-item label="结束时间" prop="endTime">
       <el-date-picker v-model="dataForm.endTime"
            type="datetime" placeholder="结束时间" style="width: 100%" default-time="00:00:00"></el-date-picker>
    </el-form-item>
     <el-form-item label="客户端" prop="clientType">
      <el-select v-model="dataForm.clientType" placeholder="客户端类型" style="width:100%;">
        <el-option v-for="item in clients" :key="item.dictItemValue" :label="item.dictItemKey" :value="item.dictItemValue"></el-option>
      </el-select>
    </el-form-item>
    <el-form-item label="CPC" prop="cpc">
      <el-input-number v-model="dataForm.cpc" :precision="3" :step="0.001" :max="1000" style="width: 100%;"></el-input-number>
    </el-form-item>
    <el-form-item label="CPM" prop="cpm">
       <el-input-number v-model="dataForm.cpm" :precision="3" :step="0.001" :max="1000" style="width: 100%;"></el-input-number>
    </el-form-item>
    <el-form-item label="消耗金额" prop="consumeMoney">
      <el-input-number v-model="dataForm.consumeMoney" :precision="3" :step="0.001" :max="999999" style="width: 100%;"></el-input-number>
    </el-form-item>
    <el-form-item label="到达率" prop="arrivesRate">
      <el-input-number v-model="dataForm.arrivesRate" :precision="3" :step="0.001" :max="1" style="width: 100%;"></el-input-number>
    </el-form-item>
    </el-form>
    <span slot="footer" class="dialog-footer">
      <el-button @click="visible = false">取消</el-button>
      <el-button type="primary" @click="dataFormSubmit()">动态</el-button>
      <el-button type="warning" @click="staticdataFormSubmit()">静态</el-button>
    </span>
  </el-dialog>
</template>

<script>
  export default {
    data () {
      return {
        visible: false,
        clients: [],
        dataForm: {
          id: 0,
          userId: '',
          userName: '',
          jobName: '',
          startTime: '',
          endTime: '',
          stopTime: '',
          cpc: '',
          cpm: '',
          consumeMoney: '',
          clicks: '',
          clicksRate: '',
          exposures: '',
          arrives: '',
          arrivesRate: '',
          oneUpdateClicks: '',
          oneUpdateExposures: '',
          oneUpdateConsumeMoney: '',
          oneUpdateArrives: '',
          createYear: '',
          createMonth: '',
          createDay: '',
          createId: '',
          createName: '',
          createTime: '',
          clientType: ''
        },
        dataRule: {
          userId: [
            { required: true, message: '账户ID不能为空', trigger: 'blur' }
          ],
          userName: [
            { required: true, message: '账户名称不能为空', trigger: 'blur' }
          ],
          jobName: [
            { required: true, message: '活动名称不能为空', trigger: 'blur' }
          ],
          startTime: [
            { required: true, message: '开始时间不能为空', trigger: 'blur' }
          ],
          endTime: [
            { required: true, message: '结束时间不能为空', trigger: 'blur' }
          ],
          stopTime: [
            { required: true, message: '停止时间不能为空', trigger: 'blur' }
          ],
          cpc: [
            { required: true, message: 'CPC不能为空', trigger: 'blur' }
          ],
          cpm: [
            { required: true, message: 'CPM不能为空', trigger: 'blur' }
          ],
          consumeMoney: [
            { required: true, message: '消耗金额不能为空', trigger: 'blur' }
          ],
          clicks: [
            { required: true, message: '点击次数不能为空', trigger: 'blur' }
          ],
          clicksRate: [
            { required: true, message: '点击率不能为空', trigger: 'blur' }
          ],
          exposures: [
            { required: true, message: '曝光次数不能为空', trigger: 'blur' }
          ],
          arrives: [
            { required: true, message: '到达次数不能为空', trigger: 'blur' }
          ],
          arrivesRate: [
            { required: true, message: '到达率不能为空', trigger: 'blur' }
          ],
          oneUpdateClicks: [
            { required: true, message: '每次刷新点击次数不能为空', trigger: 'blur' }
          ],
          oneUpdateExposures: [
            { required: true, message: '每次刷新曝光次数不能为空', trigger: 'blur' }
          ],
          oneUpdateConsumeMoney: [
            { required: true, message: '每次刷新花费金额不能为空', trigger: 'blur' }
          ],
          oneUpdateArrives: [
            { required: true, message: '每次刷新到达次数不能为空', trigger: 'blur' }
          ],
          createYear: [
            { required: true, message: '创建年份不能为空', trigger: 'blur' }
          ],
          createMonth: [
            { required: true, message: '创建月份不能为空', trigger: 'blur' }
          ],
          createDay: [
            { required: true, message: '创建日期不能为空', trigger: 'blur' }
          ],
          createId: [
            { required: true, message: '创建人ID不能为空', trigger: 'blur' }
          ],
          createName: [
            { required: true, message: '创建人名称不能为空', trigger: 'blur' }
          ],
          createTime: [
            { required: true, message: '创建时间不能为空', trigger: 'blur' }
          ]
        }
      }
    },
    mounted () {
      this.getDictType()
    },
    methods: {
      init (id,userId,userName) {
        this.dataForm.id = id || 0
        this.dataForm.userId = userId;
        this.dataForm.userName = userName;
        this.visible = true
        this.$nextTick(() => {
          this.$refs['dataForm'].resetFields()
          if (this.dataForm.id) {
            this.$http({
              url: this.$http.adornUrl(`/generator/srealdata/info/${this.dataForm.id}`),
              method: 'get',
              params: this.$http.adornParams()
            }).then(({data}) => {
              if (data && data.code === 0) {
                this.dataForm.userId = data.srealdata.userId
                this.dataForm.userName = data.srealdata.userName
                this.dataForm.jobName = data.srealdata.jobName
                this.dataForm.startTime = data.srealdata.startTime
                this.dataForm.endTime = data.srealdata.endTime
                this.dataForm.stopTime = data.srealdata.stopTime
                this.dataForm.cpc = data.srealdata.cpc
                this.dataForm.cpm = data.srealdata.cpm
                this.dataForm.consumeMoney = data.srealdata.consumeMoney
                this.dataForm.clicks = data.srealdata.clicks
                this.dataForm.clicksRate = data.srealdata.clicksRate
                this.dataForm.exposures = data.srealdata.exposures
                this.dataForm.arrives = data.srealdata.arrives
                this.dataForm.arrivesRate = data.srealdata.arrivesRate
                this.dataForm.oneUpdateClicks = data.srealdata.oneUpdateClicks
                this.dataForm.oneUpdateExposures = data.srealdata.oneUpdateExposures
                this.dataForm.oneUpdateConsumeMoney = data.srealdata.oneUpdateConsumeMoney
                this.dataForm.oneUpdateArrives = data.srealdata.oneUpdateArrives
                this.dataForm.createYear = data.srealdata.createYear
                this.dataForm.createMonth = data.srealdata.createMonth
                this.dataForm.createDay = data.srealdata.createDay
                this.dataForm.createId = data.srealdata.createId
                this.dataForm.createName = data.srealdata.createName
                this.dataForm.createTime = data.srealdata.createTime
              }
            })
          }
        })
      },
      formatDate(str) {
        var date = new Date(str);
        var myyear = date.getFullYear();
        var mymonth = date.getMonth() + 1;
        var myweekday = date.getDate();
        var myhours = date.getHours();
        var mymin = date.getMinutes();
        var mysec = date.getSeconds();

     
        if (mymonth < 10) {
            mymonth = "0" + mymonth;
        }
        if (myweekday < 10) {
            myweekday = "0" + myweekday;
        }

        if (myhours < 10) {
            myhours = "0" + myhours;
        }
        if (mymin < 10) {
            mymin = "0" + mymin;
        }
        if (mysec < 10) {
            mysec = "0" + mysec;
        }
        return (myyear + "-" + mymonth + "-" + myweekday+" "+myhours+":"+mymin+":"+mysec);
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
            this.dataList = []
          }
        })
      },
       // 静态表单提交
      staticdataFormSubmit () {
        this.$refs['dataForm'].validate((valid) => {
          if (valid) {
            this.$http({
              url: this.$http.adornUrl(`/generator/srealdata/static`),
              method: 'post',
              data: this.$http.adornData({
                'id': this.dataForm.id || undefined,
                'userId': this.dataForm.userId,
                'userName': this.dataForm.userName,
                'jobName': this.dataForm.jobName,
                'startTime': this.formatDate(this.dataForm.startTime),
                'endTime': this.formatDate(this.dataForm.endTime),
                'cpc': this.dataForm.cpc,
                'cpm': this.dataForm.cpm,
                'consumeMoney': this.dataForm.consumeMoney,
                'clicks': this.dataForm.clicks,
                'clicksRate': this.dataForm.clicksRate,
                'exposures': this.dataForm.exposures,
                'arrives': this.dataForm.arrives,
                'arrivesRate': this.dataForm.arrivesRate,
                'oneUpdateClicks': this.dataForm.oneUpdateClicks,
                'oneUpdateExposures': this.dataForm.oneUpdateExposures,
                'oneUpdateConsumeMoney': this.dataForm.oneUpdateConsumeMoney,
                'oneUpdateArrives': this.dataForm.oneUpdateArrives,
                'createYear': this.dataForm.createYear,
                'createMonth': this.dataForm.createMonth,
                'createDay': this.dataForm.createDay,
                'createId': this.dataForm.createId,
                'createName': this.dataForm.createName,
                'createTime': this.dataForm.createTime,
                'clientType': this.dataForm.clientType
              })
            }).then(({data}) => {
              if (data && data.code === 0) {
                this.$message({
                  message: '操作成功',
                  type: 'success',
                  duration: 1500,
                  onClose: () => {
                    this.visible = false
                    this.$emit('refreshDataList')
                  }
                })
              } else {
                this.$message.error(data.msg)
              }
            })
          }
        })
      },
      // 动态表单提交
      dataFormSubmit () {
        this.$refs['dataForm'].validate((valid) => {
          if (valid) {
            this.$http({
              url: this.$http.adornUrl(`/generator/srealdata/${!this.dataForm.id ? 'save' : 'update'}`),
              method: 'post',
              data: this.$http.adornData({
                'id': this.dataForm.id || undefined,
                'userId': this.dataForm.userId,
                'userName': this.dataForm.userName,
                'jobName': this.dataForm.jobName,
                'startTime': this.formatDate(this.dataForm.startTime),
                'endTime': this.formatDate(this.dataForm.endTime),
                'cpc': this.dataForm.cpc,
                'cpm': this.dataForm.cpm,
                'consumeMoney': this.dataForm.consumeMoney,
                'clicks': this.dataForm.clicks,
                'clicksRate': this.dataForm.clicksRate,
                'exposures': this.dataForm.exposures,
                'arrives': this.dataForm.arrives,
                'arrivesRate': this.dataForm.arrivesRate,
                'oneUpdateClicks': this.dataForm.oneUpdateClicks,
                'oneUpdateExposures': this.dataForm.oneUpdateExposures,
                'oneUpdateConsumeMoney': this.dataForm.oneUpdateConsumeMoney,
                'oneUpdateArrives': this.dataForm.oneUpdateArrives,
                'createYear': this.dataForm.createYear,
                'createMonth': this.dataForm.createMonth,
                'createDay': this.dataForm.createDay,
                'createId': this.dataForm.createId,
                'createName': this.dataForm.createName,
                'createTime': this.dataForm.createTime,
                'clientType': this.dataForm.clientType
              })
            }).then(({data}) => {
              if (data && data.code === 0) {
                this.$message({
                  message: '操作成功',
                  type: 'success',
                  duration: 1500,
                  onClose: () => {
                    this.visible = false
                    this.$emit('refreshDataList')
                  }
                })
              } else {
                this.$message.error(data.msg)
              }
            })
          }
        })
      }
    }
  }
</script>
