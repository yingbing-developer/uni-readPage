<template>
	<div :prop="flipPageProp" :change:prop="flipPage.propChange" id="flipPage" class="flip-page"
		:style="{background: bgColor}" @touchstart="flipPage.pageTouchstart" @touchmove="flipPage.pageTouchmove"
		@touchend="flipPage.pageTouchend">
		<div class="flip-content">
			<div class="flip-item" :chapter="item.chapter" :start="item.start" :end="item.end" :dataId="item.dataId"
				:class="{'flip-item-actived': currentPageDataId == item.dataId}" v-for="(item, index) in pages"
				:key="item.dataId">
				<div class="flip-item-content" :style="{
					padding: `${topGap}px ${slide}px ${bottomGap}px ${slide}px`,
					background: bgColor,
					color: color,
					'font-size': fontSize + 'px',
					'z-index': -item.dataId
				}">
					<template v-if="item.type == 'text'">
						<p class="flip-text" :style="{'margin-top': lineHeight + 'px', height: fontSize + 'px'}"
							v-for="(text, i) in item.text" :key="i">{{text}}</p>
					</template>
					<template v-else-if="item.type == 'loading'">
						<div class="loading">
							<page-refresh>正在加载内容</page-refresh>
						</div>
					</template>
					<template v-else>
						<div class="loading">
							{{item.type == 'top' ? '已经到第一章了' : '已经到最后一章了'}}
						</div>
					</template>
				</div>
				<div class="flip-item-bg" :style="{background: bgColor}"></div>
				<div class="flip-item-shadow"></div>
			</div>
		</div>
		<computed-page ref="computedPage" :pageType="pageType" :fontSize="fontSize" :lineHeight="lineHeight"
			:slide="slide" :topGap="topGap" :bottomGap="bottomGap"></computed-page>
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
			//字体大小（单位px）
			fontSize: {
				type: String | Number,
				default: 15
			},
			//背景颜色
			bgColor: {
				type: String,
				default: '#fcd281'
			},
			//翻页方式
			pageType: {
				type: String,
				default: 'scroll'
			},
			//行间距（单位px）
			lineHeight: {
				type: Number | String,
				default: 15
			},
			//页面左右边距（单位px）
			slide: {
				type: Number | String,
				default: 40
			},
			//页面上边距（单位px）
			topGap: {
				type: Number | String,
				default: 10
			},
			//页面下边距（单位px）
			bottomGap: {
				type: Number | String,
				default: 10
			},
			//开启预加载
			enablePreload: {
				type: Boolean,
				default: false
			}
		},
		data() {
			return {
				contents: [],
				pages: [],
				currentPageDataId: -1,
				initStatus: false,
				pullupStatus: 'none',
				pulldownStatus: 'none'
			}
		},
		computed: {
			flipPageProp() {
				return {
					currentPageDataId: this.currentPageDataId,
					initStatus: this.initStatus,
					pageType: this.pageType,
					pullupStatus: this.pullupStatus,
					pulldownStatus: this.pulldownStatus
				}
			}
		},
		methods: {
			init(data) {
				this.contents = data.contents;
				this.resetPage(data);
			},
			//重绘页面
			resetPage(data) {
				this.pages = [];
				setTimeout(() => {
					//一次最多渲染3章的内容，根据定位的章节剪切出3章内容渲染
					const nowIndex = this.contents.findIndex(item => item.chapter == data.currentChapter);
					let prevIndex = -1;
					let nextIndex = -1;
					let contents = [];
					if (!this.contents[nowIndex].isStart) prevIndex = this.contents.findIndex(item => item
						.chapter == data.currentChapter - 1);
					if (!this.contents[nowIndex].isEnd) nextIndex = this.contents.findIndex(item => item.chapter ==
						data.currentChapter + 1);
					if (prevIndex > -1) {
						contents.push(this.contents[prevIndex])
					}
					contents.push(this.contents[nowIndex])
					if (nextIndex > -1) {
						contents.push(this.contents[nextIndex])
					}
					let arr = [];
					const dowhile = (i) => {
						let item = contents[i];
						this.computedChapter(item).then(pages => {
							if (data.currentChapter == item.chapter) {
								let index = Object.keys(pages).findIndex(key => data.start >= pages[
										key].start &&
									data.start < pages[key].end)
								this.currentPageDataId = pages[index].dataId;
							}
							arr = arr.concat(pages)
							if (i == contents.length - 1) {
								arr.unshift({
									chapter: contents[0].chapter,
									type: contents[0].isStart ? 'top' : 'loading',
									dataId: arr[0].dataId - 1,
									start: 0,
									end: 0
								})
								arr.push({
									chapter: item.chapter,
									type: item.isEnd ? 'bottom' : 'loading',
									dataId: arr[arr.length - 1].dataId + 1,
									start: 0,
									end: 0
								})
								this.pages = arr;
								this.$nextTick(() => {
									this.initStatus = true;
									this.preload(data.currentChapter);
								})
							} else {
								setTimeout(() => {
									dowhile(i + 1)
								}, 100)
							}
						})
					}
					dowhile(0)
				}, 50)
			},
			//加载上个章节
			scrolltoUpper(chapter) {
				this.$emit('loadmore', chapter, (status, content) => {
					if (status == 'success') {
						let index = this.contents.findIndex(item => item.chapter == content.chapter)
						if (index > -1) {
							this.contents[index] = content;
						} else {
							this.contents.push(content);
						}
						const data = {
							content: content,
							type: 'prev'
						}
						this.computedPage(data);
						this.preload(chapter)
					}
					this.pulldownStatus = status;
				})
			},
			//加载下个章节
			scrolltoLower(chapter) {
				this.$emit('loadmore', chapter, (status, content) => {
					if (status == 'success') {
						let index = this.contents.findIndex(item => item.chapter == content.chapter)
						if (index > -1) {
							this.contents[index] = content;
						} else {
							this.contents.push(content);
						}
						const data = {
							content: content,
							type: 'next'
						}
						this.computedPage(data);
						this.preload(chapter)
					}
					this.pullupStatus = status;
				})
			},
			//预加载章节
			preload(chapter) {
				if (!this.enablePreload) return false
				const nowIndex = this.contents.findIndex(item => item.chapter == chapter);
				let prevIndex = -2;
				let nextIndex = -2;
				let chapters = [];
				if (!this.contents[nowIndex].isStart) prevIndex = this.contents.findIndex(item => item.chapter == chapter -
					1);
				if (!this.contents[nowIndex].isEnd) nextIndex = this.contents.findIndex(item => item.chapter == chapter +
					1);
				if (prevIndex == -1) {
					chapters.push(chapter - 1);
				}
				if (nextIndex == -1) {
					chapters.push(chapter + 1);
				}
				if (chapters.length > 0) {
					this.$emit('preload', chapters, (status, contents) => {
						if (status == 'success') {
							contents.forEach(item => {
								let index = this.contents.findIndex(content => content.chapter == item
									.chapter)
								if (index > -1) {
									this.contents[index] = item;
								} else {
									this.contents.push(item);
								}
							})
						}
					})
				}
			},
			computedChapter(content) {
				return new Promise((resolve) => {
					this.$refs.computedPage.computed({
						content: content.content,
						chapter: content.chapter
					}).then((pages) => {
						resolve(pages);
					})
				}).catch(() => {
					resolve([])
				})
			},
			computedPage(e) {
				this.computedChapter(e.content).then((pages) => {
					let arr = [];
					let newPages = [];
					const pagesSync = e.type == 'prev' ? pages.concat(this.pages) : this.pages.concat(pages);
					pagesSync.forEach(item => {
						if (arr.indexOf(item.chapter) == -1) arr.push(item.chapter)
					})
					if (arr.length > 3) {
						let reChapter = e.type == 'prev' ? pagesSync[pagesSync.length - 1].chapter : pagesSync[0]
							.chapter;
						newPages = pagesSync.filter(item => item.chapter != reChapter && item.type == 'text');
					} else {
						newPages = pagesSync.filter(item => item.type == 'text');
					}
					const prevIndex = this.contents.findIndex(content => content.chapter == newPages[0].chapter);
					const nextIndex = this.contents.findIndex(content => content.chapter == newPages[newPages.length - 1].chapter);
					newPages.unshift({
						chapter: this.contents[prevIndex].chapter,
						type: this.contents[prevIndex].isStart ? 'top' : 'loading',
						dataId: newPages[0].dataId - 1,
						start: 0,
						end: 0
					})
					newPages.push({
						chapter: this.contents[nextIndex].chapter,
						type: this.contents[nextIndex].isEnd ? 'bottom' : 'loading',
						dataId: newPages[newPages.length - 1].dataId + 1,
						start: 0,
						end: 0
					})
					this.pages = newPages;
				});
			},
			scrollEnd(e) {
				this.$emit('scrollEnd', e);
			},
			resetPulldownStatus() {
				this.pulldownStatus = 'none';
			},
			resetPullupStatus() {
				this.pullupStatus = 'none';
			},
			resetInitStatus () {
				this.initStatus = false;
			}
		}
	}
