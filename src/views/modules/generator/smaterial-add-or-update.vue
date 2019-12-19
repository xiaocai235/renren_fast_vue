<template>
  <el-dialog
    :title="!dataForm.id ? '素材管理' : '素材管理'"
    :close-on-click-modal="false"
    :visible.sync="visible">
    <el-form :model="dataForm" :rules="dataRule" ref="dataForm" @keyup.enter.native="dataFormSubmit()" label-width="80px">
    <el-form-item label="账户ID" prop="userId" v-if="false">
      <el-input v-model="dataForm.userId" placeholder="账户ID"></el-input>
    </el-form-item>
    <el-form-item label="账户姓名" prop="userName" v-if="false">
      <el-input v-model="dataForm.userName" placeholder="账户姓名"></el-input>
    </el-form-item>
    <el-form-item label="素材名称" prop="fileName" v-if="">
      <el-input v-model="dataForm.fileName" placeholder="素材名称"></el-input>
    </el-form-item>
    <el-form-item label="预先审核" prop="preAudit">  
      <el-select v-model="dataForm.preAudit" placeholder="预先审核"  style="width:100%;">
        <el-option v-for="item in preAudits" :key="item.dictItemValue" :label="item.dictItemKey" :value="item.dictItemValue"></el-option>
      </el-select>
    </el-form-item>
    <el-form-item label="审核状态" prop="audit">
      <el-select v-model="dataForm.audit" placeholder="审核状态"  style="width:100%;">
        <el-option v-for="item in audits" :key="item.dictItemValue" :label="item.dictItemKey" :value="item.dictItemValue"></el-option>
      </el-select>
    </el-form-item>
    <el-form-item label="客户端" prop="clientType">
      <el-select v-model="dataForm.clientType" placeholder="客户端类型"  style="width:100%;">
        <el-option v-for="item in clients" :key="item.dictItemValue" :label="item.dictItemKey" :value="item.dictItemValue"></el-option>
      </el-select>
    </el-form-item>
    <el-form-item label="文件状态" prop="fileState">
      <el-select v-model="dataForm.fileState" placeholder="文件状态"  style="width:100%;">
        <el-option v-for="item in fileStates" :key="item.dictItemValue" :label="item.dictItemKey" :value="item.dictItemValue"></el-option>
      </el-select>
    </el-form-item>
    <el-form-item label="推广链接" prop="extensionUrl">
      <el-input v-model="dataForm.extensionUrl" placeholder="推广链接"></el-input>
    </el-form-item>
    <el-form-item label="备注" prop="remark">
      <el-input v-model="dataForm.remark" placeholder="备注"></el-input>
    </el-form-item>
    <el-form-item>
       <el-upload :action="uploadurl"
          ref='upload'
          list-type="picture-card"
          :file-list="fileList"
          :on-preview="handlePictureCardPreview"
          :on-remove="handleRemove"
          :on-success="handleSuccess">
          <i class="el-icon-plus"></i>
        </el-upload>
        <el-dialog :visible.sync="dialogVisible" size="tiny">
          <img width="100%" :src="dialogImageUrl" alt="">
        </el-dialog>
    </el-form-item>
    </el-form>
    <span slot="footer" class="dialog-footer">
      <el-button @click="onClose()">取消</el-button>
      <el-button type="primary" @click="dataFormSubmit()">确定</el-button>
    </span>
  </el-dialog>
</template>

