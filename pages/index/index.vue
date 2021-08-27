<template>
	<view class="scroll">
		<page
		ref="page"
		:page-type="pageType"
		:font-size="fontsize"
		:line-height="lineHeight"
		:color="color"
		:bg-color="bgColor"
		:slide="slide"
		no-chapter
		@loadmore="loadmoreContent"
		@preload="preloadContent"
		@currentChange="currentChange"></page>
	</view>
</template>

<script>
	import Page from '@/components/page/page.vue'
	export default {
		data() {
			return {
				pages: [],
				list: [],
				pageType: 'real',
				scrollTop: 400,
				fontsize: 15,
				lineHeight: 15,
				color: '#333',
				slide: 40,
				bgColor: '#fcd281'
			}
		},
		onLoad() {
			this.list.push({
				start: 0
			})
		},
		onReady() {
			let contents = [{
				chapter: 1,
				start: 0,
				content: this.getContent(1) + this.getContent(2) + this.getContent(3),
				isEnd: false
			},{
				chapter: 2,
				start: 0,
				content: this.getContent(2),
				isEnd: false
			},{
				chapter: 3,
				start: 100,
				content: this.getContent(3),
				isEnd: false
			}]
			const { page } = this.$refs;
			page.init({
				contents: contents,
				current: 1
			})
		},
		methods: {
			currentChange (e) {
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
					start: 0,
					content: this.getContent(3),
					isEnd: false
				},{
					chapter: 4,
					start: 0,
					content: this.getContent(4),
					isEnd: false
				},{
					chapter: 5,
					start: 0,
					content: this.getContent(5),
					isEnd: false
				}]
				const { page } = this.$refs;
				page.change({
					contents: contents,
					current: 5
				})
			},
			loadmoreContent (chapter, next) {
				console.log(chapter);
				next({
					chapter: chapter,
					start: 0,
					content: this.getContent(chapter),
					isEnd: chapter == 7
				});
			},
			preloadContent (chapters, next) {
				let contents = []
				for ( let i in chapters ) {
					contents.push({
						chapter: chapters[i],
						start: 0,
						content: this.getContent(chapters[i]),
						isEnd: chapters[i] == 7
					})
				}
				setTimeout(() => {
					next(contents);
				}, 500)
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
		},
		components: {
			Page
		}
	}
</script>

<style scoped>
</style>
