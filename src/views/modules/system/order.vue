<template>
  <div>
      <el-input v-model.number="searchContent" style="width: 220px" placeholder="输入用户ID搜索"></el-input>
      <el-button type="primary" round @click="initOrders(1)">查询</el-button>
      <p></p>
    <el-table :data="orders" stripe border style="width: 100%">
      <el-table-column type="expand">
        <template slot-scope="props">
          <el-form label-position="left" inline class="demo-table-expand">
            <h3>乘客信息</h3>
            <el-form-item v-for="item in props.row.itemList" :key="item.itemId">
              <span>真实姓名：{{ item.pRealname }}</span
              ><br />
              <span>身份证号：{{ item.pIdnum }}</span
              ><br />
              <span>机票单价：{{ item.itemPrice }}￥</span><br />
              <span v-if="item.seat == 'none'">暂未分配座位</span><br />
              <span v-if="item.seat != 'none'">飞机座位：{{ item.seat }}</span
              ><br />
              <div class="tableTitle">
                <span class="midText">我是分割线</span>
              </div>
            </el-form-item>
          </el-form>
        </template>
      </el-table-column>
      <el-table-column prop="oId" label="订单号"> </el-table-column>
      <el-table-column prop="uId" sortable label="用户ID"></el-table-column>
      <el-table-column prop="from" label="出发地"> </el-table-column>
      <el-table-column prop="destination" label="目的地"> </el-table-column>
      <el-table-column prop="startTime" width="155" sortable label="出发时间">
      </el-table-column>
      <el-table-column prop="endTime" width="155" sortable label="预计抵达时间">
      </el-table-column>
      <el-table-column prop="createTime" width="155" label="订单创建时间">
      </el-table-column>
      <el-table-column prop="payTime" width="155" sortable label="支付时间">
      </el-table-column>
      <el-table-column prop="totalPrice" sortable label="总金额">
      </el-table-column>
      <el-table-column label="操作">
        <template slot-scope="scope">
          <span v-if="scope.row.status == 0" style="color: green"
            >订单未支付</span
          >
          <span v-if="scope.row.status == 1" style="color: green"
            >订单已支付</span
          >
          <span v-if="scope.row.status == 2" style="color: red"
            >订单已过期</span
          >
          <span v-if="scope.row.status == 3" style="color: orange"
            >订单已取消</span
          >
        </template>
      </el-table-column>
    </el-table>
    <el-pagination
      background
      :current-page="curP"
      @current-change="initOrders"
      layout="prev, pager, next"
      :total="orderCount"
    >
    </el-pagination>
  </div>
</template>
<script>
export default {
  name: "order",
  data() {
    return {
      orders: [],
      searchContent:null,
      orderCount: 0,
      curP: 1,
    };
  },
  mounted: function () {
    this.getOrderCount();
    this.initOrders(1);
  },
  methods: {
    alertHelper: function (msg, mType) {
      this.$message({
        type: mType,
        message: msg,
      });
    },
    getOrderCount: function () {
      var that = this;
      this.$http
        .get("https://localhost:8088/admin/getAllOrderCount")
        .then(function (result) {
          if (result.data.code == 200) {
            that.orderCount = result.data.data;
          } else {
            that.alertHelper(result.data.msg, "error");
          }
        })
        .catch(function (error) {
          that.alertHelper("系统内部异常", "error");
        });
    },
    initOrders: function (val) {
      var that = this;
      var url = "https://localhost:8088/admin/getAllOrder?pageNo=" + val;
      if(this.searchContent!=null){
          url = url+"&uid="+this.searchContent;
      }
      this.$http
        .get(url)
        .then(function (result) {
          if (result.data.code == 200) {
            that.orders = result.data.data;
          } else {
            that.alertHelper(result.data.msg, "error");
          }
        })
        .catch(function (error) {
          that.alertHelper("系统内部异常", "error");
        });
    },
  },
};
</script>
<style>
.el-table th.gutter{
    display: table-cell!important;
 }
.tableTitle {
  position: relative;
  margin: 0 auto;
  width: 600px;
  height: 1px;
  background-color: #d4d4d4;
  text-align: center;
  font-size: 16px;
  color: rgba(101, 101, 101, 1);
}
.midText {
  position: absolute;
  left: 50%;
  background-color: #ffffff;
  padding: 0 15px;
  transform: translateX(-50%) translateY(-50%);
}
</style>