<template>
    <div class="detail-container">
        <el-button @click="ReturnToUserInfo()" type="primary">返回 </el-button>
        <div style="margin: 15px;">
            <div class="detail-text">用户名：<div style="width:80%">{{ user_info.user_name }}</div></div>
            <div class="detail-text">用户真名：<div style="width:80%">{{ user_info.realname }}</div></div>
            <div class="detail-text">邮箱：<div style="width:80%">{{ user_info.email }}</div></div>
            <div class="detail-text">手机号：<div style="width:80%">{{ user_info.phone }}</div></div>
          </div>
      <div class="detail-text">
      </div>
            <h3>学生课件学习情况：</h3>
            <el-table 
                :data="files_view_times" 
                v-loading="uplistLoading"
                style="width: 100%"
                :height="300">
                <el-table-column 
                prop="name" 
                label="文件名">
                <template slot-scope="scope">
                    {{ scope.row.filename}}
                </template>
                </el-table-column>
                <el-table-column 
                prop="name" 
                label="学习时间">
                <template slot-scope="scope">
                    {{ scope.row.view_time}}
                </template>
                </el-table-column>
            </el-table>
            <h3>学生实验情况：</h3>
            <el-table 
                :data="experiment_condition" 
                v-loading="userlistLoading"
                style="width: 100%"
                :height="300">
                <el-table-column 
                prop="name" 
                label="实验名称">
                <template slot-scope="scope">
                    {{ scope.row.experiment_name}}
                </template>
                </el-table-column>
                <el-table-column 
                prop="name" 
                label="是否提交实验文件">
                <template slot-scope="scope">
                    {{ scope.row.isUploaded}}
                </template>
                </el-table-column>
                <el-table-column 
                prop="name" 
                label="分数">
                <template slot-scope="scope">
                    {{ scope.row.score}}
                </template>
                </el-table-column>
            </el-table>

    </div>
</template>


<script>
import timer from '@/components/timer'
import axios from 'axios';
export default{
    components:{
        timer
    },
    data(){
        return {
            course_id: '',
            user_info: '',
            files_view_times: '',
            experiment_condition: '',
            timer: null,
            time: 0,
            config_list: [
                '0.5核CPU 1G内存',
                '1核CPU 2G内存',
                '2核CPU 4G内存',
                '4核CPU 8G内存',
                '8核CPU 16G内存',
            ],
            config: '1核CPU 2G内存',
            uploadDialogVisible: false,
            uploaded_files: [],
            files_uploaded_to_teacher: [],
            selectedFiles: [],
            selectedExpFiles: [],
            listLoading: false,
            uplistLoading: false,
            userlistLoading: false,
            users_have_uploaded: [],
            user_files: [],
            userfileDialogVisible: false,
            current_rate_student: '',
            rateDialogVisible: false,
            form: {
                score: 0
            },
            status: ''
        }
    },
    methods:{
        getData(){
            let user_id = this.$route.query.user_id
            let formData = new FormData()
            formData.append('user_id', user_id)
            this.$axios({
                method: 'post',
                url: '/teacher/get_student_info/',
                data: formData,
            }).then(
                res => {
                    console.log(res)
                    this.user_info = res.data.data.user_info
                    this.files_view_times = res.data.data.files_view_times
                    this.experiment_condition = res.data.data.experiment_condition
                }
            )
        },
        ReturnToUserInfo(){
            this.$router.push("/manage/user")
        },
    },
    created(){
        this.getData()
    },
}
</script>



<style lang="scss" scoped>
.detail {
  &-container {
    margin: 30px;
  }
  &-text {
    font-size: 20px;
    line-height: 36px;
    white-space: pre-wrap;
    display:flex;
  }
}
</style>