<template>
  <div class="draft">
    <div class="tableWrapper">
      <el-table :data="tableData" border style="width: 100%" v-loading="loading">
        <el-table-column type="index" label="序号" width="100" align="center"></el-table-column>
        <el-table-column prop="topicname" label="课题名称" width="200" align="center"></el-table-column>
        <el-table-column prop="paperdraftSuggest" label="初稿状态" width="200" align="center"></el-table-column>
        <el-table-column prop="teachername" label="审核导师" align="center"></el-table-column>
        <el-table-column label="操作" width="240" fixed="right">
          <template slot-scope="scope" >
            <div style="float: left;padding-right: 10px">
              <el-upload
                class="upload-demo"
                :action="uploadUrl"
                :on-exceed="handleExceedContract"
                :on-success="handleContractUpload"
                :on-change="handleChange"
                :show-file-lis="false"
                :limit="1">
                <el-button class="deepbluebtn" size="small" type="primary">上传</el-button>
              </el-upload>
            </div>
            <a :href="downloadUrl" class="deepbluebtn downStyle" size="small" v-if="buttonShow === 1">下载</a>
            <el-button class="deepbluebtn" type="success" size="small" @click="openSuggest(scope.row)">查看意见</el-button>
          </template>
        </el-table-column>
      </el-table>
      <!--<el-pagination-->
      <!--background-->
      <!--@size-change="handleSizeChange"-->
      <!--@current-change="handleCurrentChange"-->
      <!--:current-page="tableParams.pageNum"-->
      <!--:page-sizes="[10, 20, 30]"-->
      <!--:page-size="tableParams.pageSize"-->
      <!--layout="total, sizes, prev, pager, next, jumper"-->
      <!--:total="paginationTotal">-->
      <!--</el-pagination>-->
    </div>
    <el-dialog title="查看审批意见" class='editdia' width="500px" :visible.sync="topicDialog" :close-on-press-escape='false' :close-on-click-modal='false'>
      <div class="diabody diascroll" style="height: 400px">
        <div class="detail-row">
          <div class="row-title">审批意见</div>
          <div class="row-content">
            <span class="input-wrapper">
              <el-input type="textarea" v-model="showData.paperdraftSuggest" :disabled="true"></el-input>
            </span>
          </div>
        </div>
      </div>
      <div class="diafoot flex">
        <el-button type="primary" class="truebutton deepbluebtn"@click="closeDialog()">确定</el-button>
      </div>
    </el-dialog>
  </div>
</template>

<script>
  import { topicSelect, studentSelect,draftList } from '@/api/student'
  import CMethods from '../../../commonJS/Methods'
  export default {
    data() {
      return {
        page: 1,
        size: 10,
        loading: false,
        topicDialog: false,
        tableData: [],
        showData: {},
        downloadUrl:'',
        uploadUrl:'',
        filename:'',
        topicname:'',
        buttonShow:""
        // fileList3:[{
        //   name:'docName',
        //   url:""
        // }]
      }
    },
    methods: {
      getData() {
        draftList(this.getUserId()).then(res => {
          this.tableData = res.data.data
          this.buttonShow = res.data.data[0].paperdraftIspass;
          this.filename = res.data.data[0].paperdraftPath;
          this.topicname = res.data.data[0].topicname;
          this.downloadUrl = CMethods.spliceDownloadUrl(this.topicname,this.filename);
          this.uploadUrl = CMethods.uploadDraftReport(this.getUserId());
        })
      },
      closeDialog() {
        this.topicDialog = !this.topicDialog
      },
      handleChange(){
        console.log('上传文件!!!!')
      },
      handleContractBefore(){
        let size = file.size/1024/1024 < 20;
        if (file.type.indexOf('doc')<0) {
          this.$message({message:'只能上传DOC格式的文件！',type:'waning'});
          return false;
        }
        if(!size){
          this.$message({message:'文件大小不能超过20MB',type:'waning'});
          return false;
        }
      },
      handleExceedContract(){
        this.$message({message:'开题报告只能上传一份！',type:'warning'})
      },
      handleContractUpload(resp,file,filelist){
        // if(resp.status===1){
        //   console.log('合同上传地址')
        // }
        this.getData();
        this.$message({message:'上传文件成功!',type:'success'})
      },
      openSuggest(val) {
        if (val.paperdraftSuggest) {
          this.$confirm(`${val.paperdraftSuggest}`, '审核意见', {
            confirmButtonText: '好的',
            customClass: 'blueMessage',
            showCancelButton: false
          })
        } else {
          this.$confirm('暂无意见', '审核意见', {
            confirmButtonText: '好的',
            customClass: 'blueMessage',
            showCancelButton: false
          })
        }
      },
      handleSelect(id, name) {
        this.$confirm(`确定要选择「${name}」课题吗?`, '确定选择？', {
          confirmButtonText: '确定',
          cancelButtonText: '取消',
          type: 'warning',
          customClass: 'blueMessage'
        })
          .then(() => {
            studentSelect(id, this.getUserId()).then(res => {
              this.$message({ type: 'success', message: res.message })
            })
          })
          .catch(() => {
            this.$message({
              type: 'info',
              message: '已取消'
            })
          })
      }
    },
    created() {
      this.getData()
    }
  }
</script>

<style rel="stylesheet/scss" lang="scss" scoped>
  @import '../../../styles/common';
</style>
<style rel="stylesheet/scss" lang="scss">
  .draft .cell{
    line-height: 60px;
    height: 60px;
  }
  .draft .el-upload-list.el-upload-list--text{
    display: none;
  }
  .draft .cell .downStyle{
    float: right;
    margin-top: 11px;
    height: 40px;
    width: 55px;
    border-radius: 2px;
    text-align: center;
    line-height: 40px;
    margin-right: 10px;
  }
  .draft .cell .upload-demo{
    float: left;
  }
  .draft .el-table td, .el-table th{
    padding: 0;
  }
</style>
