<template>
  <div>
    <el-button type="primary" @click="handleReturn()" style="margin: 15px;"> 返回容器列表 </el-button>
    <div style="margin: 15px;">
      <div class="detail-text">名称：{{ container_info.container_name}} </div>
      <div class="detail-text">配置：{{ container_info.cpu_num }}核CPU, {{ container_info.mem_size }}G内存 </div>
      <div class="detail-text">SSH端口: {{ container_info.ssh_port }}</div>
      <div class="detail-text">HTTP端口: {{ container_info.http_port }}</div>
      <div class="detail-text">SSH命令: ssh root@121.40.203.195 -p {{ container_info.ssh_port }}</div>
      <div class="detail-text">SSH密码: {{ container_info.ssh_password }}</div>
    </div>
    <div >
      <h3 style="position: relative; left:15px">上传文件至镜像：<el-button type="primary" @click="handleEnter()"> 进入容器创建新文件/预载软件 </el-button></h3>
      <div class="upload-box" style="margin: 15px;">
        选择文件：<input type="file" ref="singleFile" multiple @change="handleSingleFileUpload" style="margin: 15px;"/>
          <el-button v-on:click="submitSingleFile()" :disabled="listLoading" style="margin: 15px;" type="primary">上传文件</el-button>
        选择文件夹：<input type="file" ref="folderFiles" multiple webkitdirectory @change="handleFileUpload" style="margin: 15px;"/>
          <el-button v-on:click="submitFile()" :disabled="listLoading" style="margin: 15px;" type="primary">上传文件夹</el-button>
          <br>若文件大小较大（如视频文件），请通过此方式上传：<input type="file" ref="largeFile" @change="handleLargeFileUpload" style="margin: 15px;"/>
          <div style="margin: 15px;" v-if="uploadstatus === 1"> 正在上传大文件： {{percentage}}%，中途请不要离开或者刷新本界面</div>
          <div style="margin: 15px;" v-if="uploadstatus === 2"> 正在合并大文件</div>
          <el-button v-on:click="submitLargeFile()" :disabled="listLoading" style="margin: 15px;" type="primary">上传大文件</el-button>
          <div class="fileUpload">
            指定文件上传路径：
            <el-input placeholder="请输入路径，注意路径不以/结尾，文件夹名不以.开头" v-model="path" style="width: 70%;">
              <template slot="prepend"> {{ container_info.workdir }}/ </template>
            </el-input>
          </div>
      </div>
    </div>
    <!-- <ul v-if="uploaded_files.length > 0">
      <li v-for="(file, index) in uploaded_files" :key="index">
        {{ file.filename }} &nbsp;&nbsp; &nbsp; &nbsp; &nbsp;  {{ file.path }}  
        <button @click="onDelete(file.path)">Delete</button>
      </li>
    </ul> -->
    <div>
      <div style="margin: 15px;">
        <h3>管理已上传文件&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <el-button @click="onDeleteSelectedFiles" type="danger">删除所选文件</el-button></h3>
        <el-table 
          :data="uploaded_files" 
          v-loading="listLoading"
          @selection-change="handleSelectionChange"
          style="width: 100%"
          :height="300">
          <el-table-column
            type="selection"
            :selectable="isSelectable"
            width="55">
          </el-table-column>
          <el-table-column 
            prop="name" 
            label="文件名">
            <template slot-scope="scope">
              {{ scope.row.filename}}
            </template>
          </el-table-column>
          <el-table-column 
            prop="path" 
            label="文件路径">
            <template slot-scope="scope">
              {{ scope.row.path}}
            </template>
          </el-table-column>
          <el-table-column 
            prop="operation" 
            label="操作">
            <template slot-scope="scope">
              <el-button @click="onDelete([scope.row.path])" type="danger">删除</el-button>
            </template>
          </el-table-column>
        </el-table>
      </div>
    </div>
  </div>
</template>
  
