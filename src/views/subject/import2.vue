<template>
  <div class="app-container">



    <el-row :gutter="20">
      <el-col :span="8" :offset="1"><div class="grid-content bg-purple">
<!--        <el-form label-width="120px" >-->
        <el-form  >
        <el-form-item label="信息描述">
          <el-tag type="info">excel模版说明:subject模板</el-tag>
          <el-tag>
            <i class="el-icon-download"/>
            <a :href="OSS_PATH + '/avatar/2020/01/19/%E8%AF%BE%E7%A8%8B%E5%88%86%E7%B1%BB.xls'">点击下载模版</a>
          </el-tag>
        </el-form-item>
          <el-form-item label="选择Excel">
          <el-upload
            class="upload-demo"
            ref="upload"
            :on-success="fileUploadSuccess"
            :on-error="fileUploadError"
            :disabled="importBtnDisabled"
            :action="BASE_API+'/subject/import'"
            :on-preview="handlePreview"
            :on-remove="handleRemove"
            :auto-upload="false"
            accept="application/vnd.ms-excel">

            <el-button slot="trigger" size="small" type="primary">选取文件</el-button>
            <el-button style="margin-left: 10px;" size="small" type="success" @click="submitUpload">{{ fileUploadBtnText }}</el-button>
            <div slot="tip" class="el-upload__tip">只能上传xls/xlsx文件</div>
          </el-upload>

          </el-form-item>
        </el-form>

      </div></el-col>
    </el-row>
  </div>

</template>



<script>
  export default {
    data() {
          return {
            BASE_API: process.env.BASE_API, // 接口API地址
            OSS_PATH: process.env.OSS_PATH, // 阿里云OSS地址
            fileUploadBtnText: '上传到服务器', // 按钮文字
            importBtnDisabled: false, // 按钮是否禁用,
            loading: false,

        // fileList: [{name: 'food.jpeg', url: 'https://fuss10.elemecdn.com/3/63/4e7f3a15429bfda99bce42a18cdd1jpeg.jpeg?imageMogr2/thumbnail/360x360/format/webp/quality/100'}, {name: 'food2.jpeg', url: 'https://fuss10.elemecdn.com/3/63/4e7f3a15429bfda99bce42a18cdd1jpeg.jpeg?imageMogr2/thumbnail/360x360/format/webp/quality/100'}]
      };
    },
    methods: {
      submitUpload() {
        this.fileUploadBtnText = '正在上传'
        this.importBtnDisabled = true
        this.loading = true
        this.$refs.upload.submit()
      }
     ,
      handleRemove(file, fileList) {
        // console.log(file.name, fileList);
        this.$message({
          type: 'success',
          message: '移除文件'+file.name
        })
      },
      handlePreview(file) {
        console.log(file.name,'handlePreview');

        this.$message({
          type: 'success',
          message: '已选择文件'+file.name,
        })      },

    fileUploadSuccess(response) {
      if (response.success === true) {
        this.fileUploadBtnText = '导入成功'
        this.loading = false
        this.$message({
          type: 'success',
          message: response.message
        })
      } else {
        this.fileUploadBtnText = '导入失败'
        this.loading = false
        const messages = response.data.message
        let msgString = '<ul>'
        messages.forEach(msg => {
          msgString += `<li>${msg}</li>`
        })
        msgString += '</ul>'

        // <ul>
        //     <li>第四行第二列为空</li>
        //     <li>第六行第二列为空</li>
        // </ul>

        this.$alert(msgString, response.message, {
          dangerouslyUseHTMLString: true
        })
      }
    },

    fileUploadError(response) {
      this.fileUploadBtnText = '导入失败'
      this.loading = false
      this.$message({
        type: 'error',
        message: '导入失败'
      })
    }

    }
  }
</script>
