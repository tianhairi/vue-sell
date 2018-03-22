<template>
  <div class="goods">
      <div class="menu-wrapper" v-el:menu-wrapper>
          <ul>
              <li class="menu-item" v-for="item in goods" :class="{'current':currentIndex===$index}" @click="_scrollTo($index,$event)">
                  <span class="text border-bottom-1px">
                      <span v-show="item.type>0" class="icon" :class="classMap[item.type]"></span>{{item.name}}
                  </span>
              </li>
          </ul>
      </div>
      <div class="foods-wrapper" v-el:foods-wrapper>
          <ul>
              <li class="food-item food-list-hook" v-for="item in goods">
                  <div class="item-title">{{item.name}}</div>
                  <div class="item-wrapper border-bottom-1px" v-for="foodItem in item.foods" @click="selectFood(foodItem,$event)">
                      <div class="icon">
                          <img :src="foodItem.icon">
                      </div>
                      <div class="content">
                          <div class="food-title">{{foodItem.name}}</div>
                          <div v-show="foodItem.description" class="des">{{foodItem.description}}</div>
                          <div class="message">
                              <span v-show="foodItem.sellCount" class="food-sellCount">月售{{foodItem.sellCount}}份</span>
                              <span v-show="foodItem.rating" class="food-rating">好评率{{foodItem.rating}}%</span>
                          </div>
                          <div class="food-price">
                              <span class="price">¥{{foodItem.price}}</span>
                              <span v-show="foodItem.oldPrice" class="oldPrice">¥{{foodItem.oldPrice}}</span>
                          </div> 
                      </div>
                      <div class="cartcontrol-wrapper">
                        <cartcontrol :food="foodItem"></cartcontrol>
                      </div>
                  </div>
              </li>
          </ul>
      </div>
      <shopcart v-ref:shopcart :select-foods="selectFoods" :delivery-price="seller.deliveryPrice" :min-price="seller.minPrice"></shopcart>
  </div>
    <food :food="selectedFood" v-ref:food></food>
</template>
<script>
    import BScroll from 'better-scroll';
    import shopcart from '../shopcart/shopcart';
    import cartcontrol from 'components/cartcontrol/cartcontrol';
    import food from 'components/food/food';
    const ERR_OK = 0;
    export default{
        props: {
            seller: {
                type: Object
            }
        },
        data() {
            return {
                goods: [],
                listHeight: [],
                scrolly: 0,
                selectedFood: {}
            };
        },
        computed: {
            currentIndex() {
                for (let i = 0; i < this.listHeight.length; i++) {
                    let height1 = this.listHeight[i];
                    let height2 = this.listHeight[i + 1];
                    if (!height2 || this.scrolly >= height1 && this.scrolly < height2) {
                        return i;
                    }
                }
                return 0;
            },
            selectFoods() {
                let selectFoods = [];
                this.goods.forEach((good) => {
                    good.foods.forEach((food) => {
                        if (food.count) {
                            selectFoods.push(food);
                        }
                    });
                });
                return selectFoods;
            }
        },
        created() {
            this.$http.get('/api/goods').then((response) => {
                response = response.body;
                if (response.errno === ERR_OK) {
                    this.goods = response.data;
                    this.$nextTick(() => {
                        this._initScroll();
                        this._calculate();
                    });
                }
            });
            this.classMap = [ 'decrease', 'discount', 'special', 'invoice', 'guarantee' ];
        },
        methods: {
            _initScroll() {
                this.menuWrapper = new BScroll(this.$els.menuWrapper, {
                    click: true
                });
                this.foodsWrapper = new BScroll(this.$els.foodsWrapper, {
                    click: true,
                    probeType: 3
                });
                this.foodsWrapper.on('scroll', (pos) => {
                    this.scrolly = Math.abs(Math.floor(pos.y));
                });
            },
            _calculate() {
                let classList = this.$els.foodsWrapper.getElementsByClassName('food-list-hook');
                let height = 0;
                this.listHeight.push(height);
                for (let i = 0; i < classList.length; i++) {
                    let item = classList[i].clientHeight;
                    height += item;
                    this.listHeight.push(height);
                };
            },
            _scrollTo(index, event) {
                if (!event._constructed) {
                    return;
                }
                let classList = this.$els.foodsWrapper.getElementsByClassName('food-list-hook');
                let el = classList[index];
                this.foodsWrapper.scrollToElement(el, 500);
            },
            _drop(target) {
                // 体验优化 异步执行
                this.$nextTick(() => {
                     this.$refs.shopcart.drop(target);
                });
            },
            selectFood(foodItem, event) {
                if (!event._constructed) {
                    return;
                }
                this.selectedFood = foodItem;
                this.$refs.food.show();
            }
        },
        components: {
            shopcart,
            cartcontrol,
            food
        },
        events: {
           'cart.add'(target) {
               this._drop(target);
            }
        }
    };
