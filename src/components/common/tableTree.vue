<template>
  <div class="tableTree-cot">
    <div v-show="tableData && tableData.length" style="position: relative;">
      <div ref="wrapper" style="width: 100%;position: relative;">
        <table>
          <thead>
          <tr>
            <th v-for="(item,key) in tableHead"
            >
                            <span
                              class="arrowCom"
                              :class="{
                                          'arrowUpActive':item.ascending === true,
                                          'arrowDownActive':item.ascending === false,
                                          'arrowCom':(item.canSort && item.ascending==='')
                                         }"
                            >
                                        {{item.name}}
                            </span>
            </th>
          </tr>

          </thead>
          <tbody >
              <tr  class="lvTableTr" v-for="(item,itemKey) in lvTableData"
                   v-if="item.isShow "
                   :tableLv="item.lv"
                   :class="arrowClassGet(item)"
                   @click.stop ="showChild(item,lvTableData)"

              >
                <td >
                  {{item.isShow || item.lv==1}}
                  <div class="hideMoreText">
                    {{item[tableHead[0].sortName]}}
                  </div>
                </td>
                <td v-for="(i,idx) in tableHead" v-if="idx!==0"
                    v-html="vualeFormat(item[i.sortName],i.bit,i.symbol,'tdNum','tdUnit') "
                >
                </td>
              </tr>

          </tbody>
        </table>

      </div>
    </div>
  </div>
</template>

