<template>
  <div class="rating-select">
   	<div class="rating-type border-1px">
   		<span class="block all" @click="select(2)" :class={'active':selectType===2}>{{desc.all}} <span class="count">{{ratings.length}}</span></span>
   		<span class="block positive" @click="select(0)" :class={'active':selectType===0}>{{desc.positive}} <span class="count">{{positiveRatings.length}}</span></span>
   		<span class="block negative" @click="select(1)" :class={'active':selectType===1}>{{desc.negative}} <span class="count">{{negativeRatings.length}}</span></span>
   	</div>
   	<div class="switch" :class="{'on':onlyContent}">
   		<i @click="switchToggle" class="icon-check_circle"></i>
   		<span class="text">只看有内容的评价</span>
   		<!-- {{ratings|json}} -->
   	</div>
  </div>
</template>

<script type="text/ecmascript-6">
	/* eslint-disable no-unused-vars*/
	const NEGATIVE = 1
	const POSITIVE = 0
	const ALL = 2
	export default {
		props: {
			ratings: {
				type: Array,
				default() {
					return []
				}
			},
			selectType: {
				type: Number,
				default: ALL
			},
			onlyContent: {
				type: Boolean,
				default: false
			},
			desc: {
				type: Object,
				default() {
					return {
						all: '全部',
						positive: '满意',
						negative: '不满意'
					}
				}
			}
		},
		computed: {
			positiveRatings() {
				return this.ratings.filter((rating) => {
					return rating.rateType === 0
				})
			},
			negativeRatings() {
				return this.ratings.filter((rating) => {
					return rating.rateType === 1
				})
			}
		},
		methods: {
			select(type) {
				this.selectType = type
				this.$dispatch('ratings.select', type)
			},
			switchToggle() {
				this.onlyContent = !this.onlyContent
				this.$dispatch('content.switch', this.onlyContent)
			}
		}
	}
</script>


<style lang="stylus">
	@import "../../common/stylus/mixin.styl"
	.rating-select
		.rating-type
			padding:18px 0
			margin:0 18px
			border-1px(rgba(7,17,27,0.1))
			font-size: 0
			.block
				display: inline-block
				padding: 8px 12px
				border-radius:1px;
				margin-right:8px
				font-size:12px
				color:rgb(77,85,93)
				&.active
					color:#fff
				.count
					font-size:8px
					margin-left:2px
					line-height:16px
				&.positive,&.all
					background:rgba(0,160,220,0.2)
					&.active
						background:rgb(0,160,220)
				&.negative
					background:rgba(77,85,93,0.2)
					&.active
						background:rgb(77,85,93)
		.switch
			padding:12px 18px
			line-height:24px
			border-border:1px solid rgba(7,17,27,0.1)	
			color:rgb(147,153,159)	
			font-size:0
			.icon-check_circle
				display:inline-block
				vertical-align:top
				margin-right:4px
				font-size:24px
			&.on
				.icon-check_circle
					color:#00c850
			.text
				font-size:12px



</style>
