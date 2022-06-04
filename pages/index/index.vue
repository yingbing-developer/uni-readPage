<template>
	<view>
		<yingbing-ReadPage
		ref="page"
		:page-type="pageType"
		:font-size="fontsize"
		:line-height="lineHeight"
		:color="color"
		:bg-color="bgColor"
		:slide="slide"
		:enablePreload="enablePreload"
		:noChapter="noChapter"
		enableClick
		@clickTo="clickTo"
		@loadmore="loadmoreContent"
		@preload="preloadContent"
		@currentChange="currentChange"
		@setCatalog="setCatalog"
		@clickme="clickme"
		@clickher="clickher"
		></yingbing-ReadPage>
		
		<view class="pageInfo">{{currentPage + '/' + totalPage}}</view>
	</view>
</template>

<script>
	export default {
		data() {
			return {
				pageType: 'none',
				scrollTop: 400,
				fontsize: 20,
				lineHeight: 15,
				color: '#333',
				slide: 40,
				bgColor: '#fcd281',
				enablePreload: true,
				noChapter: false,
				totalPage: 0,
				currentPage: 0,
				custom: [//自定义页面
					`<div style="width: 100%;height: 100%;text-align: center;">
						<p style="margin-top: 50px">这是一个自定义的页面可以用于:</p>
						<p>广告展示</p>
						<p>付费章节</p>
						<p>插图展示</p>
						<p>...</p>
						<p style="margin-top: 50px">自定义页面上的元素可以添加点击事件:</p>
						<button onclick="clickme(123, 124)">点击我</button>
						<button onclick="clickher(123, 124)">点击她</button>
						<p>尽量不要将有点击事件的元素定位到中间，否则会和clikTo事件重合</p>
						<p>如果一定要定位到中间，也可以将clikTo事件定位到其它位置</p>
						<p style="margin-top: 50px">需要注意的是:</p>
						<p>图片最好将高度定好</p>
						<p>避免滚动模式下定位异常</p>
						<p>自定义页面只能放入页面章节最后展示</p>
					</div>`,
					`<div style="width: 100%;height: 100px;text-align: center;line-height: 100px;">这是一个自定义页面的第二页</div>`,
					`<div style="width: 100%;height: 100px;text-align: center;line-height: 100px;">这是一个自定义页面的第三页</div>`
				]
			}
		},
		onReady() {
			let contents = [{
				chapter: 3,
				content: this.getContent(3),
				title: '第三章',
				isStart: false,
				isEnd: false,
			},{
				chapter: 4,
				custom: this.custom,//自定义页面
				title: '第四章',
				isStart: false,
				isEnd: false,
			},{
				chapter: 5,
				content: this.getContent(5),
				title: '第五章',
				isStart: false,
				isEnd: false,
			}]
			const { page } = this.$refs;
			if ( this.noChapter ) {
				page.init({
					content: this.getContent(2),
					start: 0
				})
			} else {
				page.init({
					contents: contents,
					start: 0,
					currentChapter: 4
				})
			}
		},
		methods: {
			clickTo () {
				console.log('点击')
			},
			currentChange (e) {
				this.currentPage = e.currentPage
				this.totalPage = e.totalPage
				console.log(e);
			},
			setCatalog (e) {
				console.log(e);
			},
			//自定义页面内的点击事件
			clickme (num, num2) {
				uni.showToast({
					icon: 'none',
					title: '你点击了我'
				})
			},
			//自定义页面内的点击事件
			clickher (num, num2) {
				uni.showToast({
					icon: 'none',
					title: '你点击了她'
				})
			},
			addFontsize () {
				this.fontsize += 4;
			},
			changePageType () {
				this.pageType = this.pageType == 'real' ? 'scroll' : 'real';
			},
			reduceFontSize () {
				this.fontsize -= 4;
			},
			changeLineHeight () {
				this.lineHeight += 4;
			},
			changeSkin () {
				this.color = '#f5f5f5';
				this.bgColor = '#999';
			},
			changeChapter () {
				if ( this.noChapter ) {
					page.change({
						start: 100
					})
				} else {
					setTimeout(() => {
						let contents = [{
							chapter: 3,
							content: this.getContent(3),
							title: '第三章',
							isStart: false,
							isEnd: false
						},{
							chapter: 4,
							custom: this.custom,
							title: '第四章',
							isStart: false,
							isEnd: false
						},{
							chapter: 5,
							content: this.getContent(5),
							title: '第五章',
							isStart: false,
							isEnd: false
						}]
						const { page } = this.$refs;
						page.change({
							contents: contents,
							start: 0,
							currentChapter: 5
						})
					}, 2000)
				}
			},
			loadmoreContent (chapter, callback) {
				setTimeout(() => {
					callback('success', {
						chapter: chapter,
						content: chapter == 4 ? '' : this.getContent(chapter),
						custom: chapter == 4 ? this.custom : [],
						title: '第' + chapter + '章',
						isStart: chapter == 1,
						isEnd: chapter == 7
					});
					// callback('fail');
					// callback('timeout');
				}, 2000)
			},
			preloadContent (chapters, callback) {
				setTimeout(() => {
					let contents = []
					for ( let i in chapters ) {
						contents.push({
							chapter: chapters[i],
							start: 0,
							content: chapters[i] == 4 ? '' : this.getContent(chapters[i]),
							custom: chapters[i] == 4 ? this.custom : [],
							title: '第' + chapters[i] + '章',
							isStart: chapters[i] == 1,
							isEnd: chapters[i] == 7
						})
					}
					callback('success', contents);
					// callback('fail');
					// callback('timeout');
				}, 2000)
			},
			getContent (chapter = 1) {
return `第${chapter}章
                你们好啊你们好啊你们好啊你们好啊你们好啊你们好啊你们好啊
你们好啊你们好啊你们好啊你们好啊你们好啊你们好啊你们好啊
你们好啊你们好啊你们好啊你们好啊你们好啊你们好啊你们好啊
你们好啊你们好啊你们好啊你们好啊你们好啊你们好啊你们好啊
你们好啊你们好啊你们好啊你们好啊你们好啊你们好啊你们好啊
你们好啊你们好啊你们好啊你们好啊你们好啊你们好啊你们好啊
你们好啊你们好啊你们好啊你们好啊你们好啊你们好啊你们好啊
你们好啊你们好啊你们好啊你们好啊你们好啊你们好啊你们好啊
你们好啊你们好啊你们好啊你们好啊你们好啊你们好啊你们好啊
你们好啊你们好啊你们好啊你们好啊你们好啊你们好啊你们好啊
你们好啊你们好啊你们好啊你们好啊你们好啊你们好啊你们好啊
你们好啊你们好啊你们好啊你们好啊你们好啊你们好啊你们好啊
你们好啊你们好啊你们好啊你们好啊你们好啊你们好啊你们好啊
你们好啊你们好啊你们好啊你们好啊你们好啊你们好啊你们好啊
你们好啊你们好啊你们好啊你们好啊你们好啊你们好啊你们好啊

神经病啊神经病啊神经病啊神经病啊神经病啊神经病啊神经病啊
神经病啊神经病啊神经病啊神经病啊神经病啊神经病啊神经病啊
神经病啊神经病啊神经病啊神经病啊神经病啊神经病啊神经病啊
神经病啊神经病啊神经病啊神经病啊神经病啊神经病啊神经病啊
神经病啊神经病啊神经病啊神经病啊神经病啊神经病啊神经病啊
神经病啊神经病啊神经病啊神经病啊神经病啊神经病啊神经病啊
神经病啊神经病啊神经病啊神经病啊神经病啊神经病啊神经病啊
神经病啊神经病啊神经病啊神经病啊神经病啊神经病啊神经病啊
神经病啊神经病啊神经病啊神经病啊神经病啊神经病啊神经病啊
神经病啊神经病啊神经病啊神经病啊神经病啊神经病啊神经病啊
神经病啊神经病啊神经病啊神经病啊神经病啊神经病啊神经病啊

疯子啊疯子啊疯子啊疯子啊疯子啊疯子啊疯子啊疯子啊疯子啊啊
疯子啊疯子啊疯子啊疯子啊疯子啊疯子啊疯子啊疯子啊疯子啊啊
疯子啊疯子啊疯子啊疯子啊疯子啊疯子啊疯子啊疯子啊疯子啊啊
疯子啊疯子啊疯子啊疯子啊疯子啊疯子啊疯子啊疯子啊疯子啊啊
疯子啊疯子啊疯子啊疯子啊疯子啊疯子啊疯子啊疯子啊疯子啊啊
疯子啊疯子啊疯子啊疯子啊疯子啊疯子啊疯子啊疯子啊疯子啊啊
疯子啊疯子啊疯子啊疯子啊疯子啊疯子啊疯子啊疯子啊疯子啊啊
疯子啊疯子啊疯子啊疯子啊疯子啊疯子啊疯子啊疯子啊疯子啊啊
疯子啊疯子啊疯子啊疯子啊疯子啊疯子啊疯子啊疯子啊疯子啊啊
疯子啊疯子啊疯子啊疯子啊疯子啊疯子啊疯子啊疯子啊疯子啊啊
疯子啊疯子啊疯子啊疯子啊疯子啊疯子啊疯子啊疯子啊疯子啊啊
疯子啊疯子啊疯子啊疯子啊疯子啊疯子啊疯子啊疯子啊疯子啊啊
疯子啊疯子啊疯子啊疯子啊疯子啊疯子啊疯子啊疯子啊疯子啊啊
疯子啊疯子啊疯子啊疯子啊疯子啊疯子啊疯子啊疯子啊疯子啊啊

傻子啊傻子啊傻子啊傻子啊傻子啊傻子啊傻子啊傻子啊傻子啊啊
傻子啊傻子啊傻子啊傻子啊傻子啊傻子啊傻子啊傻子啊傻子啊啊
傻子啊傻子啊傻子啊傻子啊傻子啊傻子啊傻子啊傻子啊傻子啊啊
傻子啊傻子啊傻子啊傻子啊傻子啊傻子啊傻子啊傻子啊傻子啊啊
傻子啊傻子啊傻子啊傻子啊傻子啊傻子啊傻子啊傻子啊傻子啊啊
傻子啊傻子啊傻子啊傻子啊傻子啊傻子啊傻子啊傻子啊傻子啊啊
傻子啊傻子啊傻子啊傻子啊傻子啊傻子啊傻子啊傻子啊傻子啊啊
傻子啊傻子啊傻子啊傻子啊傻子啊傻子啊傻子啊傻子啊傻子啊啊
傻子啊傻子啊傻子啊傻子啊傻子啊傻子啊傻子啊傻子啊傻子啊啊
傻子啊傻子啊傻子啊傻子啊傻子啊傻子啊傻子啊傻子啊傻子啊啊
傻子啊傻子啊傻子啊傻子啊傻子啊傻子啊傻子啊傻子啊傻子啊啊
傻子啊傻子啊傻子啊傻子啊傻子啊傻子啊傻子啊傻子啊傻子啊啊
傻子啊傻子啊傻子啊傻子啊傻子啊傻子啊傻子啊傻子啊傻子啊啊
傻子啊傻子啊傻子啊傻子啊傻子啊傻子啊傻子啊傻子啊傻子啊啊

神经病啊神经病啊神经病啊神经病啊神经病啊神经病啊神经病啊
神经病啊神经病啊神经病啊神经病啊神经病啊神经病啊神经病啊
神经病啊神经病啊神经病啊神经病啊神经病啊神经病啊神经病啊
神经病啊神经病啊神经病啊神经病啊神经病啊神经病啊神经病啊
神经病啊神经病啊神经病啊神经病啊神经病啊神经病啊神经病啊
神经病啊神经病啊神经病啊神经病啊神经病啊神经病啊神经病啊
神经病啊神经病啊神经病啊神经病啊神经病啊神经病啊神经病啊
神经病啊神经病啊神经病啊神经病啊神经病啊神经病啊神经病啊
神经病啊神经病啊神经病啊神经病啊神经病啊神经病啊神经病啊
神经病啊神经病啊神经病啊神经病啊神经病啊神经病啊神经病啊
神经病啊神经病啊神经病啊神经病啊神经病啊神经病啊神经病啊

疯子啊疯子啊疯子啊疯子啊疯子啊疯子啊疯子啊疯子啊疯子啊啊
疯子啊疯子啊疯子啊疯子啊疯子啊疯子啊疯子啊疯子啊疯子啊啊
疯子啊疯子啊疯子啊疯子啊疯子啊疯子啊疯子啊疯子啊疯子啊啊
疯子啊疯子啊疯子啊疯子啊疯子啊疯子啊疯子啊疯子啊疯子啊啊
疯子啊疯子啊疯子啊疯子啊疯子啊疯子啊疯子啊疯子啊疯子啊啊
疯子啊疯子啊疯子啊疯子啊疯子啊疯子啊疯子啊疯子啊疯子啊啊
疯子啊疯子啊疯子啊疯子啊疯子啊疯子啊疯子啊疯子啊疯子啊啊
疯子啊疯子啊疯子啊疯子啊疯子啊疯子啊疯子啊疯子啊疯子啊啊
疯子啊疯子啊疯子啊疯子啊疯子啊疯子啊疯子啊疯子啊疯子啊啊
疯子啊疯子啊疯子啊疯子啊疯子啊疯子啊疯子啊疯子啊疯子啊啊
疯子啊疯子啊疯子啊疯子啊疯子啊疯子啊疯子啊疯子啊疯子啊啊
疯子啊疯子啊疯子啊疯子啊疯子啊疯子啊疯子啊疯子啊疯子啊啊
疯子啊疯子啊疯子啊疯子啊疯子啊疯子啊疯子啊疯子啊疯子啊啊
疯子啊疯子啊疯子啊疯子啊疯子啊疯子啊疯子啊疯子啊疯子啊啊

傻子啊傻子啊傻子啊傻子啊傻子啊傻子啊傻子啊傻子啊傻子啊啊
傻子啊傻子啊傻子啊傻子啊傻子啊傻子啊傻子啊傻子啊傻子啊啊
傻子啊傻子啊傻子啊傻子啊傻子啊傻子啊傻子啊傻子啊傻子啊啊
傻子啊傻子啊傻子啊傻子啊傻子啊傻子啊傻子啊傻子啊傻子啊啊
傻子啊傻子啊傻子啊傻子啊傻子啊傻子啊傻子啊傻子啊傻子啊啊
傻子啊傻子啊傻子啊傻子啊傻子啊傻子啊傻子啊傻子啊傻子啊啊
傻子啊傻子啊傻子啊傻子啊傻子啊傻子啊傻子啊傻子啊傻子啊啊
傻子啊傻子啊傻子啊傻子啊傻子啊傻子啊傻子啊傻子啊傻子啊啊
傻子啊傻子啊傻子啊傻子啊傻子啊傻子啊傻子啊傻子啊傻子啊啊
傻子啊傻子啊傻子啊傻子啊傻子啊傻子啊傻子啊傻子啊傻子啊啊
傻子啊傻子啊傻子啊傻子啊傻子啊傻子啊傻子啊傻子啊傻子啊啊
傻子啊傻子啊傻子啊傻子啊傻子啊傻子啊傻子啊傻子啊傻子啊啊
傻子啊傻子啊傻子啊傻子啊傻子啊傻子啊傻子啊傻子啊傻子啊啊
傻子啊傻子啊傻子啊傻子啊傻子啊傻子啊傻子啊傻子啊傻子啊啊

傻子啊傻子啊傻子啊傻子啊傻子啊傻子啊傻子啊傻子啊傻子啊啊
傻子啊傻子啊傻子啊傻子啊傻子啊傻子啊傻子啊傻子啊傻子啊啊
傻子啊傻子啊傻子啊傻子啊傻子啊傻子啊傻子啊傻子啊傻子啊啊
傻子啊傻子啊傻子啊傻子啊傻子啊傻子啊傻子啊傻子啊傻子啊啊
傻子啊傻子啊傻子啊傻子啊傻子啊傻子啊傻子啊傻子啊傻子啊啊
傻子啊傻子啊傻子啊傻子啊傻子啊傻子啊傻子啊傻子啊傻子啊啊
傻子啊傻子啊傻子啊傻子啊傻子啊傻子啊傻子啊傻子啊傻子啊啊
傻子啊傻子啊傻子啊傻子啊傻子啊傻子啊傻子啊傻子啊傻子啊啊
傻子啊傻子啊傻子啊傻子啊傻子啊傻子啊傻子啊傻子啊傻子啊啊
傻子啊傻子啊傻子啊傻子啊傻子啊傻子啊傻子啊傻子啊傻子啊啊
傻子啊傻子啊傻子啊傻子啊傻子啊傻子啊傻子啊傻子啊傻子啊啊
傻子啊傻子啊傻子啊傻子啊傻子啊傻子啊傻子啊傻子啊傻子啊啊
傻子啊傻子啊傻子啊傻子啊傻子啊傻子啊傻子啊傻子啊傻子啊啊
傻子啊傻子啊傻子啊傻子啊傻子啊傻子啊傻子啊傻子啊傻子啊啊
`
			}
		}
	}
</script>

<style scoped>
	.pageInfo {
		position: fixed;
		left: 20rpx;
		bottom: 20rpx;
		padding: 10rpx 20rpx;
		border-radius: 20rpx;
		background-color: rgba(0,0,0,0.5);
		z-index: 99;
		color: white;
		font-size: 16rpx;
	}
</style>
