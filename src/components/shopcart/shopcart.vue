<template>
	<div class="shopcart">
		<div class="content">
			<div class="content-left">
				<div class="logo-wrapper">
					<div class="logo" :class="{'high-light':totalCount>0}">
						<i class="icon-shopping_cart" :class="{'high-light':totalCount>0}"></i>
					</div>
					<div class="num" v-show="totalCount>0">{{totalCount}}</div>
				</div>
				<div class="price" :class="{'high-light':totalPrice>0}">￥{{totalPrice}}</div>
				<div class="desc" @click="_toggleList">另需配送费￥{{deliveryPrice}}元</div>
			</div>
			<div class="content-right">
				<div class="pay" :class="payClass" @click.stop.prevent="_pay">
					{{payDesc}}
				</div>
			</div>
		</div>
		<div class="ball-container">
			<div class="ball" v-for="ball in balls" v-show="ball.show" transition="drop">
				<div class="inner inner-hook"></div>
			</div>
		</div>
		<div class="shopcart-list" v-show="listShow" transition="fold">
			<div class="list-header">
				<h3 class="title">购物车</h1>
					<span class="empty" @click="_empty">清空</span>
			</div>
			<div class="list-content" v-el:list-content>
				<ul>
					<li class="food" v-for="food in selectedFoods">
						<span class="name">{{food.name}}</span>
						<div class="price">
							<span>￥{{food.count*food.price}}</span>
						</div>
						<div class="cartcontrol-wrapper">
							<cartcontrol :food="food"></cartcontrol>
						</div>
					</li>
				</ul>
			</div>
		</div>
	</div>
	<div class="list-mask" @click="_hideList" v-show="listShow" transition="mask"></div>
</template>

<script type="text/ecmascript-6">
	import BScroll from 'better-scroll'
	import cartcontrol from 'components/cartcontrol/cartcontrol'
	export default {
		data() {
				return {
					balls: [{
						show: false
					}, {
						show: false
					}, {
						show: false
					}, {
						show: false
					}, {
						show: false
					}, {
						show: false
					}, {
						show: false
					}],
					dropBalls: [],
					fold: true
				}
			},
			props: {
				selectedFoods: {
					type: Array,
					default () {
						return [{
							price: 0,
							count: 0
						}]
					}
				},
				deliveryPrice: {
					type: Number,
					default: 0
				},
				minPrice: {
					type: Number,
					default: 0
				}
			},
			created() {},
			methods: {
				_drop(targetEle) {
					for (let i = 0; i < this.balls.length; i++) {
						let ball = this.balls[i]
						if (!ball.show) {
							ball.show = true
							ball.el = targetEle
							this.dropBalls.push(ball)
							return
						}
					}
				},
				_toggleList() {
					if (this.totalCount === 0) {
						this.fold = true
						return
					}
					this.fold = !this.fold
				},
				_hideList() {
					this.fold = true
				},
				_empty() {
					this.selectedFoods.forEach((food) => {
						food.count = 0
					})
				},
				_pay() {
					if (this.totalPrice < this.minPrice) {
						return
					}
					window.alert(`需要支付￥${this.totalPrice}`)
				}
			},
			transitions: {
				drop: {
					beforeEnter(el) {
						let count = this.balls.length
						while (count--) {
							let ball = this.balls[count]
							if (ball.show) {
								let rect = ball.el.getBoundingClientRect()
								let x = rect.left - 32
								let y = -(window.innerHeight - rect.top - 22)

								el.style.display = ''
								el.style.webkitTransform = `translate3d(0,${y}px,0)`
								el.style.transform = `translate3d(0,${y}px,0)`
								let inner = el.getElementsByClassName('inner-hook')[0]
								inner.style.webkitTransform = `translate3d(${x}px,0,0)`
								inner.style.transform = `translate3d(${x}px,0,0)`
							}
						}
					},
					enter(el) {
						/* 触发一次浏览器重绘*/
						/* eslint-disable no-unused-vars*/
						let rf = el.offsetHeight
						this.$nextTick(() => {
							el.style.webkitTransform = 'translate3d(0,0,0)'
							el.style.transform = 'translate3d(0,0,0)'
							let inner = el.getElementsByClassName('inner-hook')[0]
							inner.style.webkitTransform = 'translate3d(0,0,0)'
							inner.style.transform = 'translate3d(0,0,0)'
						})
					},
					afterEnter(el) {
						let ball = this.dropBalls.shift()
						if (ball) {
							ball.show = false
							el.style.display = 'none'
						}
					}
				}
			},
			computed: {
				totalPrice() {
					let totalPrice = 0
					this.selectedFoods.forEach((food) => {
						totalPrice += food.price * food.count
					})
					return totalPrice
				},
				totalCount() {
					let totalCount = 0
					this.selectedFoods.forEach((food) => {
						totalCount += food.count
					})
					return totalCount
				},
				payDesc() {
					if (this.totalPrice === 0) {
						return `￥${this.minPrice}元起送`
					} else if (this.totalPrice < 20) {
						let diff = this.minPrice - this.totalPrice
						return `还差￥${diff}元起送`
					} else {
						return '去结算'
					}
				},
				payClass() {
					if (this.totalPrice < 20) {
						return 'not-enough'
					} else {
						return 'enough'
					}
				},
				listShow() {
					this.$nextTick(() => {
						if (!this.listScroll) {
							this.listSrcoll = new BScroll(this.$els.listContent, {
								click: true
							})
						} else {
							this.listSrcoll.refresh()
						}
					})
					if (this.totalCount === 0) {
						this.fold = true
						return false
					}
					let show = !this.fold
					return show
				}
			},
			components: {
				cartcontrol
			}
	}
