<template>
  <div>
    <symbol id="icon-arrow-short" viewBox="0 0 25 32">
        <title>arrow-short</title>
        <path class="path1" d="M24.487 18.922l-1.948-1.948-8.904 8.904v-25.878h-2.783v25.878l-8.904-8.904-1.948 1.948 12.243 12.243z"></path>
    </symbol>
    <symbol id="icon-status-ok" viewBox="0 0 32 32">
      <title>status-ok</title>
      <path class="path1" d="M22.361 10.903l-9.71 9.063-2.998-2.998c-0.208-0.209-0.546-0.209-0.754 0s-0.208 0.546 0 0.754l3.363 3.363c0.104 0.104 0.241 0.156 0.377 0.156 0.131 0 0.261-0.048 0.364-0.143l10.087-9.414c0.215-0.201 0.227-0.539 0.026-0.754s-0.539-0.226-0.754-0.026z"></path>
      <path class="path2" d="M16 30.933c-8.234 0-14.933-6.699-14.933-14.933s6.699-14.933 14.933-14.933c8.234 0 14.933 6.699 14.933 14.933s-6.699 14.933-14.933 14.933zM16 0c-8.822 0-16 7.178-16 16 0 8.823 7.178 16 16 16s16-7.177 16-16c0-8.822-7.178-16-16-16z"></path>
    </symbol>
    <nav-header></nav-header>
    <nav-bread>
      <span>商品</span>
    </nav-bread>
    <div class="accessory-result-page accessory-page">
      <div class="container">
        <div class="filter-nav">
          <span class="sortby">排序：</span>
          <a href="javascript:void(0)" class="default cur" @click="handleSort(1)">默认</a>
          <a href="javascript:void(0)" class="price" @click="handleSort(0)">
            价格
            <svg class="icon-arrow-short"  :class="{'up-arrow': this.sortFlag == '1' ? true : false}">
              <use xmlns:xlink="http://www.w3.org/1999/xlink" xlink:href="#icon-arrow-short"></use>
            </svg>
          </a>
          <a href="javascript:void(0)" class="filterby stopPop" @click="handleFilterBy">Filter by</a>
        </div>
        <div class="accessory-result">
          <!-- filter -->
          <div class="filter stopPop" id="filter" :class="{'filterby-show': filterBy}">
            <dl class="filter-price">
              <dt>筛选：</dt>
              <dd>
                <a
                  href="javascript:void(0)"
                  :class="{'cur': priceChecked == 'all'}"
                  @click="handlePriceRange('all')"
                >全部</a>
              </dd>
              <dd v-for="(item, index) in priceRange" :key="index" @click="handlePriceRange(index)">
                <a
                  href="javascript:void(0)"
                  :class="{'cur': priceChecked == index}"
                >{{item.startPrice}} - {{item.endPrice}}</a>
              </dd>
            </dl>
          </div>

          <!-- search result accessories list -->
          <div class="accessory-list-wrap">
            <div class="accessory-list col-4">
              <ul>
                <li v-for="(item, index) in productList" :key="index">
                  <div class="pic">
                    <a href="#">
                      <img v-lazy="'static/' + item.productImage" :key="item.productImage" alt="item.productName">
                    </a>
                  </div>
                  <div class="main">
                    <div class="name">{{item.productName}}</div>
                    <div class="price">{{item.salePrice}} 元</div>
                    <div class="btn-area">
                      <a
                        href="javascript:;"
                        class="btn btn--m"
                        @click="handleAddCart(item.productId)"
                      >加入购物车</a>
                    </div>
                  </div>
                </li>
              </ul>
              <div
                style="text-align: center;"
                v-infinite-scroll="loadMore"
                infinite-scroll-disabled="busy"
                infinite-scroll-distance="0"
                infinite-scroll-throttle-delay="500"
                v-show="loading"
              >
                <img src="static/loading-svg/loading-bars.svg" alt>
              </div>
            </div>
          </div>
        </div>
      </div>
    </div>
    <common-model :modelShow="banModelShow" @closeModel="closeModel">
      <p slot="message">请先登录，否则无法加入购物车</p>
      <div slot="btnGroup">
        <a href="javascript:;" class="btn btn--m" @click="closeModel">关闭</a>
      </div>
    </common-model>
    <common-model :modelShow="cartModelShow" @closeModel="closeModel">
      <p slot="message">
        <svg class="icon-status-ok">
          <use xmlns:xlink="http://www.w3.org/1999/xlink" xlink:href="#icon-status-ok"></use>
        </svg>
        加入购物车成功！
      </p>
      <div slot="btnGroup">
        <a href="javascript:;" class="btn btn--m" @click="closeModel">继续购物</a>
        <router-link href="javascript:;" class="btn btn--m" to="/cart">前往购物车</router-link >
      </div>
    </common-model>
    <nav-footer></nav-footer>
  </div>
