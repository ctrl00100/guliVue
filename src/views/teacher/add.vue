<template>
  <div class="app-container">
    <el-form label-width="90px">
      <el-form-item label="讲师名称">
        <el-input v-model="teacher.name"/>
      </el-form-item>
      <el-form-item label="讲师排序">
        <el-input-number v-model="teacher.sort" controls-position="right" min="0"/>
      </el-form-item>
      <el-form-item label="讲师头衔">
        <el-select v-model="teacher.level" clearable placeholder="请选择">
          <!--
            数据类型一定要和取出的json中的一致，否则没法回填
            因此，这里value使用动态绑定的值，保证其数据类型是number
          -->
          <el-option :value="1" label="高级讲师"/>
          <el-option :value="2" label="首席讲师"/>
        </el-select>
      </el-form-item>
      <el-form-item label="讲师资历">
        <el-input v-model="teacher.career"/>
      </el-form-item>
      <el-form-item label="讲师简介">
        <el-input v-model="teacher.intro" :rows="10" type="textarea"/>
      </el-form-item>

      <!-- 讲师头像：TODO -->
      <!-- 讲师头像：TODO -->
      <el-form-item label="讲师头像">

        <!-- 头衔缩略图 -->
        <pan-thumb :image="teacher.avatar"/>
        <!-- 文件上传按钮 -->
        <el-button type="primary" icon="el-icon-upload" @click="imagecropperShow=true">更换头像
        </el-button>

        <!--
    v-show：是否显示上传组件
    :key：类似于id，如果一个页面多个图片上传控件，可以做区分
    :url：后台上传的url地址
    @close：关闭上传组件
    @crop-upload-success：上传成功后的回调 -->
        <image-cropper
          v-show="imagecropperShow"
          :width="300"
          :height="300"
          :key="imagecropperKey"
          :url="BASE_API+'/oss/file/upload'"
          field="file"
          @close="close"
          @crop-upload-success="cropSuccess"/>

      </el-form-item>

      <el-form-item>
        <el-button :disabled="saveBtnDisabled" type="primary" @click="saveOrUpdate">保存</el-button>
<!--        <el-button  type="primary" @click="saveOrUpdate">保存</el-button>-->
      </el-form-item>
    </el-form>
  </div>
</template>
<script>
  import teacher from '../../api/edu/teacher'
import  ImageCropper from '../../components/ImageCropper'
import  PanThumb from '../../components/PanThumb'
  import {BASE_API} from '../../../config/dev.env'

  // const defaultForm = {
  //   name: '',
  //   sort: 0,
  //   level: '',
  //   career: '',
  //   intro: '',
  //   // avatar: '',
  //   avatar: process.env.OSS_PATH + '/avatar/default.jpg'
  // }

  const defaultForm = {
    name: '',
    sort: 0,
    level: 1,
    career: '',
    intro: '',
    avatar: 'http://ctrl010.k1772714.club/avatar/2020/01/17/4c39cdfa-84f6-4543-a708-3e7883adebdb.jpg'
  }
  export default {
    //声明一下这个插件
    components: { ImageCropper, PanThumb },
    data () {
      return {
        teacher: defaultForm,
        saveBtnDisabled: false, // 不启用disabled， 保存按钮为亮色
        BASE_API: process.env.BASE_API, // 接口API地址
        imagecropperShow: false, // 是否显示上传组件
        imagecropperKey: 0 // 上传组件id

      }
    },
    watch: {
      $route(to, from) {
        console.log('watch $route')
        this.init()
      }
    },
    created () {
      this.init()
    },
    methods: {
      init() {
        //在页面加载之前，判断路由里面是否有id值
        //如果有id值，调用方法根据id查询
        //从路由里面获取值
        if(this.$route.params && this.$route.params.id) {//修改数据回显
          const id = this.$route.params.id
          //调用方法根据id查询
          this.getTeacherById(id)
        } else {//添加
          //表单数据清空
          this.teacher = { ...defaultForm }
        }
      },
      //实现添加和修改使用同一个表单
      //预留做修改
      saveOrUpdate() {
        //判断点击保存，执行添加操作 还是修改操作
        if(!this.teacher.id) {
          //调用添加的方法
          this.saveTeacher()
        } else {
          this.updateTeacher()
        }
      },

      //根据id查询
      getTeacherById(id) {
        teacher.getById(id)
          .then(response => {
            this.teacher = response.data.teacher
          })
      },
      // //添加讲师的方法
      // saveTeacher() {
      //   //调用后台接口的方法实现添加
      //   teacher.save(this.teacher)
      //     .then(() => {
      //       //请求之后，添加之后，提示用户,
      //       return this.$message({
      //         type: 'success',
      //         message: '添加成功!'
      //       })
      //     }).then(() => {
      //     //回到列表页面中，通过路由跳转回到列表页面中
      //     this.$router.push({path: '/teacher'})
      //   })
      //     .catch(() => {
      //       this.$message({
      //         type: 'success',
      //         message: '添加失败!'
      //       })
      //     })
      // },
      saveTeacher() {
        teacher.save(this.teacher).then(response => {
          return this.$message({
            type: 'success',
            message: '保存成功!'
          })
        }).then(resposne => {
          this.$router.push({ path: '/teacher/list' })
        }).catch((response) => {
          // console.log(response)
          this.$message({
            type: 'error',
            message: '保存失败'
          })
        })
      },


      //修改讲师的方法
      updateTeacher() {
        // teacher.updateById(this.teacher.id,this.teacher)
        teacher.updateById(this.teacher)
          .then(() => {
            //请求之后，添加之后，提示用户,
            return this.$message({
              type: 'success',
              message: '修改成功!'
            })
          }).then(() => {
          //路由跳转
          this.$router.push({path: '/teacher'})
        }).catch(() => {
          this.$message({
            type: 'success',
            message: '修改失败!'
          })
        })
      },
      close(){
        //1、关闭了这个上传图片框
        this.imagecropperShow = false, // 是否显示上传组件
          //2、给框ID变化一次
          this.imagecropperKey = this.imagecropperKey + 1

      },
      //保存图片成功
      cropSuccess(data){
        this.teacher.avatar =  data.url
        //1、关闭了这个上传图片框
        this.imagecropperShow = false, // 是否显示上传组件
          //2、给框ID变化一次
          this.imagecropperKey = this.imagecropperKey + 1
      }
    }
  }

</script>