</script>

<style lang="stylus">
	@import "../../common/stylus/mixin"
	.shopcart
		position: fixed
		left:0
		bottom:0
		z-index:10
		width:100%
		height:48px
		background:#141d27
		.content
			display:flex
			background:141d27
			font-size:0
			.content-left
				flex:1
				.logo-wrapper
					display:inline-block
					position:relative
					top:-10px
					margin:0 12px
					padding:6px
					width:56px
					height:56px
					box-sizing:border-box
					vertical-align:top
					border-radius:50%
					background:#000
					.logo
						width:100%
						height:100%
						border-radius:50%
						background:#2b343c
						text-align:center
						&.high-light
							background:rgb(0,160,220)
						.icon-shopping_cart
							line-height:44px
							font-size:24px
							color:#80858a
							&.high-light
								color:#fff
					.num
						position:absolute
						left:30px
						top:0
						width:24px
						height:16px
						line-height:16px
						text-align:center
						border-radius:16px
						font-size:9px
						font-weight:700px
						color:#fff
						background:rgb(240,20,20)
						box-shadow:0 4px 8px 0 rgba(0,0,0,0.4)

				.price
					display:inline-block
					vertical-align:top
					line-height:24px
					margin-top:12px
					padding-right:12px
					box-sizing:border-box
					border-right:1px solid rgba(255,255,255,0.1)
					color:rgba(255,255,255,0.5)
					font-size:16px
					font-weight:700px
					&.high-light
						color:#fff

				.desc
					display:inline-block
					vertical-align:top
					line-height:24px
					margin:12px 0 0 12px
					font-size:12px
					color:rgba(255,255,255,0.5)
			.content-right
				flex:0 0 105px
				width:105px
				.pay
					height:48px
					line-height:48px
					text-align:center
					font-size:12px
					color:rgba(255,255,255,0.5)
					font-weight:700
					background:#2b333b
					&.not-enough
						background:#2b333b
					&.enough
						background:#00b43c
						color:#fff
		.ball-container
			.ball
				position:fixed
				left:32px
				bottom:22px
				z-index:13
				.inner
					width:16px
					height:16px
					border-radius:50%
					background:rgb(0,160,220)
					transition:all 0.4s	linear
				&.drop-transition
					transition:all 0.4s	cubic-bezier(0.49,-0.29,0.75,0.41)
		.shopcart-list
			position:absolute
			top:0
			left:0
			z-index:-1
			width:100%
			&.fold-transition
				transition:all 0.5s
				transform:translate3d(0,-100%,0)
			&.fold-enter,&.fold-leave
				transform:translate3d(0,0,0)
			.list-header
				height:40px
				line-height:40px
				padding:0 18px
				background:#f3f5f7
				border-bottom:1px solid rgba(7,17,27,0.1)
				.title
					float:left
					font-size:14px
					color:rgb(7,17,27)
				.empty
					float:right
					font-size:12px
					color:rgb(0,160,220

						)
			.list-content
				padding:0 18px
				max-height:217px
				background:#fff
				overflow:hidden
				.food
					position:relative
					padding:12px 0
					box-sizing:border-box
					border-1px(rgba(7,17,27,0.1))
					.name
						line-height:24px
						font-size:14px
						color:rgb(7,17,27)
					.price
						position:absolute
						right:90px
						bottom:12px
						line-height:24px
						font-size:14px
						font-weight:700
						color:rgb(240,20,20)
					.cartcontrol-wrapper
						position:absolute
						right:0
						bottom:6px
	.list-mask
		position:fixed
		left:0
		top:0
		width:100%
		height:100%
		blur:10px
		background:rgba(7,17,27,0.6)
		transition:all 0.4s
		&.mask-transition
			opacity:1
		&.mask-enter,&.mask-leave
			opacity:0





</style>