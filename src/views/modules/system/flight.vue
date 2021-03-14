<template>
  <div>
    <el-input
      v-model="searchContent"
      style="width: 220px"
      placeholder="输入航班编号搜索"
    ></el-input>
    <el-button type="success" @click="getPageData(1)">查询</el-button>
    <el-button type="primary" @click="visible = true">添加航班</el-button>
    <p></p>
    <el-table
      :data="flights"
      :header-cell-style="{ 'text-align': 'center' }"
      :cell-style="{ 'text-align': 'center' }"
      v-loading="dataListLoading"
      stripe
      border
      style="width: 100%"
    >
      <el-table-column
        prop="fName"
        width="90"
        label="航班编号"
      ></el-table-column>
      <el-table-column prop="from" label="出发地" width="80"></el-table-column>
      <el-table-column
        prop="destination"
        label="目的地"
        width="80"
      ></el-table-column>
      <el-table-column prop="startTime" label="出发时间"></el-table-column>
      <el-table-column prop="endTime" label="预计抵达时间"></el-table-column>
      <el-table-column prop="fEco" label="经济舱" width="180"></el-table-column>
      <el-table-column prop="fBus" label="商务舱" width="180"></el-table-column>
      <el-table-column prop="fFc" label="头等舱" width="180"></el-table-column>
      <el-table-column label="操作">
        <template slot-scope="scope">
          <el-row>
            <el-button size="small" type="primary" @click="toUpdate(scope.row)"
              >修改</el-button
            >
          </el-row>
          <el-row>
            <el-popconfirm
              title="确定删除吗？"
              @confirm="deleteFlight(scope.row.fId)"
            >
              <el-button slot="reference" size="small" type="danger"
                >删除</el-button
              >
            </el-popconfirm>
          </el-row>
        </template>
      </el-table-column>
    </el-table>

    <el-pagination
      background
      :current-page="curP"
      @current-change="getPageData"
      layout="prev, pager, next"
      :total="dataCount"
    >
    </el-pagination>

    <el-dialog
      :title="formMode == 'add' ? '新增航班' : '修改航班'"
      :close-on-click-modal="false"
      :visible.sync="visible"
      @close="clearForm()"
    >
      <el-form
        :model="newFlight"
        ref="newFlight"
        :rules="flightRules"
        label-width="120px"
      >
        <el-form-item label="航班名字" prop="fName">
          <el-input v-model="newFlight.fName" placeholder="航班名字"></el-input>
        </el-form-item>
        <el-form-item label="航班飞机" prop="pId">
          <el-select v-model="newFlight.pId" placeholder="请选择航班飞机">
            <el-option
              v-for="item in planes"
              :key="item.pName"
              :label="item.pName"
              :value="item.pId"
            >
            </el-option>
          </el-select>
        </el-form-item>
        <el-form-item label="出发地" prop="from">
          <el-input v-model="newFlight.from" placeholder="出发地"></el-input>
        </el-form-item>
        <el-form-item label="目的地" prop="destination">
          <el-input
            v-model="newFlight.destination"
            placeholder="目的地"
          ></el-input>
        </el-form-item>
        <el-form-item label="出发时间" prop="startTime">
          <el-date-picker
            v-model="newFlight.startTime"
            type="datetime"
            placeholder="选择出发时间"
          >
          </el-date-picker>
        </el-form-item>
        <el-form-item label="预计抵达时间" prop="endTime">
          <el-date-picker
            v-model="newFlight.endTime"
            type="datetime"
            placeholder="选择预计抵达时间"
          >
          </el-date-picker>
        </el-form-item>
        <el-row>
          <el-col :span="12">
            <el-form-item label="经济舱座位数量" prop="fEco">
              <el-input
                v-model.number="newFlight.fEco"
                placeholder="默认飞机的经济舱座位数量"
              ></el-input>
            </el-form-item>
          </el-col>
          <el-col :span="12">
            <el-form-item label="经济舱价格" prop="ecoPrice">
              <el-input
                v-model.number="newFlight.ecoPrice"
                placeholder="经济舱价格(单位:￥)"
              ></el-input>
            </el-form-item>
          </el-col>
        </el-row>
        <el-row>
          <el-col :span="12">
            <el-form-item label="商务舱座位数量" prop="fBus">
              <el-input
                v-model.number="newFlight.fBus"
                placeholder="默认飞机的商务舱座位数量"
              ></el-input>
            </el-form-item>
          </el-col>
          <el-col :span="12">
            <el-form-item label="商务舱价格" prop="busPrice">
              <el-input
                v-model.number="newFlight.busPrice"
                placeholder="商务舱价格(单位:￥)"
              ></el-input>
            </el-form-item>
          </el-col>
        </el-row>
        <el-row>
          <el-col :span="12">
            <el-form-item label="头等舱座位数量" prop="fFc">
              <el-input
                v-model.number="newFlight.fFc"
                placeholder="默认飞机的头等舱座位数量"
              ></el-input>
            </el-form-item>
          </el-col>
          <el-col :span="12">
            <el-form-item label="头等舱价格" prop="fcPrice">
              <el-input
                v-model.number="newFlight.fcPrice"
                placeholder="头等舱价格(单位:￥)"
              ></el-input>
            </el-form-item>
          </el-col>
        </el-row>
        <el-row>
          <div style="margin-left: 75%">
            <el-button type="success" @click="submitForFlight('newFlight')"
              >提交</el-button
            >
            <el-button @click="resetForm('newFlight')">重置</el-button>
          </div>
        </el-row>
      </el-form>
    </el-dialog>
  </div>
