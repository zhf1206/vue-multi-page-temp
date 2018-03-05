<template>
  <div class="place_select_win" id="place_select_win">
    <div class="place_select_search">
      <a class="backbtn" @click="hideWin"><img src="./ic-back-b.png"></a>
      <div class="search_input clearfix">
        <!--//鼠标获取焦点时-->
        <input type="search" v-model="searchKey" placeholder="中文/拼音" @focus="startInput" @input="searchCity()" @blur="outInput" />
        <span class="icon_search"><img src="./ic-search-hig.png"></span>
        <span class="icon_clear" @click="clearSearch"><img src="./search-clear.png"></span>
      </div>
    </div>
    <div class="place_select_tabs" v-if="showG&&rangeType">
      <div class="tab" @click="changeRange(0)" :class="range === 0?'on':''">国内<div class="choice"></div></div>
      <div class="tab" @click="changeRange(1)" :class="range === 1?'on':''">国际/港澳台<div class="choice"></div></div>
    </div>
    <div v-show="historySesult" class="main" :class="rangeType?'':'man-noInternal'">
      <div class="place_select_section" id="place_select_history">
        <div class="place_select_label">搜索历史</div>
        <ul class="clearfix">
          <li v-for="history in data_showHistory" @click="selectCity(history)">{{history.cityName}}</li>
        </ul>
      </div>
      <div class="place_select_section" id="place_select_hotCity">
        <div class="place_select_label">热门城市</div>
        <ul class="clearfix">
          <li v-for="city in data_show.recommendCity" @click="selectCity(city)">{{city.cityName}}</li>
        </ul>
      </div>
      <div class="maincontent" id="maincontent">
        <ul class="place_select_citys" v-for="(citys,letter) in data_show.result">
          <li class="place_select_label" :id="['letter_'+letter]">{{letter}}</li>
          <li v-for="city in citys" @click="selectCity(city)" class="select_city"><div>{{city.cityName}}</div></li>
        </ul>
        <ul class="place_select_point"  :class="{fixed:fixedslidar}">
          <li v-for="letter in letters" @click.stop.prevent="gotoLink(letter)">{{letter}}</li>
        </ul>
      </div>
    </div>
    <search v-show="inputstatus" ref="searchInput" @close="hideSearch" @selectCity="selectCity"></search>
  </div>
</template>

