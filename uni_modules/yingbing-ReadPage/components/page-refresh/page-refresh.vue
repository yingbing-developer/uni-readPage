<template>
	<div id="page-refresh" :prop="refreshProp" :change:prop="pageRefresh.refreshPropChange">
	</div>
</template>

<script>
	export default {
		props: {
			//字体颜色
			color: {
				type: String,
				default: '#333333'
			},
			padding: {
				type: String,
				default: '0 0 0 0'
			}
		},
		computed: {
			refreshProp () {
				return {
					color: this.color,
					padding: this.padding
				}
			}
		}
	}
</script>

<script lang="renderjs" type="module" module="pageRefresh">
	import Vue from 'vue'
	let myRefreshDom
	const animationRotate = `@keyframes animationRotate{
	    0% {
	      transform: rotateZ(0);
	    }
	    100% {
	      transform: rotateZ(360deg);
	    }
	}`
	export default {
		data () {
			return {
				colorSync: '',
				paddingSync: ''
			}
		},
		mounted () {
			this.colorSync = this.refreshProp.color
			this.paddingSync = this.refreshProp.padding
			new Vue({
				el: '#page-refresh',
				render: (h) => {
					const title = this.$slots.default
					return h('div', {
						class: 'page-refresh',
						style: {
							width: '100%',
							height: '100rpx',
							display:' flex',
							'align-items': 'center',
							'justify-content': 'center',
							padding: this.paddingSync
						}
					}, [
						h('div', {
							style: {
								'border-top': '5rpx solid' + this.colorSync,
								'border-left': '5rpx solid' + this.colorSync,
								'border-bottom': '5rpx solid' + this.colorSync,
								'border-right': '5rpx solid transparent',
								width: '30rpx',
								height: '30rpx',
								'border-radius': '30rpx',
								'margin-right': '10rpx',
								animation: 'animationRotate 1s linear infinite'
							}
						}, [
							h('style', {
								attrs: {
									type: 'text/css'
								}
							}, animationRotate)
						]),
						h('p', {
							style: {
								color: this.colorSync
							}
						}, title)
					])
				}
			})
		},
		methods: {
			initDom () {
				myRefreshDom = pageRefresh.init(document.getElementById('computedPage' + this.computedPageProp.dataId));
				// 观测更新的数据在 view 层可以直接访问到
				myRefreshDom.setOption(this.refreshProp);
			},
			refreshPropChange (newValue, oldValue) {
				if ( newValue.color != oldValue.color ) {
					this.colorSync = newValue.color
				}
				if ( newValue.padding != oldValue.padding ) {
					this.paddingSync = newValue.padding
				}
			}
		}
	}
</script>
