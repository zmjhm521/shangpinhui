<template>
  <div>
    <!-- 商品分类三级列表 -->
    <TypeNav />
    <div class="main">
      <div class="py-container">
        <!--bread面包屑，带有x的结构 -->
        <div class="bread">
          <ul class="fl sui-breadcrumb">
            <li>
              <a href="#">全部结果</a>
            </li>
          </ul>
          <ul class="fl sui-tag">
            <!-- 分类面包屑 -->
            <li class="with-x" v-if="searchParams.categoryName">{{ searchParams.categoryName }}<i
                @click="removeCategoryName">×</i></li>
            <!-- 关键字面包屑 -->
            <li class="with-x" v-if="searchParams.keyword">{{ searchParams.keyword }}<i @click="removeKeyword">×</i>
            </li>
            <!-- 品牌面包屑 -->
            <li class="with-x" v-if="searchParams.trademark">{{ trade.tmName }}<i @click="removeTrade">×</i>
            </li>
            <!-- 属性面包屑 -->
            <li class="with-x" v-for="(prop, index) in searchParams.props" :key="index">{{ prop.split(':')[1] }}<i
                @click="removeProps(index)">×</i>
            </li>
          </ul>
        </div>

        <!--selector-->
        <!-- <SearchSelector :attrsList="attrsList" :trademarkList="trademarkList" /> -->
        <SearchSelector @trademarkInfo="trademarkInfo" @attInfo="attInfo"></SearchSelector>
        <!--details-->
        <div class="details clearfix">
          <div class="sui-navbar">
            <div class="navbar-inner filter">
              <ul class="sui-nav">
                <li :class="{ active: isOne }" @click="changeOrder('1')">
                  <a>综合<span v-show="isOne" class="iconfont"
                      :class="{ 'icon-up': isAsc, 'icon-todown': isDesc }"></span></a>
                </li>
                <li :class="{ active: isTwo }" @click="changeOrder('2')">
                  <a>价格<span v-show="isTwo" class="iconfont"
                      :class="{ 'icon-up': isAsc, 'icon-todown': isDesc }"></span></a>
                </li>
              </ul>
            </div>
          </div>
          <!-- 销售产品列表 -->
          <div class="goods-list">
            <ul class="yui3-g">
              <li class="yui3-u-1-5" v-for="(good, index) in goodsList" :key="good.id">
                <div class="list-wrap">
                  <div class="p-img">
                    <a @click="goDetail(good.id)"><img v-lazy=good.defaultImg /></a>
                  </div>
                  <div class="price">
                    <strong>
                      <em>¥</em>
                      <i>{{ good.price }}</i>
                    </strong>
                  </div>
                  <div class="attr">
                    <a target="_blank" href="item.html" title="促销信息，下单即赠送三个月CIBN视频会员卡！【小米电视新品4A 58 火爆预约中】">{{
                      good.title
                    }}</a>
                  </div>
                  <div class="commit">
                    <i class="command">已有<span>{{ good.hootScore }}</span>人评价</i>
                  </div>
                  <div class="operate">
                    <a href="success-cart.html" target="_blank" class="sui-btn btn-bordered btn-danger">加入购物车</a>
                    <a href="javascript:void(0);" class="sui-btn btn-bordered">收藏</a>
                  </div>
                </div>
              </li>
            </ul>
          </div>
          <!-- 分页器，测试分页器阶段，这里的数据需要替换 -->
          <Pagination :pageNo="searchParams.pageNo" :pageSize="searchParams.pageSize" :total="total" :continues="5"
            @getPageNo="getPageNo"></Pagination>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import SearchSelector from '@/components/Search/SearchSelector.vue'
