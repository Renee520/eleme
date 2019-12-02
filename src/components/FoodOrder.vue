<template>
  <div :class="prefixCls">
      <!-- 推荐商家 -->
      <div :class="`${prefixCls}-recommend`">
        <h4 :class="`${prefixCls}-recommend__title`">商家推荐</h4>
        <div :class="`${prefixCls}-recommend__content`">
          <div class="recommend-item" v-for="(item, index) in selectRecommendFood(foodList)" :key="index">
            <div class="recommend-item__inner" v-if="item.recommend">
              <img @click="showFoodPanel(item)" src="../assets/imgs/food/recommend.webp" />
              <div @click="showFoodInfo(item)">
                <p class="food-title text-overflow">{{ item.name }}</p>
                <p class="small">月售{{ item.monthSales }} 好评率{{ item.rate }}%</p>
                <div class="footer flex">
                  <p class="price">￥<span class="large">{{ item.price }}</span></p>
                  <div
                    @click.stop=""
                    v-if="item.quantity"
                    :class="['add-goods', item.count===0 ? 'noGoods' : '']">
                    <van-stepper
                      v-model="item.count"
                      :min="0"
                      :max="item.quantity"
                      :disable-input="true"
                      @plus="addFoods(item)"
                      @minus="minusFoods(item)"
                    />
                  </div>
                  <span v-else class="sold-out">已售完</span>
                </div>
              </div>
            </div>
          </div>
          <van-popup
            v-model="showFoodPanelFlag"
            position="bottom"
            closeable
            close-icon="cross"
          >
            <div class="food-panel flex">
              <div class="header flex">
                <img src="../assets/imgs/food/recommend.webp" alt="">
                <div class="info">
                  <p class="title bold">{{ foodPanel.name }}</p>
                  <p class="select">已选：</p>
                  <p class="price">￥<span class="large">{{ foodPanel.price }}</span></p>
                </div>
              </div>
              <button @click="addFoodsInFoodPanel(foodPanel)" class="v-button v-button--block v-button--primary bold">选好了</button>
            </div>
          </van-popup>
        </div>
      </div>
      <!-- 食物列表 -->
      <van-sticky ref="container" :class="`${prefixCls}-content`" :offset-top="44">
        <div class="inner-container">
          <!-- 侧边导航 -->
          <van-sidebar :container="container" class="foodMenu-sidebar" v-model="menuActive">
            <van-sidebar-item
              v-for="(item, i) in foodList"
              :key="i"
              :title="item.name"
              @click="clickToChangeSidebar"
              :info="item.menuCount"
              :class="item.menuCount===0 ? 'menuTag-hidden' : 'menu-tag'"
            />
          </van-sidebar>
          <!-- 食物列表 -->
          <div :container="container" ref="foodList" class="food-list">
            <div class="foodmenu" v-for="(item, index) in foodList" :key="index">
              <p ref="foodMenuItem" class="foodmenu-title">
                <span class="foodmenu-title__tag bold">{{ item.name }}</span>{{ item.description }}
              </p>
              <div class="foodmenu-content">
                <section v-for="(innerItem, i) in item.foods" :key="i" @click.stop="showFoodInfo(innerItem)" class="foodmenu-content__item flex">
                  <div class="img">
                    <img src="../assets/imgs/food/recommend.webp" alt="">
                    <div class="sign" v-if="innerItem.hotLevel">招牌</div>
                  </div>
                  <div class="content">
                    <h4 class="title text-overflow">{{ innerItem.name }}</h4>
                    <p class="info text-overflow">{{ innerItem.description }}</p>
                    <p class="info">
                      <span class="sale">月售200份</span>
                      <span v-if="innerItem.rate" class="rate">好评率100%</span>
                    </p>
                    <p>
                      <span class="quantity" v-if="innerItem.quantity<10">剩{{ innerItem.quantity}}份</span>
                    </p>
                    <span class="price">￥{{ innerItem.price }}</span>
                    <div
                      @click.stop=""
                      v-if="innerItem.quantity"
                      :class="['add-goods', innerItem.count===0 ? 'noGoods' : '']">
                      <van-stepper
                        v-model="innerItem.count"
                        :disable-input="true"
                        :min="0"
                        :max="innerItem.quantity"
                        @plus="addFoods(innerItem)"
                        @minus="minusFoods(innerItem)"
                      />
                    </div>
                    <span v-else class="sold-out">已售完</span>
                  </div>
                </section>
              </div>
            </div>
            <!-- 食物详情 -->
            <transition name="popup">
              <div class="foodInfo-popup box" v-if="showFoodInfoPopup">
                <div class="closeFoodInfoPopup" @click="closeFoodInfoPopup">
                  <van-icon name="cross" />
                </div>
                <img src="../assets/imgs/food/recommend.webp" alt="">
                <div class="text-container">
                  <p class="title text-overflow bold">
                    <span class="sign" v-if="popupFoodInfo.hotLevel">招牌</span>{{ popupFoodInfo.name }}
                  </p>
                  <p class="sale">
                    <span>月售{{ popupFoodInfo.monthSales }}份</span>
                    <span>好评率{{ popupFoodInfo.rate }}%</span>
                  </p>
                  <p class="price-line flex">
                    <span class="price">￥{{ popupFoodInfo.price }}</span>
                    <van-stepper
                      v-model="popupFoodInfo.count"
                      v-if="popupFoodInfo.quantity"
                      :min="0"
                      :max="popupFoodInfo.quantity"
                      :disable-input="true"
                      :class="['add-goods', popupFoodInfo.count===0 ? 'noGoods' : '']"
                      @plus="addFoods(popupFoodInfo)"
                      @minus="minusFoods(popupFoodInfo)"
                    />
                  </p>
                  <p class="description">{{ popupFoodInfo.description }}</p>
                  <p class="price-notice">价格说明<van-icon name="question-o" /></p>
                </div>
              </div>
            </transition>
          </div>
        </div>
      </van-sticky>
      <!-- 购物车 -->
      <div :class="`${prefixCls}-footer`">
        <div @click="showCarPopup" :class="`${prefixCls}-footer__discount-tip text--center`" v-if="foodsCount">
          已减去{{ (totalPrice - payMoney) | toFixed }}元
        </div>
        <van-action-sheet v-model="popupShopCarList" :round="false" @click-overlay="closeCarPopup" :close-on-click-overlay="true" :lock-scroll="false">
          <p class="header">已选商品
            <span class="float--right small" @click.stop="clearShopCar">
              <van-icon name="delete" />清空
            </span>
          </p>
          <ul class="food-list">
            <li v-for="(item, index) in foods" :key="index" :class="[item.count ? '' : 'display-none', 'item']">
              <span class="name text-overflow">{{ item.name }}</span>
              <span class="price bold">
                <span class="small">￥</span>
                {{ item.price * item.count | toFixed }}
              </span>
              <van-stepper
                v-model="item.count"
                :min="0"
                :max="item.quantity"
                :disable-input="true"
                class="add-goods"
                @plus="addFoods(item)"
                @minus="minusFoods(item)"
              />
            </li>
          </ul>
        </van-action-sheet>
        <div @click="showCarPopup" :class="`${prefixCls}-footer__bottomNav flex`">
          <div class="shopcar  text--center">
            <div :class="['inner', foodsCount===0 ? '' : 'hasfood']">
              <van-icon name="shopping-cart" />
            </div>
            <van-tag round type="danger" :class="foodsCount===0 ? 'display-none' : ''">{{foodsCount}}</van-tag>
          </div>
          <div class="info">
            <p v-if="foodsCount===0">未选购商品</p>
            <p class="total-price" v-else>￥{{ payMoney | toFixed }}<span class="line-through">￥{{ totalPrice | toFixed }}</span></p>
            <p class="send-price">另需配送费2.5元</p>
          </div>
          <div class="button">
            <button v-if="foodsCount===0" class="noGoods">￥0起送</button>
            <button v-else class="settlement">去结算</button>
          </div>
        </div>
      </div>
  </div>
