<template>
  <div>
    <el-input
      v-model="searchContent"
      style="width: 220px"
      placeholder="输入飞机名字搜索"
    ></el-input>
    <el-button type="success" @click="getPageData(1)">查询</el-button>
    <el-button type="primary" @click="visible = true">添加飞机</el-button>
    <p></p>
    <el-table
      :data="planes"
      :header-cell-style="{ 'text-align': 'center' }"
      :cell-style="{ 'text-align': 'center' }"
      v-loading="dataListLoading"
      stripe
      border
      style="width: 100%"
    >
      <el-table-column prop="pId" width="90" label="飞机ID"></el-table-column>
      <el-table-column
        prop="pName"
        label="飞机名字"
        width="180"
      ></el-table-column>
      <el-table-column
        prop="pEco"
        label="经济舱座位数"
      ></el-table-column>
      <el-table-column prop="pBus" label="商务舱座位数"></el-table-column>
      <el-table-column prop="pFc" label="头等舱座位数"></el-table-column>

      <el-table-column label="操作">
        <template slot-scope="scope">
          <el-button size="small" type="primary" @click="toUpdate(scope.row)"
            >修改</el-button
          >
          <el-popconfirm
            title="确定删除吗？"
            @confirm="deletePlane(scope.row.pId)"
          >
            <el-button slot="reference" size="small" type="danger"
              >删除</el-button
            >
          </el-popconfirm>
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
      :title="formMode == 'add' ? '新增飞机' : '修改飞机'"
      :close-on-click-modal="false"
      :visible.sync="visible"
      @close="clearForm()"
    >
      <el-form
        :model="newPlane"
        ref="newPlane"
        :rules="planeRules"
        label-width="120px"
      >
        <el-form-item label="飞机名字" prop="pName">
          <el-input v-model="newPlane.pName" placeholder="飞机名字"></el-input>
        </el-form-item>
        <el-form-item label="经济舱数量" prop="pEco">
          <el-input
            v-model.number="newPlane.pEco"
            placeholder="经济舱数量"
          ></el-input>
        </el-form-item>
        <el-form-item label="商务舱数量" prop="pBus">
          <el-input
            v-model.number="newPlane.pBus"
            placeholder="商务舱数量"
          ></el-input>
        </el-form-item>
        <el-form-item label="头等舱数量" prop="pFc">
          <el-input
            v-model.number="newPlane.pFc"
            placeholder="头等舱数量"
          ></el-input>
        </el-form-item>
        <el-row>
          <div style="margin-left: 75%">
            <el-button type="success" @click="submitForAdd('newPlane')"
              >提交</el-button
            >
            <el-button @click="resetForm('newPlane')">重置</el-button>
          </div>
        </el-row>
      </el-form>
    </el-dialog>
  </div>
