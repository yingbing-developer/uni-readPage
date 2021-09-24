<template>
	<view class="scroll-page" :prop="scrollPageProp" :change:prop="scrollPage.propChange" :id="'scrollPage' + dataId">
		<!-- <view class="scroll-page-content" :id="'scrollPageContent' + dataId"> -->
			<slot></slot>
		<!-- </view> -->
	</view>
</template>

<script>
	export default {
		props: {
			dataId: {
				type: String,
				default () {
					let mydate = new Date();
					return 'cms' + mydate.getMinutes() + mydate.getSeconds() + mydate.getMilliseconds() + Math.round(Math.random() * 10000);
				}
			}
		},
		data () {
			return {
				changeScrollTop: false,
				scrollTop: 0
			}
		},
		computed: {
			scrollPageProp () {
				return {
					dataId: this.dataId,
					changeScrollTop: this.changeScrollTop,
					scrollTop: this.scrollTop
				}
			}
		},
		methods: {
			scrolltoUpper (e) {
				this.$emit('scrolltoUpper', e);
			},
			scrolltoLower (e) {
				this.$emit('scrolltoLower', e);
			},
			scrollTo (scrollTop) {
				this.changeScrollTop = true;
				this.scrollTop = scrollTop;
			},
			//内容防抖
			anchoring () {
				this.changeScrollTop = true;
				this.scrollTop = -1;
			},
			resetScroll () {
				this.changeScrollTop = false;
				this.scrollTop = 0;
			}
		}
	}
</script>

<script lang="renderjs" type="module" module="scrollPage">
	let myScrollPageDom
	export default {
		data () {
			scrollInfo: '';
		},
		mounted () {
			this.initDom.bind(this);
			this.bindScrollEvent();
		},
		methods: {
			initDom () {
				myScrollPageDom = scrollPage.init(document.getElementById('scrollPage' + this.scrollPageProp.dataId));
				// 观测更新的数据在 view 层可以直接访问到
				myScrollPageDom.setOption(this.scrollPageProp);
			},
			//参数改变
			propChange (newValue, oldValue) {
				if ( newValue.changeScrollTop != oldValue.changeScrollTop ) {
					if ( newValue.changeScrollTop ) {
						this.changeScrollTo(this.scrollPageProp.scrollTop);
					}
				}
			},
			//绑定滚动事件
			bindScrollEvent () {
				let scroll = document.getElementById('scrollPage' + this.scrollPageProp.dataId);
				if ( scroll ) {
					scroll.onscroll = () => {
						this.scrollInfo = {
							scrollTop: scroll.scrollTop,
							scrollHeight: scroll.scrollHeight,
							offsetHeight: scroll.offsetHeight
						}
						if ( Math.ceil(scroll.scrollTop + scroll.offsetHeight) >= scroll.scrollHeight ) {//触底
							this.triggerScrolltoLower(this.scrollInfo);
						}
						if ( scroll.scrollTop <= 0 ) {//触顶
							this.triggerScrolltoUpper(this.scrollInfo);
						}
					};
				}
			},
			changeScrollTo (scrollTop) {
				let scroll = document.getElementById('scrollPage' + this.scrollPageProp.dataId);
				if ( scrollTop > -1 ) {
					scroll.scrollTop = scrollTop;
				} else {
					scrollTop = scroll.scrollHeight - this.scrollInfo?.scrollHeight || 0;
					scroll.scrollTop = scrollTop;
				}
				this.triggerResetScroll();
			},
			triggerResetScroll () {
				// #ifndef H5
				this.$ownerInstance.callMethod('resetScroll');
				// #endif
				// #ifdef H5
				this.resetScroll();
				// #endif
			},
			triggerScrolltoUpper (e) {
				// #ifndef H5
				this.$ownerInstance.callMethod('scrolltoUpper', e);
				// #endif
				// #ifdef H5
				this.scrolltoUpper(e);
				// #endif
			},
			triggerScrolltoLower (e) {
				// #ifndef H5
				this.$ownerInstance.callMethod('scrolltoLower', e);
				// #endif
				// #ifdef H5
				this.scrolltoLower(e);
				// #endif
			}
		}
	}
</script>

<style scoped>
	.scroll-page {
		overflow-y: auto;
		height: 400rpx;
		width: 100%;
	}
	.scroll-page-content {
		min-height: 100%;
		min-width: 100%;
	}
</style>
