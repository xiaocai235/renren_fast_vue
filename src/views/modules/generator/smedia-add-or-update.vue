<template>
  <el-dialog
    :title="!dataForm.id ? '新增' : '修改'"
    :close-on-click-modal="false"
    :visible.sync="visible">
    <el-form :model="dataForm" :rules="dataRule" ref="dataForm" @keyup.enter.native="dataFormSubmit()" label-width="80px">
   
    <el-form-item label="客户端" prop="clientType">
      <el-select  v-model="dataForm.clientType" placeholder="客户端" style="width:100%;">
        <el-option v-for="item in clients" :key="item.dictItemValue" :label="item.dictItemKey" :value="item.dictItemValue"></el-option>
      </el-select>
    </el-form-item>
    <el-form-item label="广告平台" prop="advertisingPlatform">
      <el-input v-model="dataForm.advertisingPlatform" placeholder="广告平台"></el-input>
    </el-form-item>
    <el-form-item label="媒体类型" prop="mediaType">
      <el-input v-model="dataForm.mediaType" placeholder="媒体类型"></el-input>
    </el-form-item>
    <el-form-item label="媒体名称" prop="mediaName">
      <el-input v-model="dataForm.mediaName" placeholder="媒体名称"></el-input>
    </el-form-item>
    <el-form-item label="广告位" prop="advertisingSpace">
      <el-input v-model="dataForm.advertisingSpace" placeholder="广告位"></el-input>
    </el-form-item>
    <el-form-item label="尺寸" prop="advertisingSize">
      <el-input v-model="dataForm.advertisingSize" placeholder="广告位"></el-input>
    </el-form-item>
    <el-form-item label="尺寸要求" prop="sizeRequirement">
      <el-input v-model="dataForm.sizeRequirement" placeholder="尺寸要求"></el-input>
    </el-form-item>
    <el-form-item label="流量类型" prop="trafficType">
      <el-select  v-model="dataForm.trafficType" placeholder="流量类型" style="width:100%;">
        <el-option v-for="item in traffics" :key="item.dictItemValue" :label="item.dictItemKey" :value="item.dictItemValue"></el-option>
      </el-select>
    </el-form-item>
    <el-form-item label="广告位" prop="advertisingSpaceType">
      <el-select  v-model="dataForm.advertisingSpaceType" placeholder="广告位类型" style="width:100%;">
        <el-option v-for="item in spaces" :key="item.dictItemValue" :label="item.dictItemKey" :value="item.dictItemValue"></el-option>
      </el-select>
    </el-form-item>
    <el-form-item label="日点击量" prop="volumeOfExhibition">
      <el-input v-model="dataForm.volumeOfExhibition" placeholder="日点击量"></el-input>
    </el-form-item>
    <el-form-item label="重定向" prop="redirectType">
      <el-select  v-model="dataForm.redirectType" placeholder="重定向" style="width:100%;">
        <el-option v-for="item in redirects" :key="item.dictItemValue" :label="item.dictItemKey" :value="item.dictItemValue"></el-option>
      </el-select>
    </el-form-item>
    <el-form-item label="链接地址" prop="url">
      <el-input v-model="dataForm.url" placeholder="链接地址"></el-input>
    </el-form-item>
    <el-form-item label="私有" prop="isComment">
      <el-switch
        v-model="dataForm.isComment"
        :active-value="active"
        :inactive-value="inactive"
        active-color="#13ce66"
        inactive-color="#ccc">
      </el-switch>
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
        active: 2,
        inactive: 1,
        dataForm: {
          id: 0,
          clientType: '',
          advertisingPlatform: '',
          mediaType: '',
          mediaName: '',
          advertisingSpace: '',
          advertisingSize: '',
          sizeRequirement: '',
          trafficType: '',
          advertisingSpaceType: '',
          volumeOfExhibition: '',
          redirectType: '',
          url: '',
          isComment: '',
          createYear: '',
          createMonth: '',
          createDay: '',
          createId: '',
          createName: '',
          createTime: ''
        },
        clients: [],
        traffics: [],
        spaces: [],
        redirects: [],
        dataRule: {
          clientType: [
            { required: true, message: '客户端类型不能为空', trigger: 'blur' }
          ],
          advertisingPlatform: [
            { required: true, message: '广告平台不能为空', trigger: 'blur' }
          ],
          mediaType: [
            { required: true, message: '媒体类型不能为空', trigger: 'blur' }
          ],
          mediaName: [
            { required: true, message: '媒体名称不能为空', trigger: 'blur' }
          ],
          advertisingSpace: [
            { required: true, message: '广告位不能为空', trigger: 'blur' }
          ],
          advertisingSize: [
            { required: true, message: '尺寸不能为空', trigger: 'blur' }
          ],
          sizeRequirement: [
            { required: true, message: '尺寸要求不能为空', trigger: 'blur' }
          ],
          trafficType: [
            { required: true, message: '流量类型不能为空', trigger: 'blur' }
          ],
          advertisingSpaceType: [
            { required: true, message: '广告位类型不能为空', trigger: 'blur' }
          ],
          volumeOfExhibition: [
            { required: true, message: '日点击量不能为空', trigger: 'blur' }
          ],
          redirectType: [
            { required: true, message: '是否可以重定向不能为空', trigger: 'blur' }
          ],
          url: [
            { required: true, message: '链接地址不能为空', trigger: 'blur' }
          ],
          isComment: [
            { required: true, message: '下放不能为空', trigger: 'blur' }
          ]
        }
      }
    },
    mounted () {
      this.getDictType()
    },
    methods: {
      init (id) {
        this.dataForm.id = id || 0
        this.visible = true
        this.$nextTick(() => {
          this.$refs['dataForm'].resetFields()
          if (this.dataForm.id) {
            this.$http({
              url: this.$http.adornUrl(`/generator/smedia/info/${this.dataForm.id}`),
              method: 'get',
              params: this.$http.adornParams()
            }).then(({data}) => {
              if (data && data.code === 0) {
                this.dataForm.clientType = data.sMedia.clientType+""
                this.dataForm.advertisingPlatform = data.sMedia.advertisingPlatform
                this.dataForm.mediaType = data.sMedia.mediaType
                this.dataForm.mediaName = data.sMedia.mediaName
                this.dataForm.advertisingSpace = data.sMedia.advertisingSpace
                this.dataForm.advertisingSize = data.sMedia.advertisingSize+""
                this.dataForm.sizeRequirement = data.sMedia.sizeRequirement+""
                this.dataForm.trafficType = data.sMedia.trafficType+ ""
                this.dataForm.advertisingSpaceType = data.sMedia.advertisingSpaceType+""
                this.dataForm.volumeOfExhibition = data.sMedia.volumeOfExhibition
                this.dataForm.redirectType = data.sMedia.redirectType+""
                this.dataForm.url = data.sMedia.url
                this.dataForm.isComment = parseInt(data.sMedia.isComment)
                this.dataForm.createYear = data.sMedia.createYear
                this.dataForm.createMonth = data.sMedia.createMonth
                this.dataForm.createDay = data.sMedia.createDay
                this.dataForm.createId = data.sMedia.createId
                this.dataForm.createName = data.sMedia.createName
                this.dataForm.createTime = data.sMedia.createTime
              }
            })
          }
        })
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
      // 表单提交
      dataFormSubmit () {
        this.$refs['dataForm'].validate((valid) => {
          if (valid) {
            this.$http({
              url: this.$http.adornUrl(`/generator/smedia/${!this.dataForm.id ? 'save' : 'update'}`),
              method: 'post',
              data: this.$http.adornData({
                'id': this.dataForm.id || undefined,
                'clientType': this.dataForm.clientType,
                'advertisingPlatform': this.dataForm.advertisingPlatform,
                'mediaType': this.dataForm.mediaType,
                'mediaName': this.dataForm.mediaName,
                'advertisingSpace': this.dataForm.advertisingSpace,
                'advertisingSize': this.dataForm.advertisingSize,
                'sizeRequirement': this.dataForm.sizeRequirement,
                'trafficType': this.dataForm.trafficType,
                'advertisingSpaceType': this.dataForm.advertisingSpaceType,
                'volumeOfExhibition': this.dataForm.volumeOfExhibition,
                'redirectType': this.dataForm.redirectType,
                'url': this.dataForm.url,
                'isComment': this.dataForm.isComment,
                'createYear': this.dataForm.createYear,
                'createMonth': this.dataForm.createMonth,
                'createDay': this.dataForm.createDay,
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
