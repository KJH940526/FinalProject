<template lang="">
<div>
    <side-bar></side-bar>
    <div id='content'>
    <div class="title-wrapper">
      <span class="title">고장 리포트 통계</span>
    </div>
    <div class="select-condition">
    <table class="condition-table">
          <colgroup>
            <col width="15%">
          </colgroup>
          <tbody>
            <tr class="condition-table-body">
              <td>검색기간</td>
              <td>
        <!-- <input type="date" v-model="firstDate"/><input type="time" v-model="firstTime"/> ~ 
        <input type="date" v-model="lastDate"/><input type="time" v-model="lastTime"/>
             -->
        <!-- {{firstDate + " " + firstTime}}
        {{lastDate + " " + lastTime}} -->
                    <input type="date" v-model="firstDate"/><input type="time" v-model="firstTime"/> ~
                   <input type="date" v-model="lastDate"/><input type="time" v-model="lastTime"/>

              </td>
            </tr>
            <tr class="condition-table-body">
                <td>CCTV</td>
                <td>
                    <select name="selectingCCTV" v-model="cctvName">
                        <option v-for="(cctvs,index) in getCCTVNames" :key="index">
                            {{cctvs.name}}
                        </option>
                    </select>
        <button class="button-add" v-on:click="addCCTV(cctvName)">추가</button>
                </td>
            </tr>
            <tr>
              <td></td>
              <td>
                    <span class="groupbutton" v-for="(cctvName,index) in cctvNames" :key="index">
                        {{cctvName}}
                        <span class="groupremove" type="button" v-on:click="removeCCTV(index)">
                            <i class="closeBtn fas fa-times"></i>
                        </span>
                    </span>
              </td>
            </tr>
            </tbody>
            </table>

    <div class="searchBtn">
            <button class="button-cancle" v-on:click="makeExcelFile()">
                내보내기
            </button>
            <button class="button-add" v-on:click="searchFailReport(firstDate,firstTime,lastDate,lastTime)">
                조회
            </button>
        </div>
        </div>

    <!-- <br/>
    <br/>
    <br/> -->
    
    <div class="content-result">
        <table class="table1">
          <colgroup>
                <col width="200px">
                <col width="200px">
                <col width="200px">
                <col width="150px">
                <col width="600px">
            </colgroup>
            <thead>
                <tr class="table1-head-title">
                    <th>번호</th>
                    <th>보고 일자</th>
                    <th>장치 이름</th>
                    <th>주소</th>
                    <th>수리 현황</th>
                </tr>
            </thead>
            <tbody>
                <tr class="table1-body" v-for="(data,index) in searchData" :key="index">
                    <td>{{index+1}}</td>
                    <td>{{data.date}}</td>
                    <td>{{data.cctvName}}</td>
                    <td>{{data.region}}</td>
                    <td>{{data.repairStat}}</td>
                </tr>
            </tbody>
        </table>
        <table class="secondTable">
            <thead>
                <tr class="tTitle2">
                    <th>월별 통계</th>
                </tr>
            </thead>
            <tbody>
                <tr class="tBody2">
                    <td v-if='showChart'><chart  :data="chartData" :options="chartOptions"></chart></td>      
                </tr>
            </tbody>
        </table>
    </div>
    </div>
</div>
</template>

