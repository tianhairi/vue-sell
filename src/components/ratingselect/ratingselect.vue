<template>
  <div>
      <div class="ratingselect">
           <div class="rating-type border-bottom-1px">
               <span @click="select(2,$event)" class="block positive" :class="{'active':selectType===2}">{{desc.all}}<span class="count">{{ratings.length}}</span></span>
               <span @click="select(0,$event)" class="block positive" :class="{'active':selectType===0}">{{desc.positive}}<span class="count">{{positives.length}}</span></span>
               <span @click="select(1,$event)" class="block negative" :class="{'active':selectType===1}">{{desc.negative}}<span class="count">{{negative.length}}</span></span>
           </div>
           <div class="switch" @click="toggleContent" :class="{'on':onlyContent}">
               <span class="icon-check_circle"></span>
               <span class="text">只看有内容的评价</span>
           </div>
      </div>
  </div>
</template>
<script>
     const ALL = 2;
     const POSITIVE = 0;
     const NEGATIVE = 1;
    export default{
        props: {
           ratings: {
               type: Array,
               dafault() {
                   return [];
               }
           },
           selectType: {
               type: Number,
               dafault: ALL
           },
           onlyContent: {
               type: Boolean,
               default: false
           },
           desc: {
               type: Object,
               default() {
                   return {
                       positive: '满意',
                       negative: '不满意',
                       all: '全部'
                   };
               }
           }
        },
        methods: {
            select(type, event) {
                if (!event._constructed) {
                    return;
                }
                this.selectType = type;
                this.$dispatch('ratingtype.select', type);
            },
            toggleContent(event) {
                 if (!event._constructed) {
                    return;
                }
                this.onlyContent = !this.onlyContent;
                this.$dispatch('content.toggle', this.onlyContent);
            }
        },
        computed: {
            positives() {
                return this.ratings.filter((rating) => {
                    return rating.rateType === POSITIVE;
                });
            },
            negative() {
                return this.ratings.filter((rating) => {
                    return rating.rateType === NEGATIVE;
                });
            }
        }
    };
</script>
<style lang="stylus" rel="stylesheet/stylus">
    @import "../../common/stylus/mixin.styl"

    .ratingselect
        .rating-type
            padding: 18px 0
            margin: 0 18px
            border-bottom-1px(rgba(7,17,27,0.1))
            font-size: 0
            .block
                display: inline-block
                border-radius: 1px
                margin-right: 8px
                color: rgb(77,85,93)
                font-size: 12px
                padding: 8px 12px
                line-height: 16px
                &.active
                    color: #ffffff
                .count
                    font-size: 8px
                    margin-left: 2px
                &.positive
                    background: rgba(0,160,220,0.2)
                    &.active
                        background: rgb(0,160,220)
                &.negative
                    background: rgba(77,85,93,0.2)
                    &.active
                        background: rgb(77,85,93)
        .switch
            padding: 12px 18px
            line-height: 24px
            border-bottom: 1px solid rgba(7,17,27,0.1)
            color: rgb(147,153,159)
            font-size: 0
            &.on
                .icon-check_circle
                    color: rgb(0,160,220)
            .icon-check_circle
                margin-right: 4px
                font-size: 24px
                display: inline-block
                vertical-align: top
            .text
                font-size: 12px
            




</style>