<!--扫码支付-->
<template>
  <div id="scan-container">
    <header>
      <i class="iconfont pay-icon" :style="{color:payTypeObj[payType]['color']}"
         v-html="payTypeObj[payType]['icon']"></i>
      <span class="pay-way-name">{{payTypeObj[payType]['name']}}</span>
    </header>
    <div class="qrcode-container">
      <div id="qrcode" ref="qrcode"></div>
    </div>
    <div class="info-container">
      <ul>
        <li><span>产品名称：{{orderData.tradeName}}</span></li>
        <li><span>订单编号：{{orderData.outTradeNo}}</span></li>
        <li><span>订单金额：{{orderData.amount / 100}}</span></li>
        <li><span>实付金额：{{orderData.amount / 100}}</span></li>
      </ul>
    </div>
    <div class="close" @click="close();">
      <i class="iconfont icon-close">&#xe625;</i>
    </div>
    <alert-tip :text="alertText" :showTip.sync="showTip"></alert-tip>
  </div>
</template>

<script>
  import QRCode from '@/plugins/qrcode'
  import {listenStatus} from '@/api/order'

  export default {
    data() {
      return {
        payTypeObj: {
          1: {
            icon: '&#xe60f;',
            color: '#3d91e4',
            name: '支付宝支付'
          },
          2: {
            icon: '&#xe62a;',
            color: '#2aaf90',
            name: '微信支付'
          },

        },
        qrcode: null,
        timer: null,
        alertText: '',
        showTip: ''
      }
    },
    methods: {
      // 关闭该支付就清除监听支付状态
      close() {
        clearInterval(this.timer)
        this.$emit('close');
      },
      // out_trade_no是指商户网站唯一订单号，监听该订单号的状态，状态是支付成功还是未成功
      listenStatus(outTradeNo) {
        clearInterval(this.timer);
        let _this = this;
        // 每三秒就发起一次请求，判断支付的状态
        this.timer = setInterval(() => {
          listenStatus({outTradeNo}).then((response) => {
            if (response.data.status === 200) {
              clearInterval(this.timer);
              this.alertText = '支付成功，准备跳转';
              this.showTip = true;
              // 如果支付成功就进入订单详细列表页，但是刚拿到的
              _this.$router.push({path: '/order_detail', query: {id: _this.orderData.order_id}})
              // 解决新支付的订单回到order_detail页面后，请求拿不到最新支付的进去的订单
              if(location.href.indexOf("#reloaded")==-1){
                  location.href=location.href+"#reloaded";
                    location.reload()
              }
            }
          })
        }, 3000);
      }
    },
    props: ['payType', 'orderData'],
    watch: {
      orderData(val) {
        this.orderData = val;
        if (this.qrcode) {
          this.qrcode.makeCode(val.data.qrcode);
        } else {
          this.qrcode = new QRCode(this.$refs['qrcode'], {
            text: val.data.qrcode,

            width: 200,
            height: 200,
          });
        }
        this.listenStatus(val.outTradeNo);
      }
    }
  }
</script>

<style rel="stylesheet/scss" lang="scss" scoped>
  #scan-container {
    background: #fff;
    position: fixed;
    top: 0;
    right: 0;
    left: 0;
    bottom: 0;
    header {
      margin: 0.5rem 0;
      padding-left: 1rem;
      .pay-icon {
        font-size: 1.2rem;
      }
      .pay-way-name {
        font-weight: normal;
        font-size: 0.7rem;
      }
    }
    .qrcode-container {
      width: 200px;
      height: 200px;
      margin: 1rem auto;
    }
    .info-container {
      padding-left: 1rem;
      ul {
        li {
          margin: 0.4rem 0;
          span {
            font-size: 0.4rem;
          }
        }
      }
    }
    .close {
      margin: 0.5rem 0;
      text-align: center;
      .icon-close {
        font-size: 1rem;
      }
    }
  }
</style>
