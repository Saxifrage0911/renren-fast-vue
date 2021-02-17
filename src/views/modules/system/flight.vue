<template>
  <div>
    <el-button type="info" @click="visible = true">添加航班</el-button>
    <p></p>
    <el-table :data="flights" :header-cell-style="{ 'text-align': 'center' }"
              :cell-style="{ 'text-align': 'center' }" v-loading="dataListLoading" stripe border style="width: 100%;">
      <el-table-column
        prop="fName"
        width="90"
        label="航班编号"
      ></el-table-column>
      <el-table-column
        prop="from"
        label="出发地"
        width="80"
      ></el-table-column>
      <el-table-column
        prop="destination"
        label="目的地"
        width="80"
      ></el-table-column>
      <el-table-column prop="startTime" label="出发时间"></el-table-column>
      <el-table-column prop="endTime" label="预计抵达时间"></el-table-column>
      <el-table-column
        prop="fEco"
        label="经济舱"
        width="180"
      ></el-table-column>
      <el-table-column
        prop="fBus"
        label="商务舱"
        width="180"
      ></el-table-column>
      <el-table-column
        prop="fFc"
        label="头等舱"
        width="180"
      ></el-table-column>
      <el-table-column label="操作">
        <template slot-scope="scope">
            <el-row>
            <el-button size="small" type="primary">修改</el-button>
            </el-row>
            <el-row>
            <el-button size="small" type="danger">删除</el-button>
            </el-row>
        </template>
      </el-table-column>
    </el-table>
    <el-dialog :title="formMode == 'add' ? '新增航班' : '修改航班'"
    :close-on-click-modal="false"
    :visible.sync="visible">
    <el-form :model="newFlight" ref="newFlight" label-width="120px">
        <el-form-item label="航班名字" prop="fName">
            <el-input v-model="newFlight.fName" placeholder="航班名字"></el-input>
        </el-form-item>
        <el-form-item label="航班飞机" prop="pId">
            <el-select v-model="newFlight.pId" placeholder="请选择航班飞机">
                <el-option v-for="item in planes" :key="item.pName" :label="item.pName" :value="item.pId">
                </el-option>
            </el-select>
        </el-form-item>
        <el-form-item label="出发地" prop="from">
            <el-input v-model="newFlight.from" placeholder="出发地"></el-input>
        </el-form-item>
        <el-form-item label="目的地" prop="destination">
            <el-input v-model="newFlight.destination" placeholder="目的地"></el-input>
        </el-form-item>
        <el-form-item label="出发时间" prop="startTime">
            <el-date-picker v-model="newFlight.startTime" type="datetime" placeholder="选择出发时间">
            </el-date-picker>
        </el-form-item>
        <el-form-item label="预计抵达时间" prop="endTime">
            <el-date-picker v-model="newFlight.endTime" type="datetime" placeholder="选择预计抵达时间">
            </el-date-picker>
        </el-form-item>
        <el-row>
            <el-col :span="12">
                <el-form-item label="经济舱座位数量" prop="fEco">
                    <el-input v-model="newFlight.fEco" placeholder="经济舱座位数量"></el-input>
                </el-form-item>
            </el-col>
            <el-col :span="12">
                <el-form-item label="经济舱价格" prop="ecoPrice">
                    <el-input v-model="newFlight.ecoPrice" placeholder="经济舱价格(单位:￥)"></el-input>
                </el-form-item>
            </el-col>
        </el-row>
        <el-row>
            <el-col :span="12">
                <el-form-item label="商务舱座位数量" prop="fBus">
                    <el-input v-model="newFlight.fBus" placeholder="商务舱座位数量"></el-input>
                </el-form-item>
            </el-col>
            <el-col :span="12">
                <el-form-item label="商务舱价格" prop="busPrice">
                    <el-input v-model="newFlight.busPrice" placeholder="商务舱价格(单位:￥)"></el-input>
                </el-form-item>
            </el-col>
        </el-row>
        <el-row>
            <el-col :span="12">
                <el-form-item label="头等舱座位数量" prop="fFc">
                    <el-input v-model="newFlight.fFc" placeholder="头等舱座位数量"></el-input>
                </el-form-item>
            </el-col>
            <el-col :span="12">
                <el-form-item label="头等舱价格" prop="fcPrice">
                    <el-input v-model="newFlight.fcPrice" placeholder="头等舱价格(单位:￥)"></el-input>
                </el-form-item>
            </el-col>
        </el-row>
        <el-row>
            <div style="margin-left:75%">
                <el-button type="success">提交</el-button>
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
      dataListLoading: false,
      formMode: 'add',
      visible: false,
      newFlight:{
          fId:null,
          fName:null,
          pId:null,
          fEco:null,
          ecoPrice:null,
          fBus:null,
          busPrice:null,
          fFc:null,
          fcPrice:null,
          startTime:null,
          endTime:null,
          from:null,
          destination:null
      },
      flightRules:{
          fName:[{required:true, message:'航班名字不能为空', trigger:'blur'}],
          
      },
      fIds:[],
      planes:[],
    };
  },
  mounted: function () {
      this.initFlight();
      this.initPlane();
  },
  methods: {
      alertHelper:function(msg, mType) {
        this.$message({
        type: mType,
        message: msg,
        });
      },
      initFlight:function(){
          this.dataListLoading = true;
          var that = this;
          this.$http.get('https://localhost:8088/admin/getAllFlight')
                    .then(function(result){
                            if(result.data.code == 200){
                                that.flights = result.data.data;
                                for(let i = 0;i<that.flights.length;i++){
                                    that.flights[i].startTime = that.timeFormate(that.flights[i].startTime);
                                    that.flights[i].endTime = that.timeFormate(that.flights[i].endTime);
                                    that.fIds.push(that.flights[i].fId);
                                }
                                that.initTicket();
                            }
                            else{
                                that.alertHelper(result.data.msg,'error');
                            }
                          })
                    .catch(function(error){
                            that.alertHelper('系统内部异常，请稍后重试','error')
                          })
            this.dataListLoading = false;
      },
      initTicket:function(){
        var that = this;
        this.$http.post('https://localhost:8088/getByFidList',that.fIds)
                  .then(function(result){
                      if(result.data.code == 200){
                        for(var i=0;i<result.data.data.length;i=i+3){
                            that.flights[i/3].fEco = "票余量："+that.flights[i/3].fEco+"，价格："+result.data.data[i].tPrice;
                            that.flights[i/3].fBus = "票余量："+that.flights[i/3].fBus+"，价格："+result.data.data[i+1].tPrice;
                            that.flights[i/3].fFc = "票余量："+that.flights[i/3].fFc+"，价格："+result.data.data[i+2].tPrice;
                        }
                      }
                      else{
                          that.alertHelper(result.data.msg,'error');
                      }
                  })
                  .catch(function(error){
                      that.alertHelper('系统内部异常，请稍后重试','error');
                  })
      },

      initPlane:function(){
          var that = this;
          this.$http.get('https://localhost:8088/admin/getAllPlane')
                    .then(function(result){
                        if(result.data.code == 200){
                            that.planes = result.data.data;
                        }
                        else{
                            that.alertHelper(result.data.msg, 'error');
                        }
                    })
                    .catch(function(error){
                        that.alertHelper('系统内部异常', 'error');
                    })
      },

      resetForm(formName) {
        this.$refs[formName].resetFields();
      },

      timeFormate(data){
        var date = new Date(data);
        var Y = date.getFullYear() + '-'
        var M = (date.getMonth() + 1 < 10 ? '0' + (date.getMonth() + 1) : date.getMonth() + 1) + '-'
        var D = (date.getDate() < 10 ? '0' + date.getDate() : date.getDate()) + ' '
        var h = (date.getHours() < 10 ? '0' + date.getHours() : date.getHours()) + ':'
        var m = (date.getMinutes() < 10 ? '0' + date.getMinutes() : date.getMinutes())
        return Y + M + D + h + m
      },
  },
};
</script>