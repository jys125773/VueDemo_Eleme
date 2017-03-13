<template>
  <div v-show="showFlag" class="food" transition="slide" v-el:food>
  	<div class="food-content">
  		<div class="image-header">
  			<img :src="food.image">  
  			<div class="back" @click="hideDetail"> <i class="icon-arrow_lift"></i>
  			</div>
  		</div>
  		<div class="content">
  			<div class="title">{{food.name}}</div>
  			<div class="detail">
  				<span class="sell-count">月售{{food.sellCount}}份</span>
  				<span class="rating">好评率{{food.rating}}</span>
  			</div>
  			<div class="price">
  				<span class="now">￥{{food.price}}</span><span class="old" v-if="food.oldPrice">￥{{food.oldPrice}}</span>
  			</div>
  			<div class="cartcontrol-wrapper" v-show="food.count">
				<cartcontrol :food="food"></cartcontrol>
			</div>
			<div class="buy" transition="fade" @click.stop="addFirst($event)" v-show="!food.count">加入购物车</div>
  		</div>
  		<split v-if="food.info"></split>
  		<div class="info" v-if="food.info">
  			<h4 class="title">商品信息</h4>
  			<p class="text">{{food.info}}</p>
  		</div>
  		<split></split>
  		<div class="rating">
  			<h4 class="title">商品评价</h4>
  			<ratingselect :select-type="selectType" :desc="desc" :only-content="onlyContent" :ratings="food.ratings"></ratingselect>
  		</div>
  		<div class="rating-wrapper">
  			<ul v-show="food.ratings && food.ratings.length">
  				<li v-show="needShow(rating.rateType,rating.text)" class="rating-item border-1px" v-for="rating in food.ratings">
  					<div class="user">
  						<span class="user-name">{{rating.username}}</span><img :src="rating.avatar" width="12" height="12" class="user-avatar"></img>
  					</div>
  					<div class="time">{{rating.rateTime | formatDate}}</div>
  					<p class="text">
  						<i :class="{'icon-thumb_up':rating.rateType===0,'icon-thumb_down':rating.rateType===1}"></i>{{rating.text}}
  					</p>
  				</li>
  			</ul>
  		</div>
  		<div class="no-rating" v-show="!food.ratings || !food.ratings.length">
  				暂无评价
  		</div>
  	</div>
  </div>
</template>

<script type="text/ecmascript-6">
	import Vue from 'vue'
	import BScroll from 'better-scroll'
	import cartcontrol from 'components/cartcontrol/cartcontrol'
	import split from 'components/split/split'
	import ratingselect from 'components/ratingselect/ratingselect'
	import {formatDate} from 'common/js/formatDate.js'
	/* eslint-disable no-unused-vars*/
	const NEGATIVE = 1
	const POSITIVE = 0
	const ALL = 2
	export default {
		props: {
			food: {
				type: Object
			}
		},
		data() {
			return {
				showFlag: false,
				selectType: ALL,
				onlyContent: false,
				desc: {
					all: '全部',
					positive: '推荐',
					negative: '吐槽'
				}
			}
		},
		events: {
			'ratings.select' (type) {
				this.selectType = type
				this.$nextTick(() => {
					this.scroll.refresh()
				})
			},
			'content.switch' (onlyContent) {
				this.onlyContent = onlyContent
				this.$nextTick(() => {
					this.scroll.refresh()
				})
			}
		},
		methods: {
			showDetail() {
				this.selectType = ALL
				this.onlyContent = false
				this.showFlag = true
				this.$nextTick(() => {
					if (!this.scroll) {
						this.scroll = new BScroll(this.$els.food, {
							click: true
						})
					} else {
						this.scroll.refresh()
					}
				})
			},
			hideDetail() {
				this.showFlag = false
			},
			addFirst(event) {
				Vue.set(this.food, 'count', 1)
				this.$dispatch('cart.add', event.target)
			},
			needShow(type, text) {
				if (this.onlyContent && !text) {
					return false
				}
				if (this.selectType === ALL) {
					return true
				} else {
					return type === this.selectType
				}
			}
		},
		components: {
			cartcontrol,
			split,
			ratingselect
		},
		filters: {
			formatDate(time) {
				let date = new Date(time)
				return formatDate(date, 'yyyy-MM-DD hh:mm')
			}
		}
	}
</script>


<style lang="stylus">
	@import "../../common/stylus/mixin.styl"
	.food 
		position: fixed
		left:0
		top:0
		bottom:48px
		width:100%
		background:#fff
		transition:all 0.25s cubic-bezier(0.49, 1.13, 1, 1)
		&.slide-transition
			transform:translate3d(0,0,0)
		&.slide-enter,&.slide-leave
			transform:translate3d(-100%,0,0)
		.image-header
			position:relative
			width:100%
			height:0
			padding-top:100%
			img
				position:absolute
				left:0
				top:0
				width:100%
				height:100%
			.back
				position:absolute
				top:10px
				left:0
				.icon-arrow_lift
					color:#0078c8
					display:block
					padding:10px
					font-size:26px
		.content
			padding:18px
			position:relative
			.title
				line-height:14px
				margin-bottom:8px
				font-size:14px
				font-weight:700
				color:rgb(7,17,27)
			.detail
				margin-bottom:18px
				line-height:10px
				font-size:0
				.sell-count,.rating
					font-size:10px
					color:rgb(147,153,159)
				.sell-count
					margin-right:12px
			.price
				font-weight:700
				line-height:24px
				.now
					margin-right:18px
					font-size:14px
					color:rgb(240,20,20)
				.old
					text-decoration:line-through
					font-size:10px
					color:rgb(147,153,159)
			.cartcontrol-wrapper
				position:absolute
				right:12px
				bottom:12px
			.buy
				position:absolute
				right:18px
				bottom:18px
				z-index:2
				height:24px
				line-height:24px
				padding:0 12px
				box-sizing:border-box
				border-radius:12px
				font-size:10px
				color:#fff
				background:rgb(0,160,200)


				transition:all 0.2s
				&.fade-transition
					opacity:1
				&.fade-enter,&.fade-leave
					opacity:0
		.info
			padding:18px	
			.title
				line-height:14px
				margin-bottom:6px
				font-size:14px
				color:rgb(7,17,27)	
			.text
				line-height:24px
				font-size:12px
				padding:0 8px
				color:rgb(77,85,93)	
		.rating
			padding-top:18px
			border-1px(rgba(7,17,27,0.1))
			.title
				line-height:14px
				margin-left:18px
				font-size:14px
				color:rgb(7,17,27)	
		.rating-wrapper
			padding:0 18px
			.rating-item
				padding:16px 0
				position:relative
				border-1px(rgba(7,17,27,0.1))
				.user
					position:absolute
					right:0
					top:16px
					.user-name
						font-size:10px
						line-height:12px
						color:rgb(147,153,159)
						margin-right:6px
					.user-avatar
						border-radius:50%
				.time
					font-size:10px
					line-height:12px
					color:rgb(147,153,159)
					margin-bottom:6px
				.text
					font-size:12px
					line-height:16px
					color:rgb(7,27,27)
					i
						line-height:24px
						margin-right:4px
						font-size:12px
						color:rgb(147,153,159)
						&.icon-thumb_up
							color:rgb(0,160,220)
		.no-rating
			padding:16px 0
			font-size:12px
			color:rgb(147,153,159)
			text-align:center
					
						
						
						

</style>
