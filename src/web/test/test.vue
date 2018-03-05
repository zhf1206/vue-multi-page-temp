<template>
    <div class="content">
        <popover>
            <div slot="content" class="popover-demo-content">
                包含：快速值机、快速安检、快速通道三项服务
              </div>
             <div class="button"></div>
        </popover>
        <div>
          <pre>倒计时：</pre>
          <clocker time="2018-08-01" v-if="flag">
            <span style="color:red">%D 天</span>
            <span style="color:green">%H 小时</span>
            <span style="color:blue">%M 分 %S 秒</span>
          </clocker>
        </div>
        <div class="baidumap">
          <input type="text" id="baidumap" @change="changeAds"/>
        </div>
    </div>
</template>

<script type="text/ECMAScript-6">
    import Util from 'assets/js/global';
    import Popover from 'components/popover/index';
    import { Clocker } from 'vux'
    export default{
      data(){
        return {
          flag:true,
        }
      },
      components:{Popover,Clocker},
      mounted:function(){
        let _this = this;
        setTimeout(() => {
          _this.flag = false;
        }, 3000);
        AMap.plugin('AMap.Autocomplete',function(){//回调函数
          //实例化Autocomplete
          var autoOptions = {
            city: "北京", //城市，默认全国
            input:"baidumap"//使用联想输入的input的id
          };
          let autocomplete= new AMap.Autocomplete(autoOptions);
          console.log(autocomplete)
          //TODO: 使用autocomplete对象调用相关功能
        })
      },
      methods:{
        changeAds: function (evt) {//将地址转化成经纬度
          AMap.service('AMap.Geocoder',function(){//回调函数
            //实例化Geocoder
            let geocoder = new AMap.Geocoder();
            geocoder.getLocation("北京市海淀区苏州街", function(status, result) {
              if (status === 'complete' && result.info === 'OK') {
                //经度
                console.log(result.geocodes[0].location.lng)
                //纬度
                console.log(result.geocodes[0].location.lat)
              }else{
                //获取经纬度失败
              }
            });
          })
        }
      }
    }
</script>

<style lang="less" rel="stylesheet/less">
    .content{
        margin-top:10px;
        .button{
            width: 10px;
            height:10px;
            border-radius: 100%;
            background: red;
            margin-left:10px;
        }
      .baidumap{
        margin-top:100px;
        input[type=text]{
          border: 1px solid #aaa;
        }
      }
    }
</style>

