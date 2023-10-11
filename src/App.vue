<script setup>
</script>

<template>
  <el-row :gutter="20">
  <el-col :span="12">
    <el-row :gutter="20">
      <el-col :span="12">
        <el-card shadow="always">
          <el-statistic title="上传总量" :value="SumUp" />
        </el-card>
      </el-col>
      <el-col :span="12">
        <el-card shadow="always">
          <el-statistic title="下载总量" :value="SumDown" />
        </el-card>
      </el-col>
    </el-row>
    <el-row :gutter="20">
      <el-col :span="24">
        <el-card shadow="always">
          <template #header>
            <div class="card-header">
              <span>带宽监控</span>
              <div>
              <el-button class="button" type="primary" disabled>U: {{ NetUp }}</el-button>
              <el-button class="button" type="success" disabled>D: {{ NetDown }}</el-button></div>
            </div>
          </template>
          <canvas id="chart0" class="w-auto" style="min-height: 100px"/>
        </el-card>
      </el-col>
    </el-row>
  </el-col>
  <el-col :span="12">
    <el-row :gutter="20">
      <el-col :span="12">
        <el-card shadow="always">
          <el-statistic title="连接总数" :value="ConnNum" />
        </el-card>
      </el-col>
      <el-col :span="12">
        <el-card shadow="always">
          <el-statistic title="内核版本" :value="CoreV" />
        </el-card>
      </el-col>
    </el-row>
    <el-row :gutter="20">
      <el-col :span="24">
        <el-card shadow="always">
          <template #header>
            <div class="card-header">
              <span>内存监控</span>
              <el-button class="button" type="primary" disabled>{{ MemUse }} MB</el-button>
            </div>
          </template>
          <canvas id="chart1" class="w-auto" style="min-height: 100px"/>
<!--          <div>-->
<!--&lt;!&ndash;            <Line :data="data2" :options="options" />&ndash;&gt;-->
<!--          </div>-->
        </el-card>
      </el-col>
    </el-row>
  </el-col>
  </el-row>
</template>

<style scoped>
.el-row {
  margin-bottom: 10px;
}
.el-row:last-child {
  margin-bottom: 0;
}
.el-col {
  border-radius: 4px;
}

.grid-content {
  border-radius: 4px;
  min-height: 36px;
}

.el-card {
  --el-card-padding: 8px;
}

.card-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
}

</style>

<script>
import Chart from "chart.js/auto"
import axios from "axios";

const VITE_CLASH_URL = import.meta.env.VITE_CLASH_URL;
const VITE_CLASH_TOKEN = import.meta.env.VITE_CLASH_TOKEN;

document.addEventListener("DOMContentLoaded", function () {
  const lineCtx0 = document.getElementById('chart0')
  const lineCtx1 = document.getElementById('chart1')
  const lineConfig0 = {
    type: 'line',
    data: {
      labels: ['', '', '', '', '', '', '', '', '', '', ''],
      datasets: [
        {
          fill: true,
          backgroundColor: 'rgb(179, 225, 157, 0.2)',
          borderColor: '#B3E19D',
          data: [0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0],
        },
        {
          fill: true,
          backgroundColor: 'rgb(160, 207, 255, 0.2)',
          borderColor: '#A0CFFF',
          data: [0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0],
        }
      ],
    },
    options: {
      responsive: true,
      plugins: {
        legend: {
          display: false,
        }
      },
      scales: {
        x: {
          display: true,
        },
        y: {
          display: true,
          min: 0,
          ticks: {
            // Include a dollar sign in the ticks
            callback: function(value, index, ticks) {
              if (value > 1000){
                return value/1000 + 'MB';
              }else {
                return value + 'KB';
              }

            }}
        },
        // myScale: {
        //   axis: 'r'
        // }
      },
    }
  }
  const lineConfig1 = {
    type: 'line',
    data: {
      labels: ['', '', '', '', '', '', '', '', '', '', ''],
      datasets: [
        {
          fill: true,
          backgroundColor: 'rgb(160, 207, 255, 0.2)',
          borderColor: '#A0CFFF',
          data: [0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0],
        }
      ],
    },
    options: {
      responsive: true,
      plugins: {
        legend: {
          display: false,
        }
      },
      scales: {
        x: {
          display: true,
          scaleLabel: {
            display: true,
            labelString: 'Month',
          },
        },
        y: {
          display: true,
          min: 0,
          scaleLabel: {
            display: true,
            labelString: 'Value',
          },
        },
      },
    }
  }
  window.myLine0 = new Chart(lineCtx0, lineConfig0)
  window.myLine1 = new Chart(lineCtx1, lineConfig1)
});




