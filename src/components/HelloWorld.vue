<template>
  <div class="hello">

    <tableTree  ref="tableTree"   :tableHead="tableHead" :tableData="tableData">

    </tableTree>
  </div>
</template>

<script>
  import tableTree from './common/tableTree.vue'
export default {
  name: 'HelloWorld',
  data () {
    return {
        tableHead:[
          {
            name: '产品',//表头名字
            canSort: false, //能否升序降序
            sortName: 'productName', //升序降序按照那个名字为标准
            bit: '', //保留多少位数字
            symbol: '', //单位
            ascending:'',//升序降序  true 升序 false 降序
            childName:'child',
          },
          {
            name: '数量',//表头名字
            canSort: false, //能否升序降序
            sortName: 'number', //升序降序按照那个名字为标准
            bit: '0', //保留多少位数字
            symbol: '万台', //单位
            ascending:'',//升序降序  true 升序 false 降序
          },
          {
            name: '金额',//表头名字
            canSort: false, //能否升序降序
            sortName: 'money', //升序降序按照那个名字为标准
            bit: '2', //保留多少位数字
            symbol: '万元', //单位
            ascending:'',//升序降序  true 升序 false 降序
          }
        ],
      tableData:[
        {
            productName:'1',
            number:'10000',
            money:'11111.662',
            child:[
              {
                productName:'2',
                number:'10000',
                money:'11111.662',
                child:[
                  {
                    productName:'3',
                    number:'10000',
                    money:'11111.662',
                  }
                ]
              },
              {
                productName:'5',
                number:'10000',
                money:'11111.662',
              }
            ]
        },
        {
            productName:'6',
            number:'666',
            money:'1111.777',
        },
        {
            productName:'8',
            number:'7991',
            money:'8888.62',
        },
      ]
    }
  },
  mounted(){
      this.$refs.tableTree.initTableData(this.tableData)

  },
  components:{
      tableTree,
  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>

</style>
