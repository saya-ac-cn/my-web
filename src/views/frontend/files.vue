<template>
  <div class="base-content">
    <div class="container">
      <div class="menu-title">
        <div class="menu-name">
          <i class="el-icon-caret-right"></i>文档下载
        </div>
      </div>
      <div id="datagrid" v-loading="listLoading">
        <div v-if="datas !='null'" v-for="data in datas" class="newsitem">
          <el-col :xs="2" :sm="1" :md="1" :lg="1" :xl="1">
            <div class="yearmonthday"><span>{{data.day}}</span> {{data.year}}.{{data.month}}</div>
          </el-col>
          <el-col :offset="1" class="newstitle">
            {{data.filename}}
          </el-col>
          <div style="float: right">
            <a href="javascript:void(0)" @click="downloadFile(data.id)" target="_blank" title="下载" style="color: #752b7d;"><i class="el-icon-download"></i></a>
          </div>
        </div>
        <div v-else class="newsitem" style="justify-content:center;">
          文档好像没有了诶
        </div>
        <div v-if="nextpage != null" class="row newsitem" style="justify-content:center;">
          <a v-on:click="loadMore(nextpage)" title="加载下一页" style="font-size: 2em"><i class="el-icon-view"></i></a>
        </div>
        <div v-else class="newsitem" style="justify-content:center;">
          已经加载完文档了
        </div>
      </div>
    </div>
  </div>
</template>

<script>
  import { queryFile, downloadFiles } from '../../api/api';
  export default {
    name: 'home',
    data () {
      return {
        nextpage: '',
        datas: '',
        downloadUrl: downloadFiles,
        listLoading: false
      }
    },
    methods: {
      loadMore:function(nextpage){
        this.loadData(nextpage);
      },
      //获取列表数据
      loadData(nowpage) {
        let para = {
          nowPage: nowpage,
          pageSize: 10
        };
        this.listLoading = true;
        //NProgress.start();
        queryFile(para).then((datas) => {
          this.listLoading = false;
          let { msg, code, data } = datas;
          if(code === 0)
          {
            // 总数据量
            //this.dataTotal = data.dateSum;
            // 表格数据
            //this.datas = data.grid;
            this.rendering(data);
            console.log(data)
          }else {
            this.$message({
              message: msg,
              type: 'error'
            });
          }
        });
      },
      //渲染文件
      rendering(data) {
        if(JSON.stringify(data) !== '{}')
        {
          //对文件进行二次处理
          for(var i in data.grid) {
            var obj = data.grid[i];
            var b = (obj.date).substr(0,10).split("-");//分割日期，先把空格后的分钟切开
            data.grid[i].month=b[1];
            data.grid[i].year=b[0];
            data.grid[i].day=b[2];
          }
          console.log(data)
          //第一页采用直接覆盖的显示方式
          if(data.pageNow == 1 || data.pageNow == '1')
          {
            this.datas = data.grid;//绑定到Vue
          }
          else
          {
            this.datas= this.datas.concat(data.grid);//追加，合并
          }
        }
        else
        {
          console.log('-----')
          this.datas = null;
        }
        //显示是否加载下一页(当前页是最后一页)
        if(data.pageNow == data.totalPage)
        {
          this.nextpage = null;
        }
        else
        {
          this.nextpage = data.pageNow + 1;
        }
      },
      downloadFile(id){
        // 下载文件
        const form = document.createElement('form')
        form.id = 'form'
        form.name = 'form'
        document.body.appendChild(form);
        form.method = "GET";//请求方式
        form.action = this.downloadUrl + id ;
        form.submit();
        document.body.removeChild(form);
      }
    },
    mounted() {
      this.loadData();
    }
  }
</script>

<style lang="scss" scoped>
  .container{
    .menu-title{
      border-bottom: 2px solid #edeaf1;
    }

    #datagrid{
      /*在动态页引入新的样式*/
      .yearmonthday {
        border: 1px solid #edeaf1;
        color: #5b317d;
        background: #edeaf1;
        width: 58px;
        text-align: center;
        font-size: 12px;
        float: left;
        _display: inline;
        padding-bottom: 5px;

        span {
          display: block;
          font-size: 24px;
          line-height: 24px;
          padding: 8px 0 2px 0;
          zoom: 1;
        }
      }
      .newstitle{
        font-size: 1.2em;
        vertical-align: middle;
        display: inline-block;
        a{
          color: #5b317d;
          text-decoration: none;
        }
      }
      .newsitem
      {
        border-bottom: 1px dotted #e8e6e6;
        align-items:center;
        display:flex;/*Flex布局*/
        display:-webkit-flex;
        padding-top: 1em;
        padding-bottom: 1em;
      }
    }
  }
</style>
