<template>
  <div id="home" >
    <nav-bar class="home-nav">
      <div slot="center">购物街</div>
    </nav-bar>

    <scroll class="content"
            ref="scroll"
            :probe-type="3"
            @scroll="contentScroll"
            :pull-up-load="true"
            @pullingUp="loadMore">
      <home-swiper :banners="banners"/>
      <recommend-view :recommends="recommends"/>
      <featur-view/>
      <tab-control :titles="['流行', '新款', '精选']"
                   class="tab-control"
                   @tabClick="tabClick"/>

      <goods-list :goods="goods[currentType].list"/>
    </scroll>
  </div>
</template>

<script>
  import NavBar from "components/common/navbar/NavBar"
  import HomeSwiper from "./childComps/HomeSwiper"
  import RecommendView from "./childComps/RecommendView"
  import FeaturView from "./childComps/FeatureView"

  import TabControl from "components/content/tabControl/TabControl"

  import GoodsList from "components/content/goods/GoodsList"
  import Scroll from "components/common/scroll/Scroll";

  import {getHomeMultidata, getHomeGoods} from "network/home"

  export default {
    name: "Home",
    components: {
        NavBar,
        HomeSwiper,
        RecommendView,
        FeaturView,
        TabControl,
        GoodsList,
        Scroll
    },
    data() {
      return {
          banners: [],
          recommends: [],
          goods: {
              'pop': {page: 0, list: []},
              'new': {page: 0, list: []},
              'sell': {page: 0, list: []}
          },
          currentType: 'pop'
      }
    },
    created() {
        // 请求轮播与类型多个数据
        this.getHomeMultidata()

        // 请求商品数据
        this.getHomeGoods('pop')
        this.getHomeGoods('new')
        this.getHomeGoods('sell')
    },
    methods: {
        /**
         *事件监听相关方法
         */
        tabClick(index) {
            switch (index) {
                case 0:
                    this.currentType = 'pop'
                    break
                case 1:
                    this.currentType = 'new'
                    break
                case 2:
                    this.currentType = 'sell'
                    break
            }
        },

        backClick() {
            this.$refs.scroll.scrollTo(0, 0)
        },
        contentScroll(position) {
            this.isShowBackTop = (-position.y) > 1000
        },
        loadMore() {
            this.getHomeGoods(this.currentType)
        },

        /**
         * 网络请求相关方法
         */
        getHomeMultidata() {
            getHomeMultidata().then(res => {
                this.banners = res.data.banner.list;
                this.recommends = res.data.recommend.list;
            })
        },
        getHomeGoods(type) {
            const page = this.goods[type].page + 1;
            getHomeGoods(type, page).then(res => {
                this.goods[type].list.push(...res.data.list);
                this.goods[type].page += 1

                this.$refs.scroll.finishPullUp()
            })
        }
    }
  }
</script>

<style scoped>
  #home {
    /*padding-top: 44px;*/
    height: 100vh;
  }
  .home-nav {
    background-color: var(--color-tint);
    color: #ffffff;

    position: fixed;
    left: 0;
    right: 0;
    top: 0;
    z-index: 9;
  }
  .tab-control {
    position: sticky;
    top: 44px;
    z-index: 9;
  }
  .content {
    overflow: hidden;
    position: absolute;
    margin-top: 44px;
    bottom: 49px;
    left: 0;
    right: 0;

    background-color: #ffffff;
  }
</style>
