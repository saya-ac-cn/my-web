<template>
  <el-container class="this-container">
    <div class="background">
      <div class="item-div" v-for="item in dataList">
        <img v-bind:style="{ 'background-image': 'url(' + item.src + ')',}" v-show="item.isShow">
      </div>
    </div>
    <el-header class="this-header">
        <el-col :span="19" :offset="1" class="header-center">
          <ul v-for="(item,index) in $router.options.routes" :key="index" v-if="item.path === '/index'">
            <li v-for="(child,index) in item.children" :index="(index+1).toString()" v-if="!child.hidden" :key="child.path">
              <a v-bind:href="child.path">{{child.name}}</a>
            </li>
          </ul>
        </el-col>
        <el-col :span="3" :offset="1" style="color:#dbdbdb" class="header-center">
          {{ date | formatDate }}
        </el-col>
    </el-header>
    <el-main class="this-main">
      <el-row :gutter="20" class="search-block">
        <el-col :span="20" :offset="2" class="search-div">
          <div class="search-line1">
            <div class="search-div-tag1">全网搜</div>
            <div class="search-div-tag2"></div>
          </div>
          <div class="search-line2">
            <el-autocomplete
              class="inline-input search-input"
              v-model="search"
              :clearable="true"
              :fetch-suggestions="querySearch"
              @select="handleSelect"
              placeholder="请输入内容"
              :trigger-on-focus="false"
              @keyup.enter.native="goSearch"
            ></el-autocomplete>
            <el-button class="search-button" icon="el-icon-search" v-on:click="goSearch"></el-button>
          </div>
        </el-col>
      </el-row>
    </el-main>
    <el-footer class="this-copright">
      <el-row :gutter="20" style="width: 100%">
        <el-col :span="20" :offset="2">
          Copyright &copy; 2016-{{copyrightDate}} &nbsp; Saya.ac.cn-暖心阁 版权所有<br>国家工信部域名备案信息：蜀ICP备16013222号-2
        </el-col>
      </el-row>
    </el-footer>
  </el-container>
</template>

<script>
  import { baiduSearchSelect, baiduSearchWord } from '../../api/api';
  export default {
    name: 'index',
    data() {
      return {
        date: new Date(),
        search: '',
        restaurants: [],
        interval:'',
        // 打印到版权区域的时间
        copyrightDate: new Date().getFullYear(),
        dataList:[
          {name:'1',src:require('../../assets/picture/login/login_background1.jpg'),isShow:true,seq:1},
          {name:'2',src:require('../../assets/picture/login/login_background2.jpg'),isShow:false,seq:2},
          {name:'3',src:require('../../assets/picture/login/login_background3.jpg'),isShow:false,seq:3},
          {name:'4',src:require('../../assets/picture/login/login_background4.jpg'),isShow:false,seq:4},
          {name:'5',src:require('../../assets/picture/login/login_background5.jpg'),isShow:false,seq:5}
        ],
      };
    },
    filters:{
      formatDate(value) {
        var date=new Date(value);
        var year=date.getFullYear();
        /*
        *在日期格式中，月份是从0开始的，因此要加0
        *使用三元表达式在小于10的前面加0，以达到格式统一  如 09:11:05
        */
        var month= date.getMonth()+1<10 ? "0"+(date.getMonth()+1) : date.getMonth()+1;
        var day=date.getDate()<10 ? "0"+date.getDate() : date.getDate();
        var hours=date.getHours()<10 ? "0"+date.getHours() : date.getHours();
        var minutes=date.getMinutes()<10 ? "0"+date.getMinutes() : date.getMinutes();
        var seconds=date.getSeconds()<10 ? "0"+date.getSeconds() : date.getSeconds();
        // 拼接
        return year+"-"+month+"-"+day+" "+hours+":"+minutes+":"+seconds;
      },
    },
    methods: {
      changeImg(seq) {
        var newData = this.dataList;
        for (var i = 0;i <newData.length; i++) {
          if(newData[i].seq==seq)
            newData[i].isShow=true;
          else
            newData[i].isShow=false;
        }
        this.dataList = newData;
      },
      scollImg() {
        var newData = this.dataList.filter(function(val){return val.isShow})[0];
        if(newData.seq==this.dataList.length){
          this.changeImg(1);
        }else{
          this.changeImg(newData.seq+1);
        }
      },
      startInterval() {
        let that = this;
        this.interval = setInterval(function(){
          that.scollImg();
        },10000)
      },
      querySearch(queryString, cb) {
        if(queryString.trim() === ''){
          this.search = ''
          cb([]);
        }
        this.$http.jsonp(baiduSearchSelect,{
          params:{
            wd:queryString
          },
          jsonp:'cb'
        }).then(res=>{
          //console.log(res.data.s);//把百度服务器返回的数据传给arr数组
          var getData = res.data.s;
          if(getData.length > 0){
            var result = new Array()
            for(var i = 0; i < getData.length ; i++){
              result.push({"value":getData[i]})
            }
            cb(result)
          }else{
            cb([]);
          }
        })
      },
      goSearch(){
        window.location.href = baiduSearchWord + '?wd=' + this.search
      },
      handleSelect(item) {
        // 用户选择
        //console.log(item.value);
        window.location.href = baiduSearchWord + '?wd=' + item.value
      }
    },
    created() {
      this.startInterval();
    },
    mounted () {
      var _this = this; //声明一个变量指向vue实例this,保证作用域一致
      this.timer = setInterval(function() {
        _this.date = new Date();//修改数据date
      }, 1000);
    },
    beforeDestroy () {
      if(this.timer) {
        clearInterval(this.timer);//在vue实例销毁钱，清除我们的定时器
      }
    }
  }
