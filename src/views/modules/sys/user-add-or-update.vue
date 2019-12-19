<template>
  <el-dialog
    :title="!dataForm.id ? '新增' : '修改'"
    :close-on-click-modal="false"
    :visible.sync="visible">
    <el-form :model="dataForm" :rules="dataRule" ref="dataForm" @keyup.enter.native="dataFormSubmit()" label-width="80px">
      <el-form-item label="用户名" prop="userName">
        <el-input v-model="dataForm.userName" placeholder="登录帐号"></el-input>
      </el-form-item>
      <el-form-item label="密码" prop="password" :class="{ 'is-required': !dataForm.id }">
        <el-input v-model="dataForm.password" type="password" placeholder="密码"></el-input>
      </el-form-item>
      <el-form-item label="确认密码" prop="comfirmPassword" :class="{ 'is-required': !dataForm.id }">
        <el-input v-model="dataForm.comfirmPassword" type="password" placeholder="确认密码"></el-input>
      </el-form-item>
      <el-form-item label="邮箱" prop="email">
        <el-input v-model="dataForm.email" placeholder="邮箱"></el-input>
      </el-form-item>
      <el-form-item label="公司名称" prop="companyName">
        <el-input v-model="dataForm.companyName" placeholder="公司名称"></el-input>
      </el-form-item>
      <el-form-item label="官网链接" prop="companyUrl">
        <el-input v-model="dataForm.companyUrl" placeholder="官网链接"></el-input>
      </el-form-item>
      <el-form-item label="联系人" prop="companyPerson">
        <el-input v-model="dataForm.companyPerson" placeholder="联系人"></el-input>
      </el-form-item>
      <el-form-item label="手机号" prop="mobile">
        <el-input v-model="dataForm.mobile" placeholder="手机号"></el-input>
      </el-form-item>
      <el-form-item label="备注" prop="remark">
        <el-input v-model="dataForm.remark" placeholder="备注"></el-input>
      </el-form-item>
      <el-form-item label="备案信息">
         <el-input v-model="dataForm.recordMessge" type="textarea" :rows="5" placeholder="ICP备案信息"></el-input>
      </el-form-item>
      <el-form-item label="营业执照" size="mini">
        <el-upload
          class="avatar-uploader"
          style="float: left;margin:10px;"
          :action="uploadurl"
          :show-file-list="false"
          :on-success="businessHandleAvatarSuccess"
          :before-upload="beforeAvatarUpload">
          <img v-if="businessImg" :src="businessImg" class="avatar">
          <i v-else class="el-icon-plus avatar-uploader-icon"></i>
        </el-upload>
      </el-form-item>
      <el-form-item label="身份证" size="mini">
        <el-upload
          class="avatar-uploader" style="float: left;margin:10px;"
          :action="uploadurl"
          :show-file-list="false"
          :on-success="idcardPositiveImgHandleAvatarSuccess"
          :before-upload="beforeAvatarUpload">
          <img v-if="idcardPositiveImg" :src="idcardPositiveImg" class="avatar">
          <i v-else class="el-icon-plus avatar-uploader-icon" style="font-size: 16px;">正面</i>
        </el-upload>
        <el-upload
          class="avatar-uploader" style="float: left;margin:10px;"
          :action="uploadurl"
          :show-file-list="false"
          :on-success="idcardNegativeImgHandleAvatarSuccess"
          :before-upload="beforeAvatarUpload">
          <img v-if="idcardNegativeImg" :src="idcardNegativeImg" class="avatar">
          <i v-else class="el-icon-plus avatar-uploader-icon" style="font-size: 16px;">反面</i>
        </el-upload>
      </el-form-item>

      <el-form-item label="角色" size="mini" prop="roleIdList">
        <el-checkbox-group v-model="dataForm.roleIdList">
          <el-checkbox v-for="role in roleList" :key="role.roleId" :label="role.roleId">{{ role.roleName }}</el-checkbox>
        </el-checkbox-group>
      </el-form-item>
      <el-form-item label="状态" size="mini" prop="status">
        <el-radio-group v-model="dataForm.status">
          <el-radio :label="0">禁用</el-radio>
          <el-radio :label="1">正常</el-radio>
        </el-radio-group>
      </el-form-item>
    </el-form>
    
    <span slot="footer" class="dialog-footer">
      <el-button @click="visible = false">取消</el-button>
      <el-button type="primary" @click="dataFormSubmit()">确定</el-button>
    </span>
  </el-dialog>
