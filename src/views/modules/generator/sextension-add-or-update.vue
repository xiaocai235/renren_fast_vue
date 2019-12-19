<template>
  <el-dialog
    :title="!dataForm.id ? '推广管理' : '推广管理'"
    :close-on-click-modal="false"
    :visible.sync="visible">
    <el-form :model="dataForm" :rules="dataRule" ref="dataForm" @keyup.enter.native="dataFormSubmit()" label-width="80px">
    <el-form-item label="账户ID" prop="userId" v-if="1==2">
      <el-input v-model="dataForm.userId" placeholder="账户ID"></el-input>
    </el-form-item>
    <el-form-item label="账户名称" prop="userName" v-if="1==2">
      <el-input v-model="dataForm.userName" placeholder="账户名称"></el-input>
    </el-form-item>
    <el-form-item label="推广名称" prop="extensionName">
      <el-input v-model="dataForm.extensionName" placeholder="推广页名称"></el-input>
    </el-form-item>
    <el-form-item label="推广URL" prop="extensionUrl">
      <el-input v-model="dataForm.extensionUrl" placeholder="推广页面URL"></el-input>
    </el-form-item>
    <el-form-item label="推广类型" prop="extensionType">
      <el-select v-model="dataForm.extensionType" placeholder="推广页面类型"  style="width:100%;">
        <el-option v-for="item in extensions" :key="item.dictItemValue" :label="item.dictItemKey" :value="item.dictItemValue"></el-option>
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
        extensions:[],
        dataForm: {
          id: 0,
          userId: '',
          userName: '',
          extensionName: '',
          extensionUrl: '',
          extensionType: '',
          remark: '',
          createId: '',
          createName: '',
          createTime: ''
        },
        dataRule: {
          userId: [
            { required: true, message: '账户ID不能为空', trigger: 'blur' }
          ],
          userName: [
            { required: true, message: '账户名称不能为空', trigger: 'blur' }
          ],
          extensionName: [
            { required: true, message: '推广页名称不能为空', trigger: 'blur' }
          ],
          extensionUrl: [
            { required: true, message: '推广页面URL不能为空', trigger: 'blur' }
          ],
          extensionType: [
            { required: true, message: '推广页面类型不能为空', trigger: 'blur' }
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
      init(id,userId,userName) {
        // 新增
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
              url: this.$http.adornUrl(`/generator/sextension/info/${this.dataForm.id}`),
              method: 'get',
              params: this.$http.adornParams()
            }).then(({data}) => {
              if (data && data.code === 0) {
                console.log(data);
                this.dataForm.userId = data.sExtension.userId
                this.dataForm.userName = data.sExtension.userName
                this.dataForm.extensionName = data.sExtension.extensionName
                this.dataForm.extensionUrl = data.sExtension.extensionUrl
                this.dataForm.extensionType = data.sExtension.extensionType+"";

                this.dataForm.remark = data.sExtension.remark
                this.dataForm.createId = data.sExtension.createId
                this.dataForm.createName = data.sExtension.createName
                this.dataForm.createTime = data.sExtension.createTime
              }
            })
          }
        })
        }
      },

      getDictType () {
        // 推广页面类型
        this.$http({
          url: this.$http.adornUrl(`/generator/sysdictitem/name`),
          method: 'get',
          params: this.$http.adornParams({
            'dictName': 'EXTENSION_TYPE',
          })
        }).then(({data}) => {
          if (data && data.code === 0) {
           this.extensions = data.list;
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
              url: this.$http.adornUrl(`/generator/sextension/${!this.dataForm.id ? 'save' : 'update'}`),
              method: 'post',
              data: this.$http.adornData({
                'id': this.dataForm.id || undefined,
                'userId': this.dataForm.userId,
                'userName': this.dataForm.userName,
                'extensionName': this.dataForm.extensionName,
                'extensionUrl': this.dataForm.extensionUrl,
                'extensionType': this.dataForm.extensionType+"",
                'remark': this.dataForm.remark
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