</script>
<style>
  .search-input .el-input {
    width: 100% !important;
    height: 100% !important;
  }
  .search-input .el-input input{
    height: 100%;
    border: none;
  }
</style>
<style lang="scss" scoped>
  /*主要的字体色(导航菜单)*/
  $main-text-color:#8FBC8B;
  /*主要的字体色鼠标(导航菜单)hover*/
  $main-text-hov-color:#90EE90;
  /*版权区域背景色*/
  $copyright-color:#D3D3D3;
  /*搜索区域背景色*/
  $search-backcolor:#fff;
  .this-container::-webkit-scrollbar {
    display: none;
  }
  .this-container {
    position: absolute;
    top: 0px;
    bottom: 0px;
    width: 100vw;
    overflow-x: hidden;
    overflow-y: hidden;
    .background{
      position: absolute;
      z-index: -999;
      width: 100vw;
      height: 100vh;
      top: 0px;
      bottom: 0px;
      .item-div{
        width: 100vw;
        height: 100vh;
        padding: 0;
        margin: 0;
        top: 0;
        bottom: 0;
        position: absolute;
        img{
          width: 100vw;
          height: 100vh;
          padding: 0;
          margin: 0;
          top: 0;
          bottom: 0;
          position: absolute;
          /* 背景图垂直、水平均居中 */
          background-position: center center;
          /* 背景图不平铺 */
          background-repeat: no-repeat;
          /* 当内容高度大于图片高度时，背景图像的位置相对于viewport固定 */
          background-attachment: fixed;
          /* 让背景图基于容器大小伸缩 */
          background-size: cover;
          background-attachment: fixed;
        }
      }
    }
    .this-header{
      height: 70px !important;
      width: 100vw;
      padding: 0;
      font-family: Arial,Helvetica,Sans-Serif;
      /*opacity: 0.8;*/
      /*filter: alpha(opacity=70);*/
      /*background-color: #dbdbdb;*/
      .header-center{
        height: 100%;
        /*flex 布局*/
        display: flex;
        /*实现垂直居中*/
        align-items: center;
        /*实现水平居中*/
        /*justify-content: center;*/
        text-align: justify;
      }
      ul{
        width: 100%;
        height: 100%;
        li{
          width: 6.5em;
          height: 100%;
          float: left;
          list-style: none;
          /*flex 布局*/
          display: flex;
          /*实现垂直居中*/
          align-items: center;
          /*实现水平居中*/
          /*justify-content: center;*/
          text-align: justify;
          a{
            opacity: .8;
            font-weight: 400;
            color: $main-text-color;
            text-decoration: none;
          }
          a:hover{
            color: $main-text-hov-color;
            text-decoration: none;
          }
        }
      }
    }
    .this-main{
      width: 100vw;
      height: auto;
      min-height: 300px;
      overflow-x: hidden;
      overflow-y: hidden;
      .search-block{
        width: 100%;
        padding-top: 3em;
        .search-div{
          height: 80px;
          padding: 0px !important;
          .search-line1{
            /*width: 75px;*/
            height: 30px;
            .search-div-tag1{
              float: left;
              width: 60px;
              height: 100%;
              background-color: $search-backcolor;
              line-height: 30px;
              text-align: center;
              color: $main-text-color;
              border-top-left-radius: 4px;
            }
            .search-div-tag2{
              float: left;
              /*margin-left: 10px;*/
              float: left;
              width: 0;
              height: 0;
              border-width: 3px;
              border-style: solid;
              width: 0;
              height: 0;
              border-width: 15px;
              border-style: solid;
              border-color:transparent transparent $search-backcolor $search-backcolor;
            }
          }
          .search-line2{
            height: 50px;
            width: 45em;
            border-bottom-left-radius: 4px;
            border-bottom-right-radius: 4px;
            border-top-right-radius: 4px;
            background-color: $search-backcolor;
            .search-input{
              float: left;
              height: 100%;
              border: none;
              width: 90%;
              .el-input{
                width: 100% !important;
                height: 100% !important;
                input{
                  height: 100%;
                  border: none;
                }
              }
            }
            .search-button{
              width: 4em;
              height: 100%;
              border: none;
              float: right;
            }
            .search-button:hover{
              background: transparent;
            }
          }
        }
      }
    }
    .this-main::-webkit-scrollbar {
      display: none;
    }
    .this-copright{
      width: 100vw;
      height: 100px;
      /*flex 布局*/
      display: flex;
      /*实现垂直居中*/
      align-items: center;
      /*实现水平居中*/
      /*justify-content: center;*/
      text-align: justify;
      opacity: 0.8;
      filter: alpha(opacity=70);
      background-color: $copyright-color;
    }
  }
</style>