<script>
  export default {
    data () {
      return {
        fileList: [],
        visible: false,
        dialogImageUrl: '',
        uploadurl: '',
        dialogVisible: false,
        preAudits: [],
        audits: [],
        fileStates: [],
        clients: [],
        dataForm: {
          id: 0,
          userId: '',
          userName: '',
          fileName: '',
          fileUrl: '',
          preAudit: '',
          audit: '',
          clientType: '',
          fileState: '',
          remark: '',
          createId: '',
          createName: '',
          createTime: '',
          extensionUrl: ''
        },
        dataRule: {
          userId: [
            { required: true, message: '账户ID不能为空', trigger: 'blur' }
          ],
          userName: [
            { required: true, message: '账户姓名不能为空', trigger: 'blur' }
          ],
          fileName: [
            { required: true, message: '素材名称不能为空', trigger: 'blur' }
          ],
          fileUrl: [
            { required: true, message: '请选择上传文件', trigger: 'blur' }
          ],
          preAudit: [
            { required: true, message: '预先审核不能为空', trigger: 'blur' }
          ],
          audit: [
            { required: true, message: '审核状态不能为空', trigger: 'blur' }
          ],
          clientType: [
            { required: true, message: '客户端类型不能为空', trigger: 'blur' }
          ],
          fileState: [
            { required: true, message: '文件状态不能为空', trigger: 'blur' }
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
          extensionUrl: [
            { required: true, message: '推广链接不能为空', trigger: 'blur' }
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
        this.uploadurl = this.$http.adornUrl(`/sys/oss/upload?token=${this.$cookie.get('token')}`)
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
                url: this.$http.adornUrl(`/generator/smaterial/info/${this.dataForm.id}`),
                method: 'get',
                params: this.$http.adornParams()
              }).then(({data}) => {
                if (data && data.code === 0) {
                  this.dataForm.userId = data.sMaterial.userId
                  this.dataForm.userName = data.sMaterial.userName
                  this.dataForm.fileName = data.sMaterial.fileName
                  this.dataForm.fileUrl = data.sMaterial.fileUrl
                  this.dataForm.preAudit = data.sMaterial.preAudit+""
                  this.dataForm.audit = data.sMaterial.audit+""
                  this.dataForm.clientType = data.sMaterial.clientType+""
                  this.dataForm.fileState = data.sMaterial.fileState+""
                  this.dataForm.remark = data.sMaterial.remark
                  this.dataForm.createId = data.sMaterial.createId
                  this.dataForm.createName = data.sMaterial.createName
                  this.dataForm.createTime = data.sMaterial.createTime
                  this.dataForm.extensionUrl = data.sMaterial.extensionUrl;

                  this.listLoading = true;
                  data.sMaterial.urls.forEach(url => {
                    var urlObj = {};
                    urlObj.url = url;
                    this.fileList.push(urlObj);
                  })
                  this.listLoading = false;
                }
              })
            }
          })
        }
      },
      getDictType () {
        // 预先审核状态
        this.$http({
          url: this.$http.adornUrl(`/generator/sysdictitem/name`),
          method: 'get',
          params: this.$http.adornParams({
            'dictName': 'PRE_AUDIT',
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
      handleRemove(file, fileList) {
        if(this.dataForm.fileUrl.indexOf("-item-"))
        {
          var updateUrl = this.dataForm.fileUrl.split("-item-");
          updateUrl.forEach((item,index) => {
           
            // 刚添加的 图片是从缓存中读取blob，因此 需要转换为 图片的网络地址，然后在进行维护 fileUrl 字符串，以此来完成更改操作
            if(file.url.indexOf("localhost") > -1)
            {
              if(item.indexOf(file.realUrl) > -1)
              {
                updateUrl.splice(index,1);
              }
            }
            else
            {
              if(item.indexOf(file.url.split("?")[0]) > -1)
              {
                updateUrl.splice(index,1);
              }
            }
            
          })
          if(updateUrl.length > 1)
          {
            this.dataForm.fileUrl = updateUrl.join("-item-");
          }
          else
          {
            this.dataForm.fileUrl = updateUrl.join("");
          }
        }
        else
        {
          this.dataForm.fileUrl = "";
        }
      },
      handlePictureCardPreview(file) {
        this.dialogImageUrl = file.url;
        this.dialogVisible = true;
      },
      handleSuccess (data,file)
      {
        if(data.code == 0)
        {
          if(this.dataForm.fileUrl)
          {
            this.dataForm.fileUrl += "-item-"+data.id + "-id-" +data.url;
          }
          else
          {
            this.dataForm.fileUrl = data.id + "-id-" +data.url;         
          }
          file.realUrl =  data.id + "-id-" +data.url;
        }
      },
      onClose() {
        this.visible = false
        this.$emit('refreshDataList')
        this.$refs.upload.clearFiles();
        this.dataForm.fileUrl = '';
        this.dataForm.urls = [];
        this.fileList = [];
      },
      // 表单提交
      dataFormSubmit () {
        this.$refs['dataForm'].validate((valid) => {
          if (valid) {
            if(typeof this.dataForm.fileUrl == "undefined" || this.dataForm.fileUrl == null || this.dataForm.fileUrl == "")
            {
              this.$message.error("请选择上传文件");
              return false;
            }
            this.$http({
              url: this.$http.adornUrl(`/generator/smaterial/${!this.dataForm.id ? 'save' : 'update'}`),
              method: 'post',
              data: this.$http.adornData({
                'id': this.dataForm.id || undefined,
                'userId': this.dataForm.userId,
                'userName': this.dataForm.userName,
                'fileName': this.dataForm.fileName,

                'fileUrl': this.dataForm.fileUrl,
                'preAudit': this.dataForm.preAudit,

                'audit': this.dataForm.audit,
                'clientType': this.dataForm.clientType,
                'fileState': this.dataForm.fileState,
                'remark': this.dataForm.remark,
                'createId': this.dataForm.createId,

                'createName': this.dataForm.createName,
                'createTime': this.dataForm.createTime,
                'extensionUrl': this.dataForm.extensionUrl,
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
                    this.$refs.upload.clearFiles();
                    this.dataForm.fileUrl = '';
                    this.dataForm.urls = [];
                    this.fileList = [];
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
<style>
  
</style>