<template>
  <div class="mod-demo-echarts">
    <el-card style="width:100%;margin:0 auto;" class="box-card">
      <div slot="header" class="clearfix">
        <span>基本信息</span>
      </div>
      <el-form :model="dataForm" label-width="130px">
        <el-form-item label="用户名" prop="userName">
          <el-input v-model="dataForm.userName" :disabled="true" placeholder="暂无"></el-input>
        </el-form-item>
        <el-form-item label="邮箱" prop="email">
          <el-input v-model="dataForm.email" :disabled="true" placeholder="暂无"></el-input>
        </el-form-item>
        <el-form-item label="公司名称" prop="companyName">
          <el-input v-model="dataForm.companyName" :disabled="true" placeholder="暂无"></el-input>
        </el-form-item>
        <el-form-item label="官网链接" prop="companyUrl">
          <el-input v-model="dataForm.companyUrl" :disabled="true" placeholder="暂无"></el-input>
        </el-form-item>
        <el-form-item label="联系人" prop="companyPerson">
          <el-input v-model="dataForm.companyPerson" :disabled="true" placeholder="暂无"></el-input>
        </el-form-item>
        <el-form-item label="手机号" prop="mobile">
          <el-input v-model="dataForm.mobile" :disabled="true" placeholder="暂无"></el-input>
        </el-form-item>
        <el-form-item label="备注" prop="remark">
          <el-input v-model="dataForm.remark" :disabled="true" placeholder="暂无"></el-input>
        </el-form-item>
      </el-form>
    </el-card>
    <el-card style="width:100%;margin:15px auto;" class="box-card">
      <div slot="header" class="clearfix">
        <span>备案信息</span>
      </div>
      <el-form :model="dataForm" ref="dataForm" @keyup.enter.native="dataFormSubmit()" label-width="130px">
        <el-form-item label="备案信息">
          <el-input v-model="dataForm.recordMessge" type="textarea" :rows="5" placeholder="暂无"></el-input>
        </el-form-item>
        <el-form-item label="营业执照" size="mini" style="text-align: center;color:red">
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
        <el-form-item label="身份证正面" size="mini" style="text-align: center;color:red">
          <el-upload
            class="avatar-uploader" style="float: left;margin:10px;"
            :action="uploadurl"
            :show-file-list="false"
            :on-success="idcardPositiveImgHandleAvatarSuccess"
            :before-upload="beforeAvatarUpload">
            <img v-if="idcardPositiveImg" :src="idcardPositiveImg" class="avatar">
            <i v-else class="el-icon-plus avatar-uploader-icon" style="font-size: 16px;">正面</i>
          </el-upload>
        </el-form-item>

         <el-form-item label="身份证反面" size="mini" style="text-align: center;color:red">
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
        <el-form-item>
          <el-button style="display: block;margin:0 auto;" type="primary" @click="dataFormSubmit()">确定</el-button>
        </el-form-item>
      </el-form>

    </el-card>
  </div>
</template>


<script>
  export default {
    data () {
      return {
        visible: false,
        roleList: [],
        uploadurl: '',
        recordMessge: '',
        businessImg: '',
        idcardPositiveImg: '',
        idcardNegativeImg: '',
        dataForm: {
          id: 0,
          userName: '',
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
        dataRule: {}
      }
    },
    activated () {
      this.init()
    },
    methods: {
      init () {
        this.uploadurl = this.$http.adornUrl(`/sys/oss/upload?token=${this.$cookie.get('token')}`)
        this.$http({
          url: this.$http.adornUrl(`/sys/user/info`),
          method: 'get'
        }).then(({data}) => {
          if (data && data.code === 0) {
            this.dataForm.id = data.user.userId;
            this.dataForm.userName = data.user.username
            this.dataForm.email = data.user.email
            this.dataForm.mobile = data.user.mobile
            this.dataForm.status = data.user.status
            this.dataForm.companyName = data.user.companyName
            this.dataForm.companyUrl = data.user.companyUrl
            this.dataForm.companyPerson = data.user.companyPerson
            this.dataForm.remark = data.user.remark
            this.dataForm.businessImg = data.user.businessImg;
            this.dataForm.recordMessge = data.user.recordMessge;
            this.dataForm.idcardPositiveImg = data.user.idcardPositiveImg;
            this.dataForm.idcardNegativeImg = data.user.idcardNegativeImg;
            this.recordMessge = data.user.recordMessge;
            this.businessImg = data.user.businessImg;
            this.idcardPositiveImg = data.user.idcardPositiveImg;
            this.idcardNegativeImg = data.user.idcardNegativeImg;
          }
        })
      },
      showPic (title,url)
      {
        const image = this.$createElement // 创建元素
        this.$msgbox({
              title: title,
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
      // 表单提交
      dataFormSubmit () {
        var flag = true;
        if(this.dataForm.id == '')
        {
          this.$message.error("系统问题，请联系管理员");
          flag = false;
        }
        if (flag) {
          this.$http({
            url: this.$http.adornUrl(`/sys/user/update`),
            method: 'post',
            data: this.$http.adornData({
              'userId': this.dataForm.id,
              'username': this.dataForm.userName,
              'email':this.dataForm.email,
              'mobile':this.dataForm.mobile,
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
                  this.init();

                }
              })
            } else {
              this.$message.error(data.msg)
            }
          })
        }
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
<style>
  .text {
    font-size: 14px;
  }

  .item {
    margin-bottom: 18px;
  }

  .clearfix:before,
  .clearfix:after {
    display: table;
    content: "";
  }
  .clearfix:after {
    clear: both
  }

  .box-card {
    width: 480px;
  }
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