</template>

<script>
  import { isEmail, isMobile } from '@/utils/validate'
  export default {
    inject: ['refresh'],
    data () {
      var validatePassword = (rule, value, callback) => {
        if (!this.dataForm.id && !/\S/.test(value)) {
          callback(new Error('密码不能为空'))
        } else {
          callback()
        }
      }
      var validateComfirmPassword = (rule, value, callback) => {
        if (!this.dataForm.id && !/\S/.test(value)) {
          callback(new Error('确认密码不能为空'))
        } else if (this.dataForm.password !== value) {
          callback(new Error('确认密码与密码输入不一致'))
        } else {
          callback()
        }
      }
      var validateEmail = (rule, value, callback) => {
        if (!isEmail(value)) {
          callback(new Error('邮箱格式错误'))
        } else {
          callback()
        }
      }
      var validateMobile = (rule, value, callback) => {
        if (!isMobile(value)) {
          callback(new Error('手机号格式错误'))
        } else {
          callback()
        }
      }
      return {
        visible: false,
        roleList: [],
        uploadurl: '',
        businessImg: '',
        idcardPositiveImg: '',
        idcardNegativeImg: '',
        dataForm: {
          id: 0,
          userName: '',
          password: '',
          comfirmPassword: '',
          salt: '',
          email: '',

          companyName: '',
          companyUrl: '',
          companyPerson: '',
          mobile: '',
          remark: '',
          roleIdList: [],
          status: 1,
          businessImg: '',
          recordMessge: '',
          idcardPositiveImg: '',
          idcardNegativeImg: '',
          annex: ''
        },
        dataRule: {
          userName: [
            { required: true, message: '用户名不能为空', trigger: 'blur' }
          ],
          password: [
            { validator: validatePassword, trigger: 'blur' }
          ],
          comfirmPassword: [
            { validator: validateComfirmPassword, trigger: 'blur' }
          ],
          email: [
            { required: true, message: '邮箱不能为空', trigger: 'blur' },
            { validator: validateEmail, trigger: 'blur' }
          ],
          mobile: [
            { required: true, message: '手机号不能为空', trigger: 'blur' },
            { validator: validateMobile, trigger: 'blur' }
          ]
        }
      }
    },
    methods: {
      init (id) {
        this.uploadurl = this.$http.adornUrl(`/sys/oss/upload?token=${this.$cookie.get('token')}`)
        this.$nextTick(() => {
            this.$refs['dataForm'].resetFields()            
        })
        this.dataForm.id = id || 0
        this.$http({
          url: this.$http.adornUrl('/sys/role/select'),
          method: 'get',
          params: this.$http.adornParams()
        }).then(({data}) => {
          this.roleList = data && data.code === 0 ? data.list : []
        }).then(() => {
          this.visible = true
          this.$nextTick(() => {
            this.$refs['dataForm'].resetFields()
          })
        }).then(() => {
          if (this.dataForm.id) {
            this.$http({
              url: this.$http.adornUrl(`/sys/user/info/${this.dataForm.id}`),
              method: 'get',
              params: this.$http.adornParams()
            }).then(({data}) => {
              if (data && data.code === 0) {
                this.dataForm.userName = data.user.username
                this.dataForm.salt = data.user.salt
                this.dataForm.email = data.user.email
                this.dataForm.mobile = data.user.mobile
                this.dataForm.roleIdList = data.user.roleIdList
                this.dataForm.status = data.user.status

                this.dataForm.companyName = data.user.companyName
                this.dataForm.companyUrl = data.user.companyUrl
                this.dataForm.companyPerson = data.user.companyPerson
                this.dataForm.remark = data.user.remark
                this.dataForm.businessImg = data.user.businessImg;
                this.dataForm.recordMessge = data.user.recordMessge;
                this.dataForm.idcardPositiveImg = data.user.idcardPositiveImg;
                this.dataForm.idcardNegativeImg = data.user.idcardNegativeImg;
                
                this.businessImg = data.user.businessImg;
                this.idcardPositiveImg = data.user.idcardPositiveImg;
                this.idcardNegativeImg = data.user.idcardNegativeImg;


              }
            })
          }
        })
      },
      // 表单提交
      dataFormSubmit () {
        this.$refs['dataForm'].validate((valid) => {
          if (valid) {
            this.$http({
              url: this.$http.adornUrl(`/sys/user/${!this.dataForm.id ? 'save' : 'update'}`),
              method: 'post',
              data: this.$http.adornData({
                'userId': this.dataForm.id || undefined,
                'username': this.dataForm.userName,
                'password': this.dataForm.password,
                'salt': this.dataForm.salt,
                'email': this.dataForm.email,
                'mobile': this.dataForm.mobile,
                'status': this.dataForm.status,
                'roleIdList': this.dataForm.roleIdList,
                'companyName': this.dataForm.companyName,
                'companyUrl': this.dataForm.companyUrl,
                'companyPerson': this.dataForm.companyPerson,
                'remark': this.dataForm.remark,
                'businessImg': this.dataForm.businessImg,
                'recordMessge': this.dataForm.recordMessge,
                'idcardPositiveImg': this.dataForm.idcardPositiveImg,
                'idcardNegativeImg': this.dataForm.idcardNegativeImg

              })
            }).then(({data}) => {
              if (data && data.code === 0) {
                this.$message({
                  message: '操作成功',
                  type: 'success',
                  duration: 1500,
                  onClose: () => {
                    this.visible = false

                    this.businessImg = '';
                    this.idcardNegativeImg = '';
                    this.idcardPositiveImg = '';

                    this.dataForm.recordMessge = '';
                    
                    this.dataForm.businessImg = '';
                    this.dataForm.idcardNegativeImg = '';
                    this.dataForm.idcardPositiveImg = '';

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
      businessHandleAvatarSuccess(res, file) {
        this.dataForm.businessImg = res.url;
        this.businessImg = URL.createObjectURL(file.raw);
      },
      idcardPositiveImgHandleAvatarSuccess(res,file) {
        this.dataForm.idcardPositiveImg =  res.url;
        this.idcardPositiveImg =  URL.createObjectURL(file.raw);
      },
      idcardNegativeImgHandleAvatarSuccess(res,file)
      {
        this.dataForm.idcardNegativeImg = res.url;
        this.idcardNegativeImg = URL.createObjectURL(file.raw);
      },
      beforeAvatarUpload(file) {
        const isJPG = file.type === 'image/jpeg';
        const isPNG = file.type === 'image/png';
        const isLt2M = file.size / 1024 / 1024 < 2;
        if (!isJPG && !isPNG) {
          this.$message.error('上传图片只能是 JPG PNG 格式!');
          return;
        }
        if (!isLt2M) {
          this.$message.error('上传图片大小不能超过 2MB!');
          return;
        }
        return (isJPG || isPNG) && isLt2M;
      }
    }
  }
</script>
<style>
  .avatar-uploader .el-upload {
    border: 1px dashed #d9d9d9;
    border-radius: 6px;
    cursor: pointer;
    position: relative;
    overflow: hidden;
  }
  .avatar-uploader .el-upload:hover {
    border-color: #409EFF;
  }
  .avatar-uploader-icon {
    font-size: 28px;
    color: #8c939d;
    width: 178px;
    height: 178px;
    line-height: 178px;
    text-align: center;
  }
  .avatar {
    width: 178px;
    height: 178px;
    display: block;
  }
</style>