import { mapGetters, mapState } from 'vuex'
export default {
  name: 'Search',
  // 组件挂载完毕执行一次「仅仅执行一次 」
  data() {
    return {
      // 带给服务器的参数
      searchParams: {
        "category1Id": "",
        "category2Id": "",
        "category3Id": "",
        // 分类名字
        "categoryName": "",
        "keyword": "",
        // 排序
        "order": "1:desc",
        // 分页器当前页
        "pageNo": 1,
        // 每一页展示数据个数
        "pageSize": 3,
        // 平台售卖属性操作的数据
        "props": [],
        // 品牌
        "trademark": ""
      },
      trade: undefined,
    }
  },
  // 在发送请求之前，把接口需要传递的参数进行整理，服务器就会返回查询的数据
  beforeMount() {
    // Object.assign:ES6新增合并对象
    Object.assign(this.searchParams, this.$route.query)
  },
  mounted() {
    
    // 在发送请求之前带给服务器参数「searchParams」参数发生变化，有数值带给服务器
    this.getData()
  },
  computed: {
    // mapGetters传递的是数字，因为getters计算是不区分模块的
    ...mapGetters(['goodsList', 'trademarkList', 'attrsList']),
    // 获取search模块展示产品一共对少数据
    ...mapState({
      total: (state) => {
        return state.search.list.total
      },
      isOne() {
        return this.searchParams.order.indexOf('1') != -1
      },
      isTwo() {
        return this.searchParams.order.indexOf('2') != -1
      },
      isAsc() {
        return this.searchParams.order.indexOf('asc') != -1
      },
      isDesc() {
        return this.searchParams.order.indexOf('desc') != -1
      }
    })
  },
  components: {
    SearchSelector
  },
  methods: {
    // 向服务器发送请求获取search模块数据（根据不同参数不同返回不同的数据进行显示）
    // 把请求封装为一个函数，需要调用时调用即可
    getData() {
      this.$store.dispatch('searchList', this.searchParams)
    },
    removeCategoryName() {
      // 把带给服务器的参数置空了，还需要向服务器发请求
      this.searchParams.categoryName = ''
      this.removeId()
      // 地址栏路径修改,进行由于跳转
      // 本意是删除query,如果路径中出现params不应该删除,路由
      this.$router.push({ name: 'search', params: this.$route.params },)
    },
    removeKeyword() {
      this.searchParams.keyword = undefined
      // 通知兄弟组件Header清楚关键字
      this.$bus.$emit("clear")
      this.$router.push({ name: 'search', query: this.$route.query },)
    },
    removeId() {
      // 带给服务器参数说明（可有可无的）如果属性值为空的字符串还是会吧相应的字段带个服务器
      // 把响应的字段变为undefined,当前这个字段不会带给服务器
      this.searchParams.category1Id = undefined
      this.searchParams.category2Id = undefined
      this.searchParams.category3Id = undefined
    },
    // 自定义事件回调
    trademarkInfo(trade) {
      // 整理品牌字段参数
      this.searchParams.trademark = `${trade.tmId}:${trade.tmName}`
      this.getData()
      this.trade = trade
    },
    removeTrade() {
      this.searchParams.trademark = undefined
      // 再次发送请求
      this.getData()
    },
    attInfo(attr, att) {
      //  ["属性ID:属性值:属性名"]
      // 数组去重
      let props = `${attr.attrId}:${att}:${attr.attrName}`
      // if语句里面只有一行代码，可以省略大花括号
      if (this.searchParams.props.indexOf(props) === -1) {
        this.searchParams.props.push(props)
        this.getData()
      }
    },
    // 删除售卖属性
    removeProps(index) {
      // console.log(index);
      // 再次整理参数
      this.searchParams.props.splice(index, 1)
      // console.log(this.searchParams.props);
      // 再次发送请求
      this.getData()
    },
    // 排序
    changeOrder(flag) {
      // flag形参标记用户点击的是综合（1）还是价格（2）
      // 获取oreder初始状态，会随着用户的操作进行更新 
      let originOrder = this.searchParams.order
      let originFlag = originOrder.split(':')[0]
      let originSort = originOrder.split(':')[1]
      // 准备一个新的order属性值
      let newOrder = ''
      // 判断的是多次点击的是不是同一个li
      if (flag == originFlag) {
        newOrder = `${originFlag}:${originSort == 'desc' ? 'asc' : 'desc'}`
      } else {
        // 判断的是不同的按钮
        newOrder = `${flag}:'desc'`
      }
      // 整理参数
      this.searchParams.order = newOrder
      // 发请求
      this.getData()
    },
    // 自定义事件获取当前第几页
    getPageNo(pageNo) {
      // 整理参数
      this.searchParams.pageNo = pageNo
      // 发送请求
      this.getData()
    },
    // 路由跳转时不要忘记带id（params）
    goDetail(skuid) {
      if (this.$route.params) {
        let location = { name: "detail", params: { skuid:skuid || undefined } }
        this.$router.push(location)
      }
    }
  },
  // 数据监听，监听组件实例身上的属性的属性值变化
  watch: {
    // 监听路由的信息是否发生变化，如果发生变化，再次发起请求
    $route(newValue, oldValue) {
      // 在此发送请求之前整理带给服务器的参数
      Object.assign(this.searchParams, this.$route.params, this.$route.query)
      this.getData()
      // 每一次请求完毕，应当把相应的1、2、3级分类置空，让其接受下一次的响应1、2、3id
      this.removeId()
      // this.searchParams.keyword=''
      // console.log(this.searchParams);
    }
  }
}
</script>