<script>
import Chart from './Chart'
import SideBar from '../../common/SideBar.vue'
import Xlsx from 'xlsx'
export default {
    components:{
        SideBar,
        Chart
    },
    data(){
        return{
            firstDate:'',
            lastDate:'',
            firstTime:'',
            lastTime:'',
            cctvName:'',
            cctvNames:[],
            getCCTVs:[],
            getCCTVNames:[],
            
            searchData:[],



            
            barStatistic:[0,0,0,0,0,0,0,0,0,0,0,0],

            showChart:false,
            chartData:{
                labels: ["1월","2월","3월","4월","5월","6월","7월","8월","9월","10월","11월","12월"],
                datasets: [
                    {
                        label: "리포트 통계",
                        backgroundColor: "#62B9FB",
                        data: []
                    }
                ]
          },
          chartOptions: {
              title:{
                    display:true,
                    text: '리포트 통계',
                    position: "bottom"
                },
                responsive: true,
                legend: {
                    display: false
                },
                maintainAspectRatio: true,
           },

        }
    },
    methods:{
        getCCTVsToJson(){
            this.$http.get('http://localhost:3000/cctv_infos')
            .then((res) => {
                console.log('getCCTVs:', res.data)
                this.getCCTVs = res.data
            })            
        },
        getCCTVInfoToJson(){
            this.$http.get('http://localhost:3000/cctv_info')
            .then((res) => {
                console.log('getCCTVName:', res.data)
                this.getCCTVNames = res.data
            })            
        },
        addCCTV(cctvName){
            if(!this.isExistCCTV(cctvName)){
                this.cctvNames.push(cctvName);
            }else{
                alert("이미 선택한 CCTV그룹입니다.");
            }
        },
        isExistCCTV(cctvName){
            var returnFlag = false;
            for(var i in this.cctvNames){
                if(this.cctvNames[i] == cctvName){
                    returnFlag = true;
                }
            }
            return returnFlag;
        },
        removeCCTV(index){
            this.cctvNames.splice(index,1);
        },
        searchFailReport(firstDate,firstTime,lastDate,lastTime){
            this.barStatistic=[0,0,0,0,0,0,0,0,0,0,0,0]
            this.searchData.splice(0)
            this.chartArr = [];  // 차트에 들어가는 수 초기화 // 추가해줌!!
            let fDateTime = firstDate+" "+firstTime
            let lDateTime = lastDate+" "+lastTime
            if(!firstDate&&!firstTime&&!lastDate&&!lastTime){
                alert("기간을 입력하세요")
            }else{
                for(let i=0; i<this.getCCTVs.length; i++){
                    if(this.isbetweenDate(fDateTime,lDateTime,this.getCCTVs[i].updated_at)){
                        for(let j=0; j<this.cctvNames.length; j++){
                            if(this.getCCTVs[i].name==this.cctvNames[j]){
                                this.searchData.push({
                                    date:this.getCCTVs[i].updated_at,
                                    cctvName:this.getCCTVs[i].name,
                                    region:this.getCCTVs[i].area1+" "+this.getCCTVs[i].area2+" "+this.getCCTVs[i].area3,
                                    repairStat:this.getCCTVs[i].ptz_control_usage
                                })
                            }
                        }
                    }
                }
            }
            for(let i=0; i<this.searchData.length; i++){
                if(this.searchData[i].repairStat==0){
                    this.searchData[i].repairStat="수리 대기"
                }else if(this.searchData[i].repairStat==1){
                    this.searchData[i].repairStat="수리 완료"
                }
            }
            for(let i=0; i<this.getCCTVs.length; i++){
                let repairNum = Number(this.getCCTVs[i].updated_at.substr(5,2))
                if(repairNum==1){
                    this.barStatistic[repairNum-1]++
                }else if(repairNum==2){
                    this.barStatistic[repairNum-1]++
                }else if(repairNum==3){
                    this.barStatistic[repairNum-1]++
                }else if(repairNum==4){
                    this.barStatistic[repairNum-1]++
                }else if(repairNum==5){
                    this.barStatistic[repairNum-1]++
                }else if(repairNum==6){
                    this.barStatistic[repairNum-1]++
                }else if(repairNum==7){
                    this.barStatistic[repairNum-1]++
                }else if(repairNum==8){
                    this.barStatistic[repairNum-1]++
                }else if(repairNum==9){
                    this.barStatistic[repairNum-1]++
                }else if(repairNum==10){
                    this.barStatistic[repairNum-1]++
                }else if(repairNum==11){
                    this.barStatistic[repairNum-1]++
                }else if(repairNum==12){
                    this.barStatistic[repairNum-1]++
                }
            }
            console.log("여기까지 왔다네~");
            this.chartDatePush();
        },
        isbetweenDate(fDateTime,lDateTime,searchDate){
            let returnFlag=false
            const moment = require('moment');
            if(moment(searchDate).isBetween(fDateTime, lDateTime, undefined, '()')){
                returnFlag = true
            }
            return returnFlag
        },
        // 엑셀 내보내기
        makeExcelFile(){
        console.log("엑셀내보내기");
        const workBook = Xlsx.utils.book_new();
        const workSheet = Xlsx.utils.json_to_sheet(this.searchData);
        Xlsx.utils.book_append_sheet(workBook, workSheet, '시트이름');
        Xlsx.writeFile(workBook, '고장리포트통계.xlsx')
        },

        chartDatePush(){
            this.showChart = true;
            this.chartData.datasets[0].data = [];  //배열초기화
            for(let i = 0; i<this.barStatistic.length; i++){
                console.log(this.barStatistic[i]);
                this.chartData.datasets[0].data.push(this.barStatistic[i]);        
            }
            console.log(this.chartData.datasets[0].data);
        }

    },
    mounted(){
        console.log(this.barStatistic);
        this.getCCTVsToJson()
        this.getCCTVInfoToJson()
    }
}
</script>

<style lang="">
.table1{
    width:1300px;
    float:left;
    
}
.secondTable{
    
    width:600px;
    /* border-left: 1px;
    border-bottom: 1px solid; */
    text-align:center;
    vertical-align: middle;
    height: 40px;
    background-color: #f6f7fb;
}
 .table1-head-title-status {
    text-align:center;
    vertical-align: middle;
    height: 40px;
    background-color: #f6f7fb;
  }
.groupbutton {
  display: inline-block;
  width: 130px;
  border: 1px solid #0b6ad5;
  line-height: 35px;
  border-radius: 1px;
  padding: 0px 10px;
  margin-right: 15px;
  font-size: 15px;
  letter-spacing: 0px;
  color: #0b6ad5;
}
  
</style>