export default {
  name: 'App',

  data() {
    return {
      NetUp: '0 KB',
      NetDown: '0 KB',
      SumUp: '0 KB',
      SumDown: '0 KB',
      MemUse: 0,
      ConnNum: 0,
      CoreV: 'Meta',
    };
  },
  created() {
    this.getVersion();
    this.initWebSocket0();
    this.initWebSocket1();
    this.initWebSocket2();
  },
  // mounted() {
  //   this.getVersion();
  // },
  // mounted() {
  //   //this.initWebSocket();
  // },
  // destroyed: function () { // 离开页面生命周期函数
  //   this.websocketclose();
  // },
  methods: {
    // updateChart() {
    //   this.$data._chart.update();
    // },
    getVersion: function () {
      var that = this
      axios.get(`http://${VITE_CLASH_URL}/version`, {
        headers: {
          'Authorization': `Bearer ${VITE_CLASH_TOKEN}`,
        },
      }).then(function (response){
        that.CoreV = response.data['version']
      })
    },
    initWebSocket0: function () {
      var url = `ws://${VITE_CLASH_URL}/traffic?token=${VITE_CLASH_TOKEN}`;
      console.log(url);
      this.websock = new WebSocket(url);
      this.websock.onopen = this.websocketOnopen;
      this.websock.onerror = this.websocketOnerror;
      this.websock.onmessage = this.websocketOnmessage0
      this.websock.onclose = this.websocketOnclose;
    },
    initWebSocket1: function () {
      var url = `ws://${VITE_CLASH_URL}/memory?token=${VITE_CLASH_TOKEN}`;
      console.log(url);
      this.websock = new WebSocket(url);
      this.websock.onopen = this.websocketOnopen;
      this.websock.onerror = this.websocketOnerror;
      this.websock.onmessage = this.websocketOnmessage1
      this.websock.onclose = this.websocketOnclose;
    },
    initWebSocket2: function () {
      var url = `ws://${VITE_CLASH_URL}/connections?token=${VITE_CLASH_TOKEN}`;
      console.log(url);
      this.websock = new WebSocket(url);
      this.websock.onopen = this.websocketOnopen;
      this.websock.onerror = this.websocketOnerror;
      this.websock.onmessage = this.websocketOnmessage2
      this.websock.onclose = this.websocketOnclose;
    },
    websocketOnopen: function () {
      console.log("WebSocket连接成功");
      //心跳检测重置
      //this.heartCheck.reset().start();
    },
    websocketOnerror: function (e) {
      console.log("WebSocket连接发生错误");
      this.reconnect();
    },
    websocketOnmessage0: function (e) {
      console.log('监听关闭' + e)
      console.log("接收流量消息：", e.data);
      var data0 = eval("(" + e.data + ")"); //解析对象
      window.myLine0.data.datasets[0].data.shift()
      window.myLine0.data.datasets[0].data.push(data0['down']/1000)
      window.myLine0.data.datasets[1].data.shift()
      window.myLine0.data.datasets[1].data.push(data0['up']/1000)
      window.myLine0.update();
      var nd =  (data0['down']/1000).toFixed(2)
      var nu =  (data0['up']/1000).toFixed(2)
      if(nd>1000){
        this.NetDown = (nd/1000).toFixed(2).toString() + ' MB'
      }else {
        this.NetDown = nd.toString() + ' KB'
      }
      if(nu>1000){
        this.NetUp = (nu/1000).toFixed(2).toString() + ' MB'
      }else {
        this.NetUp = nu.toString() + ' KB'
      }
    },
    websocketOnmessage1: function (e) {
      console.log('监听关闭' + e)
      console.log("接收内存消息：", e.data);
      var data = eval("(" + e.data + ")"); //解析对象
      window.myLine1.data.datasets[0].data.shift()
      window.myLine1.data.datasets[0].data.push(data['inuse']/1000/1000)
      window.myLine1.update();
      this.MemUse = (data['inuse']/1000/1000).toFixed(2)
    },
    websocketOnmessage2: function (e) {
      console.log('监听关闭' + e)
      console.log("接收连接消息：", e.data);
      var data = eval("(" + e.data + ")"); //解析对象
      if (data['downloadTotal']/1000 > 1000){
          if (data['downloadTotal']/1000/1000> 1000){
            this.SumDown = (data['downloadTotal']/1000/1000/1000).toFixed(2).toString() + 'GB'
          }else {
            this.SumDown = (data['downloadTotal']/1000/1000).toFixed(2).toString() + 'MB'
          }
        }else {
          this.SumDown = (data['downloadTotal']/1000).toFixed(2).toString() + 'KB'
        }
      if (data['uploadTotal']/1000 > 1000){
        if (data['uploadTotal']/1000/1000> 1000){
          this.SumUp = (data['uploadTotal']/1000/1000/1000).toFixed(2).toString() + 'GB'
        }else {
          this.SumUp = (data['uploadTotal']/1000/1000).toFixed(2).toString() + 'MB'
        }
      }else {
        this.SumUp = (data['uploadTotal']/1000).toFixed(2).toString() + 'KB'
      }
      this.ConnNum = data['connections'].length
    },
    websocketOnclose: function (e) {
      console.log("connection closed (" + e.code + ")");
      this.reconnect();
    },
    reconnect() {
      var that = this;
      if (that.lockReconnect) return;
      that.lockReconnect = true;
      //没连接上会一直重连，设置延迟避免请求过多
      setTimeout(function () {
        console.info("尝试重连...");
        that.initWebSocket0();
        that.initWebSocket1();
        that.initWebSocket2();
        that.lockReconnect = false;
      }, 5000);
    },
    // websocketclose: function (){
    //   this.initWebSocket1()
    // }
  }
}
</script>
