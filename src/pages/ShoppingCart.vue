<template>

  <el-main>
    <div class="w-full grid grid-cols-9">
      <div></div>

      <el-table
          :data="tableData"
          height="700"
          show-summary
          :summary-method="getSummaries"
          stripe="true"
          style="width: 100%" class="col-span-7 mt-16 mb-8 rounded-lg border border-grey-200">


        <el-table-column
            prop="image"
            label="商品图片" width="275" align="center">

          <template #default="scope">
          <span style="margin-left: 10px" class="text-xl font-semibold ">
            <center><img
                class="w-44 h-44 rounded-b-none rounded-xl"
                :src="scope.row.image ? scope.row.image : 'https://i.loli.net/2021/05/18/vWptQgAlsTqdxrK.png'"
            ></center>
          </span>
          </template>
        </el-table-column>

        <el-table-column
            prop="commodity"
            label="商品名" width="225" align="center">
          <template #default="scope">
          <span style="margin-left: 10px" class="text-xl font-semibold ">
            {{ scope.row.commodity }}
          </span>
          </template>
        </el-table-column>

        <el-table-column
            prop="price"
            label="单价" width="150" class="col-span-1" align="center">
        </el-table-column>

        <el-table-column
            prop="count"
            label="购物车中数量" width="150" class="col-span-1" align="center">
        </el-table-column>

        <el-table-column
            fixed="right"
            label="操作"
            class="col-span-3" align="center">
          <template #default="scope">
            <el-button
                @click.prevent="deleteRow(scope.$index, tableData)"
                type="primary" class="mr-8">
              移除
            </el-button>

            <el-button type="primary" class="ml-8"
                       @click.prevent="buyCommodity(scope.$index, tableData)">
              结算
            </el-button>


          </template>
        </el-table-column>

      </el-table>

    </div>
  </el-main>
</template>
<script>
import {useStore} from "vuex";
import {api} from "@/request";
import {ElMessage} from "element-plus";

export default {
  name: "ShoppingCart",
  props: {
    title: String,
    img: String,
  },
  data() {
    return {
      tableData: [],
    };
  },
  // mounted() {
  //   this.$nextTick(() => {
  //     let user = this.store.getters['user/userInfo']
  //     let userId = user.id
  //     console.log("shoppingCart/" + userId.replaceAll('"', ''))
  //     const that = this
  //     api({
  //       url: "shoppingCart/" + userId.replaceAll('"', ''),
  //       method: "get",
  //     })
  //         .then(function (response) {
  //           console.log(JSON.stringify(response.data));
  //           that.tableData = []
  //           for (let i = 0; i < response.data['ItemList'].length; i++) {
  //             let temp = {}
  //             console.log('count', response.data['ItemList'][i]['Count'])
  //             temp['count'] = Number(response.data['ItemList'][i]['Count'])
  //             temp['price'] = Number(response.data['PriceList'][i])
  //             temp['commodity'] = response.data['NameList'][i]
  //             temp['id'] = response.data['ItemList'][i]['CommodityId']
  //             temp['image'] = 'https://139.196.20.137:5001/' + response.data['ImageList'][i]
  //             that.tableData.push(temp)
  //           }
  //         })
  //         .catch(function (error) {
  //           console.log(error);
  //           ElMessage.error({
  //             message: '服务器在开小差...',
  //             type: 'error'
  //           })
  //         });
  //   })
  // },
  methods: {
    deleteRow(index, rows) {
      let commodityId = rows[index]['id']
      let user = this.store.getters['user/userInfo']
      let userId = user.id

      var FormData = require('form-data');
      var data = new FormData();
      data.append('commodityId', commodityId);
      data.append('userId', userId);

      api({
        url: "shoppingCart",
        method: "delete",
        data: data
      })
          .then(function (response) {
            console.log(JSON.stringify(response.data));
            if (String(response.data['Code']) === '200') {
              rows.splice(index, 1);
              ElMessage.success({
                message: '删除成功',
                type: 'success'
              });
            }
          })
          .catch(function (error) {
            console.log(error);
            ElMessage.error({
              message: '服务器在开小差...',
              type: 'error'
            })
          });
    },
    buyCommodity(index, rows) {
      console.log(rows[index]);
      this.$router.push({
        path: '/CommodityDetail/' + rows[index]['id'],
      })
    },
    getSummaries(param) {
      const {columns, data} = param;
      const sums = [];
      columns.forEach((column, index) => {
        if (index === 0) {
          sums[index] = '总价';
          return;
        }
        if (index === 3) {
          sums[index] = 0
          for (let i = 0; i < this.tableData.length; i++) {
            sums[index] += Number(data[i]["price"]) * Number(data[i]["count"])
          }
          sums[index] += ' 元';
        }
      });
      return sums;
    },
  },

  mounted() {
    let id = this.store.getters['user/userInfo'].id
    api({
      method: "get",
      url: "ShoppingCart/"+id,
    }).then( response => {
      if (response.data.Code === '200') {
        for (let i = 0; i < response.data['ItemList'].length; i++) {
          let temp = {}
          console.log('count', response.data['ItemList'][i]['Count'])
          temp['count'] = Number(response.data['ItemList'][i]['Count'])
          temp['price'] = Number(response.data['PriceList'][i])
          temp['commodity'] = response.data['NameList'][i]
          temp['id'] = response.data['ItemList'][i]['CommodityId']
          temp['image'] = 'https://139.196.20.137:5001/' + response.data['ImageList'][i]
          this.tableData.push(temp)
        }
      } else {
        ElMessage.error("服务器在开小差...")
      }
    })
  },

  setup() {
    let store = useStore()
    return {
      store
    }
  }
}
</script>


<style scoped>
@import url("//unpkg.com/element-plus/lib/theme-chalk/index.css");
.el-main {
  background: -webkit-linear-gradient(left top, #2f90b9 2.5%, #1781b5 7.5%, #93b5cf 20%, #d8e3e7 50%, #93b5cf 70%, #1781b5 92.5%, #2f90b9 97.5%); /* Safari 5.1 - 6.0 */
  background: -o-linear-gradient(bottom right, #000, #fff); /* Opera 11.1 - 12.0 */
  background: -moz-linear-gradient(bottom right, #000, #fff); /* Firefox 3.6 - 15 */
  background-color: #93b5cf;
}

</style>