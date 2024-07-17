<template>
  <div class="app-container">
    <h3 style="position: relative; left:15px">课程回放</h3>
    <el-collapse v-model="activeNames">
      <el-collapse-item 
          v-for="chapter in chapters" 
          :key="chapter.chapter_num" 
          :name="chapter.chapter_num.toString()" 
          :title=" chapter.chapter_num + '  ' + chapter.chapter_name"
      >
          <div style="display: flex; margin: 15px;"><h3 style="width: 90%;">{{ chapter.chapter_intro }} </h3>
          </div>

          <el-table 
              :data="chapter.section_list" 
              style="width: 100%" 
              stripe
          >
              <el-table-column 
                  prop="chapter_number"
                  label="序号" 
                  width="50"
              >
              </el-table-column>
              <el-table-column
                  prop="section_number"
                  label="小节号"
                  width="75">
              </el-table-column>
              <el-table-column
                  prop="section_name"
                  label="小节名称"
                  width="300"
                  >
              </el-table-column>
              <el-table-column
                  prop="section_intro"
                  label="小节介绍">
              </el-table-column>
              <el-table-column
                  label="操作"
                  width="300"
                  align="center">
                  <template slot-scope="scope">
                    <el-button type="primary" @click="uploadDialogOpen(scope.row.chapter_number, scope.row.section_number)">查看课件</el-button>

                  </template>
              </el-table-column>
          </el-table>
      </el-collapse-item>
    </el-collapse>

  <el-dialog title="查看课件" :visible.sync="uploadDialogVisible" width="70%" center>
    <el-table 
              :data="file_list" 
              v-loading="listLoading"
              style="width: 100%"
              :height="400">
              <el-table-column 
              prop="name" 
              label="文件名">
              <template slot-scope="scope">
                  {{ scope.row.filename}}
              </template>
              </el-table-column>
              <el-table-column 
              prop="name" 
              label="操作">
              <template slot-scope="scope">
                  <el-button @click="viewFile(scope.row.filename)" type="primary">查看课件</el-button>
              </template>
              </el-table-column>
          </el-table>
    <div slot="footer" class="dialog-footer">
      <el-button @click="uploadDialogVisible = false"> 关闭 </el-button>
    </div>
  </el-dialog>


  <el-dialog title="查看练习" :visible.sync="uploadExerciseDialogVisible" width="60%" center>
    <div style="margin: 15px;">
      <pre>{{exercise_stem}}</pre>
    </div>
    <div slot="footer" class="dialog-footer">
      <el-button @click="uploadExerciseDialogVisible = false"> 关闭 </el-button>
    </div>
  </el-dialog>

  </div>
</template>

<script>
import { getList } from '@/api/table'
import Axios from 'axios'
import tarStream from 'tar-stream';
import bl from 'bl';
export default {
  filters: {
    statusFilter(status) {
      const statusMap = {
        published: 'success',
        draft: 'gray',
        deleted: 'danger'
      }
      return statusMap[status]
    }
  },
  data() {
    return {
      activeNames: [],
      chapters: [],
      file_list: [],
      singleFile: [], //files uploading
      folderFiles: [],
      largeFile: [],
      tarData: null,
      chapter_info: {
        chapter_num: 1,
        chapter_name: '',
        chapter_intro: '',
      },
      section_info: {
        chapter_num: 1,
        section_num: 1,
        section_name: '',
        section_intro: '',
      },
      listLoading: false,
      percentage: 0,
      uploadstatus: 0,
      addChapterDialogVisible: false,
      addSectionDialogVisible: false,
      uploadDialogVisible: false,
      uploadExerciseDialogVisible: false,
      current_chapter: 1,
      current_section: 1,
      exercise_stem: '',
    }
  },
  created() {
    this.fetchData()
  },
  methods: {
    fetchData() {
      this.listLoading = true
      this.$axios({
          method: 'post',
          url: '/teacher/list_replay_sections/',
        }).then(
          res => {
            // // console.log(res)
            this.chapters = res.data.data
            this.listLoading = false
          }
        )
      // getList().then(response => {
      //   this.list = response.data.items
      //   this.listLoading = false
      // })
    },
    uploadDialogOpen(chapter_number, section_number){
      this.current_chapter = chapter_number
      this.current_section = section_number
      this.uploadDialogVisible = true
      const formData = new FormData()
      formData.append('chapter_number', chapter_number)
      formData.append('section_number', section_number)
      this.$axios({
          method: 'post',
          url: '/teacher/load_replay_files_by_section/',
          data: formData
        }).then(
          res => {
            this.file_list = res.data.data
          }
        )
    },
    fetchFiles(){
      const formData = new FormData()
      formData.append('chapter_number', this.current_chapter)
      formData.append('section_number', this.current_section)
      this.$axios({
          method: 'post',
          url: '/teacher/load_replay_files_by_section/',
          data: formData
        }).then(
          res => {
            this.file_list = res.data.data
          }
        )
    },  
    isSelectable(file) {
      // 这个函数决定哪些文件可以被选中
      // 不写代表允许所有文件被选中
      return true;
    },
    uploadExerciseDialogOpen(chapter_num, section_num){
      this.uploadExerciseDialogVisible = true
      this.current_chapter = chapter_num
      this.current_section = section_num
      const formData = new FormData()
      formData.append('chapter_num', chapter_num)
      formData.append('section_num', section_num)
      this.$axios({
        method: 'post',
        url: '/teacher/load_exercise/',
        data: formData
      }).then(res=>{
        this.exercise_stem = res.data.data
      })
    },
    handleSelectionChange(selected) {
      this.selectedFiles = selected.map(file => file.path);
    },
    handleExerciseUpload(){
      const formData = new FormData();
      formData.append('chapter_num', this.current_chapter)
      formData.append('section_num', this.current_section)
      formData.append('exercise_stem', this.exercise_stem)
      this.$axios({
          method: 'post',
          url: '/teacher/upload_exercise/',
          data: formData,
        }).then(response => {
          window.alert(response.data.msg)
          this.uploadExerciseDialogVisible = false
        })
    },
    viewFile(filename){
      this.$router.push({
        path:'/view_replay_file',
        query:{
          filename: filename,
          filetype: 2
        }
      })
    },
  }
}
</script>

<style scoped>
   /deep/.el-collapse {
    background-color: #D5E8FF !important;
  }
  /deep/.el-collapse,.el-collapse-item__wrap {
    border: 40px solid white;
    width: 100%;
    position: relative;
    left:0;
  }
  /deep/ .el-collapse-item__header {
      color: black;
      background-color: #D5E8FF !important;
      height: 80px;
      font-size: 20px;
      margin-left: 10px;
  }
  /deep/ .el-collapse-item__content {
      color: black;
      background-color: #e1edfd !important;
      position: relative;
      top:-20px;
  }
</style>