<style lang="less" scoped>
.main {
  margin: 10px 0;

  .py-container {
    width: 1200px;
    margin: 0 auto;

    .bread {
      margin-bottom: 5px;
      overflow: hidden;

      .sui-breadcrumb {
        padding: 3px 15px;
        margin: 0;
        font-weight: 400;
        border-radius: 3px;
        float: left;

        li {
          display: inline-block;
          line-height: 18px;

          a {
            color: #666;
            text-decoration: none;

            &:hover {
              color: #4cb9fc;
            }
          }
        }
      }

      .sui-tag {
        margin-top: -5px;
        list-style: none;
        font-size: 0;
        line-height: 0;
        padding: 5px 0 0;
        margin-bottom: 18px;
        float: left;

        .with-x {
          font-size: 12px;
          margin: 0 5px 5px 0;
          display: inline-block;
          overflow: hidden;
          color: #000;
          background: #f7f7f7;
          padding: 0 7px;
          height: 20px;
          line-height: 20px;
          border: 1px solid #dedede;
          white-space: nowrap;
          transition: color 400ms;
          cursor: pointer;

          i {
            margin-left: 10px;
            cursor: pointer;
            font: 400 14px tahoma;
            display: inline-block;
            height: 100%;
            vertical-align: middle;
          }

          &:hover {
            color: #28a3ef;
          }
        }
      }
    }

    .details {
      margin-bottom: 5px;

      .sui-navbar {
        overflow: visible;
        margin-bottom: 0;

        .filter {
          min-height: 40px;
          padding-right: 20px;
          background: #fbfbfb;
          border: 1px solid #e2e2e2;
          padding-left: 0;
          border-radius: 0;
          box-shadow: 0 1px 4px rgba(0, 0, 0, 0.065);

          .sui-nav {
            position: relative;
            left: 0;
            display: block;
            float: left;
            margin: 0 10px 0 0;

            li {
              float: left;
              line-height: 18px;

              a {
                display: block;
                cursor: pointer;
                padding: 11px 15px;
                color: #777;
                text-decoration: none;
              }

              &.active {
                a {
                  background: #e1251b;
                  color: #fff;
                }
              }
            }
          }
        }
      }

      .goods-list {
        margin: 20px 0;

        ul {
          display: flex;
          flex-wrap: wrap;

          li {
            height: 100%;
            width: 20%;
            margin-top: 10px;
            line-height: 28px;

            .list-wrap {
              .p-img {
                padding-left: 15px;
                width: 215px;
                height: 255px;

                a {
                  color: #666;

                  img {
                    max-width: 100%;
                    height: auto;
                    vertical-align: middle;
                  }
                }
              }

              .price {
                padding-left: 15px;
                font-size: 18px;
                color: #c81623;

                strong {
                  font-weight: 700;

                  i {
                    margin-left: 0px;
                  }
                }
              }

              .attr {
                padding-left: 15px;
                width: 85%;
                overflow: hidden;
                margin-bottom: 8px;
                min-height: 38px;
                cursor: pointer;
                line-height: 1.8;
                display: -webkit-box;
                -webkit-box-orient: vertical;
                -webkit-line-clamp: 2;

                a {
                  color: #333;
                  text-decoration: none;
                }
              }

              .commit {
                padding-left: 15px;
                height: 22px;
                font-size: 13px;
                color: #a7a7a7;

                span {
                  font-weight: 700;
                  color: #646fb0;
                }
              }

              .operate {
                padding: 12px 15px;

                .sui-btn {
                  display: inline-block;
                  padding: 2px 14px;
                  box-sizing: border-box;
                  margin-bottom: 0;
                  font-size: 12px;
                  line-height: 18px;
                  text-align: center;
                  vertical-align: middle;
                  cursor: pointer;
                  border-radius: 0;
                  background-color: transparent;
                  margin-right: 15px;
                }

                .btn-bordered {
                  min-width: 85px;
                  background-color: transparent;
                  border: 1px solid #8c8c8c;
                  color: #8c8c8c;

                  &:hover {
                    border: 1px solid #666;
                    color: #fff !important;
                    background-color: #666;
                    text-decoration: none;
                  }
                }

                .btn-danger {
                  border: 1px solid #e1251b;
                  color: #e1251b;

                  &:hover {
                    border: 1px solid #e1251b;
                    background-color: #e1251b;
                    color: white !important;
                    text-decoration: none;
                  }
                }
              }
            }
          }
        }
      }
    }
  }
}
</style>