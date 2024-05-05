<template>
  <div style="padding: 25px;">
  <el-button style="margin-bottom: 20px;" type="primary" @click="showMessageAdd=true">新增留言</el-button>

  <div style="display: flex;justify-content: space-between;flex-wrap: wrap;">
    <el-card v-for="(item,index) in dataList" :key="index" class="box-card" style="margin-bottom: 10px; width: 48%;">
      <div slot="header" class="clearfix">
        <span>{{item.title}}</span>
        <el-button v-if="admin" style="float: right; padding: 3px 0;margin-left: 10px;color: red;" type="text" @click="delMessage(item.id)">删除</el-button>
        <el-button style="float: right; padding: 3px 0" type="text" @click="showCommentData(item.id)">回复</el-button>

      </div>
      <div class="text item">
        <div style="height: 100px;">{{item.content}}</div>

        <div style="display: flex;justify-content: space-between;flex-wrap: wrap;">
          <div><span style="color: gray;">留言人：</span>{{item.nickname}}</div>
          <div style="color: gray;">{{item.createTime}}</div>
        </div>
      </div>
    </el-card>
  </div>



    <el-dialog title="回复列表" :visible.sync="showContentList">
        <el-input
          type="textarea"
          :rows="2"
          placeholder="请输入回复内容"
          v-model="comment">
        </el-input>
        <el-button style="margin:20px 20px 20px 0px;" type="primary" @click="commentSub">回复</el-button>

        <div style="padding: 15px;">
          <div style="padding: 10px;">全部回复</div>

          <el-card v-for="(item,index) in commentDataList" :key="index" class="box-card" style="margin-bottom: 5px;">
            <div class="text item">
              {{item.content}}
              <el-button v-if="admin" style="float: right; padding: 3px 0;margin-left: 10px;color: red;" type="text" @click="delComment(item.id)">删除</el-button>

              <div style="margin-top: 10px; display: flex;justify-content: space-between;flex-wrap: wrap;font-size: 12px;">
                <div><span style="color: gray;">回复人：</span>{{item.nickname}}</div>
                <div style="color: gray;">{{item.createTime}}</div>
              </div>

            </div>
          </el-card>


        </div>
    </el-dialog>

    <el-dialog title="新增留言" :visible.sync="showMessageAdd">
      <el-input style="margin-bottom: 20px;"
        :rows="2"
        placeholder="请输入留言标题"
        v-model="message.title">
      </el-input>
        <el-input
          type="textarea"
          :rows="2"
          placeholder="请输入留言内容"
          v-model="message.content">
        </el-input>
        <el-button style="margin:20px 20px 20px 0px;" type="primary" @click="messageSub">提交</el-button>
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
      message:{
        title:"",
        content:""
      },
      comment:"",
      showTime:false,
      showMessageAdd:false,
      showContentList:false,
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
      dataList:[],
      uid:"",
      realName:"",
      pid:0,
      commentDataList:[]
    }
  },
  created:async function(){
    const { roles,userName,realName,id } = await store.dispatch('user/getInfo');
    this.uid = id;
    this.realName = realName;
    if(roles[0]&&roles[0]=="sa"){
      this.admin = true;
    }
    console.log(realName)
    console.log(id)
    this.getData();

  }
  ,
  methods: {
    messageSub(){
      let _this = this;
      this.message.creater = this.uid;
      this.message.nickname = this.realName;
      if(!this.message.title||!this.message.content){
        this.$message('请填写完整');
        return;
      }
      _this.showMessageAdd =false;
      post('/exam/api/elMessage/addElMessage', this.message).then((res)=>{
        _this.getData();
        _this.message.title="";
        _this.message.content="";
      })
    },
    delMessage(id){
      let _this = this;
      let pid = this.pid;
      post('/exam/api/elMessage/editElMessage', {validFlag:0,id:id}).then((res)=>{
        _this.getData(pid);
        _this.$forceUpdate();
        _this.comment="";
      })
    },
    delComment(id){
      let _this = this;
      let pid = this.pid;
      post('/exam/api/elComment/editElComment', {validFlag:0,id:id}).then((res)=>{
        _this.getComment(pid);
        _this.$forceUpdate();
        _this.comment="";
      })
    },
    commentSub(){
      if(!this.comment){
        this.$message('请填写完整');
        return;
      }
      let _this = this;
      let pid = this.pid;
      post('/exam/api/elComment/addElComment', {validFlag:1,pid:pid,content:this.comment,nickname:this.realName,creater:this.uid}).then((res)=>{
        _this.getComment(pid);
        _this.$forceUpdate();
        _this.comment="";
      })
    },
    showCommentData(pid){
      this.showContentList = true;
      this.pid = pid;
      this.getComment(pid);
    },
    getComment(pid){
      let _this = this;
      post('/exam/api/elComment/queryList', {validFlag:1,pid:pid}).then((res)=>{
        _this.commentDataList = res.data;
        _this.$forceUpdate()
      })
    },
    getData(){
      let _this = this;
      this.showContent = false;
      post('/exam/api/elMessage/queryList', {validFlag:1}).then((res)=>{
        _this.dataList = res.data;
        _this.$forceUpdate();
      })
    },
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
