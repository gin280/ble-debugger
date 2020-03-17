<template>
  <view class="page">
    <cu-custom bgColor="bg-gradual-blue" :isBack="false">
      <block slot="backText">返回</block>
      <block slot="content">蓝牙开发</block>
    </cu-custom>
    <button @tap="scan" class="margin cu-btn bg-blue lg flex justify-center">gogogo</button>
    <view class="text-lg margin">已发现{{ devices.length }}个外围设备</view>
    <scroll-view class="solid margin" style="height: 70vh;" scroll-y="true">
      <view @tap="connect(device)" :key="index" v-for="(device, index) of devices" class="solid-bottom padding">
        <view class="text-lg">{{ device.name }}</view>
        <view class="text-sm text-gray">信号强度：{{ device.RSSI }}</view>
        <view class="text-sm text-gray">UUID：{{ device.deviceId }}</view>
        <!-- <view class="text-sm text-gray">Service数量：{{ device.advertisData }}</view> -->
      </view>
    </scroll-view>
  </view>
</template>

<script>
export default {
  onLoad() {
    this.watchChange();
  },
  onHide() {
    this.$bt.stopScan()
  },
  data() {
    return {
      PageCur: 'basics',
      isConnected: false,
      devices: []
    };
  },
  methods: {
    NavChange: function(e) {
      this.PageCur = e.currentTarget.dataset.cur;
    },
    connect(device) {
      uni.navigateTo({
        url: `/pages/detail/detail?device=${JSON.stringify(device)}`
      });
    },
    scan() {
      // 开始扫描
      this.$bt
        .scan({
          services: [], // 主service的uuid列表
          allowDuplicatesKey: false, // 是否允许重复上报设备
          interval: 0, // 上报新设备的间隔，默认为0
          timeout: 15000, // 扫描超时时间，毫秒
          deviceName: '', // 需要匹配的设备名称
          containName: '' // 需要包含的设备名称
        })
        .then(res => {
          console.log('scan success', res);
        })
        .catch(e => {
          console.log('scan fail', e);
        });
    },
    watchChange() {
      // 注册发现外设回调，当扫描到设备时回调，或者达到超时时间回调。
      this.$bt.registerDidDiscoverDevice(res => {
        if (res.timeout) {
          console.log('home didDiscoverDevice', '扫描超时');
          uni.showModal({
            content: '扫描超时',
            showCancel: false
          });
        } else {
          let device = res.device;
          // 检查重复上报设备，更新信息
          function checkDuplicateDevice(d, ds) {
            for (let v of ds) {
              if (v.deviceId === d.deviceId) {
                Object.assign(v, d);
                return true;
              }
            }
            return false;
          }
          // 更新 & 过滤
          if (!checkDuplicateDevice(device, this.devices)) {
            this.devices.push(device);
          }
          // console.info(this.devices, 'jojo');
        }
      });
    }
  }
};
</script>

<style lang="scss">
.page {
  width: 100vw;
  min-height: 100vh;
}
</style>