<script>
  import {throttle} from 'vux';
  import search from './search'
  import {showMessage,getToken,Hi} from 'assets/js/ddbase';
  export default {
    data() {
      return {
        letters:[],
        historys:[],
        hotcitys:[],
        citys:[],
        range:0,//0 国内；1国际
        data_gn:null,
        data_gj:null,
        data_show:[],
        data_showHistory:[],
        fixedslidar:false,//字母列表是否悬浮
        inputstatus:false,//是否是输入状态
        showG:true,  //鼠标获取焦点隐藏国内搜索
        historySesult:true,//搜索历史、热门城市开关
        searchKey:''
      }
    },
    props:{
        type:{
            default:'single'
        },
        formatstr:{
            default:'YYYY-MM-DD'
        },
       value:{
            default:function () {
              return [];
            }
       },
      rangeType:{
        default:true // true - 国内和国际    false  -  不显示国际
      }
    },
    components:{search},
    mounted(){
        this.getLetters();
        this.getCity(this.range);
        this.scrollSlidar();
    },
    methods:{
      hideWin: function () {
        this.$emit('close');
      },
      updateHistory:function (city) {
        let history = JSON.parse(localStorage.getItem('flight_place_searchHistory_'+this.range)) || [];
        for (let his in history) {
            let item = history[his];
          if (item.cityCode == city.cityCode) {
            history.splice(his,1);
          }
        }
        history.unshift(city);
        history = history.slice(0,10);
        localStorage.setItem('flight_place_searchHistory_'+this.range,JSON.stringify(history));
        if(this.range) this.data_gj.history = history;
        else this.data_gn.history = history;
        this.data_showHistory = history;
//        this.data_show = Object.assign({}, this.data_show,{history:history});
      },
      clearSearch: function () {
        this.searchKey = '';
        this.showG = true;
        this.inputstatus = false;
        this.historySesult=true;
        Hi.removeClass(Hi.body,'overhidden_hi');
        Hi.body.scrollTop = this.scrollTop;
      },
      hideSearch:function () {
        this.inputstatus = false;
      },
      searchCity:function () {
        this.$refs.searchInput.getResult(this.searchKey,this.range);
        this.historySesult=false;
        Hi.addClass(Hi.body,'overhidden_hi');
      },
      startInput: function () {
        this.showG = false;
        this.inputstatus = true;
        this.historySesult=false;
        Hi.addClass(Hi.body,'overhidden_hi');
      },
      outInput: function () {
        //当搜索框没关键字时，恢复成默认的推荐样式
        if(!this.searchKey || this.searchKey==""){
        this.showG = true;
        this.inputstatus = false;
        this.historySesult=true;
        Hi.removeClass(Hi.body,'overhidden_hi');
        Hi.body.scrollTop = this.scrollTop;
        }
      },
      selectCity:function (city) {
        this.updateHistory(city);
        city.range = this.range;
        this.$emit('selectCity',city);
        this.clearSearch();
      },
      gotoLink: function (letter) {
        var letterDom = document.getElementById('letter_'+letter);
        if(letterDom){
          let offsetTop = letterDom.offsetTop;
          let height1 = document.getElementById('place_select_history').clientHeight;
          let height2 = document.getElementById('place_select_hotCity').clientHeight
          document.getElementById('place_select_win').scrollTop = offsetTop + height1 + height2 + 16;
        }
      },
      scrollSlidar:function () {
        let winDom = document.getElementById('place_select_win');
        winDom.onscroll = throttle(()=>{
          let scrollTop = winDom.scrollTop;
          let offsetTop = document.getElementById('maincontent').offsetTop;
          if(scrollTop + 100 >= offsetTop) this.fixedslidar = true;
          else this.fixedslidar = false;
        },50)
      },
      changeRange: function (range) {
        if(this.range === range) return;
        this.range = range;
        if(range){
          this.data_showHistory = [];
          try {
            this.data_showHistory = this.data_gj.history;
          } catch (e) {

          }
          return this.data_gj ? this.data_show = this.data_gj : this.getCity(range);

        }else {
          this.data_showHistory = [];
          //搜索历史
          let historys = JSON.parse(localStorage.getItem('flight_place_searchHistory_'+this.range)) || [];
          //遍历历史
          for(var i=0;i<historys.length;i++){
            if(historys.length>10){
              historys.splice(0,1)
            }
            this.data_showHistory.push(historys[i]);
          }
         return this.data_gn ? this.data_show = this.data_gn : this.getCity(range);
        }

      },
      getLetters: function () {
        for(var i=65;i<91;i++){
          this.letters.push(String.fromCharCode(i));
        }
      },
      getCity:function (flag) {
        let action = flag ? 'queryInternationalCityList' : 'queryCityList';
        this.$ajax({
          url:'/hiflight/api.go?action='+action,
          showLoading: flag? true :false,
          success:(response)=>{
            let result = response.data;
            this.data_show = result;
            //搜索历史
            let historys = JSON.parse(localStorage.getItem('flight_place_searchHistory_'+this.range)) || [];
            //遍历历史
            for(var i=0;i<historys.length;i++){
              if(historys.length>10){
                historys.splice(0,1)
              }
                this.data_showHistory.push(historys[i]);
            }
            let history_gn = localStorage.getItem('flight_place_searchHistory_0');
              if(flag){
                result.history = JSON.parse(localStorage.getItem('flight_place_searchHistory_1'));
                this.data_gj = result;
              }else{
                result.history = JSON.parse(localStorage.getItem('flight_place_searchHistory_0'));
                this.data_gn = result;
              }
          }
        })
      }
    }
  }
</script>

