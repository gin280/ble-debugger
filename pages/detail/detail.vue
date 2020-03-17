<template>
  <view class="page">
    <cu-custom bgColor="bg-gradual-blue" :isBack="true">
      <block slot="backText">返回</block>
      <block slot="content">设备详情</block>
    </cu-custom>
    <view v-if="info.deviceId" class="intro margin">
      <view class="text-lg">{{ info.name }}</view>
      <view class="text-gray">UUID:{{ info.deviceId }}</view>
    </view>
    <view v-if="info.deviceId" class="list solid margin">
      <view class="solid-bottom padding" :key="index" v-for="(service, index) of info.services">
        <view>
          <h3 class="margin">服务ID:{{ service.serviceId }}</h3>
          <view @tap="write(service.serviceId, i)" class="text-gray margin-left text-gray" :key="k" v-for="(i, k) of service.characteristics">
            <view>特征UUID：{{ i.uuid }}</view>
            <view>
              notify:
              <text class="text-blue margin-left-sm">{{ i.properties.notify }}</text>
            </view>
            <view>
              read:
              <text class="text-blue margin-left-sm">{{ i.properties.read }}</text>
            </view>
            <view>
              write:
              <text class="text-blue margin-left-sm">{{ i.properties.write }}</text>
            </view>
          </view>
        </view>
      </view>
    </view>
  </view>
</template>

<script>
import { ConnectStatus } from 'wx-ant-ble';
export default {
  onLoad(config) {
    console.info(config);
    this.device = JSON.parse(config.device);
    this.connect();
  },
  onUnload() {
    this.$bt.disconnect();
  },
  data() {
    return {
      device: {},
      info: {}
    };
  },
  methods: {
    write(serviceId, item) {
      // 向蓝牙模块写入数据
      this.$bt
        .write({
          suuid: serviceId, // 特征对应的服务uuid
          cuuid: item.uuid, // 特征uuid
          value: 'jojo' // 写入的数据
        })
        .then(res => {
          // 监听/停止监听 特征值改变，改变特征值从registerDidUpdateValueForCharacteristic注册的方法上报
          this.$bt
            .notify({
              suuid: serviceId, // 特征对应的服务uuid
              cuuid: item.uuid, // 特征uuid
              state: true // 是否监听
            })
            .then(res => {
              console.log('notify success', res);
              uni.showToast({
                title: res.message,
                icon: 'none'
              });
            })
            .catch(e => {
              console.log('notify fail', e);
              uni.showToast({
                title: res.message,
                icon: 'none'
              });
            });
          uni.showToast({
            title: res.message,
            icon: 'none'
          });
          console.log('write success', res);
        })
        .catch(e => {
          console.log('write fail', e);
          uni.showToast({
            title: e.message,
            icon: 'none'
          });
        });
    },
    watchChange() {
      // 注册特征值改变回调，当监听的特征值改变时回调，或者读特征值时回调。
      this.$bt.registerDidUpdateValueForCharacteristic(res => {
        console.log('registerDidUpdateValueForCharacteristic', res);
      });
      // 注册状态更新回调
      this.$bt.registerDidUpdateConnectStatus(res => {
        // 参数结构注意查看log
        console.log('registerDidUpdateConnectStatus', res);
        if (res.connectStatus === ConnectStatus.connected) {
          this.isConnected = true;
          this.info = res.device;
        } else if (res.connectStatus === ConnectStatus.disconnected) {
          this.isConnected = false;
        }
      });
    },
    connect() {
      this.$bt
        .connect(this.device)
        .then(res => {
          uni.showToast({
            title: '连接成功',
            icon: 'none'
          });
          console.log('home connect success', res);
          this.watchChange();
        })
        .catch(e => {
          uni.showToast({
            title: e.message,
            icon: 'none'
          });
          this.$bt.disconnect();
          console.log('home connect fail', e);
        });
      uni.showLoading({
        title: '连接' + this.device.name
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
