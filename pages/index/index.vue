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
		no-chapter
		@loadmore="loadmoreContent"
		@preload="preloadContent"
		@currentChange="currentChange"
		@setCatalog="setCatalog"></yingbing-ReadPage>
	</view>
</template>

<script>
	export default {
		data() {
			return {
				pages: [],
				pageType: 'real',
				scrollTop: 400,
				fontsize: 15,
				lineHeight: 15,
				color: '#333',
				slide: 40,
				bgColor: '#fcd281',
				enablePreload: true
			}
		},
		onReady() {
			let contents = [{
				chapter: 3,
				content: this.getContent(3),
				isStart: false,
				isEnd: false
			}]
			const { page } = this.$refs;
			page.init({
				contents: contents,
				start: 730,
				currentChapter: 3
			})
		},
		methods: {
			currentChange (e) {
				console.log(e);
			},
			setCatalog (e) {
				console.log(e);
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
				let contents = [{
					chapter: 3,
					content: this.getContent(3),
					isEnd: false
				},{
					chapter: 4,
					content: this.getContent(4),
					isEnd: false
				},{
					chapter: 5,
					content: this.getContent(5),
					isEnd: false
				}]
				const { page } = this.$refs;
				page.change({
					contents: contents,
					start: 600,
					current: 5
				})
			},
			loadmoreContent (chapter, callback) {
				setTimeout(() => {
					callback('success', {
						chapter: chapter,
						content: this.getContent(chapter),
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
							content: this.getContent(chapters[i]),
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
`
			}
		}
	}
</script>

<style scoped>
</style>