</script>

<script lang="renderjs" type="module" module="flipPage">
	let myFlipPageDom
	export default {
		data() {
			return {
				pageWating: false,
				viewWidth: 0,
				viewHeight: 0
			}
		},
		mounted() {
			this.initDom.bind(this);
		},
		methods: {
			initDom() {
				myFlipPageDom = flipPage.init(document.getElementById('flipPage'));
				// 观测更新的数据在 view 层可以直接访问到
				myFlipPageDom.setOption(this.flipPageProp);
			},
			//参数改变
			propChange(newValue, oldValue) {
				if (newValue.pulldownStatus != oldValue.pulldownStatus) {
					this.pulldownStatusChange(newValue.pulldownStatus, oldValue.pulldownStatus);
				}
				if (newValue.pullupStatus != oldValue.pullupStatus) {
					this.pullupStatusChange(newValue.pullupStatus, oldValue.pullupStatus);
				}
				//初始化时，定位到设置的位置
				if (newValue.initStatus != oldValue.initStatus) {
					if ( newValue.initStatus ) {
						const flip = document.getElementById('flipPage');
						this.viewWidth = flip.offsetWidth;
						this.viewHeight = flip.offsetHeight;
						const flipItems = flip.getElementsByClassName('flip-item');
						Object.keys(flipItems).forEach(key => {
							if (flipItems[key].getAttribute('dataId') < this.flipPageProp.currentPageDataId) {
								this.pageAnimation(flipItems[key], 'next', -this.viewWidth)
							}
						})
						this.triggerResetInitStatus();
					}
				}
			},
			//翻页动画
			pageAnimation(el, direction, moveX, rotateZ = 0) {
				let lateX = direction == 'next' ? moveX : moveX - this.viewWidth;
				const content = el.getElementsByClassName('flip-item-content')[0];
				const bg = el.getElementsByClassName('flip-item-bg')[0];
				const shadow = el.getElementsByClassName('flip-item-shadow')[0];
				el.style.transform = `translateX(${lateX}px)`;
				content.style.transform = this.flipPageProp.pageType == 'real' ? `translateX(${-lateX}px)` : content.style
					.transform;
				bg.style.transform = this.flipPageProp.pageType == 'real' ?
					`translate(${lateX}px, -50%) rotateZ(${rotateZ}deg)` : bg.style.transform;
				shadow.style.boxShadow = '0 0 60px ' + (this.flipPageProp.pageType == 'real' ? Math.abs(lateX) > 30 ? 30 :
					Math.abs(lateX) : 0) + 'px rgba(0,0,0,0.5)';
			},
			//监听下拉加载状态变化
			pulldownStatusChange(newValue) {
				switch (newValue) {
					case 'success':
						bs.finishPullDown();
						bs.enable();
						break;
					case 'fail':
						document.getElementsByClassName('pulldown-loading')[0].style.display = 'none';
						document.getElementsByClassName('pulldown-finish')[0].innerHTML = '------请求内容失败------'
						document.getElementsByClassName('pulldown-finish')[0].style.display = 'block'
						window.setTimeout(() => {
							bs.finishPullDown();
							bs.enable();
							window.setTimeout(() => {
								document.getElementsByClassName('pulldown-loading')[0].style.display =
									'flex';
								document.getElementsByClassName('pulldown-finish')[0].innerHTML = ''
								document.getElementsByClassName('pulldown-finish')[0].style.display =
									'none'
							}, 50)
						}, TIME_WATING)
						break;
					case 'timeout':
						document.getElementsByClassName('pulldown-loading')[0].style.display = 'none';
						document.getElementsByClassName('pulldown-finish')[0].innerHTML = '------请求超时------'
						document.getElementsByClassName('pulldown-finish')[0].style.display = 'block'
						window.setTimeout(() => {
							bs.finishPullDown();
							bs.enable();
							window.setTimeout(() => {
								document.getElementsByClassName('pulldown-loading')[0].style.display =
									'flex';
								document.getElementsByClassName('pulldown-finish')[0].innerHTML = ''
								document.getElementsByClassName('pulldown-finish')[0].style.display =
									'none'
							}, 50)
						}, TIME_WATING)
						break;
					default:
						console.log('重置pulldown')
				}
				this.triggerResetPulldownStatus();
			},
			//监听上拉加载状态变化
			pullupStatusChange(newValue) {
				switch (newValue) {
					case 'success':
						bs.finishPullUp();
						bs.enable();
						break;
					case 'fail':
						bs.enable();
						document.getElementsByClassName('pullup-loading')[0].style.display = 'none';
						document.getElementsByClassName('pullup-finish')[0].innerHTML = '------请求失败,点击重试------'
						document.getElementsByClassName('pullup-finish')[0].style.display = 'block'
						document.getElementsByClassName('pullup-finish')[0].addEventListener('touchend', function() {
							bs.finishPullUp();
							document.getElementsByClassName('pullup-loading')[0].style.display = 'flex';
							document.getElementsByClassName('pullup-finish')[0].innerHTML = ''
							document.getElementsByClassName('pullup-finish')[0].style.display = 'none';
							bs.autoPullUpLoad();
							document.getElementsByClassName('pullup-finish')[0].removeEventListener('touchend',
								function() {}, false);
						}, false)
						break;
					case 'timeout':
						bs.enable();
						document.getElementsByClassName('pullup-loading')[0].style.display = 'none';
						document.getElementsByClassName('pullup-finish')[0].innerHTML = '------请求超时,点击重试------'
						document.getElementsByClassName('pullup-finish')[0].style.display = 'block'
						document.getElementsByClassName('pullup-finish')[0].addEventListener('touchend', function() {
							bs.finishPullUp();
							document.getElementsByClassName('pullup-loading')[0].style.display = 'flex';
							document.getElementsByClassName('pullup-finish')[0].innerHTML = ''
							document.getElementsByClassName('pullup-finish')[0].style.display = 'none';
							bs.autoPullUpLoad();
							document.getElementsByClassName('pullup-finish')[0].removeEventListener('touchend',
								function() {}, false);
						}, false)
						break;
					default:
						console.log('重置pullup')
				}
				this.triggerResetPullupStatus();
			},
			diff(obj1, obj2) {
				var o1 = obj1 instanceof Object;
				var o2 = obj2 instanceof Object;
				// 判断是不是对象
				if (!o1 || !o2) {
					return obj1 === obj2;
				}

				//Object.keys() 返回一个由对象的自身可枚举属性(key值)组成的数组,
				//例如：数组返回下表：let arr = ["a", "b", "c"];console.log(Object.keys(arr))->0,1,2;
				if (Object.keys(obj1).length !== Object.keys(obj2).length) {
					return false;
				}

				for (var o in obj1) {
					var t1 = obj1[o] instanceof Object;
					var t2 = obj2[o] instanceof Object;
					if (t1 && t2) {
						return this.diff(obj1[o], obj2[o]);
					} else if (obj1[o] !== obj2[o]) {
						return false;
					}
				}
				return true;
			},
			pageTouchstart(e) {
				if (this.pageWating) {
					return;
				}
				if (e.touches.length == 1) {
					this.touchTimer = window.setInterval(() => {
						this.touchTime += 50;
					}, 50)
					let touch = e.touches[0];
					this.touchstart.x = touch.pageX;
					this.touchstart.y = touch.pageY;
					if (this.touchstart.x > (this.viewWidth / 4) * 3) {
						this.pageEl = this.getPageActived(0);
						this.pageDirection = 'next'
					}
					if (this.touchstart.x < (this.viewWidth / 4)) {
						this.pageEl = this.getPageActived(-1);
						this.pageDirection = 'prev'
					}
				}
			},
			pageTouchmove(e) {
				if (this.pageWating) {
					return;
				}
				if (e.touches.length == 1) {
					if (this.pageEl) {
						let touch = e.touches[0];
						let height = this.viewHeight / 2;
						let maxDeg = height / 5;
						let rotateZ = this.pageDirection == 'next' ? ((touch.pageY - height) / maxDeg) : -((touch.pageY -
							height) / maxDeg);
						if (this.touchstart.x > (this.viewWidth / 4) * 3 || this.touchstart.x < (this.viewWidth / 4)) {
							this.moveX = touch.pageX - this.touchstart.x;
						}
						this.pageAnimation(this.moveX, rotateZ);
					}
				}
			},
			pageTouchend(e) {
				window.clearInterval(this.touchTimer);
				if (this.pageWating) {
					return;
				}
				if (this.pageEl) {
					this.pageWating = true;
					if (this.touchTime <= 200) {
						let duration = (this.pageProp.pageType == 'real' || this.pageProp.pageType == 'cover') ? 1000 : 0
						let value = this.pageDirection == 'next' ? 1 : -1;
						this.pageDuration(duration);
						this.$nextTick(() => {
							this.pageAnimation(-value * this.viewWidth);
							setTimeout(() => {
								this.changePageActived(value);
								this.resetPageMove();
							}, duration + 50)
						})
					} else {
						let duration = (this.pageProp.pageType == 'real' || this.pageProp.pageType == 'cover') ? 500 : 0
						if (Math.abs(this.moveX) >= this.viewWidth / 2.5) {
							let value = this.pageDirection == 'next' ? 1 : -1;
							this.pageDuration(duration);
							this.$nextTick(() => {
								this.pageAnimation(-value * this.viewWidth);
								setTimeout(() => {
									this.changePageActived(value);
									this.resetPageMove();
								}, duration + 50)
							})
						} else {
							this.pageDuration(duration);
							this.$nextTick(() => {
								this.pageAnimation(0);
								setTimeout(() => {
									this.resetPageMove();
								}, duration + 50)
							})
						}
					}
				}
			},
			computedPageInfo() {
				const scroll = document.getElementById('scrollPage');
				const scrollItems = scroll.getElementsByClassName('scroll-item');
				const scrollTop = this.scrollInfo.scrollTop + this.scrollPageProp.topGap + this.scrollPageProp.bottomGap;
				let dataId = -1;
				for (let i = 0; i < scrollItems.length; i++) {
					let offsetTop = scrollItems[i].offsetTop;
					let offsetBottom = scrollItems[i].offsetTop + scrollItems[i].offsetHeight;
					if (scrollTop >= offsetTop && scrollTop < offsetBottom) {
						dataId = parseInt(scrollItems[i].getAttribute('data-id'))
					}
				}
				if (!dataId) {
					if (scrollTop < scrollItems[0].offsetTop) {
						dataId = parseInt(scrollItems[0].getAttribute('data-id'))
					}
					if (scrollTop > scrollItems[scrollItems.length - 1].offsetTop + scrollItems[scrollItems.length - 1]
						.offsetHeight) {
						dataId = parseInt(scrollItems[scrollItems.length - 1].getAttribute('data-id'))
					}
				}
				let index = this.pagesSync.findIndex(item => item.dataId == dataId);
				let pageInfo = this.pagesSync[index];
				pageInfo.currentPage = index + 1;
				pageInfo.totalPage = this.pagesSync.filter(item => item.chapter == pageInfo.chapter).length;
				return pageInfo
			},
			triggerResetPulldownStatus() {
				// #ifndef H5
				this.$ownerInstance.callMethod('resetPulldownStatus');
				// #endif
				// #ifdef H5
				this.resetPulldownStatus();
				// #endif
			},
			triggerResetPullupStatus() {
				// #ifndef H5
				this.$ownerInstance.callMethod('resetPullupStatus');
				// #endif
				// #ifdef H5
				this.resetPullupStatus();
				// #endif
			},
			triggerComputedPage(e) {
				// #ifndef H5
				this.$ownerInstance.callMethod('computedPage', e);
				// #endif
				// #ifdef H5
				this.computedPage(e);
				// #endif
			},
			triggerScrolltoUpper(chapter) {
				// #ifndef H5
				this.$ownerInstance.callMethod('scrolltoUpper', chapter);
				// #endif
				// #ifdef H5
				this.scrolltoUpper(chapter);
				// #endif
			},
			triggerScrolltoLower(chapter) {
				// #ifndef H5
				this.$ownerInstance.callMethod('scrolltoLower', chapter);
				// #endif
				// #ifdef H5
				this.scrolltoLower(chapter);
				// #endif
			},
			triggerScrollEnd(e) {
				let pageInfo = this.computedPageInfo();
				// #ifndef H5
				this.$ownerInstance.callMethod('scrollEnd', pageInfo);
				// #endif
				// #ifdef H5
				this.scrollEnd(pageInfo);
				// #endif
			},
			triggerPreload(chapter) {
				// #ifndef H5
				this.$ownerInstance.callMethod('preload', chapter);
				// #endif
				// #ifdef H5
				this.preload(chapter);
				// #endif
			},
			triggerResetInitStatus () {
				// #ifndef H5
				this.$ownerInstance.callMethod('resetInitStatus', );
				// #endif
				// #ifdef H5
				this.resetInitStatus();
				// #endif
			},
			triggerResetPage() {
				bs.disable();
				let pageInfo = this.computedPageInfo();
				this.pagesSync = [];
				const data = {
					start: pageInfo.start,
					currentChapter: pageInfo.chapter
				}
				// #ifndef H5
				this.$ownerInstance.callMethod('resetPage', data);
				// #endif
				// #ifdef H5
				this.resetPage(data);
				// #endif
			},
		}
	}
