<template>
  <v-card>
    <!--&lt;!&ndash;弹出的对话框&ndash;&gt;-->
    <v-dialog max-width="500" v-model="show" persistent>
      <!--对话框的标题-->
      <v-card>
        <v-toolbar dense dark color="primary">
          <v-toolbar-title>
            {{isEdit?'修改':'增加'}}品牌
          </v-toolbar-title>
          <v-spacer/>
          <!--关闭窗口的按钮-->
          <v-btn icon @click="closeWindow">
            <v-icon>close</v-icon>
          </v-btn>
        </v-toolbar>
        <!--对话框的内容，表单-->
        <v-card-text class="px-5">
          <div>
            <MyBrand-Form @close="closeWindow" :oldBrand="oldBrand" :isEdit="isEdit"></MyBrand-Form>
          </div>
        </v-card-text>

      </v-card>
    </v-dialog>
    <v-card-title flat color="white">
      <v-btn color="primary" @click="addBrand">新增品牌</v-btn>
      <!--空间隔离组件-->
      <v-spacer/>
      <!--搜索框，与search属性关联-->
      <v-text-field label="输入关键字搜索" v-model="search" append-icon="search" hide-details/>
    </v-card-title>
    <v-data-table
      :headers="headers"
      :items="brands"
      :pagination.sync="pagination"
      :total-items="totalBrands"
      :loading="loading"
      class="elevation-1"
    >
      <template slot="items" slot-scope="props">
        <td class="text-xs-center">{{ props.item.id }}</td>
        <td class="text-xs-center">{{ props.item.name }}</td>
        <td class="text-xs-center"><img v-if="props.item.image" :src="props.item.image" width="130" height="40"/></td>
        <td class="text-xs-center">{{ props.item.letter }}</td>
        <td class="text-xs-center">
          <v-icon small class="mr-2" @click="editItem(props.item)">
            edit
          </v-icon>
          <v-icon small @click="deleteItem(props.item)">
            delete
          </v-icon>
        </td>
      </template>
    </v-data-table>
  </v-card>
</template>
<script>
  import MyBrandForm from './MyBrandForm'

  export default {
    name: "MyBrand",
    data() {
      return {
        totalBrands: 0, // 总条数
        brands: [], // 当前页品牌数据
        loading: true, // 是否在加载中
        search: "", // 查询关键字
        pagination: {}, // 分页信息
        headers: [ // 头信息
          {text: 'id', align: 'center', value: 'id'},
          {text: '名称', align: 'center', value: 'name', sortable: false},
          {text: 'LOGO', align: 'center', value: 'image', sortable: false},
          {text: '首字母', align: 'center', value: 'letter'},
          {text: '操作', align: 'center', value: 'id', sortable: false},
        ],
        show: false,
        oldBrand: {},
        isEdit: false,
      }
    },
    watch: {
      pagination: {
        deep: true, // 深度监视
        handler() {
          this.getDataFromServer();
        }
      },
      search() {
        this.pagination.page = 1;
        this.getDataFromServer();
      }
    },
    methods: {
      getDataFromServer() { // 从服务端加载数据的函数
        this.$http("/item/brand/page", {
          params: {
            page: this.pagination.page, // 当前页
            rows: this.pagination.rowsPerPage, // 每页条数
            sortBy: this.pagination.sortBy, // 排序字段
            desc: this.pagination.descending, // 是否降序
            key: this.search // 查询字段
          }
        }).then(resp => {
          this.totalBrands = resp.data.total; // 总条数
          this.brands = resp.data.items; // 品牌数据
          this.loading = false; // 加载完成
        })
      },
      addBrand() {
        this.isEdit = false;
        // 控制弹窗可见：
        this.show = true;
        // 把oldBrand变为null
        this.oldBrand = null;
      },
      closeWindow() {
        // 关闭窗口
        this.show = false;
        // 重新加载数据
        this.getDataFromServer();
      },
      editItem(oldBrand) {
        this.isEdit = true;
        this.$http.get("/item/category/bid/" + oldBrand.id).then(({data}) => {
          this.show = true;
          // 获取要编辑的brand
          this.oldBrand = oldBrand;
          // 回显商品分类
          this.oldBrand.categories = data;
        })
      },

      deleteItem(oldBrand) {
          this.$message.confirm('此操作将永久删除该品牌, 是否继续?').then(
            () => {
              //发起删除请求，删除单条数据
              this.$http.delete("/item/brand/delete/bid/" + oldBrand.id).then(({data}) => {
                this.getDataFromServer();
                this.$message.info("删除成功！");
              }).catch(()=>{
                this.$message.error("删除失败！");
              });
            }
          ).catch(() => {
            this.$message.info("删除已取消！");
          });
      }
    },
    // 渲染后执行
    mounted() {
      this.getDataFromServer() // 调用数据初始化函数
    },
    components: {
      MyBrandForm
    },
  }
</script>

<style scoped>

</style>
