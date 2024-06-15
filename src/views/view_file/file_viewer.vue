<template>
    <div>
        <div class="viewFile">
            <div style="height:10%; position: relative; left: 25%; display: flex;"> 
                <h3 style="margin: 15px; position: relative; top: 10px;">正在查看课件：{{filename}}</h3>
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
                style="width: 50%; height: 80%; position: relative; left: 25%;"
            />


            <video-player :options="playerOptions" 
                v-if="fileType.startsWith('video/')" 
                >
            </video-player>

            <div v-if="fileType !== 'application/pdf' && !fileType.startsWith('video/')" style="height:10%; position: relative; left: 30%; display: flex;">
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
        }
      };
    },
    methods: {
      getFile() {
        const formData = new FormData();
        let filename = this.$route.query.filename
        formData.append('filename', filename);
  
        this.$axios({
          method: 'post',
          url: '/teacher/download_course_file/',
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
          if (this.fileType.startsWith('video/')) {
            this.playerOptions.sources[0].src = url;
            this.playerOptions.sources[0].type = this.fileType;
          }
        });
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
            this.$router.push('/manage/course_file')
        },
    },
    created() {
      this.getFile();
      this.filename = this.$route.query.filename
    }
  }
  </script>