<script>
  export default {
    name: 'tableTree',
    props: {
      tableData: Array,
      tableHead: Array,

    },
    data () {
      return {
        msg: '',
        lvTableData:[]
      }
    },
    mounted(){
    },
    methods: {
        arrowClassGet(item){
            /*
            * 判断lv层级属于底基层
            * */
            let str =''
            switch(item.id.split('-').length){
              case 1:
                  break;
              default:
                  break;
            }
            if(item.showChild){
                str +='active';
            }
            return str;


        },
      vualeFormat(num, bit = 2, str = '', c1 = '', c2 = '') {
        /*
         *  num 数字
         *  bit 保留位数
         *   str 后缀单位
         *   c1 主class  (包含单位和数字)
         *    C2 单位class  只包含单位
         * */
        if (isNaN(parseInt(num))) {
          return num || '-' // 原形返回
        } else {
          num = num - 0 //字符串转数字类型
//                     亿以上主动转行单位
//                    if(/万/.test(str) && (num > 1000 || num < -1000)){
//                        str = str.replace(/万/,'亿');
//                        num =num/10000;
//                        bit=2;
//                    }else if(/万/.test(str) && (num <1 && num>-1)){
//                        str = str.replace(/万/,'');
//                        num =num*10000;
//                        bit =0;  //保留整数
//                    }
          return '<span class="' + c1 + '">' + num.toFixed(bit || 0) + '</span>' + '<span class="' + c2 + '"> ' + str + '</span>'
        }
      },
      saveTwoNumber (num, bit = 2, str = '') {
        if (isNaN(parseInt(num))) {
          return '-' // 原形返回
        } else {
          num = num - 0 //字符串转数字类型
          //                     亿以上主动转行单位
//                    if(/万/.test(str) && (num > 1000 || num < -1000)){
//                        str = str.replace(/万/,'亿');
//                        num =num/10000;
//                        bit=2;
//                    }else if(/万/.test(str) && (num <1 &&num >-1)){
//                        str = str.replace(/万/,'');
//                        num =num*10000;
//                        bit =0;  //保留整数
//                    }
          return num.toFixed(bit || 0) + str
        }
      },

      /*
      * 解析树形结构为列表结构
      * */
      initTableData(data,parent){
          if(data && data.length){
               let cot =[]; //初始化容器
               let _this = this;//保留this
              /*
               * 获取头部指定 子集合参数
               * */
              let childName = this.tableHead[0].childName || '';

                let  fn =function (allItem,parent,item) {
                    /*
                    * 自动附上所需参数
                    * */
                  let c = {    //自定义组件需要参数
                    showMore: false,
                    hasMore: false,
                    isLong: false,
                    isShow: false,
                    index: '',
                    lv: '', // 第几级数据级数据
                    hasGetData: false,
                    id: ''
                  };
                  for(let k in c){
//                    item[k] = c[k];
                    _this.$set(item,k,c[k]);
                  }
                  if(parent){
                    parent.hasMore=true;
                    _this.createArea(allItem,parent,item);
                    parent[childName] =[];
                  }else{
                    _this.createArea(allItem,'',item);
                  }

                  /*
                  * 保存lv 层级
                  * */
                  item.lv =item.id.split('-').length;
                  if(item.lv ==1){
                      item.isShow =true;
                  }
                  /*
                  * 递归寻找子节点
                  * */
                  if(childName && item[childName] && item[childName].length ){
                    for(let i = 0; i<item[childName].length;i++){
                      fn(allItem,item,item[childName][i]);
                    }
                  }
                };
                /*
                * 第一层数组排序
                * */
              for(let i=0;i<data.length;i++){
                   fn(cot,'',data[i]);
              }
              for(let i =0 ; i<cot.length ; i++ ){
                  cot[i].index = i;
               }
              this.lvTableData =cot;
          }else{
              console.warn('treeTable-warn：','请指定需要初始化data');
              console.warn('treeTable-warn：','please Providing effective data');
          }

      },
      createArea: function (allItem, parentItem,item) {

          /*
          * allItem :全部数组
          * parentItem ：作为父级对象
          * */

        if (parentItem != '') {
          this.getChildId(allItem, parentItem, item);
//          this.showChild(parentItem, allItem);
        } else {
          this.getChildId(allItem, '', item);
        }

      },
      getChildId: function (allItem, parentItem, item) {
        if (allItem.length) {
          if (parentItem) {
            this.getArrIndex(parentItem, allItem, item)
          } else {
            var c = allItem[allItem.length - 1].id.split('-')
            item.id = parseInt(c[0]) + 1 + ''
            allItem.push(item)
          }
        } else {
          item.id = '0'
          allItem.push(item)
        }
      },
      getArrIndex: function (parentItem, allItem, item) {
        var p = parentItem.id.split('-'), a;
        for (var i = 0; i < allItem.length; i++) {
//                搜索所有子内容
          if (allItem[i].id.indexOf(parentItem.id) == 0) {
            a = allItem[i].id.split('-');
//                    同一辈
//                    debugger;
            if (a.length == p.length) {
//                          没有下一项，且下一项不是子内容
              if ((i + 1 < allItem.length) && (allItem[i + 1].id.indexOf(parentItem.id) == 0 )) {
              } else {
                //如果下一级<=父数量
                if (a.length <= p.length) {
                  item.id = parentItem.id + '-0';
                  allItem.splice(i + 1, 0, item);
                  return;
                } else {
//                                  debugger;
                  a[a.length - 1] = parseInt(a[a.length - 1]) + 1 + '';
                  item.id = a.join('-');
                  allItem.splice(i + 1, 0, item);
                  return;
                }
              }
            } else {
//                          判断下一辈长度
//                          debugger;
              var f;
//                                保存子最后一个id
              if (a.length > p.length && a.length - 1 == p.length) {
                f = a;
              }
              if ((i + 1 < allItem.length) && (allItem[i + 1].id.indexOf(parentItem.id) == 0 )) {

              } else {
                //如果下一级<=父数量
                if (a.length <= p.length) {
                  item.id = parentItem.id + '-0'
                  allItem.splice(i + 1, 0, item);
                  return;
                } else {
                  if (f) {
                    f[f.length - 1] = parseInt(f[f.length - 1]) + 1 + '';
                    item.id = f.join('-');
                    allItem.splice(i + 1, 0, item);
                    f = null;
                    return;
                  }
                  a[a.length - 1] = parseInt(a[a.length - 1]) + 1 + '';
                  item.id = a.join('-');
                  allItem.splice(i + 1, 0, item);
                  return;
                }
              }
//
            }
          }
        }
      },
      showChild: function (parentItem, allItem) {
        if (parentItem.showMore) {
//                     隐藏所有子类
          parentItem.showMore = false;
          var p = parentItem.id.split('-'), a, c;
          for (var i = 0; i < allItem.length; i++) {
            //                搜索所有子内容
            if (allItem[i].id.indexOf(parentItem.id) == 0) {
              a = allItem[i].id.split('-');
              //                    同一辈
              if (a.length > p.length) {
                allItem[i].isShow = parentItem.showMore;

              }
            }
          }
        } else {
          parentItem.showMore = true;
          this.showNextChild(parentItem, allItem);
          for (var i = parentItem.index; i < allItem.length; i++) {
            if (parentItem === allItem[i]) continue;
            if (allItem[i].id.indexOf(parentItem.id) == 0) {
              if (allItem[i].hasMore) {
                this.showNextChild(allItem[i], allItem);
              }
            } else {
              return;
            }
          }
        }

      },
      showNextChild: function (parentItem, allItem) {
        for (var i = parentItem.index; i < allItem.length; i++) {
          if (parentItem === allItem[i]) continue;
          if (allItem[i].id.indexOf(parentItem.id) == 0) {
            allItem[i].isShow = parentItem.showMore;
          } else {
            return;
          }
        }
      },
    }
  }
