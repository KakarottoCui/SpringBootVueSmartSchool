<template>
  <div style="padding: 25px;">
    <el-table border :data="timeTable" style="width: 100%">

      <el-table-column
        label="时间"
        prop="time"
      >
      <template v-slot="scope">
        <div>
          {{scope.row.time}}
          <el-link v-if="admin" type="primary" @click="showTimeChange(scope.row)">修改</el-link>
        </div>
      </template>
      </el-table-column>

      <el-table-column
        label="星期一"
        prop="week1"
      >
      <template v-slot="scope">
        <div>
          {{scope.row.week1}}
          <el-link type="primary"  v-if="admin" @click="showContentChange(scope.row,'1')" >修改</el-link>
        </div>
      </template>

      </el-table-column>

      <el-table-column
        label="星期二"
        prop="week1"
      >
      <template v-slot="scope">
        <div>
          {{scope.row.week2}}
          <el-link type="primary" v-if="admin" @click="showContentChange(scope.row,'2')">修改</el-link>
        </div>
      </template>

      </el-table-column>

      <el-table-column
        label="星期三"
        prop="week3"
      >
      <template v-slot="scope">
        <div>
          {{scope.row.week3}}
          <el-link type="primary" v-if="admin" @click="showContentChange(scope.row,'3')">修改</el-link>
        </div>
      </template>

      </el-table-column>

      <el-table-column
        label="星期四"
        prop="week4"
      >
      <template v-slot="scope">
        <div>
          {{scope.row.week4}}
          <el-link type="primary" v-if="admin" @click="showContentChange(scope.row,'4')">修改</el-link>
        </div>
      </template>

      </el-table-column>

      <el-table-column
        label="星期五"
        prop="week5">

      <template v-slot="scope">
        <div>
          {{scope.row.week5}}
          <el-link type="primary" v-if="admin" @click="showContentChange(scope.row,'5')">修改</el-link>
        </div>
      </template>

      </el-table-column>

      <el-table-column
        label="星期六"
        prop="week6"
      >
      <template v-slot="scope">
        <div>
          {{scope.row.week6}}
          <el-link type="primary" v-if="admin" @click="showContentChange(scope.row,'6')">修改</el-link>
        </div>
      </template>

      </el-table-column>

      <el-table-column
        label="星期日"
        prop="week7"
      >
      <template v-slot="scope">
        <div>
          {{scope.row.week7}}
          <el-link type="primary" v-if="admin" @click="showContentChange(scope.row,'7')">修改</el-link>
        </div>
      </template>

      </el-table-column>

    </el-table>

    <el-dialog title="修改时间范围" :visible.sync="showTime">
      <el-time-picker
          is-range
          v-model="selData.value"
          range-separator="至"
          start-placeholder="开始时间"
          end-placeholder="结束时间"
          placeholder="选择时间范围"
          format="HH:mm"
          value-format="HH:mm">
        </el-time-picker>
        <el-button style="margin-left: 20px;" type="primary" @click="timeSub">确认</el-button>
    </el-dialog>

    <el-dialog title="修改课程" :visible.sync="showContent">
      <div style="display: flex;justify-content: space-between;">
        <el-input v-model="selData.title" placeholder="请输入课程名"></el-input>
        <el-button style="margin-left: 20px;" type="primary" @click="contentSub">确认</el-button>
      </div>

    </el-dialog>

  </div>
</template>

<script>
import DataTable from '@/components/DataTable'
import store from '@/store'
import { post } from '@/utils/request'
export default {
  name: 'ExamList',
  components: { DataTable },
  data() {
    return {
      showTime:false,
      showContent:false,
      selData:{
        value:[new Date(),new Date()],
        time:"",
        id:"",
        title:""
      },
      listQuery: {
        current: 1,
        size: 10,
        params: {
          title: ''
        }
      },
      admin:false,
      timeTable:[]
    }
  },
  created:async function(){
    const { roles,userName } = await store.dispatch('user/getInfo');
    console.log(userName);
    if(roles[0]&&roles[0]=="sa"){
      this.admin = true;
    }
    this.getTimeTable();

  }
  ,
  methods: {
    showContentChange(row,id){
      this.showContent = true;
      this.selData.title = row['week'+id];
      this.selData.id = row['id-'+id];
    },
    showTimeChange(row){
      let scope = row.time.split("-");
      this.selData.time = row.time;
      this.selData.id = row.id;
      this.selData.value = [new Date("2024/01/01 "+scope[0]),new Date("2024/01/01 "+scope[1])]
      this.showTime=true
    },
    contentSub(){
      let _this = this;
      this.showContent = false;
      post('/exam/api/elTimetable/editElTimetable', {id:this.selData.id,title:this.selData.title}).then((res)=>{
        _this.getTimeTable();
      })
    },
    timeSub(){
      let _this = this;
      let res = this.selData.value[0]+"-"+this.selData.value[1];
      this.showTime = false;
      post('/exam/api/elTimetable/editTimeScope', {time:this.selData.time,content:res}).then((res)=>{

        _this.getTimeTable();
      })
    },
    getTimeTable(){
      let _this = this;
      let timeTable = []
      post('/exam/api/elTimetable/queryTimeTable', {validFlag:1}).then((res)=>{
        for(let i=0;i<res.data.length;i++){
          let week = {};
          let item = res.data[i];
          for(let j=0;j<item.length;j++){
            let obj = item[j];
            week['time'] = obj.content;
            week['week'+obj.pid] = obj.title;
            week['id-'+obj.pid]= obj.id
          }
          timeTable.push(week);
        }
        _this.timeTable = timeTable;
      })
    },
    handleExamDetail(examId) {
      this.$router.push({ name: 'ListExamUser', params: { examId: examId }})
    },

    handleUpdateExam(examId) {
      this.$router.push({ name: 'UpdateExam', params: { id: examId }})
    }
  }
}
</script>
