<template>
  <div class="dashboard-editor-container">
    <div class="see-title"> > 个人简历填写/Resume Edit</div>
    <div class="edit-container">
      <div class="steps">
        <el-steps :active="active" finish-status="success" space="300px">
          <el-step title="基本信息"></el-step>
          <el-step title="自身经历"></el-step>
          <el-step title="研究成果"></el-step>
        </el-steps>
      </div>
      <div class="right-side">
        <el-form ref="form" :model="form" label-width="100px" style="width:500px">
          <!-- 个人基本信息　-->
          <div class="basic-info infos" v-show="active === 0">
            <el-form-item label="姓名"><el-input v-model="form.name"></el-input></el-form-item>
            <el-form-item label="职称"><el-input v-model="form.title"></el-input></el-form-item>
            <el-form-item label="E-mail"><el-input v-model="form.e_mail"></el-input></el-form-item>
            <el-form-item label="研究方向"><el-input v-model="form.research_direction"></el-input></el-form-item>
            <el-form-item label="个人照片">
              <!--action为上传的地址，on-remove为文件列表移除文件时的钩子 -->
              <el-upload
                class="upload-photo"
                action="http://295uq99495.wicp.vip/fileUpload"
                :on-preview="handlePreview"
                :on-remove="handleRemove"
                :before-remove="beforeRemove"
                :on-success="successUpload"
                :limit="1"
                :on-exceed="handleExceed"
                :file-list="attachList">
                <el-button size="small" type="primary">上传照片</el-button>
                <div slot="tip" class="el-upload__tip">只能上传jpg/png文件，且不超过500kb</div>
              </el-upload>
            </el-form-item>
          </div>
           <!-- 项目经历　-->
          <div class="work-info infos" v-show="active === 1">
            <el-form-item label="所在单位">
                <el-input v-model="form.school"></el-input>
            </el-form-item>
            <el-form-item label="职位">
                <el-input v-model="form.position"></el-input>
            </el-form-item>
            <el-form-item label="时间跨度">
               <el-col :span="11">
                <el-date-picker type="date" placeholder="选择日期"  format="yyyy 年 MM 月 dd 日" v-model="form.time_start" value-format="timestamp" style="width: 100%;"></el-date-picker>
              </el-col>
              <el-col class="line" :span="2">-</el-col>
              <el-col :span="11">
                <el-date-picker type="date" placeholder="选择日期"  format="yyyy 年 MM 月 dd 日" v-model="form.time_end" value-format="timestamp" style="width: 100%;"></el-date-picker>
              </el-col>
            </el-form-item>
            <el-form-item label="描述信息">
                <el-input type="textarea" placeholder="尽量简洁，突出重点，成绩优异的话建议写上GPA及排名等信息。" v-model="form.describes"></el-input>
            </el-form-item>
          </div>
          <!-- 研究成果　-->
          <div class="project-info infos" v-show="active === 2">
            <el-form-item label="研究成果">
                <el-input
                    type="textarea"
                    :autosize="{ minRows: 4, maxRows: 5}"
                    placeholder="关于研究方向，学术论文，职称，出版书籍。"
                    v-model="form.research_findings">
                </el-input>
            </el-form-item>
          </div>
          <div class="buttons" v-show=" active!==2 ">
            <el-button style="margin-top: 12px;" @click="next">保存并下一步</el-button>
            <el-button style="margin-top: 12px;" @click="pre">返回上一步</el-button>
          </div>
          <el-form-item v-show=" active===2 ">
            <el-button style="margin-top:12px;margin-left:24px;" @click="onSubmit">保存并提交</el-button>
            <el-button style="margin-top:12px;" @click="pre">返回上一步</el-button>
          </el-form-item>
        </el-form>
      </div>
    </div>
  </div>
</template>

<script>
// import { mapGetters } from 'vuex'
import { getUid } from '@/utils/auth'

export default {
  name: 'DashboardEditor',
  components: {},
  data() {
    return {
      uid: getUid(),
      active: 0,
      imageUrl: '',
      edit: true,
      form: {
        // 基本信息
          name: '',
          title: '',
          e_mail: '',
          research_direction:'',
          image:'',
          research_findings:'',
          school:'',
          position:'',
          time_start:'',
          time_end:'',
          describes:''
      },
      attachList:[]
    }
  },
  created() {
    this.$store.dispatch('teacher/getTeacherBasic',{uid:this.uid}).then(response => {
      console.log(response)
      if(response.data) {
        this.edit = true
        this.form.name = this.$store.getters.tname
        this.form.title = this.$store.getters.title
        this.form.e_mail = this.$store.getters.temail
        this.form.research_direction = this.$store.getters.research_direction
        this.form.image = this.$store.getters.image
        this.form.research_findings = this.$store.getters.research_findings
     } else {
       this.edit = false
     }
    }),
    this.$store.dispatch('teacher/getTeacherProject',{uid:this.uid}).then(response => {
      if(response.data) {
        this.edit = true
        this.form.school = this.$store.getters.tschool
        this.form.position = this.$store.getters.tposition
        this.form.time_start = this.$store.getters.tstarttime
        this.form.time_end = this.$store.getters.tendtime
        this.form.describes = this.$store.getters.describes
      } else {
        this.edit = false
      }
    })
  },
  methods: {
    next() {
        if (this.active++ == 3) this.active = 0;
    },
    pre() {
      if (this.active-- == 0) this.active = 0;
    },
    onSubmit() {
      const {name,title,e_mail,research_direction,image,research_findings,school,position,time_start,time_end,describes} = this.form
      if(this.edit) {
        // 修改＆更新
        this.$store.dispatch('teacher/updateTeacherBasic',{name,title,e_mail,research_direction,image,research_findings})
        this.$store.dispatch('teacher/updateTeacherProject',{name:school,position,time_start,time_end,describes,pid:9})
        this.$message({
          message: '信息修改成功',
          type: 'success'
        });
      } else {
        // 添加
        this.$store.dispatch('teacher/addTeacherBasic',{name,title,e_mail,research_direction,image,research_findings})
        this.$store.dispatch('teacher/addTeacherProject',{name:school,position,time_start,time_end,describes,pid:9})
        this.$message({
          message: '信息添加成功',
          type: 'success'
        });
      }
    },
    handleRemove(file, fileList) {
        console.log(file, fileList);
    },
    handlePreview(file) {
        console.log(file);
    },
    handleExceed(files, fileList) {
        this.$message.warning(`当前限制选择1个文件，本次选择了 ${files.length} 个文件，共选择了 ${files.length + fileList.length} 个文件`);
    },
    beforeRemove(file, fileList) {
        return this.$confirm(`确定移除 ${ file.name }？`);
    },
    successUpload(response,file,fileList) {
      this.form.image = response.data
    }
  }
}
</script>

<style lang="scss" scoped>
  .dashboard-editor-container {
    min-height: 100vh;
    padding: 20px;

    .see-title {
    font-size: 22px;
    color:rgb(48,65,86);
    font-weight: bold;
    }

    .edit-container{
      margin-top:30px;
      margin-left: 50px;
    }

    .infos {
      margin-top: 20px;
    }

    .right-side{
      width:600px;
      margin-left:80px;

      .buttons{
        display: flex;
        justify-content: center;
      }

      .work-info{
          .add-button {
              margin-left: 200px;
          }
      }

    }


  }
</style>
