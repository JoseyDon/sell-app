<template>
  <div class="goods">
      <div class="menu-wrapper" ref="menuWrapper">
        <ul>
          <li v-for="(item,index) in goods" @click="menuClick(index,$event)" :class="index==menuCurrentIndex?'menu-item-selected':'menu-item'">
            <span class="text border-1px">
              <span v-show="item.type>0" class="icon" :class="iconclassMap[item.type]"></span>
              {{item.name}}
            </span>
          </li>
        </ul>
      </div>
      <div class="foods-wrapper" ref="foodsWrapper">
        <ul>
          <li v-for="item in goods" class="food-list food-list-hook">
            <h1 class="title">{{item.name}}</h1>
            <ul>
              <li v-for="food in item.foods" class="food-item border-1px" @click="selectFood(food,$event)">
                <div class="icon">
                  <img width="57px" height="57px" :src="food.icon">
                </div>
                <div class="content">
                  <h2 class="name">{{food.name}}</h2>
                  <p class="desc">{{food.description}}</p>
                  <div class="extra">
                    <span class="count">月售{{food.sellCount}}份</span><span>好评率{{food.rating}}%</span>
                  </div>
                  <div class="price">
                    <span class="now">￥{{food.price}}</span><span class="old" v-show="food.oldPrice">￥{{food.oldPrice}}</span>
                  </div>
                   <div class="carcontrol-wrapper">
                     <cartcontrol :food="food" @increment="_drop"></cartcontrol>
                   </div>
                </div>
              </li>
            </ul>
          </li>
        </ul>
      </div>
      <div>
        <shopcar :select-foods="selectFoods" ref="shopcar" :delivery-price="seller.deliveryPrice" :min-price="seller.minPrice"></shopcar>
        <food :food="selectedFood" ref="food"></food>
    </div>
      </div>
</template>

<script>
import BScroll from 'better-scroll';
import shopcar from './shopcar';
import cartcontrol from './cartcontrol';
import food from '../food/food';

const ERR_OK = 0;
export default {
  props: {
    seller: {
      type: Object
    }
  },
  data() {
    return {
      goods: [],
      listHeight: [],
      scrollY: 0,
      selectedFood: {}
    };
  },
  computed: {
    menuCurrentIndex() {
      for (let i = 0; i < this.listHeight.length; i++) {
        let height1 = this.listHeight[i];
        let height2 = this.listHeight[i + 1];
        if (!height2 || (this.scrollY >= height1 && this.scrollY < height2)) {
          return i;
        }
      }
      return 0;
    },
    selectFoods() {
      let foods = [];
      this.goods.forEach((good) => {
        good.foods.forEach((food) => {
          if (food.count) {
            foods.push(food);
          }
        });
      });
      return foods;
    }
  },
  created() {
    this.iconclassMap = ['decrease', 'discount', 'special', 'invoice', 'guarantee'];
    this.$http.get('/api/goods').then((response) => {
      response = response.body;
      if (response.errno === ERR_OK) {
        this.goods = response.data;
        this.$nextTick(() => {
          this._initScroll();
          this._calculateHeight();
        });
      }
    });
  },
  methods: {
    selectFood(food, event) {
      if (!event._constructed) {
        return;
      }
      this.selectedFood = food;
      this.$refs.food.show();
    },
    _drop(target) {
      this.$refs.shopcar.drop(target);
    },
    _initScroll() {
      this.meunScroll = new BScroll(this.$refs.menuWrapper, {
        click: true
      });
      this.foodsScroll = new BScroll(this.$refs.foodsWrapper, {
        click: true,
        probeType: 3
      });
      this.foodsScroll.on('scroll', (pos) => {
        this.scrollY = Math.abs(Math.round(pos.y));
      });
    },
    _calculateHeight() {
      let foodList = this.$refs.foodsWrapper.getElementsByClassName('food-list-hook');
      let height = 0;
      this.listHeight.push(height);
      for (let i = 0, l = foodList.length; i < l; i++) {
        let item = foodList[i];
        height += item.clientHeight;
        this.listHeight.push(height);
      }
    },
    menuClick(index, event) {
      if (!event._constructed) {
        return;
      }
      this.foodsScroll.scrollTo(0, -this.listHeight[index], 300);
    },
    goDetail(food) {
      this.selectedFood = food;
      this.$nextTick(() => {
        this.$refs.myFood.showToggle();
      });
    }
  },
  components: {
    shopcar,
    cartcontrol,
    food
  },
  events: {
    'car.add'(target) {
      this._drop(target);
    }
  }
};

</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped  lang="stylus" rel="stylesheet/stylus">
@import '../../common/stylus/mixin'
.goods
  display:flex
  position:absolute
  top:174px
  bottom:46px
  width:100%
  overflow:hidden
  .menu-wrapper
    flex:0 0 80px
    width:80px
    background:#f3f5f7
    .menu-item
      display:table
      height:54px
      width:56px
      padding:0 12px
      line-height:14px
    .menu-item-selected
      background:white
      font-weight:700
      margin-top:-1px
      .text
        border-none
    .menu-item,.menu-item-selected
      position:relative
      display:table
      height:54px
      line-height:14px
      width:56px
      padding:0 12px
      &:last-child:after
        content:none
    .menu-item:after
        position: absolute
        content: ''
        left: 12px
        width: 56px
        bottom: 0
        border-bottom: 1px solid rgba(7,17,27,0.1)
      .text
        display:table-cell
        vertical-align:middle
        font-size:12px
        font-weight:200
        white-space:normal
        line-height:14px
      .icon
        display:inline-block
        vertical-align:top
        width:12px
        height:12px
        margin-right:2px
        background-size:12px 12px
        background-repeat:no-repeat
      .decrease
        bg-image('decrease_3')
      .discount
        bg-image('discount_3')
      .guarantee
        bg-image('guarantee_3')
      .invoice
        bg-image('invoice_3')
      .special
        bg-image('special_3')
      .text
        display:table-cell
        width:56px
        vertical-align:middle
        line-height:12px
        border-1px:(rgba(7,17,27,0.1))
        font-size:12px
  .foods-wrapper
    flex:1
    margin-top: 2px;
    .title
      padding-left:14px
      height:26px
      line-height:26px
      border-left:2px solid #d9dde1
      font-size:12px
      color:rgb(147,153,159)
      background:#f3f5f7
    .food-item
      position relative
      display:flex
      margin:10px 18px
      padding-bottom:18px
      border-1px:(rgba(7,17,27,0.1))
      :last-child
        border-none()
        margin-bottom:0
      .icon
        flex: 0 0 57px
        margin-right:10px
      .content
        flex:1
        padding-left 10px
        .name
          margin:2px 0 8px 0
          height:14px
          line-height:14px
          font-size:14px
          color:rgb(7,14,27)
        .desc,.extra
          line-height:10px
          font-size:10px
          color:rgb(147,153,159)
        .desc
          margin-bottom:8px
          line-height:12px
        .extra
          line-height:10px
        .count
          margin-right:12px
        .price
          font-weight:700
          line-height:24px
          .now
            margin-right:8px
            font-size:14px
            color:rgb(240,20,20)
          .old
            text-decoration:line-through
            font-size:10px
            color:rgb(147,153,19)
        .carcontrol-wrapper
          position:absolute
          right:0
          bottom:12px
 </style>
