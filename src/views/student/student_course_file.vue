<template>
  <div class="app-container">
    <h3 style="position: relative; left:15px">课程文件</h3>

    <el-table 
              :data="file_list" 
              v-loading="listLoading"
              style="width: 100%"
              :height="1000">
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
                  <el-button @click="viewFile(scope.row.filename)" type="primary">查看文件</el-button>
              </template>
              </el-table-column>
          </el-table>

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
      file_list: [],
      singleFile: [], //files uploading
      folderFiles: [],
      tarData: null,
      listLoading: false
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
          url: '/student/load_course_files/',
        }).then(
          res => {
            console.log(res)
            this.file_list = res.data.data
            this.listLoading = false
          }
        )
      // getList().then(response => {
      //   this.list = response.data.items
      //   this.listLoading = false
      // })
    },
    viewFile(filename){
      this.$router.push({
        path:'/view_course_file',
        query:{
          filename: filename,
        }
      })
    },
  }
}
</script>
