<template>
  <div class="home">
    <div class="table_title">球队积分榜。点击切换排序：积分（升序排列），进球（降序排列），失球（降序、升序）</div>
    <tableItem  v-if="tableData" class="table_1 table" v-bind="tableData"></tableItem>
    <div class="table_title">球队积分榜-无数据表格-默认提示（暂无数据）</div>
    <tableItem  v-if="tableData2" class="table_2 table" v-bind="tableData2"></tableItem>
    <div class="table_title">球队积分榜-无数据表格-自定义提示</div>
    <tableItem  v-if="tableData3" class="table_2 table" v-bind="tableData3"></tableItem>
  </div>
</template>

<script>
import tableItem from '@/components/table'
// import {get} from '@/utils/request.js'
export default {
  name: 'Home',
  data() {
    return {
      tableData: null,
      table_1:[],
      tableData2: null,
      table_2:[],
      tableData3: null,
      table_3:[],
      columns: [
        {
          title: '索引',
          key: 'INDEX',
          render: (index, item) => `<div class="index">${item}</div>`,
        },
        {
          title: '名次',
          key: 'sort',
          render: (index, item) => `<div class="rank">${item}</div>`,
        },
        {
          title: '球队ID',
          key: 'teamId',
          render: (index, item) => `<div class="rank">${item}</div>`,
        },
        {
          title: '球队',
          key: 'teamInfo',
          subkey: ['team', 'logo'],
          render: (index, [team, logo]) => {
            let res = `<div class="teamInfo"><img class="logo" src="${logo}"/><span class="teamName">${team}</span></div>`;
            return res;
          },
        },
        {
          title: '积分',
          key: 'points',
          sort: true,
          order: "asc",
          render: (index, item) => item,
        },
        {
          title: '场次',
          key: 'games',
          render: (index, item) => item,
        },
        {
          title: '进球',
          key: 'goal',
          sort: true,
          order: "desc",  // 点击切换再降序和不排序之间切换
          render: (index, item) => item,
        },
        {
          title: '失球',
          key: 'lossGoal',
          sort: true,
          render: (index, item) => item,
        },
        {
          title: '胜/平/负',
          key: 'spf',
          subkey: ['win', 'draw', 'loss'],
          render: (index, [win, draw, loss]) => {
            return `${win}/${draw}/${loss}`;
          },
        },
        {
          title:'<span class="teamInfo">赛况</span>',
          key:'linkInfo',
          subkey:['linkNews','linkReport','roomId','teamId'],
          render:(index, [linkNews,linkReport,roomId,teamId]) => {
            let res = [];
            if(linkNews){
              res.push(`<a class="cube" href="" target="_blank" href="${this.matchDetailUrl(
                '/photo/',
                teamId
              )}">新闻</a>`)
            }
            if(linkReport){
               res.push(`<a class="cube" href="" target="_blank" href="${this.matchDetailUrl(
                '/report/',
                teamId
              )}">战报</a>`)
            }
            if(roomId){
               res.push(`<a class="cube" href="" target="_blank" href="${this.matchDetailUrl(
                '/report/',
                teamId
              )}">直播</a>`)
            }
            return res.join('')
          },
        }
      ]
    }
  },
  components: {
    tableItem
  },
  created () {
    this.getTableData();
  },
  methods: {
    async getTableData(){
      // 接口获取数据（mock/home.json里配置了假数据）
      // let data = await get('/api/table1')
      // console.log(data)
      // this.table_1 = data.data
      this.table_1 =[
      {
        "difference": 14,
        "draw": 3,
        "games": 11,
        "goal": 23,
        "groupName": "未分组",
        "logo": "https://relottery.ws.126.net/nano_team/soccer/5_10183_nano.png",
        "loss": 1,
        "lossGoal": 9,
        "points": 24,
        "sort": 1,
        "team": "热刺",
        "teamId": 10183,
        "win": 7,
        "linkNews": true,
        "linkReport": true,
        "roomId": 12
      },
      {
        "difference": 9,
        "draw": 3,
        "games": 11,
        "goal": 26,
        "groupName": "未分组",
        "logo": "https://relottery.ws.126.net/nano_team/soccer/5_10249_nano.png",
        "loss": 1,
        "lossGoal": 17,
        "points": 23,
        "sort": 2,
        "team": "利物浦",
        "teamId": 10249,
        "win": 7,
        "linkNews": true,
        "linkReport": false,
        "roomId": 12
      },
      {
        "difference": 14,
        "draw": 4,
        "games": 11,
        "goal": 25,
        "groupName": "未分组",
        "logo": "https://relottery.ws.126.net/nano_team/soccer/5_10180_nano.png",
        "loss": 1,
        "lossGoal": 11,
        "points": 22,
        "sort": 3,
        "team": "切尔西",
        "teamId": 10180,
        "win": 6
      },
      {
        "difference": 6,
        "draw": 0,
        "games": 11,
        "goal": 21,
        "groupName": "未分组",
        "logo": "https://relottery.ws.126.net/nano_team/soccer/5_10247_nano.png",
        "loss": 4,
        "lossGoal": 15,
        "points": 21,
        "sort": 4,
        "team": "莱斯特城",
        "teamId": 10247,
        "win": 7
      }
    ]
      
      // 表格1
      this.tableData={
        columns:this.columns,
        dataSource: this.table_1
      }
      // 无数据表格
      this.tableData2 ={
        columns:this.columns,
        dataSource: this.table_2
      }
      // 无数据表格 自定义无数据样式
       this.tableData3 ={
        columns:this.columns,
        dataSource: this.table_3,
        noData: '<div class="no_data_wrap">此页面无数据</div>'
      }
    },
    matchDetailUrl(path, mid) {
      if (!mid) return
      let url = `/live.html#/${path}${mid}`
      return url
    }   
  }
};
</script>
<style lang="less" scoped>
.home{
  width: 960px;
  margin: 0 auto;
}
.table_title{
  width: 100%;
  height: 40px;
  line-height: 40px;
  padding-left: 20px;
  background: #bdf9e0;
  text-align: left;
}
.table{
  margin-bottom: 30px;
}
.table_1{
  width: 100%;
  /deep/ tr{
    &:nth-child(2),
    &:nth-child(3){
      .rank {
        font-family: Arial-BoldItalicMT;
        color: #0b84db;
      }
    }
    .teamInfo{
      .logo{
        width: 30px;
        height: 30px;
        vertical-align: middle;
        margin-right: 5px;
      }
      .teamName{
        vertical-align: middle;
      }
    }
    .cube{
      margin-right: 5px;
      padding: 1px 7px;
      border: 1px solid #EAEAEA;
      border-radius: 4px;
      color: #333;
      text-decoration: unset;
      background: #FFF;
      &:hover{
        border: 1px solid #79B6FE;
        color: #fff;
        background :#79B6FE;
      }
    }
  }
}
/deep/.no_data_wrap{
  position: relative;
  width: 100%;
  margin-left: auto;
  margin-right: auto;
  padding-top: 146px;
  font-size: 18px;
  color: #41d8c6;
  text-align: center;
  background: url('../assets/nodata_bg.png') no-repeat;
  background-position: center top;
}
</style>
