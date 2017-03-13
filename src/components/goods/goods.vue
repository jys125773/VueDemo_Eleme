<template>
	<div class="goods">
		<div class="menu-wrapper" v-el:menu-wrapper>
			<ul>
				<li v-for="item in goods"class="menu-item" @click="_selectMenu($index,$event)"  :class = "{current:menuCurrenIndex === $index}">
					<span class="text border-1px">
						<span v-show="item.type >0" class="icon" :class="classMap[item.type]"></span>
						{{item.name}}
					</span>
				</li>
			</ul>
		</div>
		<div class="foods-wrapper" v-el:foods-wrapper>
			<ul>
				<li v-for="item in goods" class="food-list food-list-hook">
					<h3 class="title">{{item.name}}</h3>
					<ul>
						<li v-for="food in item.foods" @click="_selectFood(food,$event)" class="food-item">
							<div class="icon">
								<img width="57" height="57" :src="food.icon"/>
							</div>
							<div class="content">
								<h4 class="name">{{food.name}}</h4>
								<p class="desc">{{food.description}}</p>
								<div class="extra">
									<span class="count">月售{{food.sellCount}}份</span>
									<span>好评率{{food.rating}}%</span>
								</div>
								<div class="price">
									<span class="now">￥{{food.price}}</span>
									<span class="old" v-if="food.oldPrice">￥{{food.oldPrice}}</span>
								</div>
								<div class="cartcontrol-wrapper">
									<cartcontrol :food="food"></cartcontrol>
								</div>
							</div>
						</li>
					</ul>
				</li>
			</ul>
		</div>

		<shopcart  v-ref:shopcart :selected-foods = "selectedFoods" :delivery-price="seller.deliveryPrice" :min-price="seller.minPrice"></shopcart>
	</div>
	<food :food="selectFood" v-ref:food></food>
</template>

<script type="text/ecmascript-6">
	import BScroll from 'better-scroll'
	import shopcart from 'components/shopcart/shopcart'
	import cartcontrol from 'components/cartcontrol/cartcontrol'
	import food from 'components/food/food'
	const OK = 0
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
				preCurrentIndex: 0,
				selectFood: {},
				date: null
			}
		},
		computed: {
			menuCurrenIndex() {
				for (let i = 0; i < this.listHeight.length; i++) {
					let H1 = this.listHeight[i]
					let H2 = this.listHeight[i + 1]
					if (!H2 || (this.scrollY >= H1 && this.scrollY < H2)) {
						return i
					}
				}
				return 0
			},
			selectedFoods() {
				let foods = []
				this.goods.forEach((good) => {
					good.foods.forEach((food) => {
						if (food.count > 0) {
							foods.push(food)
						}
					})
				})
				return foods
			}
		},
		destroyed() {
			console.log(new Date() - this.date)
		},
		created() {
			this.date = new Date()
			this.classMap = ['decrease', 'discount', 'special', 'invoice', 'guarantee']
			this.$http.get('/api/goods').then(response => {
				if (response.data.errno === OK) {
					this.goods = response.data.data
					this.$nextTick(() => {
						this._initScroll()
						this._calculateHeight()
					})
				}
			})
		},
		methods: {
			_initScroll() {
				this.menuScroll = new BScroll(this.$els.menuWrapper, {
					click: true
				})
				this.foodsScroll = new BScroll(this.$els.foodsWrapper, {
					probeType: 3,
					click: true
				})
				this.foodsScroll.on('scroll', (pos) => {
					this.scrollY = Math.abs(Math.round(pos.y))
				})
			},
			_calculateHeight() {
				let height = 0
				this.listHeight.push(height)
				let foodList = this.$els.foodsWrapper.getElementsByClassName('food-list-hook')
				for (let i = 0; i < foodList.length; i++) {
					height += foodList[i].clientHeight
					this.listHeight.push(height)
				}
			},
			_selectMenu($index, $event) {
				if (!$event._constructed) {
					return
				}
				let targetEle = this.$els.foodsWrapper.getElementsByClassName('food-list-hook')[$index]
				this.foodsScroll.scrollToElement(targetEle, 100 * Math.abs($index - this.preCurrentIndex))
				this.preCurrentIndex = $index
			},
			_drop(targetEle) {
				// 体验优化，异步动画
				this.$nextTick(() => {
					this.$refs.shopcart._drop(targetEle)
				})
			},
			_selectFood(food, $event) {
				if (!$event._constructed) {
					return
				}
				this.selectFood = food
				this.$refs.food.showDetail()
			}
		},
		components: {
			shopcart,
			cartcontrol,
			food
		},
		events: {
			'cart.add' (targetEle) {
				this._drop(targetEle)
			}
		}
	}
</script>

<style lang="stylus">
	@import "../../common/stylus/mixin"
	.goods
		display:flex
		overflow: hidden
		position: absolute
		width:100%
		top:174px
		bottom:46px
		.menu-wrapper
			flex:0 0 80px
			width:80px
			background:#f3f5f7
			.menu-item
				padding:0 12px
				display:table
				height:54px
				width:56px
				line-height:14px
				font-size:14px
				&.current
					position:relative
					margin-top:-1px
					background:#fff
					font-weight:700
					.text
						border:0
				.text
					display:table-cell
					width:56px
					font-size:12px
					vertical-align:middle
					border-1px(rgba(7,17,27,0.1))
					.icon
						display:inline-block
						vertical-align:top
						width:12px
						height:12px
						margin-right:2px
						background-size:12px 12px
						background-repeat:no-repeat
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
			flex:1
			.title
				padding-left:14px
				height:26px
				line-height:26px
				border-left:2px solid #d9dde
				font-size:12px
				color:rgb(147,153,159)
				background:#f3f5f7
			.food-item
				display:flex
				margin:18px
				padding-bottom:18px
				border-1px(rgba(7,17,27,0.1))
				&:last-child
					margin-bottom:0
				.icon
					flex:0 0 57px
					margin-right:10px
				.content
					flex:1
					.name
						margin:2px 0 8px 0
						height:14px
						line-height:14px
						font-size:14px
						color:rgb(7,17,27)
					.desc,.extra
						line-height:12px
						font-size:10px
						color:rgb(147,153,159)
					.desc
						margin-bottom:8px
					.extra
						.count
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
						right:0
						bottom:12px

</style>
