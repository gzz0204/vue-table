<template>
  <table class="default_table_name">
    <tr>
      <th
        v-for="one in columns"
        :key="one.key"
        @click="sort(one.key, one.sort, one.order)"
        :class="[
          { cur: curKey == one.key, 
          }, 
          one.key,
          one.sort ? sortInfo[one.key].orderClassName:'']"
        v-html="one.title"
      ></th>
    </tr>
    <tr v-for="(item, k) in dataList" :key="k">
      <!-- cloumns字段中，key为INDEX始终显示为索引号排序，不受排序算法影响 -->
      <td
        v-for="(one, n) in keys"
        :key="n"
        v-html="one != 'INDEX' ? item[one] : ++k"
        :class="{ cur: curKey == one }"
      ></td>
    </tr>
    <tr v-if="showNoData">
      <td class="no_data" :colspan="columns.length" v-html="noData"></td>
    </tr>
  </table>
</template>
<script type="text/javascript">
export default {
  props: {
    columns: { type: Array, required:true}, // 定义列属性
    dataSource: { type: Array, required:true }, // 表格数据源，由各行数据组成的数组
    noData: { default: '暂无数据' }
    // sortId: String
  },
  data() {
    return {
      flag: false,
      curKey: '',
      keys: [],
      subkeys: {},
      renders: [],
      dataList: [],
      showNoData: false,
      sortInfo:{},//存储排序类名
      orginDataList:[],// 保存初始化时处理好的原始数据，用于取消排序
    };
  },
  created() {
    this.initSortInfo();
  },
  mounted() {
    if (typeof this.columns !== 'undefined') {
      this.columns_fn(this.columns);
      this.dataSource_fn(this.dataSource);
    }
  },
  beforeCreate() {
    if (this.$route && this.$route.params && this.$route.params.sortkey) {
      this.curKey = this.$route.params.sortkey;
    }
  },
  activated() {
    if (this.$route && this.$route.params && this.$route.params.sortkey) {
      this.curKey = this.$route.params.sortkey;
    }
    if (this.curKey) {
      this.dataList = this.bubbleSort(this.dataList, this.curKey, true);
      this.dataList.unshift();
    }
  },
  watch: {
    columns: function(val) {
      // if (!this.flag) {
      this.columns_fn(val);
      // }
    },
    dataSource: function(val) {
      // console.log(val)
      // console.log(this.flag)
      // if (!this.flag) {
      this.dataSource_fn(val);
      // }
    },
  },
  methods: {
    // 排序
    sort(sortId, sort) {
      if (!sort) return;
      const order = this.handleSortInfo(sortId);
      this.curKey = sortId;
      if(order === 'sortClass'){
        // 不排序
        this.dataList= [...this.orginDataList]
      }else{
        this.dataList = this.bubbleSort(this.dataList, sortId, order === 'desc'? true :false );
        this.dataList.unshift();
      }
    },
    initSortInfo(){
      this.columns.forEach(one => {
        const {key, sort, order} = one;
        if(sort){
          // 有order参数(desc或asc)指定排序时，排序在指定排序和不排序间切换
          if(order){
            this.sortInfo[key]={
              orderArray :['sortClass',order],
              orderClassName: 'sortClass' , // 存储当前排序情况的类名
            }
          }else{
            // 不指定在降序，升序，不排序之间切换
            this.sortInfo[key]={
              orderArray :['sortClass','desc','asc'],
              orderClassName: 'sortClass' , // 存储当前排序情况的类名
            }
          }
        }
      });
    },
    handleSortInfo(sortId){
      if(this.curKey && this.curKey !== sortId){
        this.sortInfo[this.curKey].orderClassName='sortClass'
      }
      const arr = this.sortInfo[sortId].orderArray;
      const index =  arr.indexOf(this.sortInfo[sortId].orderClassName);
      const nextIndex = index+1 > arr.length -1 ? 0: index+1;
      console.log(nextIndex);
      this.sortInfo[sortId].orderClassName = arr[nextIndex];
      console.log(this.sortInfo[sortId].orderClassName)
      console.log(this.sortInfo[sortId].orderArray)
      return arr[nextIndex]
    },
    // 工具函数获取对象的某个键值
    getter(obj, key) {
      var v = obj;
      var args = key.split('.');
      for (var i = 0; i < args.length; i++) {
        if (!v) return null;
        v = v[args[i]];
      }
      return v;
    },
    // 工具函数 冒泡排序
    bubbleSort(array, sortKey, ascendFlag) {
      var i = 0,
        len = array.length,
        j,
        d;
      for (; i < len; i++) {
        for (j = 0; j < len; j++) {
          var vi = this.getter(array[i], sortKey),
            vj = this.getter(array[j], sortKey);
          if (ascendFlag) {
            if (vi > vj) {
              d = array[j];
              array[j] = array[i];
              array[i] = d;
            }
          } else {
            if (vi < vj) {
              d = array[j];
              array[j] = array[i];
              array[i] = d;
            }
          }
        }
      }
      return array;
    },
    columns_fn(val) {
      this.keys = [];
      this.renders = [];
      val.forEach((item) => {
        this.keys.push(item.key);
        if (Object.hasOwnProperty.call(item, 'subkey')) {
          this.subkeys[item.key] = item.subkey;
        }
        this.renders.push(item.render);
      });
    },
    dataSource_fn(val) {
      this.showNoData = val.length < 1;
      this.dataList = [];
      for (let index = 0; index < val.length; index++) {
        let item = val[index];
        let newItem = {};
        for (let i = 0; i < this.keys.length; i++) {
          let key = this.keys[i];
          let render = this.renders[i];
          if (Object.hasOwnProperty.call(item, key)) {
            newItem[key] = render(index, item[key]);
          } else if (Object.hasOwnProperty.call(this.subkeys, key)) {
            let arr = this.subkeys[key];
            let items = arr.map((key) => item[key]);
            newItem[key] = render(index, items);
          } else {
            newItem[key] = render(index, '');
          }
        }
        this.dataList.push(newItem);
      }
      if (this.$route && this.$route.params && this.$route.params.sortkey) {
        this.curKey = this.$route.params.sortkey;
      }
      if (this.curKey) {
        this.dataList = this.bubbleSort(this.dataList, this.curKey, true);
        this.dataList.unshift();
      }
      this.orginDataList = [...this.dataList];
    },
  },
};
</script>
<style scoped lang="less">
.default_table_name{
  width: 100%;
  tr{
    height: 71px;
    font-size: 14px;
    color: #333;
    th{
      &.sortClass{
        position: relative;
        cursor: pointer;
        &:before{
          content:'';
          border-top: 5px solid transparent;
          border-bottom: 5px solid #000;
          border-left: 5px solid transparent;
          border-right: 5px solid transparent;
          position: absolute;
          right: 0px;
          top: 8px;
        }
        &:after{
          content:'';
          border-top: 5px solid #000;
          border-bottom: 5px solid transparent;
          border-left: 5px solid transparent;
          border-right: 5px solid transparent;
          position: absolute;
          right: 0px;
          bottom: 8px;
        }
      }
      &.asc{
        position: relative;
        cursor: pointer;
        &:before{
          content:'';
          border-top: 5px solid transparent;
          border-bottom: 5px solid #000;
          border-left: 5px solid transparent;
          border-right: 5px solid transparent;
          position: absolute;
          right: 0px;
          top: 8px;
        }
      }
      &.desc{
         position: relative;
        cursor: pointer;&:after{
        content:'';
        border-top: 5px solid #000;
        border-bottom: 5px solid transparent;
        border-left: 5px solid transparent;
        border-right: 5px solid transparent;
        position: absolute;
        right: 0px;
        bottom: 8px;
        }
      }
    }
    &:first-child{
      border: 1px solid #6eddc6;
    }
    &:nth-of-type(n+2){
      border-right: 1px solid #F1F1F1;
      border-bottom: 1px solid #F1F1F1;
      border-left: 1px solid #F1F1F1;
    }
     
    &:nth-child(2n+1){
      background: #F9F9F9;
    }
     
    &:nth-child(1){
      height: 43px;
      color: #fff;
      background: #6EDDC6;
    }  
    th,td{
      text-align: center;
    }
      
  }
}
</style>
