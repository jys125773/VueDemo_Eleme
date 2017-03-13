<template>
  <div class="cartcontrol">
   	<div class="decrement" @click.stop.prevent="decreseCart" v-show="food.count>0" transition="move">
   		<span class="inner icon-remove_circle_outline"></span>
   	</div>
   	<div class="count" v-show="food.count>0">{{food.count}}</div>
   	<div class="increment icon-add_circle" @click.stop.prevent="addCart"></div>
  </div>
</template>

<script type="text/ecmascript-6">
	import Vue from 'Vue'
	export default {
		props: {
			food: {
				type: Object
			}
		},
		methods: {
			addCart(event) {
				if (!this.food.count) {
					Vue.set(this.food, 'count', 1)
				} else {
					this.food.count++
				}
				this.$dispatch('cart.add', event.target)
			},
			decreseCart() {
				if (this.food.count) {
					this.food.count--
				}
			}
		}
	}
</script>


<style lang="stylus">
	.cartcontrol
		font-size:0
		.decrement
			display:inline-block
			padding:6px
			transition:all 0.4s linear
			.inner
				display: inline-block
				line-height: 24px
				font-size: 24px
				color: rgb(0, 160, 220)
				transition: all 0.4s linear
				transform: rotate(0)
			&.move-tansition
				opacity:1
				transform:translate3D(0,0,0)
			&.move-enter,&.move-leave
				opacity:0
				transform:translate3D(24px,0,0)	
				.inner
					transform: rotate(180deg)
		.increment
			display:inline-block
			line-height:24px
			font-size:24px
			padding:6px
			color:rgb(0,120,200)
		.count
			display:inline-block
			vertical-align:top
			padding-top:6px
			width:12px
			line-height:24px
			text-align:center
			font-size:10px
			color:rgb(147,153,159)
</style>
