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
      <el-form :model="dataForm" label-width="130px">
        <el-form-item label="备案信息">
          <el-input v-model="dataForm.recordMessge" :disabled="true" type="textarea" :rows="5" placeholder="暂无"></el-input>
        </el-form-item>
        <el-form-item label="营业执照" size="mini" style="text-align: center;color:red">
          <img style="margin:0 auto" @click="showPic('营业执照',dataForm.businessImg)" v-if="dataForm.businessImg" :src="dataForm.businessImg" class="avatar">
          <font v-else>暂无图片</font>
        </el-form-item>
        <el-form-item label="身份证正面" size="mini" style="text-align: center;color:red">
          <img style="margin:0 auto" @click="showPic('身份证正面',dataForm.idcardPositiveImg)" v-if="dataForm.idcardPositiveImg" :src="dataForm.idcardPositiveImg" class="avatar">
          <font v-else>暂无图片</font>
        </el-form-item>
         <el-form-item label="身份证反面" size="mini" style="text-align: center;color:red">
          <img style="margin:0 auto"  @click="showPic('身份证反面',dataForm.idcardNegativeImg)" v-if="dataForm.idcardNegativeImg" :src="dataForm.idcardNegativeImg" class="avatar">
          <font v-else>暂无图片</font>
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
        dataRule: {}
      }
    },
    activated () {
      this.init()
    },
    methods: {
      init () {
          if(this.$route.params.id) {
            this.dataForm.id = this.$route.params.id;
          }
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
</style>