<template>
  <div class="type-nav">
    <div class="container">
      <div
        @mouseleave="
          leaveIndex();
          dontShowInSearch();
        "
        @mouseenter="showInSearch"
      >
        <h2 class="all">全部商品分类</h2>
        <transition name="sort">
          <div class="sort" v-show="isSearch">
            <div class="all-sort-list2">
              <div
                class="item"
                v-for="(c1, index) in categorylist"
                :key="c1.categoryId"
                @click.prevent="goSearch"
              >
                <h3
                  @mouseenter="changeIndex(index)"
                  :class="{ cur: index === currentIndex }"
                >
                  <a
                    href=""
                    :data-categoryName="c1.categoryName"
                    :data-category1Id="c1.categoryId"
                    >{{ c1.categoryName }}</a
                  >
                </h3>
                <div
                  class="item-list clearfix"
                  :style="{
                    display: currentIndex === index ? 'block' : 'none',
                  }"
                >
                  <div class="subitem">
                    <dl
                      class="fore"
                      v-for="c2 in c1.categoryChild"
                      :key="c2.categoryId"
                    >
                      <dt>
                        <a
                          href=""
                          :data-categoryName="c2.categoryName"
                          :data-category2Id="c2.categoryId"
                          >{{ c2.categoryName }}</a
                        >
                      </dt>
                      <dd>
                        <!-- 在需要遍历的标签上使用v-for！ -->
                        <em v-for="c3 in c2.categoryChild" :key="c3.categoryId">
                          <a
                            href=""
                            :data-categoryName="c3.categoryName"
                            :data-category3Id="c3.categoryId"
                            >{{ c3.categoryName }}</a
                          >
                        </em>
                      </dd>
                    </dl>
                  </div>
                </div>
              </div>
            </div>
          </div>
        </transition>
      </div>
      <nav class="nav">
        <a href="###">服装城</a>
        <a href="###">美妆馆</a>
        <a href="###">shopping mall超市</a>
        <a href="###">全球购</a>
        <a href="###">闪购</a>
        <a href="###">团购</a>
        <a href="###">有趣</a>
        <a href="###">秒杀</a>
      </nav>
    </div>
  </div>
</template>

<script>
import throttle from "lodash/throttle";
import { computed, onMounted, ref } from "vue";
import { useStore } from "vuex";
import { useRoute, useRouter } from "vue-router";
export default {
  name: "TypeNav",
  setup() {
    const store = useStore();
    // 1、向服务器发送请求，获得展示商品数据——为避免重复发送请求，移动到app.vue中
    // 1-1 使用computed保持响应性
    let categorylist = computed(() =>
      // 1-2 使用slice()是因为页面样式已定，超过16个会影响排版
      store.state.typenav.categorylist.slice(0, 15)
    );
    // 2、通过js改变背景颜色
    let currentIndex = ref(-1);
    // 2-1 引入lodash进行节流操作
    let changeIndex = throttle(function (index) {
      currentIndex.value = index;
    }, 50);
    // 2-2 移开鼠标时恢复样式
    function leaveIndex() {
      currentIndex.value = -1;
    }
    // 3、路由跳转至search页面
    let router = useRouter();
    function goSearch(event) {
      let element = event.target;
      // 3-1 获得当前标签的自定义属性
      let { categoryname, category1id, category2id, category3id } =
        element.dataset;
      if (categoryname) {
        let location = { name: "Search" };
        let query = { categoryName: categoryname };
        if (category1id) {
          query.category1Id = category1id;
        } else if (category2id) {
          query.category2Id = category2id;
        } else if (category3id) {
          query.category3Id = category3id;
        }
        // 3-2 将搜索框输入的内容也一并传输
        if (route.params) {
          location.params = route.params;
        }
        location.query = query;
        router.push(location);
      }
    }
    // 4、在搜索页面显示typenav组件
    let route = useRoute();
    let isSearch = ref(route.meta.isShow); // computed获得的值不能直接修改，除非设定setter
    // 4-1 进入搜索页面默认不显示
    onMounted(() => {
      // 4-1-1 判断是否为搜索页面
      if (route.path.match(/^\/search/) || route.path.match(/^\/detail/)) {
        isSearch.value = false;
      }
    });
    // 4-2 鼠标移入时
    function showInSearch() {
      if (route.path.match(/^\/search/) || route.path.match(/^\/detail/)) {
        isSearch.value = true;
      }
    }
    // 4-3 鼠标移出时
    function dontShowInSearch() {
      if (route.path.match(/^\/search/) || route.path.match(/^\/detail/)) {
        isSearch.value = false;
      }
    }
    return {
      categorylist,
      currentIndex,
      changeIndex,
      leaveIndex,
      goSearch,
      isSearch,
      showInSearch,
      dontShowInSearch,
    };
  },
};
</script>

<style lang="less" scoped>
.type-nav {
  border-bottom: 2px solid #d4237a;

  .container {
    width: 1200px;
    margin: 0 auto;
    display: flex;
    position: relative;

    .all {
      width: 210px;
      height: 45px;
      background-color: #d4237a;
      line-height: 45px;
      text-align: center;
      color: #fff;
      font-size: 14px;
      font-weight: bold;
    }

    .nav {
      a {
        height: 45px;
        margin: 0 22px;
        line-height: 45px;
        font-size: 16px;
        color: #333;
      }
    }

    .sort {
      position: absolute;
      left: 0;
      top: 45px;
      width: 210px;
      height: 461px;
      position: absolute;
      background: #fafafa;
      z-index: 999;

      .all-sort-list2 {
        .item {
          h3 {
            line-height: 30px;
            font-size: 14px;
            font-weight: 400;
            overflow: hidden;
            padding: 0 20px;
            margin: 0;

            a {
              color: #333;
            }
          }
          .cur {
            background-color: #ffb5d9;
          }

          .item-list {
            display: none;
            position: absolute;
            width: 734px;
            min-height: 460px;
            background: #f7f7f7;
            left: 210px;
            border: 1px solid #ddd;
            top: 0;
            z-index: 9999 !important;

            .subitem {
              float: left;
              width: 650px;
              padding: 0 4px 0 8px;

              dl {
                border-top: 1px solid #eee;
                padding: 6px 0;
                overflow: hidden;
                zoom: 1;

                &.fore {
                  border-top: 0;
                }

                dt {
                  float: left;
                  width: 54px;
                  line-height: 22px;
                  text-align: right;
                  padding: 3px 6px 0 0;
                  font-weight: 700;
                }

                dd {
                  float: left;
                  width: 415px;
                  padding: 3px 0 0;
                  overflow: hidden;

                  em {
                    float: left;
                    height: 14px;
                    line-height: 14px;
                    padding: 0 8px;
                    margin-top: 5px;
                    border-left: 1px solid #ccc;
                  }
                }
              }
            }
          }

          // 改用js
          // &:hover {
          //   .item-list {
          //     display: block;
          //   }
          // }
        }
      }
    }

    .sort-enter-from,
    .sort-leave-to {
      height: 0px;
      opacity: 0;
    }
    .sort-enter-to,
    .sort-leave-from {
      height: 461px;
      opacity: 1;
    }
    .sort-enter-active,
    .sort-leave-active {
      transition: all 0.2s linear;
    }
  }
}
</style>