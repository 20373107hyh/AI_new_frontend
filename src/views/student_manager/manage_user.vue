<template>
    <div class="app-container">
      <h3 style="position: relative; left:15px">学生管理</h3>
      <el-button v-on:click="getViewTimeExcel()" :disabled="listLoading" style="margin: 15px;" type="primary">导出学生今日理论课件学习时间表</el-button>
      <el-button v-on:click="getReplayViewTimeExcel()" :disabled="listLoading" style="margin: 15px;" type="primary">导出学生今日课程回放学习时间表</el-button>
      <el-table
        v-loading="listLoading"
        :data="course_list"
        element-loading-text="Loading"
        border
        fit
        highlight-current-row
        class="custom-table"
        style="width: 100%; height: 600px; overflow: auto; "
      >
        <el-table-column align="center" label="用户ID" width="80">
          <template slot-scope="scope">
            {{ scope.row.user_id}}
          </template>
        </el-table-column>
        <el-table-column label="用户名">
          <template slot-scope="scope">
            {{ scope.row.username }}
          </template>
        </el-table-column>
        <el-table-column label="密码">
          <template slot-scope="scope">
            {{ scope.row.password }}
          </template>
        </el-table-column>
        <el-table-column label="用户真名">
          <template slot-scope="scope">
            {{ scope.row.realname }}
          </template>
        </el-table-column>
        <el-table-column label="邮箱">
          <template slot-scope="scope">
            {{ scope.row.email}}
          </template>
        </el-table-column>
        <el-table-column label="手机号">
          <template slot-scope="scope">
            {{ scope.row.phone }}
          </template>
        </el-table-column>
        <el-table-column label="身份">
          <template slot-scope="scope">
            {{ scope.row.status}}
          </template>
        </el-table-column>
        <el-table-column align="center" label="操作" width="140">
          <template slot-scope="scope">
            <div  style="text-align: right; position: relative; right: 20px">
              <el-button type="primary" v-if="scope.row.status==='student'" @click="TurnToDetail(scope.row.user_id)" > 查看 </el-button>
            </div>
          </template>
        </el-table-column>
      </el-table>

    </div>
  </template>
  
  <script>
  import { getList } from '@/api/table'
  import Axios from 'axios'
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
        list: [],
        listLoading: true,
        addDialogVisible: false,
        commitDialogVisible: false,
        deleteImageDialogVisible: false,
        user_form:{
          user_id: '',
          username: '',
          password: '',
          realname: '',
          email: '',
          phone: '',
          status: '',
        },
        image_list: [],
        container_name: '',
        new_image_name: '',
        new_container_name: '',
        course_list: [],
        userDialogVisible: false,
        alterDialogVisible: false,
        excelFile: []
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
            url: '/teacher/list_user/',
          }).then(
            res => {
              // console.log(res)
              let user_list = res.data.data
              const students = user_list.filter(item => item.status === 'student');
              this.course_list = students
              this.listLoading = false
            }
          )
        // getList().then(response => {
        //   this.list = response.data.items
        //   this.listLoading = false
        // })
      },
      handleDelete(user_id){
        this.listLoading = true
        // console.log(user_id)
        const formData = new FormData()
        formData.append('user_id', user_id)
        this.$axios({
            method: 'post',
            url: '/teacher/delete_user/',
            data: formData,
          }).then(
            res => {
              // console.log(res)
              this.fetchData()
              this.listLoading = false
            }
          )
      },
      isself(id){
        if(id == localStorage.getItem('user_id')){
          return true
        }
        else
          return false
      },
      AddUserDialogOpen(){
        this.userDialogVisible = true
        this.user_form = {
          user_id: '',
          username: '',
          password: '',
          realname: '',
          email: '',
          phone: '',
          status: '',
        }
      },
      handleCreate(){
        this.listLoading = true
        const formData = new FormData()
        formData.append('username', this.user_form.username)
        formData.append('password', this.user_form.password)
        formData.append('realname', this.user_form.realname)
        formData.append('email', this.user_form.email)
        formData.append('phone', this.user_form.phone)
        formData.append('status', this.user_form.status)
        this.$axios({
            method: 'post',
            url: '/teacher/add_user/',
            data: formData,
          }).then(
            res => {
              // console.log(res)
              this.fetchData()
              this.listLoading = false
              window.alert('新用户创建成功')
            }
          )
          this.userDialogVisible= false
      },
      AlterDialogOpen(row){
        this.alterDialogVisible = true
        this.user_form = {
          user_id: row.user_id,
          username: row.username,
          password: row.password,
          realname: row.realname,
          email: row.email,
          phone: row.phone,
          status: row.status,
        }
      },
      handleAlter(){
        this.listLoading = true
        const formData = new FormData()
        formData.append('user_id', this.user_form.user_id)
        formData.append('username', this.user_form.username)
        formData.append('password', this.user_form.password)
        formData.append('realname', this.user_form.realname)
        formData.append('email', this.user_form.email)
        formData.append('phone', this.user_form.phone)
        formData.append('status', this.user_form.status)
        this.$axios({
            method: 'post',
            url: '/teacher/alter_user/',
            data: formData,
          }).then(
            res => {
              // console.log(res)
              this.fetchData()
              this.listLoading = false
              window.alert('用户信息修改成功')
            }
          )
          this.alterDialogVisible = false
      },
      TurnToDetail(user_id){
        this.$router.push({
          path:'/manage/user_detail',
          query:{
            user_id: user_id,
          }
        })
      },
      handleExcelFileUpload() {
        this.excelFile = this.$refs.excelFile.files[0]; // 获取文件
      },
      AddUserByExcel() {
        let formData = new FormData();
        let excelFile = this.excelFile
        formData.append("excel_file", excelFile);
  
        this.$axios({
          method: 'post',
          url: '/teacher/add_user_by_excel/',
          data: formData,
          headers: {
            'Content-Type': 'multipart/form-data'
          }
        }).then(res => {
          window.alert(res.data.msg)
          this.fetchData()
        })
      },
      getViewTimeExcel(){
        this.$axios({
          method: 'post',
          url: '/teacher/export_view_time_excel/',
          responseType: 'blob',
        })
        .then((response) => {
                const filename = '学生今日理论课件学习时间统计表.xlsx'
                const url = window.URL.createObjectURL(new Blob([response.data]));
                const link = document.createElement('a');
                link.href = url;
                link.setAttribute('download', filename);
                document.body.appendChild(link);
                link.click();
            })
      },
      getReplayViewTimeExcel(){
        this.$axios({
          method: 'post',
          url: '/teacher/export_replay_view_time_excel/',
          responseType: 'blob',
        })
        .then((response) => {
                const filename = '学生今日课程回放学习时间统计表.xlsx'
                const url = window.URL.createObjectURL(new Blob([response.data]));
                const link = document.createElement('a');
                link.href = url;
                link.setAttribute('download', filename);
                document.body.appendChild(link);
                link.click();
            })
      },
    }
  }
  </script>
  
  <style scoped>
  .custom-table >>>.el-table__header-wrapper tr:first-child th {
  background-color: lightblue; /* 更改第一行表头的背景颜色，这里设置为红色，您可以修改为您想要的颜色 */
}


.custom-table >>>.el-table__body-wrapper tr:nth-child(even) {
  background-color: #f0f0f0; /* 偶数行背景颜色 */
}

.custom-table >>>.el-table__body-wrapper tr:nth-child(odd) {
  background-color: #ffffff; /* 奇数行背景颜色 */
}
  </style>