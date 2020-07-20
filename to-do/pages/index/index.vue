<template>
	<view class="content">
		<!-- 状态栏 -->
		<view v-if="list.length!==0" class="todo-header">
			<!-- 状态栏的左侧 -->
			<view class="todo-header__left">
				<text class="active-text">{{text}}</text>
				<text>{{listData.length}}条</text>
			</view>
			<!-- 状态栏的右侧 -->
			<view class="todo-header__right">
				<view class="todo-header__right-item" :class="{'active-tab':activeIndex===0}" @click="tab(0)">全部</view>
				<view class="todo-header__right-item" :class="{'active-tab':activeIndex===1}" @click="tab(1)">待办</view>
				<view class="todo-header__right-item" :class="{'active-tab':activeIndex===2}" @click="tab(2)">已完成</view>
			</view>
		</view>
		<!-- 没有数据的页面 -->
		<view v-if="list.length===0" class="default">
			<view class="image-default">
				<image src="../../static/default.png" mode="aspectFit"></image>
			</view>
			<view class="default-info">
				<view class="default-info__text">您还没有创建任何待办事项</view>
				<view class="default-info__text">点击+号创建一个吧</view>
			</view>
		</view>
		<!-- 代办卡片内容部分 -->
		<view v-else class="todo-content">
			<view class="todo-list" :class="{'todo__finish':item.checked}" v-for="(item,index) in listData" :key="index" @click="finish(item.id)">
				<view class="todo-list__checkbox">
					<view class="checkbox"></view>
				</view>
				<view class="todo-list__content">
					{{item.content}}
				</view>
			</view>
		</view>
		<!-- 底部——加号创建按钮 -->
		<view class="create-todo" @click="create">
			<text class="iconfont icon-add" :class="{'create-todo-active':textShow}"></text>
		</view>
		<!-- 创建代办事件输入框部分 -->
		<view v-if="active" class="create-content" :class="{'create__show':textShow}">
			<view class="create-content-box">
				<!-- 输入部分 -->
				<view class="create-input">
					<input type="text"  v-model="value" placeholder="请输入您要创建的事项"/>
				</view>
				<!-- 发布按钮 -->
				<view class="create-button" @click="add">
					创建
				</view>
			</view>
		</view>
	</view>
</template>

<script>
	export default {
		data() {
			return {
				value:'',  //输入框输入的内容
				list:[],   //卡片列表
				active:false,  //控制输入框是否显示
				activeIndex: 0  ,//控制选项卡为全部 待办 还是已完成
				
				text: '全部'  ,//控制状态栏左侧的文字是显示 全部 待办 还是已完成
				textShow: false
			}
		},
		onLoad() {

		},
		computed:{
			listData(){
				let list = JSON.parse(JSON.stringify(this.list))  // 数据拷贝
				let newList = []  // 放计算属性处理过后的数据
				
				// 点击 全部
				if(this.activeIndex === 0){
					this.text = '全部'
					return list
				}
				// 点击 待办
				if(this.activeIndex === 1){
					// 即 checked = false 的时候
					this.text = '待办'
					list.forEach((item) => {
						if(item.checked === false){
							newList.push(item)
						}
					})
					return newList
				}
				// 点击 已完成
				if(this.activeIndex === 2){
					// 即 checked = true 的时候
					this.text = '已完成'
					list.forEach((item) => {
						if(item.checked === true){
							newList.push(item)
						}
					})
					return newList
				}
				return []
			}
		},
		methods: {
			// 打开创建事件输入框
			create() {
				if (this.active) {
					this.close()
				} else {
					this.open()
				}
			},
			// 打开 动画
			open() {
				this.active = true
				this.$nextTick(() => {
					setTimeout(() => {
						this.textShow = true
					}, 50)
				})
			},
			// 关闭动画
			close() {
				this.textShow = false
				this.$nextTick(()=>{
					setTimeout(() => {
						this.active = false
					}, 350)
				})
			},
			//发布待办
			add(){
				console.log(this.value);
				if(this.value ===''){
					// 如果提交空值,则调用uni-app的一个api,提示请输入内容
					uni.showToast({
						title:"输入内容不能为空",
						icon:'none'
					})
					return
				}
				// 把新数据插到数组的最前面
				this.list.unshift({
					//使用时间戳来当作每个卡片的唯一id
					id:'id'+new Date().getTime(),  
					content: this.value,
					// 用于控制是否为已完成状态,  刚创建出来默认为false即未完成 true为已完成
					checked: false  
				})
				// 点击创建之后输入框置空
				this.value = ''
				// 收起输入框
				this.close()
			},
			// 点击待办列表完成
			finish(id){
				// 拿到index: list中id和点击的id一样时的索引
				let index = this.list.findIndex((item) => item.id === id)
				console.log('I am on click',this.list[index])
				this.list[index].checked = !this.list[index].checked
			},
			tab(index){
				this.activeIndex = index
			}
		}
	}
</script>

