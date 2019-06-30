<template>
    <div>
      <nav-header></nav-header>
      <nav-bread>
        <span>支付成功</span>
      </nav-bread>
      <div class="container">
        <div class="page-title-normal">
          <h2 class="page-title-h2"><span></span></h2>
        </div>
        <!-- 进度条 -->
        <div class="check-step">
          <ul>
            <li class="cur">选择地址</li>
            <li class="cur">查看订单</li>
            <li class="cur">订单确认</li>
            <li class="cur">付款成功</li>
          </ul>
        </div>

        <div class="order-create">
          <div class="order-create-pic"><img src="/static/ok-2.png" alt=""></div>
          <div class="order-create-main">
            <h3>亲爱的顾客 <br>您的订单正在派送中...</h3>
            <p>
              <span>订单号：{{orderId}}</span>
              <span>合计：{{orderTotal | currency('')}}</span>
            </p>
            <div class="order-create-btn-wrap">
              <div class="btn-l-wrap" @click="$router.push('/cart')">
                <a class="btn btn--m">Cart List</a>
              </div>
              <div class="btn-r-wrap" @click="$router.push('/goods')">
                <a class="btn btn--m">Goods List</a>
              </div>
            </div>
          </div>
        </div>
      </div>
      <nav-footer></nav-footer>
    </div>
</template>
<script>
  import axios from "axios";
  import NavHeader from "@/components/NavHeader";
  import NavBread from "@/components/NavBread";
  import NavFooter from "@/components/NavFooter";
    export default{
        data(){
            return{
              orderId: '',
              orderTotal: 0
            }
        },
        components: {
          NavHeader,
          NavBread,
          NavFooter
        },
        mounted () {
          let orderId = this.$route.query.orderId;
          this.orderId = orderId;
          axios.get('/users/orderSuccess', {
            params: {
              orderId
            }
          }).then(res => {
            res = res.data;
            this.orderTotal = res.result.orderTotal;
          })
        }
    }
</script>