</script>

<style scoped>
	.flip-page {
		width: 100%;
		height: 100%;
		position: absolute;
		left: 0;
		top: 0;
		box-sizing: border-box;
		overflow: hidden;
	}

	.flip-content {
		width: 100%;
		height: 100%;
		position: absolute;
		left: 0;
		top: 0;
		box-sizing: border-box;
		overflow: hidden;
		z-index: 1;
	}

	.flip-item {
		position: absolute;
		width: 100%;
		height: 100%;
		top: 0;
		left: 0;
		box-sizing: border-box;
		overflow: hidden;
	}

	.flip-item-content {
		position: absolute;
		width: 100%;
		height: 100%;
		top: 0;
		left: 0;
		box-sizing: border-box;
	}
	
	.loading {
		position: absolute;
		width: 100%;
		height: 100%;
		top: 0;
		left: 0;
		box-sizing: border-box;
		display: flex;
		align-items: center;
		justify-content: center;
	}

	.flip-item-bg {
		position: absolute;
		width: 100%;
		height: 150vh;
		top: 50%;
		left: 100%;
		transform: translateY(-50%);
	}

	.flip-item-shadow {
		position: absolute;
		width: 0;
		height: 100%;
		top: 0;
		right: 0;
		z-index: 9;
	}

	.flip-text {
		width: 100%;
		box-sizing: border-box;
		white-space: nowrap;
	}
</style>