</script>
<style lang="stylus" rel="stylesheet/stylus">

    @import '../../common/stylus/mixin'

    .goods
        display: flex
        flex-direction: row
        position: absolute
        top: 174px
        bottom: 46px
        width: 100%
        overflow: hidden
        .menu-wrapper
            width: 80px
            background: #f3f5f7
            .menu-item
                width: 56px
                height: 54px
                padding: 0 12px
                display: table
                font-size: 0
                &.current
                    position relative
                    margin-top: -1px
                    background: #fff
                    .text
                        font-weight: 700
                        &:after
                            border: none
                .text
                    font-size: 12px
                    line-height: 14px
                    display: table-cell
                    vertical-align: middle
                    width: 56px
                    color: rgb(7,17,27)
                    border-bottom-1px(rgba(7,17,27,0.1))
                    .icon
                        display: inline-block
                        width: 12px
                        height: 12px
                        margin-right: 2px
                        background-size: 12px 12px 
                        background-repeat: no-repeat
                        vertical-align: top
                        &.decrease
                            bg-image('decrease_3')
                        &.discount
                            bg-image('discount_3')
                        &.guarantee
                            bg-image('guarantee_3')
                        &.invoice
                            bg-image('invoice_3')
                        &.special
                            bg-image('special_3')
                       
        .foods-wrapper
            flex: 1
            .food-item
                width: 100%
                .item-title
                    height: 26px
                    line-height: 26px
                    background: #f3f5f7
                    font-size: 12px
                    color: rgb(147,153,159)
                    text-align: left
                    padding-left: 14px
                    border-left: 1px solid #d9dde1
                 .item-wrapper
                    margin: 0 18px
                    padding: 18px 0
                    box-sizing: border-box
                    border-bottom-1px(rgba(7,17,27,0.1))
                    display: flex
                    flex-direction: row
                    &:last-child
                        &:after
                            border: 0
                    .icon
                        height: 56px
                        width: 56px
                        margin-right: 10px
                        img 
                            height: 56px
                            width: 56px
                    .content
                        flex: 1
                        .food-title
                            font-size: 14px
                            color: rgb(7,17,27)
                            line-height: 14px
                            padding-top: 2px
                        .des
                            font-size: 10px
                            color: rgb(147,153,159)
                            margin-top: 8px
                            line-height: 10px
                        .message
                            margin: 8px 0
                            font-size: 0
                            .food-sellCount,
                            .food-rating
                                font-size: 10px
                                color: rgb(147,153,159)
                                margin: 8px 0
                                line-height: 10px
                            .food-sellCount
                                margin-right: 8px
                        .food-price
                            font-size: 0
                            line-height: 16px;
                            height: 16px;
                            .price
                                font-size: 14px
                                line-height: 16px
                                font-weight: 700
                                color: rgb(240,20,20)
                                margin-right: 8px
                            .oldPrice
                                font-size: 10px
                                line-height: 16px
                                font-weight: 700
                                color: rgb(147,153,159)
                    .cartcontrol-wrapper
                        position: absolute
                        right: 0
                        bottom: 12px


</style>