<script>
import tarStream from 'tar-stream';
import tar from 'tar-js';
import buffer from 'buffer/';
import bl from 'bl';
export default {
  data() {
    return {
      singleFile: [], //files uploading
      folderFiles: [],
      largeFile: null,
      tarData: null,
      singleTarData: null,
      selectedFiles: [],
      uploaded_files:[], //已上传的文件
      container_info: {},
      path: '',
      listLoading: true,
      percentage: 0,
      uploadstatus: 0,
    };
  },
  methods: {
    handleSelectionChange(selected) {
      this.selectedFiles = selected.map(file => file.path);
    },
    loadFiles(name){
      this.listLoading = true
      const formData = new FormData()
      let user_id = localStorage.getItem('user_id')
      formData.append('user_id', user_id)
      formData.append('container_name', name)
      this.$axios({
        method: 'post',
        url: '/teacher/load_files/',
        data: formData,
      }).then(res => {
        // // console.log(res)
        this.uploaded_files = res.data.data
        this.listLoading = false
      })
      this.files = []
    },
    handleFileUpload() {
      this.listLoading = true
      let filesArray = Array.from(this.$refs.folderFiles.files);
      let filePromises = filesArray.map(file => {
        return new Promise((resolve, reject) => {
            let reader = new FileReader();
            reader.onload = () => {
            resolve({ name: file.webkitRelativePath, data: reader.result });
            };
            reader.onerror = reject;
            reader.readAsArrayBuffer(file);
        });
        });

      Promise.all(filePromises).then(fileDataArray => {
        let pack = tarStream.pack();

        for(let fileData of fileDataArray) {
            pack.entry({ name: fileData.name }, new Buffer(fileData.data));
        }

        pack.finalize();
        pack.pipe(bl((err, data) => {
            if (err) {
              console.error(err);
              return;
            }

            let blob = new Blob([data], { type: "application/x-tar" });
            let reader = new FileReader();
            reader.onloadend = () => {
              this.tarData = reader.result;
              // 在这里加入日志
            };
            reader.readAsArrayBuffer(blob);
            this.listLoading = false
          }));
        }).catch(error => {
          console.error(error);
        });
    },
    handleSingleFileUpload() {
          let file = this.$refs.singleFile.files[0];
          let reader = new FileReader();
          reader.onload = () => {
                let pack = tarStream.pack();
                pack.entry({ name: file.name }, new Buffer(reader.result));
                pack.finalize();
                pack.pipe(bl((err, data) => {
                  if (err) {
                    console.error(err);
                    return;
                  }
                            
                  let blob = new Blob([data], { type: "application/x-tar" });
                  let reader2 = new FileReader();
                  reader2.onloadend = () => {
                    this.singleTarData = reader2.result; // Directly set the tarData for the single file
                  };
                  reader2.readAsArrayBuffer(blob);
                  this.listLoading = false
                }));
          };

          reader.onerror = (error) => {
              console.error(error);
          }
  
          reader.readAsArrayBuffer(file);
      },
    handleLargeFileUpload() {
      this.largeFile = this.$refs.largeFile.files[0]; // 获取文件
    },
    async submitLargeFile() {
      if (!this.largeFile) {
        // // console.log("未选择大文件");
        return;
      }
      this.listLoading = true
      const chunkSize = 1024 * 1024 * 20; // 设置chunk大小为20MB
      this.uploadstatus = 1
      const chunks = Math.ceil(this.largeFile.size / chunkSize); // 计算文件需要分割的块数
      let user_id = localStorage.getItem('user_id')
      let container_name = this.container_info.container_name

      for (let i = 0; i < chunks; i++) {
        let result = Math.floor((i / chunks) * 100);
        this.percentage = result
        const start = i * chunkSize;
        const end = start + chunkSize >= this.largeFile.size ? this.largeFile.size : start + chunkSize;
        const chunk = new Blob([this.largeFile.slice(start, end)], {type: 'text/plain'}); // 分割文件
        let formData = new FormData();
        formData.append('file', chunk, `chunk-${i}-${this.largeFile.name}`);
        formData.append('fileName', this.largeFile.name);
        formData.append('user_id', user_id)
        formData.append('container_name', container_name)
        formData.append('path', this.path)
          
        // 使用axios上传文件块(chunk)

        try {
            await this.$axios({
                method: 'post',
                url: '/teacher/upload_chunk/',
                data: formData,
                headers: { 'Content-Type': 'multipart/form-data' } 
            });
        } catch (error) {
            console.error('上传过程中出现错误：', error);
            this.listLoading = false;
            return;
        }
      }
      const formData1 = new FormData()
      formData1.append('filename', this.largeFile.name)
      formData1.append('chunks', chunks)
      formData1.append('user_id', user_id)
      formData1.append('container_name', container_name)
      formData1.append('path', this.path)
      this.uploadstatus = 2
      // 合并文件块
      await this.$axios({
        method: 'post',
        url: '/teacher/merge_chunks/',
        data: formData1
      });
      this.uploadstatus = 0
      this.listLoading = false
      this.loadFiles(container_name)
    },
    isSelectable(file) {
      // 这个函数决定哪些文件可以被选中
      // 不写代表允许所有文件被选中
      return true;
    },
    submitFile(){
        this.listLoading = true
        let formData = new FormData();
        let user_id = localStorage.getItem('user_id')
        let container_name = this.container_info.container_name
        let blob = new Blob([this.tarData], {type: "application/x-tar"}); 
        formData.append('tarFile', blob);
        formData.append('user_id', user_id);
        formData.append('container_name', container_name);
        formData.append('path', this.path);
        
        this.$axios({
          method: 'post',
          url: '/teacher/upload_file/',
          data: formData,
          headers: {
            'Content-Type': 'multipart/form-data'
          }
        })
        .then(response => {
          // // console.log(response);
          this.loadFiles(container_name);
          this.files = [];
          this.listLoading = false;
        })
    },

    submitSingleFile(){
        this.listLoading = true
        let formData = new FormData();
        let user_id = localStorage.getItem('user_id')
        let container_name = this.container_info.container_name
        let blob = new Blob([this.singleTarData], {type: "application/x-tar"}); // Use the tarData for the single file created in the method handleSingleFileUpload
        formData.append('tarFile', blob, this.$refs.singleFile.files[0].name + '.tar'); // Append the tarFile to the form data
        formData.append('user_id', user_id);
        formData.append('container_name', container_name);
        formData.append('path', this.path);

        this.$axios({
            method: 'post',
            url: '/teacher/upload_file/',
            data: formData,
            headers: {
                'Content-Type': 'multipart/form-data'
            }
        })
        .then(response => {
          // console.log(response);
          this.loadFiles(container_name);
          this.tarData = []; // Reset the tarData for next usage
          this.listLoading = false;
        })
    },
    onDelete(path){
      // console.log(path)
      const formData = new FormData();
      let user_id = localStorage.getItem('user_id')
      let container_name = this.container_info.container_name
      formData.append('user_id', user_id)
      formData.append('container_name', container_name)
      formData.append('file_paths', JSON.stringify(path))
      this.$axios({
          method: 'post',
          url: '/teacher/delete_file/',
          data: formData,
        }).then(response => {
          // console.log(response)
          this.loadFiles(container_name)
        })
    },
    onDeleteSelectedFiles(){
      // console.log(this.selectedFiles)
      this.onDelete(this.selectedFiles); // Call onDelete with the paths of the selected files
    },
    search_container(){
      const formData = new FormData();
      let container_id = this.$route.query.container_id
      formData.append('container_id', container_id)
      this.$axios({
          method: 'post',
          url: '/teacher/search_container/',
          data: formData,
        }).then(response => {
          // console.log(response)
          this.container_info = response.data.data
          this.loadFiles(response.data.data.container_name)
        })
    },
    handleEnter(){
      let url = this.container_info.container_url
      // console.log(url)
      window.open(url, '_blank');
    },
    handleReturn(){
      this.$router.push('/manage/image')
    },
  },
  created(){
      this.search_container()
  },
}
</script>
  

<style lang="scss" scoped>
.detail {
  &-text {
    font-size: 20px;
    line-height: 36px;
    white-space: pre-wrap;
    display:flex;
  }
}
</style>