<style>
	@import '../../common/icon.css';
	.todo-header {
		position: fixed;
		top: 0;
		left: 0;
		display: flex;
		align-items: center;
		padding: 0 15px;
		font-size: 12px;
		color: #333333;
		width: 100%;
		height: 45px;
		box-sizing: border-box;
		box-shadow: -1px 1px 5px 0 rgba(0, 0, 0, 0.1);
		background: #FFFFFF;
		z-index: 10;
	}

	.todo-header__left {
		width: 100%;
	}
	.active-text{
		font-size: 13px;
		color: #279ABF;
		padding-right: 5px;
	}
	.todo-header__right {
		flex-shrink: 0;
		display: flex;
	}
	.todo-header__right-item {
		padding: 0 5px;
	}
	.active-tab{
		color: #279abf;
	}
	
	.todo-content{
		position: relative;
		padding-top: 50px;
		padding-bottom: 100px;
	}
	/* 卡片样式 */
	.todo-list{
		display: flex;
		align-items: center;
		position: relative;
		padding: 15px;
		margin: 15px;
		border-radius: 10px;  /*圆角*/
		color: #0c3854;    /*字体颜色*/
		font-size: 14px;
		background: #cfebfd;
		box-shadow: -1px 1px 5px 1px rgba(0,0,0,0.1)/* , -1px 2px 1px 0 rgba(255,255,255) inset */;
		overflow: hidden;  /*裁剪溢出元素的内容*/
	}
	/* 使用伪类实现左侧小圆圈效果 */
	.todo-list::after{
		position: absolute;
		content: '';
		top: 0;
		left: 0;
		bottom: 0;
		width: 5px;
		background: #91d1e8;
	}
	.checkbox{
		width: 20px;
		height: 20px;
		border-radius: 50%;
		background: #FFFFFF;
		box-shadow: 0 0 5px 1px rgba(0,0,0,0.1);
	}
	.todo-list__checkbox{
		padding-right: 15px;
	}
	/* 代办事项完成后的样式 */
	.todo__finish .checkbox{
		position: relative;
		background: #eee;
		
	}
	/* 伪类实现代办完成后灰色圆球中的蓝色小圆圈 */
	.todo__finish .checkbox:after{
		content: '';
		position: absolute;
		width: 10px;
		height: 10px;
		border-radius: 50%;
		top: 0;
		left: 0;
		right: 0;
		bottom: 0;
		margin: auto;
		background: #CFEBFD;
		box-shadow: 0 0 2px 0px rgba(0,0,0,0.2) inset; 
	}
	.todo__finish .todo-list__content {
		color: #999;
	}
	.todo__finish.todo-list::before {
		content: '';
		position: absolute;
		top: 0;
		bottom: 0;
		left: 40px;
		right: 10px;
		height: 2px;
		margin: auto 0;
		background: #bdcdd8;
	}
	.todo__finish.todo-list::after{
		background: #CCCCCC;
	}
	.create-todo{
		display: flex;
		align-items: center;   /*垂直居中*/
		justify-content: center;  /*水平居中*/
 		margin: 0 auto;
		position: fixed;
		bottom: 20px;
		left: 0;
		right: 0;
		width: 50px;
		height: 50px;
		border-radius:50% ;
		background-color: #deeff5;
		box-shadow: -1px 1px 5px 2px rgba(0,0,0,0.1), -1px 1px 1px 0 rgba(255,255,255);
	}
	.icon-add{
		font-size: 35px;
		color: #add8e6;
	}
	.create-content{
		position: fixed;
		bottom: 95px;
		left: 20px;
		right: 20px;
		
		transition: all 0.3s;
		opacity: 0;
		transform: scale(0) translateY(200%);
	}
	.create__show{
		opacity: 1;
		transform: scale(1) translateY(0);
	}
	.create-content-box{
		display: flex;
		align-items: center;
		padding:0 15px;
		padding-right: 0;
		border-radius:50px;
		background: #deeff5;
		box-shadow: -1px 1px 5px 2px rgba(0,0,0,0.1)/* , -1px 1px 1px 0 rgba(255,255,255) inset */;
	}
	/* 两个伪类实现添加事项框的叠加箭头 */
	.create-content::after{
		content: '';
		position: absolute;
		right: 0;
		left: 0;
		bottom: -8px;
		width: 20px;
		height: 20px;
		margin: 0 auto;
		background: #deeff5;
		transform: rotate(45deg);
		box-shadow: 1px 1px 5px 2px rgba(0,0,0,0.1);
		z-index: -1;
	}
	.create-content-box::after{
		content: '';
		position: absolute;
		right: 0;
		left: 0;
		bottom: -8px;
		width: 20px;
		height: 20px;
		margin: 0 auto;
		background: #deeff5;
		transform: rotate(45deg);
	}
	
	.create-input{
		width: 100%;
		padding-right:15px ;
		color:  #7fb7ec;
	}
	.create-button{
		display: flex;
		justify-content: center;
		align-items: center;
		flex-shrink: 0 ;  /*让元素不被挤压*/
		width: 80px;
		height: 50px;
		border-radius: 40px;
		font-size: 16px;
		color: #88d4ec;
		box-shadow: -1px 0px 1px 1px rgba(0,0,0,0.1);
	}
	
	.default{
		padding-top: 100px;
	}
	.image-default{
		display: flex;
		justify-content: center;
	}
	.image-default image{
		width: 100%;
	}
	.default-info{
		text-align: center;
		font-size: 14px;
		color: #CCCCCC;
	}
	.icon-add{
		transform: transform 0.3s;
	}
	.create-todo-active{
		transform: rotate(135deg);
	}
</style>
