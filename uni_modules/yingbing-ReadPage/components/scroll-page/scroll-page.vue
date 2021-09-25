<template>
	<view class="scroll-page" :prop="scrollPageProp" :change:prop="scrollPage.propChange" :id="'scrollPage' + dataId">
		<slot></slot>
		<view class="scroll-ramove-node"></view>
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
			scrollEnd (e) {
				this.$emit('scrollEnd', e);
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
			return {
				scrollInfo: '',
				oldFirstChild: ''
			}
		},
		mounted () {
			this.initDom.bind(this);
			this.bindScrollEvent();
			this.bindDomAddListen();
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
			//监听元素节点变动
			bindDomAddListen () {
				let MutationObserver = window.MutationObserver || window.WebKitMutationObserver || window.MozMutationObserver;
				let scroll = document.getElementById('scrollPage' + this.scrollPageProp.dataId);
				this.oldFirstChild = scroll.firstChild ? scroll.firstChild : {'data-id': -1};
				let observer = new MutationObserver((MutationRecord) => {
				  // 指定的DOM节点(目标节点)发生变化时被调用
				  let adNodes = [];
				  let reNodes = [];
				  const fistChild = scroll.firstChild;
				  MutationRecord.forEach((mutation) => {
				    mutation.addedNodes.forEach((node) => {
						//只获取从前面增加的节点
						if (parseInt(node.attributes['data-id'].value) < parseInt(this.oldFirstChild.getAttribute('data-id'))) adNodes.push(node);
					})
					mutation.removedNodes.forEach((node) => {
						//只获取从前面移除的节点
						if (parseInt(node.attributes['data-id'].value) < parseInt(fistChild.getAttribute('data-id'))) reNodes.push(node);
					})
				  });
				  //当滚动区域前面增加或移除节点时，滚动位置会发生改变，需要重置滚动位置，以保证当前位置不变
				  if ( adNodes.length > 0 ) {
					  let height = 0;
					  for ( let i in adNodes ) {
						  height += adNodes[i].offsetHeight;
					  }
					  scroll.scrollTop = scroll.scrollTop + height
				  }
				  if ( reNodes.length > 0 ) {
				  	let height = 0;
					let removeBox = scroll.getElementsByClassName('scroll-ramove-node')[0];
				  	for ( let i in reNodes ) {
						removeBox.appendChild(reNodes[i]);//已经移除的节点不能获取到高度，所以先将节点插入页面指定的位置，再获取高度
						height += reNodes[i].offsetHeight;
				  	}
					removeBox.innerHTML = '';//高度计算完毕后清空插入的已移除的节点
				  	scroll.scrollTop = scroll.scrollTop - height;
				  }
				  this.oldFirstChild = fistChild;
				});
				observer.observe(scroll, {
				  attributes: false,// 观察属性
				  childList: true// 观察子节点
				})
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
						window.clearTimeout(this.scrollEndTimer)
						this.scrollEndTimer = window.setTimeout(() => {
							this.triggerScrollEnd(this.scrollInfo);
						}, 300)
					};
				}
			},
			changeScrollTo (scrollTop) {
				let scroll = document.getElementById('scrollPage' + this.scrollPageProp.dataId);
				scroll.scrollTop = typeof scrollTop == 'Number' ? scrollTop : 0;
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
			},
			triggerScrollEnd (e) {
				// #ifndef H5
				this.$ownerInstance.callMethod('scrollEnd', e);
				// #endif
				// #ifdef H5
				this.scrollEnd(e);
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
	.scroll-ramove-node {
		position: fixed;
		width: 100%;
		height: 100%;
		top: -100%;
		left: 0;
	}
</style>
