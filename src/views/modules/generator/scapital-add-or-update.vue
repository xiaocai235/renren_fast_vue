<template>
  <el-dialog
    :title="!dataForm.id ? '资金管理' : '资金管理'"
    :close-on-click-modal="false"
    :visible.sync="visible">
    <el-form :model="dataForm" :rules="dataRule" ref="dataForm" @keyup.enter.native="dataFormSubmit()" label-width="80px">
    <el-form-item label="账户ID" prop="userId" v-if="1==2">
      <el-input v-model="dataForm.userId" placeholder="账户ID"></el-input>
    </el-form-item>
     <el-form-item label="账户姓名" prop="userId" v-if="1==2">
      <el-input v-model="dataForm.userName" placeholder="账户姓名"></el-input>
    </el-form-item>
    <el-form-item label="金额" prop="money">
      <el-input v-model="dataForm.money" placeholder="金额"></el-input>
    </el-form-item>
    <el-form-item label="资金类型" prop="capitalType">
      <el-select v-model="dataForm.capitalType" placeholder="资金类型"  style="width:100%;">
        <el-option v-for="item in capitals" :key="item.dictItemValue" :label="item.dictItemKey" :value="item.dictItemValue"></el-option>
      </el-select>
    </el-form-item>
    <el-form-item label="客户端" prop="clientType">
      <el-select v-model="dataForm.clientType" placeholder="客户端类型" style="width:100%;">
        <el-option v-for="item in clients" :key="item.dictItemValue" :label="item.dictItemKey" :value="item.dictItemValue"></el-option>
      </el-select>
    </el-form-item>
    <el-form-item label="备注" prop="remark">
      <el-input type="textarea" :rows="7" v-model="dataForm.remark" placeholder="备注"></el-input>
    </el-form-item>
    </el-form>
    <span slot="footer" class="dialog-footer">
      <el-button @click="visible = false">取消</el-button>
      <el-button type="primary" @click="dataFormSubmit()">确定</el-button>
    </span>
  </el-dialog>
</template>

<script>
  export default {
    data () {
      return {
        visible: false,
        dataForm: {
          id: 0,
          userId: '',
          userName: '',
          money: '',
          capitalType: '',
          clientType: '',
          remark: '',
          createId: '',
          createName: '',
          createTime: ''
        },
        capitals: [],
        clients: [],
        dataRule: {
          userId: [
            { required: true, message: '账户ID不能为空', trigger: 'blur' }
          ],
          userName: [
            { required: true, message: '账户姓名不能为空', trigger: 'blur' }
          ],
          money: [
            { required: true, message: '金额不能为空', trigger: 'blur' }
          ],
          capitalType: [
            { required: true, message: '资金类型不能为空', trigger: 'blur' }
          ],
          clientType: [
            { required: true, message: '客户端类型不能为空', trigger: 'blur' }
          ],
          remark: [
            { required: true, message: '备注不能为空', trigger: 'blur' }
          ],
          createId: [
            { required: true, message: '创建人ID不能为空', trigger: 'blur' }
          ],
          createName: [
            { required: true, message: '创建人姓名不能为空', trigger: 'blur' }
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
        // 增加
        if(id == null)
        {
          this.dataForm.userId = userId || 0
          this.dataForm.userName = userName || ''

          this.visible = true
            this.$nextTick(() => {
            this.$refs['dataForm'].resetFields()
          })
        }
        // 修改
        else
        {
          this.dataForm.id = id || 0
          this.visible = true
          this.$nextTick(() => {
          this.$refs['dataForm'].resetFields()
          if (this.dataForm.id) {
            this.$http({
              url: this.$http.adornUrl(`/generator/scapital/info/${this.dataForm.id}`),
              method: 'get',
              params: this.$http.adornParams()
            }).then(({data}) => {
              if (data && data.code === 0) {
                this.dataForm.userId = data.sCapital.userId
                this.dataForm.userName = data.sCapital.userName
                this.dataForm.money = data.sCapital.money
                this.dataForm.capitalType = data.sCapital.capitalType+""
                this.dataForm.clientType = data.sCapital.clientType+""
                this.dataForm.remark = data.sCapital.remark
                this.dataForm.createId = data.sCapital.createId
                this.dataForm.createName = data.sCapital.createName
                this.dataForm.createTime = data.sCapital.createTime
                }
              })
            }
          })
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
      // 表单提交
      dataFormSubmit () {
        this.$refs['dataForm'].validate((valid) => {
          if (valid) {
            this.$http({
              url: this.$http.adornUrl(`/generator/scapital/${!this.dataForm.id ? 'save' : 'update'}`),
              method: 'post',
              data: this.$http.adornData({
                'id': this.dataForm.id || undefined,
                'userId': this.dataForm.userId,
                'userName': this.dataForm.userName,
                'money': this.dataForm.money,
                'capitalType': this.dataForm.capitalType,
                'clientType': this.dataForm.clientType,
                'remark': this.dataForm.remark,
                'createId': this.dataForm.createId,
                'createName': this.dataForm.createName,
                'createTime': this.dataForm.createTime
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