</template>
<script>
const prefixCls = 'foodOrder-index';
export default {
  data() {
    return {
      prefixCls,
      menuActive: 0, // 默认选中菜单项
      showFoodInfoPopup: false, // 显示食物详情页
      showFoodPanelFlag: false, // 控制 点击“商家推荐”里的图片 的弹出层
      foodPanel: {}, // “商家推荐”里的图片的弹出层渲染的foodItem
      timer: null,
      scrollTop: 0, // 滚轮滚动距离
      targetHeight: 0, // 目标元素高度
      count: 50, // 计时器
      pace: 0, // 步长
      container: null,
      itemHeight: null,
      foodsCount: 0, // 购物车数量
      totalPrice: 0, // 购物车总价
      payMoney: 0, // 实际应付价格（满减后需付的总价）
      popupShopCarList: false, // 弹出购物车清单
      foods: [], // 当前商家中选中的所有食物
      allFoods: [], // 所有商家中选中的所有食物
      haveFoodsInCurrStore: false, // 是否存过当前商家中的食物
    };
  },
  props: ['foodListFromFood', 'reduceActivities'],
  computed: {
    foodList: { // 从后台获取当前商家所有食物数据
      get() {
        this.foodListFromFood.map(item => { // 遍历添加属性：食物选中数量count，每个菜单中选中的食物总数menuCount
          item.menuCount = 0;
          item.foods.map(innerItem => {
              innerItem.count = 0;
              return innerItem;
          })
        });
        return this.foodListFromFood;
      },
      set() {},
    },
  },
  filters: {
    toFixed(value) { // 精确到小数点后两位
      return value.toFixed(2)
    },
  },
  watch: {
    count(newVal) {
      if (newVal === 0) {
        clearInterval(this.timer);
      }
    },
    totalPrice(newVal, oldVal) { // 总价改变，检查是否符合满减条件
      if (!this.reduceActivities) {
        return;
      }
      let obj = this.reduceActivities.attribute,
          arr = Object.keys(obj);
      for (let i=0; i<arr.length; i++) {
        /* 满减：
            总价 < 最低满减 ------ 应付 =  总价
            (满减[i] < 总价 < 满减[i+1]) || (总价 > 最高满减) ------ 应付 =  总价 - 第i个满减
        */
        if (i===arr.length - 1 || (newVal >= arr[i] && newVal < arr[i+1])) {
          this.payMoney = newVal - obj[arr[i]];
          break;
        } else if (newVal < arr[0]) {
          this.payMoney = newVal;
          break;
        }
      }
    },
  },
  methods: {
    showFoodPanel(item) { // 点击“商家推荐”item图片 弹出panel
      this.showFoodPanelFlag = true;
      this.foodPanel = item;
    },
    getTargetHeight(i) { // 获取目标元素高度（距离foodList的高度）
      return this.$refs.foodMenuItem[i].offsetTop - 8;
    },
    clickToChangeSidebar(index) { // 点击改变侧边导航选中项
      clearInterval(this.timer);
      this.count = 50;
      // 页面滚动距离 = 元素本身的位置（距离食物列表顶部的高度） + 食物列表（距离document顶部）的高度 - 顶部固定导航的高度
      this.targetHeight = this.getTargetHeight(index); // 获取目标元素高度
      this.scrollTop = this.getScrollTop(); // 获取食物列表滚动高度
      this.pace = (this.targetHeight - this.scrollTop) / 50;
      let that = this;
      if (this.pace > 0) { // 滚动条高度不够，需要向上滚
        that.timer = setInterval(() => {
          that.smoothUp();
          that.count --;
        }, 10);
      } else { // 向下滚
        that.timer = setInterval(() => {
          that.smoothDown();
          that.count --;
        }, 10);
      }
    },
    scrollChangeMenu () { // 滚动改变菜单选项
      /*
          监听滚动：
            遍历菜单，对比目标元素i和滚动条高度，
            如果 scroll >= 元素i.offsetTop，则第i个菜单选中，把i传给menuActive
       */
      this.scrollTop = this.getScrollTop(); // 获取食物列表滚动高度
      let that = this;
      this.foodList.forEach((item, index) => { // 遍历寻找滚动区域对应的目标元素
        that.itemHeight = this.getTargetHeight(index);
        if (that.itemHeight <= that.scrollTop) {
          that.menuActive = index;
        }
      });
    },
    smoothUp() { // 向上滚动（当滚动距离低于目标元素高度）
      if (this.scrollTop < this.targetHeight) {
        this.scrollTop += this.pace;
        this.$refs.foodList.scrollTop = this.scrollTop;
      } else {
        this.$refs.foodList.scrollTop = this.targetHeight;
      }
    },
    smoothDown() { // 向下滚动（当滚动距离高于目标元素高度）
      if (this.scrollTop > this.targetHeight) {
        this.scrollTop += this.pace;
       this.$refs.foodList.scrollTop = this.scrollTop;
      } else {
        this.$refs.foodList.scrollTop = this.targetHeight;
      }
    },
    changeMenuItem(index) { // 改变食物菜单选中项
      this.menuActive = index
    },
    showFoodInfo(foodItem) { // 显示“食物详情”
      this.popupFoodInfo = foodItem;
      this.showFoodInfoPopup = true;
    },
    closeFoodInfoPopup() { // 关闭“食物详情”
      this.showFoodInfoPopup = false;
    },
    getScrollTop() { // 获取滚动高度
      return this.$refs.foodList.scrollTop;
    },
    selectRecommendFood(foodList) { // 选出foodList里的“推荐商家”
      let recommendFood = [];
      foodList.forEach(item => {
        item.foods.forEach(innerItem => {
          if (innerItem.recommend) {
            recommendFood.push(innerItem);
          }
        })
      })
      return recommendFood;
    },
    addFoods(food) { // 添加商品
      if (food.count === food.quantity) {
        this.$toast('所选商品已达上限');
        return;
      }
      food.count ++;
      this.totalPrice += food.price;
      this.foodsCount ++;
      let hasSavedFlag = false; // hasSavedFlag: 之前是否添加过该商品
      // 更新当前商家购物车数据
      if (this.foods.length) { // 如果购物车不为空
        this.foods.map(item => {
          if (food._id === item._id) { // 添加过
            hasSavedFlag = true;
            return item;
          }
        })
        if (!hasSavedFlag) { // 没添加
          this.foods.push(food);
        }
      } else { // 如果购物车为空，添加该商品
        this.foods.push(food);
      }
      // 更新总购物车数据，然后存入store和本地
      if (!this.haveFoodsInCurrStore) { // 如果之前当前商家购物车为空
        // 存入当前商家id和当前购物车
        this.allFoods.push({id: this.$route.params.id, foods: [this.foods]});
        this.haveFoodsInCurrStore = true;
      } else { // 如果之前当前商家购物车不为空，更新当前购物车
        this.upDateAllFoods(this.foods);
      }
      this.$store.dispatch('saveFoods', this.allFoods).then(
        () => {
          localStorage.setItem('foods', JSON.stringify(this.allFoods));
        },
        err => {
          console.log(err.message || '失败');
        }
      );
      this.foodList = this.foodList.map(item => { // 遍历食物菜单，让食物菜单的右上标签++
        let flag = item.foods.findIndex(innerItem => {
          return innerItem._id === food._id;
        })
        if (flag >= 0) {
          item.menuCount ++;
        }
        return item;
      })
    },
    minusFoods(food) { // 去掉商品
      if (this.foodsCount > 0) { // 购物车中有商品才能删除
        food.count --;
        this.totalPrice -= food.price;
        this.foodsCount --;
        if (this.foodsCount === 0) { // 购物车删空了
          this.clearFoods();
        } else { // 购物车中还有数据
          let index, minusFlag = false; // index表示当前减去的食物索引，minusFlag表示食物数量是否减为0
          this.foods.map((item, i) => {
            if (item.count === 0) {
              minusFlag = true;
              index = i;
            }
          })
          if (minusFlag) { // 如果当前食物选中数量减为0，就从购物车中删去
            this.foods.splice(index, 1);
          }
        }
        this.upDateAllFoods(this.foods);
        this.$store.dispatch('saveFoods', this.allFoods).then(
          () => {
            localStorage.setItem('foods', JSON.stringify(this.allFoods));
          },
          err => {
            console.log(err);
          }
        )
        this.foodList.map(item => {
          let flag = item.foods.findIndex(innerItem => {
            return food._id === innerItem._id;
          })
          if (flag >= 0 && item.menuCount > 0) {
            item.menuCount --;
          }
          return item;
        })
      }
      
    },
    upDateAllFoods(foods) { // 更新总购物车
      this.allFoods.map((item, i) => {
        if (item.id === this.$route.params.id) { // 找到当前商家
          if (this.foods.length === 0) {
            index = i;
          }
          item = this.foods;
        }
        return item;
      })
    },
    showCarPopup() { // 点击切换显示购物车弹框
      if (this.foodsCount) {
        this.popupShopCarList = !this.popupShopCarList;
      }
    },
    clearFoods() { // 找到当前商家购物车，从总购物车中删去当前商家购物车
      this.foods = [];
      this.foodsCount = 0;
      this.haveFoodsInCurrStore = false;
      let index = this.allFoods.findIndex(item => {
        return item.id === this.$route.params.id;
      })
      this.allFoods.splice(index, 1);
    },
    clearShopCar() { // 清空购物车
      this.foodList.map(item => { // 食物列表项count = 0
        item.menuCount = 0;
        item.foods.map(innerItem => {
          innerItem.count = 0;
          return innerItem;
        })
        return item;
      })
      this.popupShopCarList = false;
      this.clearFoods();
      this.$store.dispatch('saveFoods', this.allFoods).then(
        () => {
          localStorage.setItem('foods', JSON.stringify(this.allFoods));
        },
        err => {
          console.log(err);
        }
      )
    },
    closeCarPopup() { // 点击购物车弹框遮罩层，关闭弹框
      this.popupShopCarList = false;
    },
    addFoodsInFoodPanel(food) { // 点击foodPanel弹框的“选好了”，添加商品，并关闭弹框
      this.addFoods(food);
      this.showFoodPanelFlag = false;
    },
  },
  created() {
    let store = this.$store.state.order.foods; // 拿到store中的foods
    if (store && store.length) { // 如果不为空
      this.allFoods = store;
      this.allFoods.some(item => {
        if (item.id === this.$route.params.id) { // 如果store之前存过当前商家的食物
          this.haveFoodsInCurrStore = true;
          this.foods = item.foods; // 存储当前商家所有选中食物和食物选中总数
          this.foodList.map(foodListItem => { // 遍历从后台获取的所有的食物列表
            let count = 0;
            foodListItem.foods.map(innerItem => { // 遍历每个menu对应的食物组
              this.foods.forEach(food => { // 遍历当前商家中选中的所有食物
                if (food._id === innerItem._id) {// 如果当前这道菜被选中过
                  innerItem.count = food.count; // 更新当前这道菜的count
                  count += food.count; // 将食物数量累加到当前menu的count上
                  this.totalPrice += food.price * food.count; // 当前食物 总价 * 总数 累加到购物车总价上
                }
              })
              return innerItem;
            })
            if (count) { // 如果当前菜单中有食物被选中
              foodListItem.menuCount = count; // 当前menu中有多少道食物被选中
              this.foodsCount += count; // 当前menu中的count累加到购物车总数上
            }
            return foodListItem;
          })
        }
      })
    }
  },
  mounted() {
    this.container = this.$refs.container;
    this.$refs.foodList.addEventListener('scroll', this.scrollChangeMenu); // 监听食物滚动，改变对应菜单选中项
  },
}
</script>
