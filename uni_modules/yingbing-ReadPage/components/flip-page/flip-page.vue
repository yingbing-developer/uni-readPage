<template>
	<div :prop="flipPageProp" :change:prop="flipPage.propChange" id="flipPage" class="flip-page"
		:style="{background: bgColor}" @touchstart="flipPage.pageTouchstart" @touchmove="flipPage.pageTouchmove"
		@touchend="flipPage.pageTouchend">
		<div class="flip-content">
			<div class="flip-item" :chapter="item.chapter" :start="item.start" :end="item.end" :dataId="item.dataId" :type="item.type"
				:class="{'flip-item-actived': currentPageDataId == item.dataId, 'flip-item-prev': index < pages.length - 1 ? currentPageDataId == pages[index + 1].dataId : false}" v-for="(item, index) in pages"
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
					<template v-else-if="item.type == 'prevLoading' || item.type == 'nextLoading'">
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
									type: contents[0].isStart ? 'top' : 'prevLoading',
									dataId: arr[0].dataId - 1,
									start: 0,
									end: 0
								})
								arr.push({
									chapter: item.chapter,
									type: item.isEnd ? 'bottom' : 'nextLoading',
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
						type: this.contents[prevIndex].isStart ? 'top' : 'prevLoading',
						dataId: newPages[0].dataId - 1,
						start: 0,
						end: 0
					})
					newPages.push({
						chapter: this.contents[nextIndex].chapter,
						type: this.contents[nextIndex].isEnd ? 'bottom' : 'nextLoading',
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
			showToast (e) {
				uni.showToast({
					title: e.title,
					icon: 'none'
				})
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
				pageEl: null,
				pageDirection: '',
				touchstart: {
					x: 0,
					y: 0
				},
				moveX: 0,
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
			getPageActived (value) {
				const flip = document.getElementById('flipPage')
				const pageActived = flip.getElementsByClassName('flip-item-actived')[0]
				const pageActivedPrev = flip.getElementsByClassName('flip-item-prev')[0]
				if ( pageActived.getAttribute('type') == 'bottom' ) {
					if ( value == 0 ) {
						this.triggerShowToast({
							title: '已经到最后了'
						})
						return false
					} else {
						return pageActivedPrev
					}
				} else if ( pageActived.getAttribute('type') == 'top' ) {
					if ( value < 0 ) {
						this.triggerShowToast({
							title: '已经到最前面了'
						})
						return false
					} else {
						return pageActived
					}
				} else if ( pageActived.getAttribute('type') == 'prevLoading' ) {
					if ( value < 0 ) {
						this.triggerShowToast({
							title: '请等待内容加载'
						})
						return false
					} else {
						return pageActived
					}
				} else if ( pageActived.getAttribute('type') == 'nextLoading' ) {
					if ( value == 0 ) {
						this.triggerShowToast({
							title: '请等待内容加载'
						})
						return false
					} else {
						return pageActivedPrev
					}
				} else {
					if ( value == 0 ) {
						return pageActived
					} else {
						return pageActivedPrev
					}
				}
				return false;
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
						this.pageAnimation(this.pageEl, this.pageDirection, this.moveX, rotateZ);
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
			triggerShowToast(e) {
				// #ifndef H5
				this.$ownerInstance.callMethod('showToast', e);
				// #endif
				// #ifdef H5
				this.showToast(e);
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
		box-shadow: -5px 0 20px rgba(0,0,0,0.1);
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
