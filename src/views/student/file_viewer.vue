<template>
    <div>
        <div class="viewFile">
            <div style="height:10%; position: relative; left: 18%; display: flex;"> 
                <h3 style="margin: 15px; position: relative; top: 10px;">正在查看课件：{{filename}} 已学习 {{watch_time}} 分钟</h3>
                <el-button @click="prevPage" :disabled="page === 1" type="primary" v-if="fileType === 'application/pdf'" style="margin: 15px;">上一页</el-button>
                <el-button @click="nextPage" :disabled="page === pages" type="primary" v-if="fileType === 'application/pdf'" style="margin: 15px;">下一页</el-button>
                <h3 style="margin: 15px; position: relative; top: 10px;" v-if="fileType === 'application/pdf'"> 第 {{ page }} 页 共 {{ pages }}页 </h3>
                <el-button type="primary" @click="handleReturn()" style="margin: 15px;"> 返回课件列表 </el-button>
            </div>
            <vuePdf
                :src="fileUrl"
                :page="page"
                @num-pages="p => pages =p"
                v-if="fileType === 'application/pdf'"
                style="width: 80%; position: relative; left: 10%;"
            />


            <video-player :options="playerOptions" 
                v-if="fileType.startsWith('video')" 
                ref="videoPlayer"
                @play="videoPlay"
                @pause="videoPause"
                >
            </video-player>
            <div v-if="fileType === '' " style="height:10%; position: relative; left: 30%; display: flex;">
              <h1 > 文件正在加载中，请稍后 </h1>
            </div>

            <div v-if="fileType !== 'application/pdf' && !fileType.startsWith('video') && fileType !=='' " style="height:10%; position: relative; left: 30%; display: flex;">
              <h1 > 该文件格式暂不支持线上查看，请下载到本地 </h1>
            </div>
        </div>

    </div>
  </template>
  
  <script>
  import vuePdf from 'vue-pdf'
  import { videoPlayer } from 'vue-video-player'

  export default {
    components: {
      vuePdf,
      videoPlayer
    },
    data() {
      return {
        fileUrl: '',
        filename: '',
        fileType: '',
        page: 1,
        pages: 0,
        playerOptions: {
            playbackRates: [0.5, 1.0, 1.5, 2.0], // 播放速度
            autoplay: false, // 如果true,浏览器准备好时开始回放。
            muted: false, // 默认情况下将会消除任何音频。
            loop: false, // 导致视频一结束就重新开始。
            preload: 'auto', // 视频文件的预加载。auto: 当页面加载后开始加载视频; metadata: 当页面加载后只加载视频信息; none: 当页面加载后不加载视频。
            language: 'zh-CN',
            aspectRatio: '16:9', // 将播放器置于流畅模式，并在计算播放器的动态大小时使用该值。值应该代表一个比例 - 用冒号分隔的两个数字（例如"16:9"或"4:3"）
            fluid: true, // 当true时，Video.js player将拥有流体大小。换句话说，它将按比例缩放以适应其容器。
            sources: [
                {
                    type: "", 
                    src: ""
                }
            ],
            controlBar: {
                timeDivider: true,
                durationDisplay: true,
                remainingTimeDisplay: false,
                fullscreenToggle: true // 全屏按钮
            }
        },
        intervalId: null,
        watch_time: 0,
        videoIsPlaying: false
      };
    },
    methods: {
      isPdf(filename) {
        return filename.toLowerCase().endsWith('.pdf');
      },
      isVideo(filename){
        return filename.toLowerCase().endsWith('.mp4') || filename.toLowerCase().endsWith('.webm');
      },
      getFile() {
        const formData = new FormData();
        let filename = this.$route.query.filename
        this.filename = filename
        let user_id = localStorage.getItem('user_id')
        formData.append('user_id', user_id)
        formData.append('filename', filename);
        this.fileType = ''

          if(!this.isVideo(filename)){
            this.$axios({
              method: 'post',
              url: '/student/download_course_file/',
              data: formData,
              responseType: 'blob',
              crossOrigin: "Anonymous"
            }).then(response => {
              const url = window.URL.createObjectURL(new Blob([response.data]));
              this.fileUrl = url;
              console.log(this.fileUrl)
              // 更新文件类型
              this.fileType = response.headers['content-type']; // 更新为实际的文件类型
              console.log(this.fileType)
              if (this.fileType === 'application/pdf') {
                // 如果文件类型为PDF，启动计时器
                this.startInterval();
              } else if (this.fileType.startsWith('video/')) {
                  this.playerOptions.sources[0].src = url;
                  this.playerOptions.sources[0].type = this.fileType;
              }
            });
          }
          else{
            let extension = filename.split('.').pop();
            this.fileType = 'video/' + extension
            console.log(this.fileType)
            let fileUrl = "http://buaacdn.xiaolegou.top/" + filename
            this.fileUrl = fileUrl
            this.playerOptions.sources[0].src = fileUrl;
            this.playerOptions.sources[0].type = this.fileType;
          }

      },
      prevPage() {
            if ( this.page > 1 )
            this.page--;
        },
        nextPage() {
            if ( this.page < this.pages )
            this.page++;
        },
        handleReturn(){
            this.$router.push('/course_file_list')
        },
        update_view_time(flag){
          const formData = new FormData();
          let user_id = localStorage.getItem('user_id')
          let filename = this.$route.query.filename
          formData.append('user_id', user_id)
          formData.append('filename', filename)
          formData.append('flag', flag)
          this.$axios({
            method: 'post',
            url: '/teacher/update_view_course_file/',
            data: formData
          }).then(response=>{
            console.log(response)
            this.watch_time = response.data.data.view_time
          })
        },
        videoPlay() {
            if (!this.intervalId) {
                this.startInterval();
            }
            console.log('play')
        },
        videoPause() {
            this.videoIsPlaying = false;
            this.clearInterval();
            console.log('pause')
        },
        startInterval() {
            this.intervalId = setInterval(() => {
                this.update_view_time(1);
            }, 60000);
            console.log(this.intervalId)
        },
        clearInterval() {
            if (this.intervalId) {
                clearInterval(this.intervalId);
                this.intervalId = null;
            }
        }
    },
    mounted() {
      this.getFile();
      this.filename = this.$route.query.filename;
      this.update_view_time(0);       
      },
    beforeDestroy() {
        // 当组件被销毁时，清除计时器
        this.clearInterval()
    }
  }
  </script>

