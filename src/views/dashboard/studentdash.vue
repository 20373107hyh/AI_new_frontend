<template>
  <div class="dashboard-container">
    <div class="dashboard-text">用户名: {{ userinfo.username }}</div>
    <div class="dashboard-text">姓名: {{ userinfo.realname }}</div>
    <div class="dashboard-text">邮箱: {{userinfo.email}}</div>
    <div class="dashboard-text">电话: {{userinfo.phone}}</div>
    <div class="dashboard-text">原密码:<el-input
          :key="passwordType"
          ref="password"
          v-model="originpassword"
          :type="passwordType"
          name="password"
          tabindex="2"
          auto-complete="on"
          style="width: 300px;"
        />
    </div>
    <div class="dashboard-text">新密码:<el-input
          :key="passwordType"
          ref="password"
          v-model="changepassword"
          :type="passwordType"
          name="password"
          tabindex="2"
          auto-complete="on"
          style="width: 300px;"
        />
    </div>
    <div class="dashboard-text">确认密码:<el-input
          :key="passwordType"
          ref="password"
          v-model="surepassword"
          :type="passwordType"
          name="password"
          tabindex="2"
          auto-complete="on"
          style="width: 300px;"
        />
    </div>
    <div class="dashboard-text"><el-button type="primary" @click="handleChangePassword"> 修改密码 </el-button></div>
  </div>
</template>

<script>
import { mapGetters } from 'vuex'

export default {
  name: 'Dashboard',
  data(){
    return{
      userinfo: {},
      originpassword: '',
      changepassword: '',
      surepassword: '',
    }
  },
  methods:{
    getInfo(){
      let user_id = localStorage.getItem('user_id');
      let formData = new FormData()
            formData.append('user_id', user_id)
            this.$axios({
                method: 'post',
                url: '/student/get_user_info/',
                data: formData,
            }).then(
                res => {
                    console.log(res)
                    this.userinfo = res.data.data
                }
            )
    },
    handleChangePassword(){
      let user_id = localStorage.getItem('user_id');
      const formData = new FormData()
      formData.append('user_id', user_id)
      formData.append('old_password', this.originpassword)
      formData.append('new_password', this.changepassword)
      formData.append('sure_password', this.surepassword)
      this.$axios({
        method: 'post',
        url:'/user/change_password/',
        data: formData,
      }).then(res=>{
        console.log(res)
        window.alert(res.data.msg)
      })
      this.originpassword = ''
      this.changepassword = ''
      this.surepassword = ''
    }

  },
  created(){
    this.getInfo()
  }
}
</script>

<style lang="scss" scoped>
.dashboard {
  &-container {
    margin: 30px;
  }
  &-text {
    font-size: 30px;
    line-height: 46px;
  }
}
</style>