</template>
<script>
export default {
  name: "flightMng",
  data() {
    return {
      flights: [],
      searchContent: null,
      curP: 1,
      dataCount: 0,
      dataListLoading: false,
      formMode: "add",
      visible: false,
      newFlight: {
        fId: null,
        fName: null,
        pId: null,
        fEco: null,
        ecoPrice: null,
        fBus: null,
        busPrice: null,
        fFc: null,
        fcPrice: null,
        startTime: null,
        endTime: null,
        from: null,
        destination: null,
      },
      flightRules: {
        fName: [
          { required: true, message: "航班名字不能为空", trigger: "blur" },
        ],
        pId: [{ required: true, message: "请选择飞机", trigger: "change" }],
        fEco: [{ type: "number", message: "需填写数字", trigger: "blur" }],
        ecoPrice: [
          { required: true, message: "请填写经济舱机票价格", trigger: "blur" },
          { type: "number", message: "需填写数字", trigger: "blur" },
        ],
        fBus: [{ type: "number", message: "需填写数字", trigger: "blur" }],
        busPrice: [
          { required: true, message: "请填写商务舱机票价格", trigger: "blur" },
          { type: "number", message: "需填写数字", trigger: "blur" },
        ],
        fFc: [{ type: "number", message: "需填写数字", trigger: "blur" }],
        fcPrice: [
          { required: true, message: "请填写头等舱机票价格", trigger: "blur" },
          { type: "number", message: "需填写数字", trigger: "blur" },
        ],
        startTime: [
          { required: true, message: "请选择出发时间", trigger: "blur" },
        ],
        endTime: [
          { required: true, message: "请选择预计抵达时间", trigger: "blur" },
        ],
        from: [{ required: true, message: "请输入出发地", trigger: "blur" }],
        destination: [
          { required: true, message: "请输入目的地", trigger: "blur" },
        ],
      },
      fIds: [],
      planes: [],
    };
  },
  mounted: function () {
    this.initFlight();
    this.initPlane();
  },
  methods: {
    alertHelper: function (msg, mType) {
      this.$message({
        type: mType,
        message: msg,
      });
    },
    initFlight: function () {
      this.dataListLoading = true;
      var that = this;
      this.$http
        .get("https://localhost:8088/admin/getAllFlight")
        .then(function (result) {
          if (result.data.code == 200) {
            that.fIds = [];
            that.flights = [];
            for (let i = 0; i < Math.min(10, result.data.data.length); i++) {
              that.flights.push(result.data.data[i]);
              that.flights[i].startTime = that.timeFormate(
                that.flights[i].startTime
              );
              that.flights[i].endTime = that.timeFormate(
                that.flights[i].endTime
              );
              that.fIds.push(that.flights[i].fId);
            }
            that.dataCount = result.data.data.length;
            that.initTicket();
          } else {
            that.alertHelper(result.data.msg, "error");
          }
        })
        .catch(function (error) {
          that.alertHelper("系统内部异常，请稍后重试", "error");
        });
      this.dataListLoading = false;
    },
    initTicket: function () {
      var that = this;
      this.$http
        .post("https://localhost:8088/getByFidList", that.fIds)
        .then(function (result) {
          if (result.data.code == 200) {
            for (var i = 0; i < result.data.data.length; i = i + 3) {
              that.flights[i / 3].fEco =
                "票余量：" +
                that.flights[i / 3].fEco +
                "，价格：" +
                result.data.data[i].tPrice;
              that.flights[i / 3].fBus =
                "票余量：" +
                that.flights[i / 3].fBus +
                "，价格：" +
                result.data.data[i + 1].tPrice;
              that.flights[i / 3].fFc =
                "票余量：" +
                that.flights[i / 3].fFc +
                "，价格：" +
                result.data.data[i + 2].tPrice;
            }
          } else {
            that.alertHelper(result.data.msg, "error");
          }
        })
        .catch(function (error) {
          that.alertHelper("系统内部异常，请稍后重试", "error");
        });
    },

    initPlane: function () {
      var that = this;
      this.$http
        .get("https://localhost:8088/admin/getAllPlane")
        .then(function (result) {
          if (result.data.code == 200) {
            that.planes = result.data.data;
          } else {
            that.alertHelper(result.data.msg, "error");
          }
        })
        .catch(function (error) {
          that.alertHelper("系统内部异常", "error");
        });
    },

    submitForFlight: function (formName) {
      var that = this;
      this.$refs[formName].validate((valid) => {
        if (valid) {
          if (that.formMode == "update") {
            that.updateFlight();
            that.formMode = "add";
            return;
          }
          that.newFlight.startTime = that.timeFormate(that.newFlight.startTime);
          that.newFlight.endTime = that.timeFormate(that.newFlight.endTime);
          this.$http
            .post("https://localhost:8088/admin/addFlight", that.newFlight)
            .then(function (result) {
              if (result.data.code != 200) {
                that.alertHelper(result.data.msg, "error");
              } else {
                that.submitForTicket(result.data.data);
              }
            })
            .catch(function (error) {
              that.alertHelper("系统内部异常，请稍后重试", "error");
            });
        } else {
          that.alertHelper("请规范填写！", "warning");
          return false;
        }
      });
    },

    submitForTicket: function (fId) {
      var that = this;
      console.log(fId);
      var tickets = [
        { fId: fId, rank: 1, tPrice: this.newFlight.ecoPrice },
        { fId: fId, rank: 2, tPrice: this.newFlight.busPrice },
        { fId: fId, rank: 3, tPrice: this.newFlight.fcPrice },
      ];
      this.$http
        .post("https://localhost:8088/admin/addTicket", tickets)
        .then(function (result) {
          if (result.data.code != 200) {
            that.alertHelper(result.data.msg, "error");
          } else {
            that.visible = false;
            that.alertHelper("添加航班成功", "success");
            that.initFlight();
          }
        })
        .catch(function (error) {
          that.alertHelper("系统内部异常，请稍后重试" + error, "error");
        });
    },

    updateFlight: function () {
      var that = this;
      this.newFlight.startTime = this.timeFormate(this.newFlight.startTime);
      this.newFlight.endTime = this.timeFormate(this.newFlight.endTime);
      this.$http
        .put("https://localhost:8088/admin/updateFlight", that.newFlight)
        .then(function (result) {
          if (result.data.code != 200) {
            that.alertHelper(result.data.msg, "error");
          } else {
            that.checkTicket(that.newFlight.fId);
          }
        })
        .catch(function (error) {
          that.alertHelper("系统内部异常，请稍后重试", "error");
        });
    },

    checkTicket: function (fId) {
      var that = this;
      this.$http
        .get("https://localhost:8088/getTicketsByFid?fid=" + fId)
        .then(function (result) {
          if (result.data.code != 200) {
            that.alertHelper(result.data.msg, "error");
          } else {
            that.updateTicket(result.data.data);
          }
        })
        .catch(function (error) {
          that.alertHelper("系统内部异常，请稍后重试", "error");
        });
    },

    updateTicket: function (tickets) {
      var that = this;
      for (var i = 0; i < tickets.length; i++) {
        if (tickets[i].rank == 1) {
          tickets[i].tPrice = this.newFlight.ecoPrice;
        } else if (tickets[i].rank == 2) {
          tickets[i].tPrice = this.newFlight.busPrice;
        } else if (tickets[i].rank == 3) {
          tickets[i].tPrice = this.newFlight.fcPrice;
        }
      }
      this.$http
        .put("https://localhost:8088/admin/updateTicket", tickets)
        .then(function (result) {
          that.alertHelper("操作成功", "success");
          that.initFlight();
          that.visible = false;
        })
        .catch(function (error) {
          that.alertHelper("系统内部异常，请稍后重试", "error");
        });
    },

    resetForm(formName) {
      this.$refs[formName].resetFields();
    },

    toUpdate: function (flight) {
      this.formMode = "update";
      this.visible = true;
      this.newFlight.fId = flight.fId;
      this.newFlight.fName = flight.fName;
      this.newFlight.pId = flight.pId;
      this.newFlight.startTime = flight.startTime;
      this.newFlight.endTime = flight.endTime;
      this.newFlight.from = flight.from;
      this.newFlight.destination = flight.destination;
      this.newFlight.fEco = Number(
        flight.fEco.substring(
          flight.fEco.indexOf("：") + 1,
          flight.fEco.indexOf("，")
        )
      );
      this.newFlight.fBus = Number(
        flight.fBus.substring(
          flight.fBus.indexOf("：") + 1,
          flight.fBus.indexOf("，")
        )
      );
      this.newFlight.fFc = Number(
        flight.fFc.substring(
          flight.fFc.indexOf("：") + 1,
          flight.fFc.indexOf("，")
        )
      );
      this.newFlight.ecoPrice = Number(
        flight.fEco.substring(flight.fEco.indexOf("价格") + 3)
      );
      this.newFlight.busPrice = Number(
        flight.fBus.substring(flight.fBus.indexOf("价格") + 3)
      );
      this.newFlight.fcPrice = Number(
        flight.fFc.substring(flight.fFc.indexOf("价格") + 3)
      );
    },

    deleteFlight(fId) {
      var that = this;
      this.$http
        .delete("https://localhost:8088/admin/delFlight?fId=" + fId)
        .then(function (result) {
          if (result.data.code != 200) {
            that.alertHelper(result.data.msg, "error");
          } else {
            that.alertHelper("删除成功", "success");
            that.initFlight();
          }
        })
        .catch(function (error) {
          that.alertHelper("系统内部异常，请稍后重试");
        });
    },

    clearForm: function () {
      this.formMode = "add";
      this.newFlight.fName = null;
      this.newFlight.fId = null;
      this.newFlight.fName = null;
      this.newFlight.pId = null;
      this.newFlight.fEco = null;
      this.newFlight.ecoPrice = null;
      this.newFlight.fBus = null;
      this.newFlight.busPrice = null;
      this.newFlight.fFc = null;
      this.newFlight.fcPrice = null;
      this.newFlight.startTime = null;
      this.newFlight.endTime = null;
      this.newFlight.from = null;
      this.newFlight.destination = null;
    },

    getPageData: function (val) {
      var that = this;
      that.dataListLoading = true;
      that.curP = val;
      var url = "https://localhost:8088/admin/getForPage";
      if (this.searchContent === null || this.searchContent.trim() == "") {
        url = "https://localhost:8088/admin/getForPage?pageNo=" + val;
      }
      else{
        url = "https://localhost:8088/admin/getForPage?pageNo=" + val + "&fName=" + that.searchContent;
      }
      this.$http
        .get(url)
        .then(function (result) {
          if (result.data.code != 200) {
            that.alertHelper(result.data.msg, "error");
          } else {
            that.fIds = [];
            that.flights = [];
            for (let i = 0; i < result.data.data.length; i++) {
              that.flights.push(result.data.data[i]);
              that.flights[i].startTime = that.timeFormate(
                that.flights[i].startTime
              );
              that.flights[i].endTime = that.timeFormate(
                that.flights[i].endTime
              );
              that.fIds.push(that.flights[i].fId);
            }
            that.initTicket();
          }
        })
        .catch(function (error) {
          that.alertHelper("系统内部异常，请稍后重试");
        });
        that.dataListLoading = false;
    },

    timeFormate(data) {
      var date = new Date(data);
      var Y = date.getFullYear() + "-";
      var M =
        (date.getMonth() + 1 < 10
          ? "0" + (date.getMonth() + 1)
          : date.getMonth() + 1) + "-";
      var D =
        (date.getDate() < 10 ? "0" + date.getDate() : date.getDate()) + " ";
      var h =
        (date.getHours() < 10 ? "0" + date.getHours() : date.getHours()) + ":";
      var m =
        date.getMinutes() < 10 ? "0" + date.getMinutes() : date.getMinutes();
      return Y + M + D + h + m;
    },
  },
};
</script>