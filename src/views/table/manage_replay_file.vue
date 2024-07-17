<template>
    <div class="app-container">
      <h3 style="position: relative; left:15px">回放文件管理</h3>
      <div style="margin: 15px;"><h4>请先添加章节和小节，然后再添加相应的课件</h4></div>
      <div class="upload-box" style="margin: 15px;">
          <el-button v-on:click="addChapterDialogOpen()" style="margin: 15px;" type="primary">添加章节</el-button>
          <el-button v-on:click="addSectionDialogOpen()" style="margin: 15px;" type="primary">添加小节</el-button>
      </div>
      
      <el-collapse v-model="activeNames">
        <el-collapse-item 
            v-for="chapter in chapters" 
            :key="chapter.chapter_num" 
            :name="chapter.chapter_num.toString()" 
            :title=" chapter.chapter_num + '  ' + chapter.chapter_name"
        >
            <div style="display: flex; margin: 15px;"><h3 style="width: 75%;">{{ chapter.chapter_intro }} </h3>
              <div>  
                <el-button v-on:click="altChapterDialogOpen(chapter)" style="margin: 15px; position: relative;" type="primary">编辑章节信息</el-button>
                <el-button v-on:click="deleteChapter(chapter.chapter_num)" style="margin: 15px; position: relative;" type="danger">删除章节</el-button>
              </div>
            </div>

            <el-table 
                :data="chapter.section_list" 
                style="width: 100%" 
                stripe
            >
                <el-table-column 
                    prop="chapter_number"
                    label="序号" 
                    width="50"
                >
                </el-table-column>
                <el-table-column
                    prop="section_number"
                    label="小节号"
                    width="75">
                </el-table-column>
                <el-table-column
                    prop="section_name"
                    label="小节名称"
                    width="300"
                    >
                </el-table-column>
                <el-table-column
                    prop="section_intro"
                    label="小节介绍">
                </el-table-column>
                <el-table-column
                    label="操作"
                    width="400"
                    align="center">
                    <template slot-scope="scope">
                      <el-button type="primary" @click="altSectionDialogOpen(scope.row)">编辑小节信息</el-button>
                      <el-button type="primary" @click="uploadDialogOpen(scope.row.chapter_number, scope.row.section_number)">上传课件</el-button>
                      <el-button type="danger" @click="deleteSection(scope.row.chapter_number, scope.row.section_number)">删除小节</el-button>

                    </template>
                </el-table-column>
            </el-table>
        </el-collapse-item>
      </el-collapse>
      
    <el-dialog title="添加新章节" :visible.sync="addChapterDialogVisible" width="40%" center>
      <el-form ref="dataForm" :model="chapter_info" label-position="left" label-width="100px" style="width: 400px; margin-left: 50px">
        <el-form-item label="课程章节">
          第<el-input-number v-model="chapter_info.chapter_num" :min="1" :max="100"></el-input-number>章
        </el-form-item>
        <el-form-item label="章节名称" prop="com">
          <el-input v-model="chapter_info.chapter_name" clearable style="width: 300px; margin-left: 20px"/>
        </el-form-item>
        <el-form-item label="章节简介">
          <el-input v-model="chapter_info.chapter_intro" :rows="8" type="textarea" style="width:300px; margin-left: 20px"/>
        </el-form-item>
      </el-form>
      <div slot="footer" class="dialog-footer">
        <el-button @click="addChapterDialogVisible = false"> 取消 </el-button>
        <el-button type="primary" @click="handleAddChapter()"> 提交 </el-button>
      </div>
    </el-dialog>

    <el-dialog title="编辑章节信息" :visible.sync="altChapterDialogVisible" width="40%" center>
      <el-form ref="dataForm" :model="chapter_info" label-position="left" label-width="100px" style="width: 400px; margin-left: 50px">
        <el-form-item label="课程章节" style="width: 300px; margin-left: 20px">
          第{{chapter_info.chapter_num}}章
        </el-form-item>
        <el-form-item label="章节名称" prop="com">
          <el-input v-model="chapter_info.chapter_name" clearable style="width: 300px; margin-left: 20px"/>
        </el-form-item>
        <el-form-item label="章节简介">
          <el-input v-model="chapter_info.chapter_intro" :rows="8" type="textarea" style="width:300px; margin-left: 20px"/>
        </el-form-item>
      </el-form>
      <div slot="footer" class="dialog-footer">
        <el-button @click="addChapterDialogVisible = false"> 取消 </el-button>
        <el-button type="primary" @click="handleAltChapter()"> 提交 </el-button>
      </div>
    </el-dialog>

    <el-dialog title="添加新小节" :visible.sync="addSectionDialogVisible" width="40%" center>
      <el-form ref="dataForm" :model="section_info" label-position="left" label-width="100px" style="width: 400px; margin-left: 50px">
        <el-form-item label="课程章节">
          <div style="display: flex;">
            第<el-select v-model="section_info.chapter_num" placeholder="请选择" style="width: 300px; margin-left: 20px">
            <el-option
              v-for="item in chapters"
              :key="item.chapter_num"
              :label="item.chapter_num"
              :value="item.chapter_num">
            </el-option>
            </el-select>章
          </div>
        </el-form-item>
        <el-form-item label="课程小节">
          第<el-input-number v-model="section_info.section_num" :min="1" :max="100"></el-input-number>节
        </el-form-item>
        <el-form-item label="小节名称" prop="com">
          <el-input v-model="section_info.section_name" clearable style="width: 300px; margin-left: 20px"/>
        </el-form-item>
        <el-form-item label="小节简介">
          <el-input v-model="section_info.section_intro" :rows="8" type="textarea" style="width:300px; margin-left: 20px"/>
        </el-form-item>
      </el-form>
      <div slot="footer" class="dialog-footer">
        <el-button @click="addSectionDialogVisible = false"> 取消 </el-button>
        <el-button type="primary" @click="handleAddSection()"> 提交 </el-button>
      </div>
    </el-dialog>

    <el-dialog title="编辑小节信息" :visible.sync="altSectionDialogVisible" width="40%" center>
      <el-form ref="dataForm" :model="section_info" label-position="left" label-width="100px" style="width: 400px; margin-left: 50px">
        <el-form-item label="课程章节" style="width: 300px; margin-left: 20px">
          <div style="display: flex;">
            第{{section_info.chapter_num}}章
          </div>
        </el-form-item>
        <el-form-item label="课程小节" style="width: 300px; margin-left: 20px">
          第{{section_info.section_num}}节
        </el-form-item>
        <el-form-item label="小节名称" prop="com">
          <el-input v-model="section_info.section_name" clearable style="width: 300px; margin-left: 20px"/>
        </el-form-item>
        <el-form-item label="小节简介">
          <el-input v-model="section_info.section_intro" :rows="8" type="textarea" style="width:300px; margin-left: 20px"/>
        </el-form-item>
      </el-form>
      <div slot="footer" class="dialog-footer">
        <el-button @click="addSectionDialogVisible = false"> 取消 </el-button>
        <el-button type="primary" @click="handleAltSection()"> 提交 </el-button>
      </div>
    </el-dialog>

    <el-dialog title="上传课件" :visible.sync="uploadDialogVisible" width="70%" center>
      <div class="upload-box" style="margin: 15px;">
        课件上传：
        <br>选择PDF文件：<input type="file" ref="singleFile" accept=".pdf" @change="handleSingleFileUpload" style="margin: 15px;"/>
          <el-button v-on:click="submitSingleFile()" :disabled="listLoading" style="margin: 15px;" type="primary">上传PDF文件</el-button>
        <br>选择文件夹（只会上传PDF文件）：<input type="file" ref="folderFiles" multiple webkitdirectory @change="handleFileUpload" style="margin: 15px;"/>
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
                style="width: 100%"
                :height="400">
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
      <div slot="footer" class="dialog-footer">
        <el-button @click="uploadDialogVisible = false"> 关闭 </el-button>
      </div>
    </el-dialog>


    <el-dialog title="添加练习" :visible.sync="uploadExerciseDialogVisible" width="60%" center>
      <div style="margin: 15px;">
        <el-input v-model="exercise_stem" :rows="15" type="textarea" style="width:100%"/>
      </div>
      <div slot="footer" class="dialog-footer">
        <el-button @click="uploadExerciseDialogVisible = false"> 取消 </el-button>
        <el-button type="primary" @click="handleExerciseUpload()"> 提交 </el-button>
      </div>
    </el-dialog>

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
        activeNames: [],
        chapters: [],
        file_list: [],
        singleFile: [], //files uploading
        folderFiles: [],
        largeFile: [],
        tarData: null,
        chapter_info: {
          chapter_num: 1,
          chapter_name: '',
          chapter_intro: '',
        },
        section_info: {
          chapter_num: 1,
          section_num: 1,
          section_name: '',
          section_intro: '',
        },
        listLoading: false,
        percentage: 0,
        uploadstatus: 0,
        addChapterDialogVisible: false,
        addSectionDialogVisible: false,
        uploadDialogVisible: false,
        uploadExerciseDialogVisible: false,
        current_chapter: 1,
        current_section: 1,
        exercise_stem: '',
        altChapterDialogVisible: false,
        altSectionDialogVisible: false,
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
            url: '/teacher/list_replay_sections/',
          }).then(
            res => {
              // console.log(res)
              this.chapters = res.data.data
              this.listLoading = false
            }
          )
        // getList().then(response => {
        //   this.list = response.data.items
        //   this.listLoading = false
        // })
      },
      addChapterDialogOpen(){
        this.addChapterDialogVisible = true
      },
      handleAddChapter(){
        const formData = new FormData()
        formData.append('chapter_num', this.chapter_info.chapter_num)
        formData.append('chapter_name', this.chapter_info.chapter_name)
        formData.append('chapter_intro', this.chapter_info.chapter_intro)
        this.$axios({
            method: 'post',
            url: '/teacher/add_replay_chapter/',
            data: formData,
          }).then(
            res => {
              // console.log(res)
              window.alert(res.data.msg)
              this.fetchData()
            }
          )
        this.addChapterDialogVisible = false
        this.chapter_info.chapter_name = ''
        this.chapter_info.chapter_num = 1
        this.chapter_info.chapter_intro = ''
      },
      addSectionDialogOpen(){
        this.addSectionDialogVisible = true
      },
      handleAddSection(){
        const formData = new FormData()
        formData.append('chapter_num', this.section_info.chapter_num)
        formData.append('section_num', this.section_info.section_num)
        formData.append('section_name', this.section_info.section_name)
        formData.append('section_intro', this.section_info.section_intro)
        this.$axios({
            method: 'post',
            url: '/teacher/add_replay_section/',
            data: formData,
          }).then(
            res => {
              window.alert(res.data.msg)
              this.fetchData()
            }
          )
        this.addSectionDialogVisible = false
        this.section_info.chapter_num = 1
        this.section_info.section_num = 1
        this.section_info.section_name = ''
        this.section_info.section_intro = ''
      },
      uploadDialogOpen(chapter_number, section_number){
        this.current_chapter = chapter_number
        this.current_section = section_number
        this.uploadDialogVisible = true
        const formData = new FormData()
        formData.append('chapter_number', chapter_number)
        formData.append('section_number', section_number)
        this.$axios({
            method: 'post',
            url: '/teacher/load_replay_files_by_section/',
            data: formData
          }).then(
            res => {
              this.file_list = res.data.data
            }
          )
      },
      deleteChapter(chapter_num){
        const formData = new FormData()
        formData.append('chapter_num', chapter_num)
        this.$axios({
          method: 'post',
          url: '/teacher/delete_replay_chapter/',
          data: formData,
        }).then(res=>{
          window.alert(res.data.msg)
          this.fetchData()
        })
      },
      deleteSection(chapter_num, section_num){
        // // console.log(chapter_num)
        // // console.log(section_num)
        const formData = new FormData()
        formData.append('chapter_num', chapter_num)
        formData.append('section_num', section_num)
        this.$axios({
          method: 'post',
          url: '/teacher/delete_replay_section/',
          data: formData,
        }).then(res=>{
          window.alert(res.data.msg)
          this.fetchData()
        })
      },
      fetchFiles(){
        const formData = new FormData()
        formData.append('chapter_number', this.current_chapter)
        formData.append('section_number', this.current_section)
        this.$axios({
            method: 'post',
            url: '/teacher/load_replay_files_by_section/',
            data: formData
          }).then(
            res => {
              this.file_list = res.data.data
            }
          )
      },  
      isSelectable(file) {
        // 这个函数决定哪些文件可以被选中
        // 不写代表允许所有文件被选中
        return true;
      },
      uploadExerciseDialogOpen(chapter_num, section_num){
        this.uploadExerciseDialogVisible = true
        this.current_chapter = chapter_num
        this.current_section = section_num
        const formData = new FormData()
        formData.append('chapter_num', chapter_num)
        formData.append('section_num', section_num)
        this.$axios({
          method: 'post',
          url: '/teacher/load_exercise/',
          data: formData
        }).then(res=>{
          this.exercise_stem = res.data.data
        })
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
          // console.log("未选择大文件");
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
                  url: '/teacher/upload_replay_chunk/',
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
        formData1.append('chapter_number', this.current_chapter)
        formData1.append('section_number', this.current_section)
        // 合并文件块
        this.uploadstatus = 2
        await this.$axios({
          method: 'post',
          url: '/teacher/merge_replay_chunks/',
          data: formData1
        }).then(res=>{
          window.alert(res.data.msg)
        });
        this.listLoading = false
        this.uploadstatus = 0
        this.fetchFiles()
      },
      submitFile(){
          this.listLoading = true
          let formData = new FormData();
          let blob = new Blob([this.tarData], {type: "application/x-tar"}); 
          formData.append('tarFile', blob);
          formData.append('chapter_number', this.current_chapter)
          formData.append('section_number', this.current_section)
          this.$axios({
            method: 'post',
            url: '/teacher/upload_replay_files/',
            data: formData,
            headers: {
              'Content-Type': 'multipart/form-data'
            }
          })
          .then(response => {
            // console.log(response);
            this.fetchFiles()
            this.files = [];
            this.listLoading = false;
          })
      },

      submitSingleFile(){
          this.listLoading = true
          let formData = new FormData();
          let blob = new Blob([this.singleTarData], {type: "application/x-tar"}); // Use the tarData for the single file created in the method handleSingleFileUpload
          formData.append('tarFile', blob, this.$refs.singleFile.files[0].name + '.tar'); // Append the tarFile to the form data
          formData.append('chapter_number', this.current_chapter)
          formData.append('section_number', this.current_section)
          this.$axios({
              method: 'post',
              url: '/teacher/upload_replay_files/',
              data: formData,
              headers: {
                  'Content-Type': 'multipart/form-data'
              }
          })
          .then(response => {
            // console.log(response);
            this.fetchFiles()
            this.tarData = []; // Reset the tarData for next usage
            this.listLoading = false;
          })
      },
      onDelete(path){
        // console.log(path)
        const formData = new FormData();
        formData.append('file_paths', JSON.stringify(path))
        this.$axios({
            method: 'post',
            url: '/teacher/delete_replay_files/',
            data: formData,
          }).then(response => {
            window.alert(response.data.msg)
            this.fetchFiles()
          })
      },
      onDeleteSelectedFiles(){
        // console.log(this.selectedFiles)
        this.onDelete(this.selectedFiles); // Call onDelete with the paths of the selected files
      },
      handleExerciseUpload(){
        const formData = new FormData();
        formData.append('chapter_num', this.current_chapter)
        formData.append('section_num', this.current_section)
        formData.append('exercise_stem', this.exercise_stem)
        this.$axios({
            method: 'post',
            url: '/teacher/upload_exercise/',
            data: formData,
          }).then(response => {
            window.alert(response.data.msg)
            this.uploadExerciseDialogVisible = false
          })
      },
      downloadFile(filename) {
            let formData = new FormData();
            formData.append('filename', filename);
            // console.log(filename)
            this.$axios({
                method: 'post',
                url: '/teacher/download_replay_file/', 
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
            filetype: 2,
          }
        })
      },
      altChapterDialogOpen(chapter){
        this.altChapterDialogVisible = true
        this.chapter_info.chapter_num = chapter.chapter_num
        this.chapter_info.chapter_name = chapter.chapter_name
        this.chapter_info.chapter_intro = chapter.chapter_intro
      },
      handleAltChapter(){
        const formData = new FormData()
        formData.append('chapter_num', this.chapter_info.chapter_num)
        formData.append('chapter_name', this.chapter_info.chapter_name)
        formData.append('chapter_intro', this.chapter_info.chapter_intro)
        this.$axios({
            method: 'post',
            url: '/teacher/alt_replay_chapter/',
            data: formData,
          }).then(
            res => {
              // console.log(res)
              window.alert(res.data.msg)
              this.fetchData()
            }
          )
        this.altChapterDialogVisible = false
        this.chapter_info.chapter_name = ''
        this.chapter_info.chapter_num = 1
        this.chapter_info.chapter_intro = ''
      },
      altSectionDialogOpen(row){
        this.altSectionDialogVisible = true
        this.section_info.chapter_num = row.chapter_number
        this.section_info.section_num = row.section_number
        this.section_info.section_name = row.section_name
        this.section_info.section_intro = row.section_intro
      },
      handleAltSection(){
        const formData = new FormData()
        formData.append('chapter_num', this.section_info.chapter_num)
        formData.append('section_num', this.section_info.section_num)
        formData.append('section_name', this.section_info.section_name)
        formData.append('section_intro', this.section_info.section_intro)
        this.$axios({
            method: 'post',
            url: '/teacher/alt_replay_section/',
            data: formData,
          }).then(
            res => {
              window.alert(res.data.msg)
              this.fetchData()
            }
          )
        this.altSectionDialogVisible = false
        this.section_info.chapter_num = 1
        this.section_info.section_num = 1
        this.section_info.section_name = ''
        this.section_info.section_intro = ''
      },
    }
  }
  </script>
  
  <style scoped>
     /deep/.el-collapse {
      background-color: #D5E8FF !important;
    }
    /deep/.el-collapse,.el-collapse-item__wrap {
      border: 40px solid white;
      width: 100%;
      position: relative;
      left:0;
    }
    /deep/ .el-collapse-item__header {
        color: black;
        background-color: #D5E8FF !important;
        height: 80px;
        font-size: 20px;
        margin-left: 10px;
    }
    /deep/ .el-collapse-item__content {
        color: black;
        background-color: #e1edfd !important;
        position: relative;
        top:-20px;
    }
  </style>