<template>
  <el-container class="dm-container">

    <el-header class="dm-header">
    </el-header>

    <!-- Main view -->
    <div>
      <input class="inputw"  @keyup.enter="search()"  v-model="searchDate" placeholder="输入日期2022-01-01" />
    </div>
    <el-main>
        <table v-if="dataList!=null" border style="display:inline-block;border-spacing: 0px 0px;">
            <tr>
              <td>行业</td>
              <td v-for="item in dataList" >
               {{item}}
              </td>
            </tr>
            <tr v-for="(key,k) in list">
              <td @click="showStockCode(k)">{{nameMap.get(k)}}</td>
              <td v-for="item in key">
                <p style="color:green" v-if="item.factor<0">{{item.factor}}</p>
                <p style="color:red" v-if="item.factor>=0">{{item.factor}}</p>
              </td>
            </tr>
        </table>
        <table v-if="stockList!=null" style="display:inline-block;height:1000px;overflow-y:auto;border-spacing: 0px 0px;" border>
            <tr>
              <td>股票名字</td>
              <td>股票代码</td>
              <td>股票价格</td>
              <td @click="sortBy()">当前涨跌幅</td>
            </tr>
            <tr v-for="stock in stockList">
              <td>{{stock.stockName}}</td>
              <td>{{stock.stockCode}}</td>
              <td>{{stock.stockPrice}}</td>
              <td>
                <p style="color:green" v-if="stock.stockQuote<0">{{-stock.stockQuote}}%</p>
                <p style="color:red" v-if="stock.stockQuote>=0">{{stock.stockQuote}}%</p>
                </td>
            </tr>
    
        </table>
    </el-main>
  </el-container>
</template>
  
<script>
import axios from "axios";
let domainUrl = "http://43.142.128.29:8070";
let getLastUrl = domainUrl + "/stock-industry-factor/industry/full/";
let getIndustry = domainUrl + "/stock-industry-factor/industry/list";
let getIndustryStockCode = domainUrl + "/stock-industry-factor/industry/stock/list/";
export default {
  name: "Home",
  components: {
  },
  data() {
    return {
       searchDate: '',
       list:null,
       dataList:null,
       nameMap: {},
       stockList:null,
       sort:false,
    };
  },
  components: {
  },
  created() { },
  watch: {},
  mounted() {
    this.initMap();
    //获取链接地址日期
    let query = window.location.search.substring(1);
    this.initPage(query);
  },
  methods: {
    sortBy(){
      let c=!this.sort;
      if(c){
        this.stockList.sort(function (a, b) {
            return a.stockQuote - b.stockQuote;
        });
      }else{
        this.stockList.sort(function (a, b) {
            return b.stockQuote - a.stockQuote;
        });
      }
      this.sort=c;
    },
    async showStockCode(code){
      const stockCodeRes = await axios.get(getIndustryStockCode+code, {
      });
      console.log(stockCodeRes);
      this.stockList=stockCodeRes.data.data
    },
    search(){
      this.initPage(this.searchDate);
    },
    async initMap(){
      const industry = await axios.get(getIndustry, {
      });
      let industryData=industry.data.data;
      let industryMap=new Map();
      for (let i=0; i<industryData.length; i++){
        industryMap.set(industryData[i].stockIndustryCode,industryData[i].stockIndustry)
      }
      this.nameMap=industryMap;
    },
    async initPage(query){
      this.stockList=null;
      let url=getLastUrl;
      if(!query){
        url=url+"last";
      }else{
        url=url+query;
      }
      console.log(query);
      const res = await axios.get(url, {
      });
      let data=res.data.data
      this.list=data;
      let header=[];
      let index=0;
      Object.getOwnPropertyNames(data).forEach(function(key){
        if(index==0){
          for (let i=0; i<data[key].length; i++){
            header.push(data[key][i].collectDate);
          }
        }
        ++index;
      });
      this.dataList=header;
    }
  },
  unmounted() {
    this.timer = {};
  },
};
</script>

<!-- common -->

.inputw {
  width: 640px;
  height: 48px;
  background: #f2f3f5;
  border-radius: 4px;
}