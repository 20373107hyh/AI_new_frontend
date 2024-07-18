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
            <h3>学生课件学习情况： 今日总学习时间 {{ total_view_time }}分钟</h3>
            <el-table :data="files_view_times" style="width: 1500px; height: 400px; overflow: auto; " class="custom-table">
                <el-table-column label="文件名" width="200">
                    <template slot-scope="scope" >
                        {{ scope.row.file_name }}
                    </template>
                </el-table-column>
                <!-- 动态生成日期列 -->
                <el-table-column
                v-for="date in uniqueDates"
                :key="date"
                :label="date"
                width="100"

                >
                    <template slot-scope="scope">
                        {{ getViewTime(scope.row.records, date) }}
                    </template>
                </el-table-column>
                
            </el-table>
            <h3>学生课程回放学习情况</h3>
            <el-table :data="replay_view_times" style="width: 1500px; height: 400px; overflow: auto; " class="custom-table">
                <el-table-column label="文件名" width="200">
                    <template slot-scope="scope" >
                        {{ scope.row.file_name }}
                    </template>
                </el-table-column>
                <!-- 动态生成日期列 -->
                <el-table-column
                v-for="date in uniqueDates"
                :key="date"
                :label="date"
                width="100"

                >
                    <template slot-scope="scope">
                        {{ getViewTime(scope.row.records, date) }}
                    </template>
                </el-table-column>
                
            </el-table>
            <h3>学生实验情况：</h3>
            <el-table 
                :data="experiment_condition" 
                v-loading="userlistLoading"
                style="width: 100%"
                :height="300"
                class="custom-table">
                <el-table-column 
                prop="name" 
                label="实验名称">
                <template slot-scope="scope">
                    {{ scope.row.experiment_name}}
                </template>
                </el-table-column>
                <el-table-column 
                prop="name" 
                label="实验次数">
                <template slot-scope="scope">
                    {{ scope.row.count}}
                </template>
                </el-table-column>
                <el-table-column 
                prop="name" 
                label="实验总时长">
                <template slot-scope="scope">
                    {{ scope.row.time}}分钟
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
            replay_view_times: '',
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
            status: '',
            uniqueDates: [],
            total_view_time: 0,
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
                    // console.log(res)
                    this.user_info = res.data.data.user_info
                    this.files_view_times = res.data.data.files_view_times
                    this.replay_view_times = res.data.data.replay_view_times
                    this.experiment_condition = res.data.data.experiment_condition
                    this.total_view_time = res.data.data.total_view_time
                    this.uniqueDates = res.data.data.files_view_times.flatMap(item => item.records.map(record => record.view_date)).filter((date, index, self) => self.indexOf(date) === index);
                }
            )
        },
        getViewTime(records, date){
            for (let record of records) {
                if (record.view_date === date) {
                return record.view_time;
                }
            }
            return null;
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



<style scoped>
.detail-container {
    margin: 30px;
}
.detail-text {
    font-size: 16px;
    line-height: 30px;
    white-space: pre-wrap;
    display:flex;
  }
.custom-table >>>.el-table__header-wrapper {
  position: sticky;
  top: 0;
  background-color: white;
  z-index: 1;
}

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