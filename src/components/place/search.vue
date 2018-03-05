<template>
  <div class="search_result">
    <!--没有搜索结果start-->
    <div class="content no_search" v-show="resultstatus">
      <div class="no_result">
        <img src="./search-nofound.png"/>
        <div class="msg">找不到您想要的结果</div>
      </div>
    </div>
    <!--没有搜索结果end-->
    <!--搜索匹配start-->
    <div class="content citys_match" v-show="showresult">
      <ul class="citys" v-for="(citys,letter) in resultList">
        <li v-for="city in citys" @click="selectCity(city)"><div>{{city.cityName}}<span>{{city.cityCode}}</span></div></li>
      </ul>
    </div>
    <!--搜索匹配end-->
  </div>
</template>

<script type="text/ECMAScript-6">
  export default{
      data: function () {
        return {
          resultstatus: false,
          showresult: false,
          resultList:{}
        }
      },
      methods:{
          selectCity: function (city) {
            this.resultList = [];
            this.$emit('selectCity',city);
          },
          hideWin: function (){
            this.$emit('close');
          },
          getResult: function (key,range) {
            if(!key){
              this.resultstatus = false;
              this.showresult = false;
              return;
            }
            let type = range ? 2 : 1;
            this.$ajax({
              url:'/hiflight/api.go?action=searchCity',
              data:{
                keyword:key,
                type:type
              },
              success: response => {
                  let result = response.data.result;
                  this.resultList = result;
                  if(JSON.stringify(result) == "{}"){
                    this.resultstatus = true;
                    this.showresult = false;
                  }else{
                    this.resultstatus = false;
                    this.showresult = true;
                  }
              }
            })
          }
      }
  }
</script>

<style lang="less" rel="stylesheet/less">
  .search_result{
    position: fixed;
    top:60px;
    left:0;
    right:0;
    bottom: 0;
    background:transparent;
    z-index: 10001;
    .content{
      position: absolute;
      top:0;
      left:0;
      right:0;
      bottom:0;
      background: #fff;
      z-index: 20;
      margin-top:0;
    }
    .no_result {
      padding-top:96px;
      font-size: 12px;
      background-size: auto 100px;
      text-align: center;
      line-height: 22px;
      color: rgba(0, 0, 0, 0.21);
      margin-top: 15px;
    }
    .citys{
      ul{
        clear:both;
      }
      li{
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
      .label{
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