</template>
<script>
import axios from "axios";
import NavHeader from "@/components/NavHeader";
import NavBread from "@/components/NavBread";
import NavFooter from "@/components/NavFooter";
import CommonModel from '@/components/CommonModel';
import getCookie from '@/util/cookies.js';
export default {
  data() {
    return {
      productList: [],
      priceRange: [
        {
          startPrice: "0",
          endPrice: "100"
        },
        {
          startPrice: "100",
          endPrice: "500"
        },
        {
          startPrice: "500",
          endPrice: "1000"
        },
        {
          startPrice: "1000",
          endPrice: "2000"
        }
      ],
      priceChecked: "all",
      filterBy: false,
      overlayShow: false,
      sortFlag: 0,
      sortPriceRange: "all",
      page: 1,
      pageSize: 8,
      busy: true,
      loading: false,
      // 模框
      banModelShow: false,
      cartModelShow: false
    };
  },
  components: {
    NavHeader,
    NavBread,
    NavFooter,
    CommonModel
  },
  mounted() {
    // console.log('mounted');
    this.getGoodsData();
  },
  methods: {
    getGoodsData(flag) {
      this.loading = true;
      let params = {
        page: this.page,
        pageSize: this.pageSize,
        sort: this.sortFlag,
        sortPriceRange: this.sortPriceRange
      };
      axios
        .get("/goods/list", {
          params
        })
        .then(res => {
          // console.log(res)
          res = res.data;
          this.loading = false;
          if (res.status == "0") {
            if (flag) {
              this.productList = this.productList.concat(res.result.list);
            } else {
              this.productList = res.result.list;
              this.busy = false;
            }
            if (res.result.count < this.pageSize) {
              this.busy = true;
            }
            // console.log(this.productList)
          } else {
            this.busy = true;
          }
        });
    },
    handleSort(def) {
      this.page = 1;
      if (!def) {
        if (this.sortFlag == 0) {
          this.sortFlag = 1;
        } else {
          this.sortFlag = this.sortFlag == 1 ? -1 : 1;
        }
      } else {
        this.sortFlag = 0;
      }
      this.getGoodsData();
    },
    loadMore() {
      // console.log('load');
      this.busy = true;
      this.page++;
      this.busy = false;
      this.getGoodsData(true);
    },
    handleFilterBy() {
      // console.log('show');
      this.filterBy = true;
      this.overlayShow = true;
    },
    handlePriceRange(index) {
      this.page = 1;
      this.closeOverlay();
      if (index != "all") {
        this.sortPriceRange =
          this.priceRange[index].startPrice +
          "-" +
          this.priceRange[index].endPrice;
      } else {
        this.sortPriceRange = "all";
      }
      this.priceChecked = index;
      this.getGoodsData();
    },
    closeOverlay() {
      this.overlayShow = false;
      this.filterBy = false;
      this.loginShow = false;
    },
    handleAddCart(id) {
      axios
        .post("/goods/addCart", {
          productId: id
        })
        .then(res => {
          res = res.data;
          if (res.status == '0') {
            this.cartModelShow = true;
            this.$store.commit('updateCartCount', 1);
          } else {
            this.banModelShow = true;
          }
        });
    },
    closeModel () {
      this.banModelShow = false;
      this.cartModelShow = false;
    }
  }
};
</script>

<style lang="stylus" scoped>
  .up-arrow
    transform rotate(180deg)
    transition all .3s ease-out
</style>

