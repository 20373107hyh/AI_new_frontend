<template>
    <div class="app-container">
      <h3 style="position: relative; left:15px">用户管理</h3>
      <el-button @click="AddUserDialogOpen()" style="margin: 20px;" type="primary"> 新建用户 </el-button>
      <br>通过EXCEL表格批量新建：<input type="file" ref="excelFile" @change="handleExcelFileUpload" style="margin: 15px;"/>
      <el-button v-on:click="AddUserByExcel()" :disabled="listLoading" style="margin: 15px;" type="primary">批量新建</el-button>
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
        <el-table-column align="center" label="操作" width="300">
          <template slot-scope="scope">
            <div  style="text-align: right; position: relative; right: 20px">
              <el-button type="primary" v-if="scope.row.status==='student'" @click="TurnToDetail(scope.row.user_id)" > 查看 </el-button>
              <el-button type="primary" @click="AlterDialogOpen(scope.row)"> 修改 </el-button>
              <el-button type="danger" @click="handleDelete(scope.row.user_id)" v-if="isself(scope.row.user_id)===false"> 删除 </el-button>
              <el-button v-if="isself(scope.row.user_id)===true" style="opacity: 0; cursor: default;"> 空白 </el-button>
            </div>
          </template>
        </el-table-column>
      </el-table>

      <el-dialog title="添加用户信息" :visible.sync="userDialogVisible" width="40%" center>
        <el-form ref="dataForm" :model="user_form" label-position="left" label-width="100px" style="width: 400px; margin-left: 50px">
            <el-form-item label="用户名" prop="com">
                <el-input v-model="user_form.username" clearable style="width: 300px; margin-left: 20px"/>
            </el-form-item>
            <el-form-item label="密码" prop="com">
                <el-input v-model="user_form.password" clearable style="width: 300px; margin-left: 20px"/>
            </el-form-item>
            <el-form-item label="用户真名" prop="com">
                <el-input v-model="user_form.realname" clearable style="width: 300px; margin-left: 20px"/>
            </el-form-item>
            <el-form-item label="邮箱" prop="com">
                <el-input v-model="user_form.email" clearable style="width: 300px; margin-left: 20px"/>
            </el-form-item>
            <el-form-item label="手机号" prop="com">
                <el-input v-model="user_form.phone" clearable style="width: 300px; margin-left: 20px"/>
            </el-form-item>
            <el-form-item label="用户身份">
                <el-radio-group v-model="user_form.status">
                    <el-radio :label="'teacher'">教师</el-radio>
                    <el-radio :label="'student'">学生</el-radio>
                    <el-radio :label="'manager'">学生管理</el-radio>
                </el-radio-group>
            </el-form-item>
        </el-form>
        <div slot="footer" class="dialog-footer">
            <el-button @click="userDialogVisible = false"> 取消 </el-button>
            <el-button type="primary" @click="handleCreate()"> 提交 </el-button>
        </div>
        </el-dialog>

        <el-dialog title="修改用户信息" :visible.sync="alterDialogVisible" width="40%" center>
            <el-form ref="dataForm" :model="user_form" label-position="left" label-width="100px" style="width: 400px; margin-left: 50px">
                <el-form-item label="用户名" prop="com">
                    <el-input v-model="user_form.username" clearable style="width: 300px; margin-left: 20px"/>
                </el-form-item>
                <el-form-item label="密码" prop="com">
                    <el-input v-model="user_form.password" clearable style="width: 300px; margin-left: 20px"/>
                </el-form-item>
                <el-form-item label="用户真名" prop="com">
                    <el-input v-model="user_form.realname" clearable style="width: 300px; margin-left: 20px"/>
                </el-form-item>
                <el-form-item label="邮箱" prop="com">
                    <el-input v-model="user_form.email" clearable style="width: 300px; margin-left: 20px"/>
                </el-form-item>
                <el-form-item label="手机号" prop="com">
                    <el-input v-model="user_form.phone" clearable style="width: 300px; margin-left: 20px"/>
                </el-form-item>
            </el-form>
            <div slot="footer" class="dialog-footer">
                <el-button @click="userDialogVisible = false"> 取消 </el-button>
                <el-button type="primary" @click="handleAlter()"> 提交 </el-button>
            </div>
        </el-dialog>
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
              this.course_list = res.data.data
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