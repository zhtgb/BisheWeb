<!--  -->
<template>
  <div class="checkStudent">
    <div>
      <el-button type="primary" class="truebutton deepbluebtn"@click="closeDialog()">新建小组</el-button>
    </div>
    <el-table
      :data="groupData"
      style="width: 100%">
      <el-table-column
        property="groupid"
        label="分组序号"
        width="250">
      </el-table-column>
      <el-table-column
        label="小组成员">
        <template slot-scope="scope">
          <div class="flex">
            <div v-for="(item,index) in scope.row.grouplist">
              {{item}}
            </div>
          </div>
        </template>
      </el-table-column>      <el-table-column
        property="phone"
        label="操作">
        <template slot-scope="scope" >
          <el-button type="success" size="small" @click="handleOpen(scope.row)">修改组员</el-button>
          <el-button type="danger" size="small" @click="handleOpen(scope.row)">查看组员</el-button>
        </template>
      </el-table-column>
    </el-table>
    <el-dialog title="中检分组" class='editdia' width="800px" :visible.sync="topicDialog" :close-on-press-escape='false' :close-on-click-modal='false'>
      <div class="flex" style="margin-bottom: 20px">
        <el-select v-model="value" placeholder="请选择学院" flex="fixed" align="left" style="width: 200px;margin-right: 25px;" @change="getCollegeValue">
          <el-option
            v-for="item in options"
            :key="item.collegeid"
            :label="item.collegename"
            :value="item.collegeid">
          </el-option>
        </el-select>
        <el-select v-model="groupid" placeholder="请选择分组" style="width: 200px;display: block;" @change="groupIdValue">
          <el-option
            v-for="item in groupOptions"
            :key="item.groupid"
            :label="item.groupName"
            :value="item.groupid">
          </el-option>
        </el-select>
      </div>
      <div flex="fixed">
        <el-transfer
          style="text-align: left; display: inline-block"
          v-model="value3"
          filterable
          :render-content="renderFunc"
          :right-default-checked="[]"
          :titles="['教师', '组员']"
          :button-texts="['到左边', '到右边']"
          :format="{ noChecked: '${total}', hasChecked: '${checked}/${total}' }"
          @change="handleChange"
          :data="teacherList">
          </el-transfer>
      </div>
      <div class="diafoot flex" style="margin-top: 15px">
        <el-button type="primary" class="truebutton deepbluebtn" @click="submitData()">确定</el-button>
        <el-button type="info" class="cancelbtn" @click="closeDialog()">取消</el-button>
      </div>
    </el-dialog>
  </div>
</template>

<script>
  import { getteacher, middlegroup, getCollegeList, checkMiddleGroup } from '@/api/admin'

  export default {
    data() {
      return {
        dataList: [],
        value3: [0],
        renderFunc(h, option) {
          return <span> { option.key } - { option.label } </span>
        },
        page: 1,
        size: 10,
        loading: false,
        topicDialog: false,
        tableData: [],
        title: '',
        showData: {},
        teacherDate: [],
        teacherList: [],
        groupid:'',
        selectValue:[],
        grouplist:[],
        data: [],
        groupData: [],
        collegeValue:'',
        groupOptions:[{
          groupid:1,
          groupName:'第一组'
        },{
          groupid:2,
          groupName:'第二组'
        },{
          groupid:3,
          groupName:'第三组'
        },{
          groupid:4,
          groupName:'第四组'
        },{
          groupid:5,
          groupName:'第五组'
        }],
        options: [{
          collegeid: '选项1',
          collegename: '黄金糕'
        }, {
          collegeid: '选项2',
          collegename: '双皮奶'
        }, {
          collegeid: '选项3',
          collegename: '蚵仔煎'
        }, {
          collegeid: '选项4',
          collegename: '龙须面'
        }, {
          collegeid: '选项5',
          collegename: '北京烤鸭'
        }],
        value: ''
      }
    },
    methods: {
      getData(){
        middlegroup().then(res =>{
          this.groupData = res.data.data
          console.log(this.tableData)
        })
      },
      getteacherData() {
        getteacher(0, 10, 1,this.collegeValue).then(res => {
          this.teacherList =[]
          this.teacherDate = res.data.data.content
          this.teacherList = this.show()
        })
      },
      groupIdValue(val){
        this.groupid = val
      },
      getCollegeList(){
        getCollegeList().then(res =>{
          this.options = res.data.data;
        })
      },
      closeDialog() {
        this.topicDialog = !this.topicDialog
        this.grouplist = []
        this.groupid = ''
        this.value3 = []
      },
      getCollegeValue(val){
        this.collegeValue = val;
        this.getteacherData();
      },
      submitData() {
        this.topicDialog = !this.topicDialog
        checkMiddleGroup({groupid:this.groupid, grouplist:this.grouplist}).then(res =>{
          this.grouplist = []
          this.groupid = ''
        })
      },
      show() {
        this.data = []
        for (let i = 0; i < this.teacherDate.length; i++) {
          this.data.push({
            key: this.teacherDate[i].userid,
            label: this.teacherDate[i].username,
          })
        }
        return this.data
      },
      handleChange(value, direction, movedKeys) {
        this.selectValue = value
        this.selectValue.forEach(item =>{
          this.grouplist.push(item)
        })

      },
      handleOpen(data) {
        this.topicDialog = !this.topicDialog
        if (data !== undefined) {
          this.showData = JSON.parse(JSON.stringify(data))
          this.showData.schedule = this.showData.schedule.replace(/<br>/g, '\n')
          this.showData.topiccontent = this.showData.topiccontent.replace(/<br>/g, '\n')
        }
      },
    },
    created() {
      this.getCollegeList();
      this.getData();
    }
  }
</script>
<style  scoped rel="stylesheet/scss" lang="scss">
  .checkStudent {
    margin-top: 60px;
  }
</style>
<style  rel="stylesheet/scss" lang="scss">
  .checkStudent {
    .el-table {
      .handle {
        .result {
          color: #ffffff;
          font-size: 12px;
          display: inline-block;
          padding: 8px 15px;
          border-radius: 4px;
          margin-right: 8px;
        }
        .reject {
          padding: 12px 32px;
        }
        .allowed {
          background-color: #6ed596;
        }
        .notAllowed {
          background-color: #b8b9c7;
        }
      }
    }
  }
</style>