<style lang="less">
  .clearfix:after{content:".";display:block;height:0;clear:both;visibility:hidden;}
  .place_select_win{
    position: fixed;
    -webkit-overflow-scrolling: touch;
    width: 100%;
    height: 100%;
    top:0;
    left: 0;
    right:0;
    bottom: 0;
    z-index: 1000;
    background-color: #fff;
    overflow: auto;
    .place_select_search{
      position: fixed;
      position: -webkit-sticky;
      top:0;
      z-index: 1000;
      height: 32px;
      padding:14px 0;
      background-color: #ffffff;
      box-shadow: inset 0 -1px 0 0 rgba(235, 235, 235, 0.5);
      clear: both;
      .backbtn{
        position: absolute;
        top:0;
        left:0;
        width: 48px;
        height: 48px;
        padding-top: 12px;
        line-height: 48px;
        text-align: center;
        float: left;
      }
      .search_input{
        position: relative;
        margin-left: 48px;
        margin-right: 80px;
        height: 32px;
        float:left;
        .icon_search{
          display: inline-block;
          position: absolute;
          width: 24px;
          height: 24px;
          left: 4px;
          top: 50%;
          opacity: 0.5;
          margin-top: -12px;
        }
        .icon_clear{
          display: inline-block;
          position: absolute;
          width: 16px;
          height: 16px;
          right: 10px;
          top:8px;
        }
        input[type=search]{
          line-height: 32px;
          white-space: nowrap;
          height: 32px;
          border: none;
          width: 300px;
          padding: 0 32px;
          border-radius: 4px;
          background: #f5f5f5;
          font-size: 16px;
          color: rgba(0, 0, 0, 0.21);
        }
        input[type=search]::-webkit-search-cancel-button{
          -webkit-appearance: none;//此处只是去掉默认的小×
        }
      }
    }
    .place_select_tabs{
      overflow: hidden;
      position: fixed;
      position: -webkit-sticky;
      top:60px;
      left:0;
      right:0;
      z-index: 10;
      height: 40px;
      background-color: #fff;
      box-shadow: inset 0 -1px 0 0 rgba(235, 235, 235, 0.5);
      clear: both;
      .tab{
        position: relative;
        font-size: 14px;
        text-align: center;
        color: #8d8d8d;
        line-height: 40px;
        width:50%;
        float: left;
      }
      .choice {
        position: absolute;
        left:41%;
        bottom:0;
        width: 32px;
        height: 2px;
        background-color: #f67808;
        display: none;
      }
      .on{
        color: #f67808;
        font-weight: 500;
        .choice{
          display: inline-block;
        }
      }

    }
    .place_select_point{
      position: absolute;
      top: 20px;
      right:0;
      font-size: 14px;
      font-weight: 500;
      color: #000;
      line-height: 125%;
      li{
        width:50px;
        text-align: center;
      }
      &.fixed{
        position: fixed;
        top:110px;
      }
    }
    .main{
      position: absolute;
      top:100px;
    }
    .man-noInternal{
      top:60px;
    }
    .maincontent{
      position: relative;
    }
    .place_select_label{
      height: 60px;
      line-height: 60px;
      padding: 0 16px;
      font-size: 14px;
      font-weight: 500;
      text-align: left;
      color: rgba(0, 0, 0, 0.54);
    }
    .place_select_section{
      ul{
        padding:0 16px 10px;
        clear: both;
      }
      li{
        padding:0 10px;
        margin: 0px 10px 10px 0;
        height: 36px;
        line-height: 36px;
        border-radius: 4px;
        background-color: rgba(0, 0, 0, 0.04);
        font-size: 14px;
        text-align: center;
        color: rgba(0, 0, 0, 0.54);
        display: inline-block;
        float: left;
      }
    }
    .place_select_citys{
      ul{
        clear:both;
      }
      li.select_city{
        padding: 0 16px;
        box-sizing: border-box;
        font-size: 14px;
        text-align: left;
        color: rgba(0, 0, 0, 0.87);
        display: block;
        div{
          width:100%;
          height: 48px;
          line-height: 48px;
          border-bottom: solid 1px #ebebeb;
          span{
            color:#afafaf;
            margin-left: 16px;
          }
        }

      }
      .place_select_label{
        height: 32px;
        line-height: 32px;
        background-color: #fafafa;
        padding: 0 16px;
        font-size: 14px;
        font-weight: 600;
        text-align: left;
        color: rgba(0, 0, 0, 0.38);
        border:0;
      }
    }
  }

</style>
