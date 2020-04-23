<template>
 <el-container>
  <el-header>
    <h1>机试题：筛选并且导出表格</h1>
  </el-header>
    <el-main>
    <div class="block">
        <span class="demonstration">请选择日期：</span>
        <el-date-picker
          v-model="value2"
          align="right"
          type="date"
          placeholder="选择日期"
          :picker-options="pickerOptions"
           format="yyyy 年 MM 月 dd 日"
      value-format="yyyy-MM-dd">
        </el-date-picker>
    </div>
    <div class="status">
        <p>项目状态 ：</p>
        <div>
            <el-button  v-for="item in choices" :key="item.id" type="primary" plain @click=category(item.status)>{{item.status}}:{{countNum(item.status)}}</el-button>
        </div>
    </div>
    <div class="search-box">
        <el-select v-model="value" placeholder="请选择">
          <el-option
            v-for="item in options"
            :key="item.value"
            :label="item.label"
            :value="item.value">
          </el-option>
        </el-select>
        <el-input v-model="input" max-length="200" placeholder="请输入内容"></el-input>
        <el-button type="success" @click="export2Excel">导出</el-button>
    </div>
    <div class="data-table">
          <el-table :data="tableData" style="width: 100%">
              <el-table-column prop="id" label="序号"  width="180" column-key="date">
              </el-table-column>
              <el-table-column
                prop="name"
                label="项目名称"
                width="180">
              </el-table-column>
            <el-table-column
              prop="date"
              label="项目时间"
            >
            </el-table-column>
            <el-table-column
              prop="person"
              label="项目负责人"
            >
            </el-table-column>
            <el-table-column
              prop="status"
              label="项目状态"
              width="100"
             >
            <template slot-scope="scope">
              <el-tag :type="styleshow(scope.row.status)">{{scope.row.status}}</el-tag>
                </template>
        </el-table-column>
       </el-table>
    </div>
  </el-main>
</el-container>
</template>

<script>
// @ is an alias to /src
export default {
  name: "Home",
  data(){
    return {
       pickerOptions: {
          disabledDate(time) {
            return time.getTime() > Date.now();
          }
        },
        value2:'',
        choices:[{id:0,status:'全部',num:0},{id:1,status:'待开始',num:0},{id:2,status:'进行中',num:0},{id:3,status:'已完成',num:0},
        {id:4,status:'延迟',num:0},],
        options:[{
          value: '0',
          label: '项目名称'
        },
        {
          value: '1',
          label: '项目负责人'
        }],
        value:'',
        input:'',
        tableData: [
          {
          id: '0',
          name: '临时项目',
          status: '待开始',
          date: '2020-04-21',
          person:'王小虎'
        }, {
          id: '1',
          name: '团建项目',
          status: '进行中',
          date: '2020-04-21',
          person:'李晓明'
        }, {
           id: '2',
          name: '海外项目',
          status: '已完成',
          date: '2020-04-20',
          person:'张三'
        }, {
          id: '3',
          name: '国内项目',
          status: '延迟',
          date: '2020-04-20',
          person:'李四'
        }, {
           id: '4',
          name: '海外项目',
          status: '已完成',
          date: '2020-04-19',
          person:'王五'
        },
         {
           id: '5',
          name: '国内项目',
          status: '已完成',
          date: '2020-04-21',
          person:'张三'
        },
         {
           id: '6',
          name: '海外项目',
          status: '已完成',
          date: '2020-04-19',
          person:'王小虎'
        },
         {
           id: '7',
          name: '临时项目',
          status: '进行中',
          date: '2020-04-20',
          person:'王小虎'
        },
         {
           id: '8',
          name: '海外项目',
          status: '已完成',
          date: '2020-04-18',
          person:'李四'
        }]
      }
  },
   computed:{
    styleshow:function(){
      return function(value){
        return this.showstyle(value)
      }
    },
    countNum:function(){
      return function(value){
        return this.count(value);
      }
    }
  },
  methods:{
    showstyle:function(status){
      if(status==="待开始"){
        return "info"
      }else if(status==="进行中"){
        return "warning"
      }else if(status==="已完成"){
        return "success"
      }else{
        return "danger"
      }
    },
    count(currStatus){
      let num=0;
      let arr = JSON.parse(window.sessionStorage.getItem('data'));
      if(currStatus==="全部"){
        num=arr.length;
      }else{
         arr.forEach(item=>{
            if(item.status===currStatus){
              num++;
            }
      });
      }
      return num;
    },
    category(status){
      if(status==="全部"){
       this.tableData=JSON.parse(window.sessionStorage.getItem('data'));
      }else{
         let arr = JSON.parse(window.sessionStorage.getItem('data'));
        this.tableData=arr.filter(ele=>ele.status===status);
      }
    },
    save(){
       let list = this.tableData;
      window.sessionStorage.setItem('data',JSON.stringify(list));
    },
    search(kind,userInput){
        let arr =JSON.parse(window.sessionStorage.getItem('data'));
        let newArr=[];
        if(kind==0){
            arr.forEach(function(va){
             if(va.name.match(userInput)){
               newArr.push(va);
             }else{
                 return false;
             }
            });
            this.tableData=newArr;
        }else if(kind==1){
          this.tableData=arr.filter(ele=>ele.person===userInput);
        }
    },
    timechange(nowDate){
       let arr =JSON.parse(window.sessionStorage.getItem('data'));
       this.tableData=arr.filter(ele=>ele.date===nowDate);
    },
    formatJson(filterVal, jsonData) {
    return jsonData.map(v => filterVal.map(j => v[j]))
},
    export2Excel(){
         const {export_json_to_excel} = require('../vendor/Export2Excel');         
         const tHeader = ['序号', '项目名称', '项目时间','项目负责人','项目状态'];
         const filterVal = ['id', 'name', 'date','person','status'];
         const list = this.tableData;
         const data = this.formatJson(filterVal, list);
         export_json_to_excel(tHeader, data, 'productExcel')
 }},
  mounted(){
    this.save();
  },
  watch:{
    value:function(){
      this.search(this.value,this.input);
    },
    input:function(){
      this.search(this.value,this.input);
    },
    value2:function(){
     this.timechange(this.value2);
    }
  }}
</script>
<style scope>
   .el-header, .el-footer {
    background-color: #B3C0D1;
    color: #333;
    text-align: center;
    line-height: 60px;
  }
  .el-main {
    background-color: #E9EEF3;
    color: #333;
  }
  .block{
    margin-bottom:20px;
  }
  .status{
    width:100%;
    display:flex;
    margin-bottom:10px;
  }
  p{
    padding-top:8px;
  }
  .el-button{
    margin-right:5px;
    margin-left:5px;
  }
  .search-box{
    width:100%;
    height:50px;
    background:#fff;
    border-radius:10px;
    display:flex;
    align-items:center;
    padding:0 10px;
  }
  .el-select{
    margin-right:6px;
  }
  .el-input{
    margin:l0 6px;
  }
  .data-table{
    margin-top:20px;
    width:100%;
    overflow:hidden;
    background:#fff;
  }
</style>