</script>

<style scoped >

  tr[tablelv ='1']{
    background-color: #FFF  !important;
  }
  tr[tablelv ='2']{
    background-color: #e6e6e6 !important;
  }
  tr[tablelv = '3']{
    background-color: #f8eaea !important;
  }

  thead > tr{
    background-color: #F1F8FF;
  }
  table{
    width: 100%;
    text-align: center;
    font-size: 12px;
    border-collapse: collapse !important;
  }
  table tr{
    height: 35px;
    vertical-align: middle;
  }
  table >thead>tr>th{
    word-break:keep-all;
    white-space:initial;
    line-height: normal;
    font-weight: bold;
  }
  table >thead>tr>th span{
    display: inline-block;
    padding: 0 7px;
    word-break:keep-all;
    white-space:nowrap;

  }
  table>tbody>tr>td:first-child{
    font-weight: bold;
  }

  table tr>td:first-child{
    padding-right: 10px;
  }

  .tableTree-cot {
    width: 100%;
    overflow-x: scroll;
  }
  .tableTree-cot table{
    width: 100%;

  }

  .arrowCom:after{
    position: absolute;
    right: 0;
    top:-1px;
    width: 0;
    height: 5px;
    content: " ";
    border-left: 2px solid transparent ;
    border-right: 2px solid transparent ;
    border-bottom: 3px solid  #b1b1b1;
  }

  .arrowCom:before{
    position: absolute;
    right: 0;
    bottom:-2px;
    width: 0;
    height: 5px;
    content: " ";
    border-left: 2px solid transparent ;
    border-right: 2px solid transparent ;
    border-top: 3px solid    #b1b1b1;
  }
  .arrowCom{
    position: relative;
  }



  .lvFirstTableTr >td:first-child{
     position: relative;
   }
  .lvFirstTableTr >td:first-child:after{
     position: absolute;
     right: 2px;
     top: 14px;
     width: 6px;
     height: 6px;
     content: "";
     border-right: 1px solid  #666666 ;
     border-bottom: 1px solid  #666666;
     -webkit-transition: all .25s;
     -moz-transition: all .25s;
     -ms-transition: all .25s;
     -o-transition: all .25s;
     transition: all .25s;
     -webkit-transform: rotate(-45deg);
     -moz-transform: rotate(-45deg);
     -ms-transform: rotate(-45deg);
     -o-transform: rotate(-45deg);
     transform: rotate(-45deg);
   }
  .active>td:first-child:after{
    top: 12px;
    -webkit-transform: rotate(90deg);
    -moz-transform: rotate(90deg);
    -ms-transform: rotate(90deg);
    -o-transform: rotate(90deg);
    transform: rotate(90deg);
  }





  .lvTableTr  >td:first-child{
     position: relative;
     padding-left: 20px;
   }
  .lvTableTr >td:first-child:after{
     position: absolute;
     right: -2px;
     top: 14px;
     width: 0;
     height: 0;
     content: "";
     border-width: 4px;
     border-style: solid;
     border-color:   transparent transparent  transparent #000 ;
   }

</style>
