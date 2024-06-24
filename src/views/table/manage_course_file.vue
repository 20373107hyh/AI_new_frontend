<template>
    <div class="app-container">
      <h3 style="position: relative; left:15px">课程文件管理</h3>
      <div class="upload-box" style="margin: 15px;">
        PDF课件上传：
        <br>选择PDF文件：<input type="file" ref="singleFile" accept=".pdf" @change="handleSingleFileUpload" style="margin: 15px;"/>
          <el-button v-on:click="submitSingleFile()" :disabled="listLoading" style="margin: 15px;" type="primary">上传PDF文件</el-button>
        选择文件夹（只会上传PDF文件）：<input type="file" ref="folderFiles" multiple webkitdirectory @change="handleFileUpload" style="margin: 15px;"/>
          <el-button v-on:click="submitFile()" :disabled="listLoading" style="margin: 15px;" type="primary">上传文件夹</el-button>
        <br>视频文件请通过此方式上传，支持MP4和WEBM格式：<input type="file" ref="largeFile" accept=".webm,.mp4" @change="handleLargeFileUpload" style="margin: 15px;"/>
          <el-button v-on:click="submitLargeFile()" :disabled="listLoading" style="margin: 15px;" type="primary">上传视频文件</el-button>
          <br>注意，请不要通过上传视频的板块上传其他类型的文件，否则会出现上传文件无法删除的情况
          <div style="margin: 15px;" v-if="uploadstatus === 1"> 正在上传视频文件： {{percentage}}%，中途请不要离开或者刷新本界面</div>
          <div style="margin: 15px;" v-if="uploadstatus === 2"> 正在合并视频文件，中途请不要离开或者刷新本界面</div>
      </div>
  
      <el-table 
                :data="file_list" 
                v-loading="listLoading"
                @selection-change="handleSelectionChange"
                style="width: 100%"
                :height="1000">
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
                prop="name" 
                label="操作">
                <template slot-scope="scope">
                    <el-button @click="viewFile(scope.row.filename)" type="primary">查看文件</el-button>
                    <el-button @click="downloadFile(scope.row.filename)" type="primary">下载文件</el-button>
                    <el-button @click="onDelete([scope.row.filename])" type="danger">删除文件</el-button>
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
        largeFile: [],
        tarData: null,
        listLoading: false,
        percentage: 0,
        uploadstatus: 0,
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
            url: '/teacher/load_course_files/',
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
      isSelectable(file) {
        // 这个函数决定哪些文件可以被选中
        // 不写代表允许所有文件被选中
        return true;
      },
      handleSelectionChange(selected) {
        this.selectedFiles = selected.map(file => file.path);
      },
      handleFileUpload() {
        this.listLoading = true
        let filesArray = Array.from(this.$refs.folderFiles.files);
        let filteredFilesArray = filesArray.filter(file => file.type === 'application/pdf');

        let filePromises = filteredFilesArray.map(file => {
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
          if (file.type !== 'application/pdf') {
            window.alert('文件不是PDF!');
            return;
          }
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
          console.log("未选择大文件");
          return;
        }
        this.listLoading = true
        this.uploadstatus = 1
        const chunkSize = 1024 * 1024 * 5; // 设置chunk大小为5MB
        const chunks = Math.ceil(this.largeFile.size / chunkSize); // 计算文件需要分割的块数

        for (let i = 0; i < chunks; i++) {
          setTimeout(() => {
            // 这里编写您要延迟执行的函数逻辑
          }, 1000);
          let result = Math.floor((i / chunks) * 100);
          this.percentage = result
          const start = i * chunkSize;
          const end = start + chunkSize >= this.largeFile.size ? this.largeFile.size : start + chunkSize;
          const chunk = new Blob([this.largeFile.slice(start, end)], {type: 'text/plain'}); // 分割文件
          let formData = new FormData();
          formData.append('file', chunk, `chunk-${i}-${this.largeFile.name}`);
          formData.append('fileName', this.largeFile.name);
          
          // 使用axios上传文件块(chunk)
          try {
              await this.$axios({
                  method: 'post',
                  url: '/teacher/upload_course_chunk/',
                  data: formData,
                  headers: { 'Content-Type': 'multipart/form-data' } 
              });
          } catch (error) {
              window.alert('上传过程中出现错误：', error);
              this.listLoading = false;
              return;
          }
        }
        const formData1 = new FormData()
        formData1.append('filename', this.largeFile.name)
        formData1.append('chunks', chunks)
        // 合并文件块
        this.uploadstatus = 2
        await this.$axios({
          method: 'post',
          url: '/teacher/merge_course_chunks/',
          data: formData1
        }).then(res=>{
          window.alert(res.data.msg)
        });
        this.listLoading = false
        this.uploadstatus = 0
        this.fetchData()
      },
      submitFile(){
          this.listLoading = true
          let formData = new FormData();
          let blob = new Blob([this.tarData], {type: "application/x-tar"}); 
          formData.append('tarFile', blob);
          
          this.$axios({
            method: 'post',
            url: '/teacher/upload_course_files/',
            data: formData,
            headers: {
              'Content-Type': 'multipart/form-data'
            }
          })
          .then(response => {
            console.log(response);
            this.fetchData();
            this.files = [];
            this.listLoading = false;
          })
          .catch(error => console.log(error));
      },

      submitSingleFile(){
          this.listLoading = true
          let formData = new FormData();
          let blob = new Blob([this.singleTarData], {type: "application/x-tar"}); // Use the tarData for the single file created in the method handleSingleFileUpload
          formData.append('tarFile', blob, this.$refs.singleFile.files[0].name + '.tar'); // Append the tarFile to the form data
          this.$axios({
              method: 'post',
              url: '/teacher/upload_course_files/',
              data: formData,
              headers: {
                  'Content-Type': 'multipart/form-data'
              }
          })
          .then(response => {
            console.log(response);
            this.fetchData();
            this.tarData = []; // Reset the tarData for next usage
            this.listLoading = false;
          })
          .catch(error => console.log(error));
      },
      onDelete(path){
        console.log(path)
        const formData = new FormData();
        formData.append('file_paths', JSON.stringify(path))
        this.$axios({
            method: 'post',
            url: '/teacher/delete_course_files/',
            data: formData,
          }).then(response => {
            window.alert(response.data.msg)
            this.fetchData()
          })
            .catch(error => console.log(error));
      },
      onDeleteSelectedFiles(){
        console.log(this.selectedFiles)
        this.onDelete(this.selectedFiles); // Call onDelete with the paths of the selected files
      },
      
      downloadFile(filename) {
            let formData = new FormData();
            formData.append('filename', filename);
            console.log(filename)
            this.$axios({
                method: 'post',
                url: '/teacher/download_course_file/', 
                data: formData,
                responseType: 'blob',
            })
            .then((response) => {
                const url = window.URL.createObjectURL(new Blob([response.data]));
                const link = document.createElement('a');
                link.href = url;
                link.setAttribute('download', filename);
                document.body.appendChild(link);
                link.click();
            })
            .catch((error) => {
                console.error(error);
            });
        },
      viewFile(filename){
        this.$router.push({
          path:'/manage/view_course_file',
          query:{
            filename: filename,
          }
        })
      },
    }
  }
  </script>
  