</template>
<script>
export default {
  name: "planeMng",
  data() {
    return {
      planes: [],
      dataListLoading: false,
      searchContent: null,
      visible: false,
      formMode: "add",
      dataCount:0,
      curP:1,
      newPlane: {
        pId: null,
        pName: null,
        pEco: null,
        pBus: null,
        pFc: null,
      },
      planeRules: {
        pName: [
          { required: true, message: "飞机名字不能为空", trigger: "blur" },
        ],
        pEco: [
          { required: true, message: "请填写经济舱数量", trigger: "blur" },
          { type: "number", message: "经济舱数量应为数字", trigger: "blur" },
        ],
        pBus: [
          { required: true, message: "请填写商务舱数量", trigger: "blur" },
          { type: "number", message: "商务舱数量应为数字", trigger: "blur" },
        ],
        pFc: [
          { required: true, message: "请填写头等舱数量", trigger: "blur" },
          { type: "number", message: "头等舱数量应为数字", trigger: "blur" },
        ],
      },
    };
  },
  mounted: function () {
    this.initPlane();
  },
  methods: {
    alertHelper: function (msg, mType) {
      this.$message({
        type: mType,
        message: msg,
      });
    },

    initPlane: function () {
      var that = this;
      this.dataListLoading = true;
      this.$http
        .get("https://localhost:8088/admin/getAllPlane")
        .then(function (result) {
          if (result.data.code == 200) {
            // that.planes = result.data.data;
            that.planes = [];
            that.dataCount = result.data.data.length;
            for(let i=0; i<Math.min(10,result.data.data.length); i++){
                that.planes.push(result.data.data[i]);
            }
          } else {
            that.alertHelper(result.data.msg, "error");
          }
        })
        .catch(function (error) {
          that.alertHelper("系统内部异常", "error");
        });
      this.dataListLoading = false;
    },

    toUpdate: function (data) {
      this.formMode = "update";
      this.newPlane.pId = data.pId;
      this.newPlane.pName = data.pName;
      this.newPlane.pEco = data.pEco;
      this.newPlane.pBus = data.pBus;
      this.newPlane.pFc = data.pFc;
      this.visible = true;
    },

    submitForAdd: function (formName) {
      var that = this;
      this.$refs[formName].validate((valid) => {
        if (valid) {
          if (that.formMode == "update") {
            that.submitForUpdate();
            that.formMode = "add";
            return;
          }
          this.$http
            .post("https://localhost:8088/admin/addPlane", that.newPlane)
            .then(function (result) {
              if (result.data.code != 200) {
                that.alertHelper(result.data.msg, "error");
              } else {
                that.visible = false;
                that.initPlane();
                that.alertHelper("添加成功", "success");
              }
            })
            .catch(function (error) {
              that.alertHelper("系统内部异常，请稍后重试", "error");
            });
        } else {
          that.alertHelper("请规范填写！", "warning");
        }
      });
    },

    submitForUpdate: function () {
      var that = this;
      this.$http
        .put("https://localhost:8088/admin/updatePlane", that.newPlane)
        .then(function (result) {
          if (result.data.code != 200) {
            that.alertHelper(result.data.msg, "error");
          } else {
            that.alertHelper("更新成功","success");
            that.initPlane();
            that.visible = false;
          }
        })
        .catch(function (error) {
          that.alertHelper("系统内部异常，请稍后重试", "error");
        });
    },

    deletePlane: function (pid) {
      var that = this;
      this.$http
        .delete("https://localhost:8088/admin/delPlane?id=" + pid)
        .then(function (result) {
          if (result.data.code != 200) {
            that.alertHelper(result.data.msg, "error");
          } else {
            that.alertHelper("删除成功", "success");
            that.initPlane();
          }
        })
        .catch(function (error) {
          that.alertHelper("系统内部异常，请稍后重试");
        });
    },

    resetForm(formName) {
      this.$refs[formName].resetFields();
    },

    clearForm: function () {
      this.formMode = "add";
      this.newPlane.pId = null;
      this.newPlane.pName = null;
      this.newPlane.pEco = null;
      this.newPlane.pBus = null;
      this.newPlane.pFc = null;
    },

    getPageData: function (val) {
      var that = this;
      that.dataListLoading = true;
      that.curP = val;
      var url = "https://localhost:8088/admin/getPlaneForPage";
      if (this.searchContent === null || this.searchContent.trim() == "") {
        url = "https://localhost:8088/admin/getPlaneForPage?pageNo=" + val;
      }
      else{
        url = "https://localhost:8088/admin/getPlaneForPage?pageNo=" + val + "&pName=" + that.searchContent;
      }
      this.$http
        .get(url)
        .then(function (result) {
          if (result.data.code != 200) {
            that.alertHelper(result.data.msg, "error");
          } else {
            that.planes = [];
            for (let i = 0; i < result.data.data.length; i++) {
              that.planes.push(result.data.data[i]);
            }
          }
        })
        .catch(function (error) {
          that.alertHelper("系统内部异常，请稍后重试");
        });
        that.dataListLoading = false;
    },
  },
};
</script>
<style>
</style>