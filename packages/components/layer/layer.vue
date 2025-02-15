<template>
	<transition :name="(animation?animation:'mvi-layer')" @before-enter="beforeEnter" @enter="enter" @after-enter="afterEnter" @leave="leave"
	@before-leave="beforeLeave" @after-leave="afterLeave">
		<div v-if="firstShow" v-show="layerShow" v-on="listeners" :class="['mvi-layer',fixed?'mvi-layer-fixed':'']" :style="layerStyle">
			<div :class="wrapperCls" :style="wrapperStyle">
				<m-triangle v-if="showTriangle" ref="triangle" class="mvi-layer-triangle" :placement="trianglePlacement" :background="background" :border-color="(border&&borderColor?borderColor:background)" size="0.14rem"></m-triangle>
				<slot></slot>
			</div>
		</div>
	</transition>
</template>

<script>
	import $util from "../../util/util"
	import mTriangle from "../triangle/triangle"
	export default {
		name: 'm-layer',
		data(){
			return {
				layerShow:false,
				firstShow:false,
				realPlacement:'bottom'
			}
		},
		model:{
			prop:'show',
			event:'model-change'
		},
		props: {
			//是否显示悬浮层
			show:{
				type:Boolean,
				default:false
			},
			//悬浮层触发元素
			target: {
				type: String,
				default: null
			},
			//悬浮层定位父元素
			root:{
				type:String,
				default:null
			},
			//悬浮层相对于触发元素的位置
			placement: {
				type: String,
				default: 'bottom',
				validator(value) {
					return ['top', 'left', 'right', 'bottom', 'top-start',
						'top-end', 'left-start', 'left-end', 'right-start', 'right-end', 'bottom-start',
						'bottom-end'
					].includes(value)
				}
			},
			//悬浮层主体距离触发元素的距离
			offset:{
				type:String,
				default:'0.2rem'
			},
			//悬浮层的z-index
			zIndex:{
				type:Number,
				default:null
			},
			//悬浮层是否为fixed模式
			fixed:{
				type:Boolean,
				default:false
			},
			//悬浮层的宽度
			width:{
				type:String,
				default:null
			},
			//悬浮层的主体额外样式
			wrapperClass:{
				type:String,
				default:null
			},
			//悬浮层显示与隐藏动画时长
			timeout:{
				type:Number,
				default:300
			},
			//点击触发元素和悬浮层以外的元素是否关闭悬浮层
			closable:{
				type:Boolean,
				default:false
			},
			//是否显示三角
			showTriangle:{
				type:Boolean,
				default:false
			},
			//自定义动画
			animation:{
				type:String,
				default:null
			},
			//是否显示阴影
			shadow:{
				type:Boolean,
				default:true
			},
			//是否显示边框
			border:{
				type:Boolean,
				default:false
			},
			//边框颜色
			borderColor:{
				type:String,
				default:'#eee'
			},
			//背景色
			background:{
				type:String,
				default:'#fff'
			},
			//适配transform父元素
			fixedAuto:{
				type:Boolean,
				default:false
			}
		},
		components:{
			mTriangle
		},
		watch:{
			placement(newValue){
				if(this.layerShow && this.firstShow){
					this.autoAdjust()
					this.reset()
					this.$nextTick(()=>{
						this.resetTriangle()
					})
				}
			},
			show(newValue){
				if(newValue){
					if(!this.firstShow){
						this.firstShow = true;
					}
				}
				this.layerShow = newValue;
			},
		},
		computed: {
			listeners(){
				return Object.assign({},this.$listeners);
			},
			trianglePlacement(){
				if(this.realPlacement == 'bottom-start' || this.realPlacement == 'bottom' || this.realPlacement == 'bottom-end'){
					return 'top'
				}else if(this.realPlacement == 'top-start' || this.realPlacement == 'top' || this.realPlacement == 'top-end'){
					return 'bottom'
				}else if(this.realPlacement == 'left-start' || this.realPlacement == 'left' || this.realPlacement == 'left-end'){
					return 'right'
				}else if(this.realPlacement == 'right-start' || this.realPlacement == 'right' || this.realPlacement == 'right-end'){
					return 'left'
				}
			},
			wrapperCls(){
				let cls = ['mvi-layer-wrapper'];
				if(this.shadow){
					cls.push('mvi-layer-shadow');
				}
				if(this.border){
					cls.push('mvi-layer-border');
				}
				if(this.wrapperClass){
					cls.push(this.wrapperClass)
				}
				return cls;
			},
			wrapperStyle(){
				let style = {}
				if(this.border && this.borderColor){
					style.borderColor = this.borderColor;
				}
				if(this.background){
					style.background = this.background;
				}
				if(this.width){
					style.width = this.width;
				}
				return style;
			},
			layerStyle(){
				let style = {};
				if(this.zIndex){
					style.zIndex = this.zIndex;
				}
				if(this.timeout){
					style.transition = 'transform ' + this.timeout + 'ms,opacity ' + this.timeout + 'ms';
					style.webkitTransition = 'transform ' + this.timeout + 'ms,opacity ' + this.timeout + 'ms';
				}
				if(!this.showTriangle){
					style.padding = 0;
					if(this.realPlacement == 'bottom-start' || this.realPlacement == 'bottom' || this.realPlacement == 'bottom-end'){
						style.paddingTop = this.offset;
					}else if(this.realPlacement == 'top-start' || this.realPlacement == 'top' || this.realPlacement == 'top-end'){
						style.paddingBottom = this.offset;
					}else if(this.realPlacement == 'left-start' || this.realPlacement == 'left' || this.realPlacement == 'left-end'){
						style.paddingRight = this.offset;
					}else if(this.realPlacement == 'right-start' || this.realPlacement == 'right' || this.realPlacement == 'right-end'){
						style.paddingLeft = this.offset;
					}
				}
				return style;
			}
		},
		mounted() {
			//初始化时是否显示
			if(this.show){
				if(!this.firstShow){
					this.firstShow = true;
				}
				this.layerShow = this.show;
			}
			//添加事件
			window.on(`resize.layer_${this._uid}`,this.resizeSet);
			window.on(`click.layer_${this._uid}`,this.hideLayer);
		},
		methods: {
			//窗口变化时处理
			resizeSet(){
				if(this.layerShow && this.firstShow){
					this.autoAdjust()
					this.reset()
					this.$nextTick(()=>{
						this.resetTriangle()
					})
				}
			},
			//悬浮层显示位置智能化
			autoAdjust(){
				//获取绑定元素位置
				let $target = this.getTargetEl();
				let point = $util.getElementBounding($target);
				this.realPlacement = this.placement;
				if(this.placement == 'bottom'){
					//如果底部距离不够
					if(point.bottom < this.$el.offsetHeight){
						//顶部距离够
						if(point.top > this.$el.offsetHeight){
							this.realPlacement = 'top';
							if(point.left + $target.offsetWidth/2 < this.$el.offsetWidth/2){
								this.realPlacement = 'top-start'
							}
							if(point.right + $target.offsetWidth/2 < this.$el.offsetWidth/2){
								this.realPlacement = 'top-end'
							}
						}else if(point.left > this.$el.offsetWidth){//左侧距离够
							this.realPlacement = 'left';
							if(point.top + $target.offsetHeight/2 < this.$el.offsetHeight/2){
								this.realPlacement = 'left-start'
							}
							if(point.bottom + $target.offsetHeight/2 < this.$el.offsetHeight/2){
								this.realPlacement = 'left-end'
							}
						}else if(point.right > this.$el.offsetWidth){//右侧距离够
							this.realPlacement = 'right';
							if(point.top + $target.offsetHeight/2 < this.$el.offsetHeight/2){
								this.realPlacement = 'right-start'
							}
							if(point.bottom + $target.offsetHeight/2 < this.$el.offsetHeight/2){
								this.realPlacement = 'right-end'
							}
						}
					}else {
						if(point.left + $target.offsetWidth/2 < this.$el.offsetWidth/2){
							this.realPlacement = 'bottom-start'
						}
						if(point.right + $target.offsetWidth/2 < this.$el.offsetWidth/2){
							this.realPlacement = 'bottom-end'
						}
					}
				}else if(this.placement == 'bottom-start'){
					//如果底部距离不够
					if(point.bottom < this.$el.offsetHeight){
						//顶部距离够
						if(point.top > this.$el.offsetHeight){
							this.realPlacement = 'top-start';
							if(point.right + $target.offsetWidth < this.$el.offsetWidth){
								this.realPlacement = 'top'
							}
							if(point.right + $target.offsetWidth/2 < this.$el.offsetWidth/2){
								this.realPlacement = 'top-end'
							}
						}else if(point.left > this.$el.offsetWidth){//左侧距离够
							this.realPlacement = 'left';
							if(point.top + $target.offsetHeight/2 < this.$el.offsetHeight/2){
								this.realPlacement = 'left-start'
							}
							if(point.bottom + $target.offsetHeight/2 < this.$el.offsetHeight/2){
								this.realPlacement = 'left-end'
							}
						}else if(point.right > this.$el.offsetWidth){//右侧距离够
							this.realPlacement = 'right';
							if(point.top + $target.offsetHeight/2 < this.$el.offsetHeight/2){
								this.realPlacement = 'right-start'
							}
							if(point.bottom + $target.offsetHeight/2 < this.$el.offsetHeight/2){
								this.realPlacement = 'right-end'
							}
						}
					}else {
						if(point.right + $target.offsetWidth < this.$el.offsetWidth){
							this.realPlacement = 'bottom'
						}
						if(point.right + $target.offsetWidth/2 < this.$el.offsetWidth/2){
							this.realPlacement = 'bottom-end'
						}
					}
				}else if(this.placement == 'bottom-end'){
					//如果底部距离不够
					if(point.bottom < this.$el.offsetHeight){
						//顶部距离够
						if(point.top > this.$el.offsetHeight){
							this.realPlacement = 'top-end';
							if(point.left + $target.offsetWidth < this.$el.offsetWidth){
								this.realPlacement = 'top'
							}
							if(point.left + $target.offsetWidth/2 < this.$el.offsetWidth/2){
								this.realPlacement = 'top-start'
							}
						}else if(point.left > this.$el.offsetWidth){//左侧距离够
							this.realPlacement = 'left';
							if(point.top + $target.offsetHeight/2 < this.$el.offsetHeight/2){
								this.realPlacement = 'left-start'
							}
							if(point.bottom + $target.offsetHeight/2 < this.$el.offsetHeight/2){
								this.realPlacement = 'left-end'
							}
						}else if(point.right > this.$el.offsetWidth){//右侧距离够
							this.realPlacement = 'right';
							if(point.top + $target.offsetHeight/2 < this.$el.offsetHeight/2){
								this.realPlacement = 'right-start'
							}
							if(point.bottom + $target.offsetHeight/2 < this.$el.offsetHeight/2){
								this.realPlacement = 'right-end'
							}
						}
					}else {
						if(point.left + $target.offsetWidth < this.$el.offsetWidth){
							this.realPlacement = 'bottom'
						}
						if(point.left + $target.offsetWidth/2 < this.$el.offsetWidth/2){
							this.realPlacement = 'bottom-start'
						}
					}
				}else if(this.placement == 'top'){
					//如果顶部距离不够
					if(point.top < this.$el.offsetHeight){
						//底部距离够
						if(point.bottom > this.$el.offsetHeight){
							this.realPlacement = 'bottom'
							if(point.left + $target.offsetWidth/2 < this.$el.offsetWidth/2){
								this.realPlacement = 'bottom-start'
							}
							if(point.right + $target.offsetWidth/2 < this.$el.offsetWidth/2){
								this.realPlacement = 'bottom-end'
							}
						}else if(point.left > this.$el.offsetWidth){//左侧距离够
							this.realPlacement = 'left';
							if(point.top + $target.offsetHeight/2 < this.$el.offsetHeight/2){
								this.realPlacement = 'left-start'
							}
							if(point.bottom + $target.offsetHeight/2 < this.$el.offsetHeight/2){
								this.realPlacement = 'left-end'
							}
						}else if(point.right > this.$el.offsetWidth){//右侧距离够
							this.realPlacement = 'right';
							if(point.top + $target.offsetHeight/2 < this.$el.offsetHeight/2){
								this.realPlacement = 'right-start'
							}
							if(point.bottom + $target.offsetHeight/2 < this.$el.offsetHeight/2){
								this.realPlacement = 'right-end'
							}
						}
					}else {
						if(point.left + $target.offsetWidth/2 < this.$el.offsetWidth/2){
							this.realPlacement = 'top-start'
						}
						if(point.right + $target.offsetWidth/2 < this.$el.offsetWidth/2){
							this.realPlacement = 'top-end'
						}
					}
				}else if(this.placement == 'top-start'){
					//如果顶部距离不够
					if(point.top < this.$el.offsetHeight){
						//底部距离够
						if(point.bottom > this.$el.offsetHeight){
							this.realPlacement = 'bottom-start'
							if(point.right + $target.offsetWidth < this.$el.offsetWidth){
								this.realPlacement = 'bottom'
							}
							if(point.right + $target.offsetWidth/2 < this.$el.offsetWidth/2){
								this.realPlacement = 'bottom-end'
							}
						}else if(point.left > this.$el.offsetWidth){//左侧距离够
							this.realPlacement = 'left';
							if(point.top + $target.offsetHeight/2 < this.$el.offsetHeight/2){
								this.realPlacement = 'left-start'
							}
							if(point.bottom + $target.offsetHeight/2 < this.$el.offsetHeight/2){
								this.realPlacement = 'left-end'
							}
						}else if(point.right > this.$el.offsetWidth){//右侧距离够
							this.realPlacement = 'right';
							if(point.top + $target.offsetHeight/2 < this.$el.offsetHeight/2){
								this.realPlacement = 'right-start'
							}
							if(point.bottom + $target.offsetHeight/2 < this.$el.offsetHeight/2){
								this.realPlacement = 'right-end'
							}
						}
					}else {
						if(point.right + $target.offsetWidth < this.$el.offsetWidth){
							this.realPlacement = 'top'
						}
						if(point.right + $target.offsetWidth/2 < this.$el.offsetWidth/2){
							this.realPlacement = 'top-end'
						}
					}
				}else if(this.placement == 'top-end'){
					//如果顶部距离不够
					if(point.top < this.$el.offsetHeight){
						//底部距离够
						if(point.bottom > this.$el.offsetHeight){
							this.realPlacement = 'bottom-end'
							if(point.left + $target.offsetWidth < this.$el.offsetWidth){
								this.realPlacement = 'bottom'
							}
							if(point.left + $target.offsetWidth/2 < this.$el.offsetWidth/2){
								this.realPlacement = 'bottom-start'
							}
						}else if(point.left > this.$el.offsetWidth){//左侧距离够
							this.realPlacement = 'left';
							if(point.top + $target.offsetHeight/2 < this.$el.offsetHeight/2){
								this.realPlacement = 'left-start'
							}
							if(point.bottom + $target.offsetHeight/2 < this.$el.offsetHeight/2){
								this.realPlacement = 'left-end'
							}
						}else if(point.right > this.$el.offsetWidth){//右侧距离够
							this.realPlacement = 'right';
							if(point.top + $target.offsetHeight/2 < this.$el.offsetHeight/2){
								this.realPlacement = 'right-start'
							}
							if(point.bottom + $target.offsetHeight/2 < this.$el.offsetHeight/2){
								this.realPlacement = 'right-end'
							}
						}
					}else {
						if(point.left + $target.offsetWidth < this.$el.offsetWidth){
							this.realPlacement = 'top'
						}
						if(point.left + $target.offsetWidth/2 < this.$el.offsetWidth/2){
							this.realPlacement = 'top-start'
						}
					}
				}else if(this.placement == 'left'){
					//如果左侧距离不够
					if(point.left < this.$el.offsetWidth){
						//右侧距离够
						if(point.right > this.$el.offsetWidth){
							this.realPlacement = 'right'
							if(point.top + $target.offsetHeight/2 < this.$el.offsetHeight/2){
								this.realPlacement = 'right-start'
							}
							if(point.bottom + $target.offsetHeight/2 < this.$el.offsetHeight/2){
								this.realPlacement = 'right-end'
							}
						}else if(point.top > this.$el.offsetHeight){//顶部距离够
							this.realPlacement = 'top'
							if(point.left + $target.offsetWidth/2 < this.$el.offsetWidth/2){
								this.realPlacement = 'top-start'
							}
							if(point.right + $target.offsetWidth/2 < this.$el.offsetWidth/2){
								this.realPlacement = 'top-end'
							}
						}else if(point.bottom > this.$el.offsetHeight){//底部距离够
							this.realPlacement = 'bottom'
							if(point.left + $target.offsetWidth/2 < this.$el.offsetWidth/2){
								this.realPlacement = 'bottom-start'
							}
							if(point.right + $target.offsetWidth/2 < this.$el.offsetWidth/2){
								this.realPlacement = 'bottom-end'
							}
						}
					}else {
						if(point.top + $target.offsetHeight/2 < this.$el.offsetHeight/2){
							this.realPlacement = 'left-start'
						}
						if(point.bottom + $target.offsetHeight/2 < this.$el.offsetHeight/2){
							this.realPlacement = 'left-end'
						}
					}
				}else if(this.placement == 'left-start'){
					//如果左侧距离不够
					if(point.left < this.$el.offsetWidth){
						//右侧距离够
						if(point.right > this.$el.offsetWidth){
							this.realPlacement = 'right-start'
							if(point.bottom + $target.offsetHeight < this.$el.offsetHeight){
								this.realPlacement = 'right'
							}
							if(point.bottom + $target.offsetHeight/2 < this.$el.offsetHeight/2){
								this.realPlacement = 'right-end'
							}
						}else if(point.top > this.$el.offsetHeight){//顶部距离够
							this.realPlacement = 'top'
							if(point.left + $target.offsetWidth/2 < this.$el.offsetWidth/2){
								this.realPlacement = 'top-start'
							}
							if(point.right + $target.offsetWidth/2 < this.$el.offsetWidth/2){
								this.realPlacement = 'top-end'
							}
						}else if(point.bottom > this.$el.offsetHeight){//底部距离够
							this.realPlacement = 'bottom'
							if(point.left + $target.offsetWidth/2 < this.$el.offsetWidth/2){
								this.realPlacement = 'bottom-start'
							}
							if(point.right + $target.offsetWidth/2 < this.$el.offsetWidth/2){
								this.realPlacement = 'bottom-end'
							}
						}
					}else {
						if(point.bottom + $target.offsetHeight < this.$el.offsetHeight){
							this.realPlacement = 'left'
						}
						if(point.bottom + $target.offsetHeight/2 < this.$el.offsetHeight/2){
							this.realPlacement = 'left-end'
						}
					}
				}else if(this.placement == 'left-end'){
					//如果左侧距离不够
					if(point.left < this.$el.offsetWidth){
						//右侧距离够
						if(point.right > this.$el.offsetWidth){
							this.realPlacement = 'right-end'
							if(point.top + $target.offsetHeight < this.$el.offsetHeight){
								this.realPlacement = 'right'
							}
							if(point.top + $target.offsetHeight/2 < this.$el.offsetHeight/2){
								this.realPlacement = 'right-start'
							}
						}else if(point.top > this.$el.offsetHeight){//顶部距离够
							this.realPlacement = 'top'
							if(point.left + $target.offsetWidth/2 < this.$el.offsetWidth/2){
								this.realPlacement = 'top-start'
							}
							if(point.right + $target.offsetWidth/2 < this.$el.offsetWidth/2){
								this.realPlacement = 'top-end'
							}
						}else if(point.bottom > this.$el.offsetHeight){//底部距离够
							this.realPlacement = 'bottom'
							if(point.left + $target.offsetWidth/2 < this.$el.offsetWidth/2){
								this.realPlacement = 'bottom-start'
							}
							if(point.right + $target.offsetWidth/2 < this.$el.offsetWidth/2){
								this.realPlacement = 'bottom-end'
							}
						}
					}else {
						if(point.top + $target.offsetHeight < this.$el.offsetHeight){
							this.realPlacement = 'left'
						}
						if(point.top + $target.offsetHeight/2 < this.$el.offsetHeight/2){
							this.realPlacement = 'left-start'
						}
					}
				}else if(this.placement == 'right'){
					//如果右侧距离不够
					if(point.right < this.$el.offsetWidth){
						//左侧距离够
						if(point.left > this.$el.offsetWidth){
							this.realPlacement = 'left'
							if(point.top + $target.offsetHeight/2 < this.$el.offsetHeight/2){
								this.realPlacement = 'left-start'
							}
							if(point.bottom + $target.offsetHeight/2 < this.$el.offsetHeight/2){
								this.realPlacement = 'left-end'
							}
						}else if(point.top > this.$el.offsetHeight){//顶部距离够
							this.realPlacement = 'top'
							if(point.left + $target.offsetWidth/2 < this.$el.offsetWidth/2){
								this.realPlacement = 'top-start'
							}
							if(point.right + $target.offsetWidth/2 < this.$el.offsetWidth/2){
								this.realPlacement = 'top-end'
							}
						}else if(point.bottom > this.$el.offsetHeight){//底部距离够
							this.realPlacement = 'bottom'
							if(point.left + $target.offsetWidth/2 < this.$el.offsetWidth/2){
								this.realPlacement = 'bottom-start'
							}
							if(point.right + $target.offsetWidth/2 < this.$el.offsetWidth/2){
								this.realPlacement = 'bottom-end'
							}
						}
					}else {
						if(point.top + $target.offsetHeight/2 < this.$el.offsetHeight/2){
							this.realPlacement = 'right-start'
						}
						if(point.bottom + $target.offsetHeight/2 < this.$el.offsetHeight/2){
							this.realPlacement = 'right-end'
						}
					}
				}else if(this.placement == 'right-start'){
					//如果右侧距离不够
					if(point.right < this.$el.offsetWidth){
						//左侧距离足够
						if(point.left > this.$el.offsetWidth){
							this.realPlacement = 'left-start'
							if(point.bottom + $target.offsetHeight < this.$el.offsetHeight){
								this.realPlacement = 'left'
							}
							if(point.bottom + $target.offsetHeight/2 < this.$el.offsetHeight/2){
								this.realPlacement = 'left-end'
							}
						}else if(point.top > this.$el.offsetHeight){//顶部距离够
							this.realPlacement = 'top'
							if(point.left + $target.offsetWidth/2 < this.$el.offsetWidth/2){
								this.realPlacement = 'top-start'
							}
							if(point.right + $target.offsetWidth/2 < this.$el.offsetWidth/2){
								this.realPlacement = 'top-end'
							}
						}else if(point.bottom > this.$el.offsetHeight){//底部距离够
							this.realPlacement = 'bottom'
							if(point.left + $target.offsetWidth/2 < this.$el.offsetWidth/2){
								this.realPlacement = 'bottom-start'
							}
							if(point.right + $target.offsetWidth/2 < this.$el.offsetWidth/2){
								this.realPlacement = 'bottom-end'
							}
						}
					}else {
						if(point.bottom + $target.offsetHeight < this.$el.offsetHeight){
							this.realPlacement = 'right'
						}
						if(point.bottom + $target.offsetHeight/2 < this.$el.offsetHeight/2){
							this.realPlacement = 'right-end'
						}
					}
				}else if(this.placement == 'right-end'){
					//如果右侧距离不够
					if(point.right < this.$el.offsetWidth){
						//左侧距离足够
						if(point.left > this.$el.offsetWidth){
							this.realPlacement = 'left-end'
							if(point.top + $target.offsetHeight < this.$el.offsetHeight){
								this.realPlacement = 'left'
							}
							if(point.top + $target.offsetHeight/2 < this.$el.offsetHeight/2){
								this.realPlacement = 'left-start'
							}
						}else if(point.top > this.$el.offsetHeight){//顶部距离够
							this.realPlacement = 'top'
							if(point.left + $target.offsetWidth/2 < this.$el.offsetWidth/2){
								this.realPlacement = 'top-start'
							}
							if(point.right + $target.offsetWidth/2 < this.$el.offsetWidth/2){
								this.realPlacement = 'top-end'
							}
						}else if(point.bottom > this.$el.offsetHeight){//底部距离够
							this.realPlacement = 'bottom'
							if(point.left + $target.offsetWidth/2 < this.$el.offsetWidth/2){
								this.realPlacement = 'bottom-start'
							}
							if(point.right + $target.offsetWidth/2 < this.$el.offsetWidth/2){
								this.realPlacement = 'bottom-end'
							}
						}
					}else {
						if(point.top + $target.offsetHeight < this.$el.offsetHeight){
							this.realPlacement = 'right'
						}
						if(point.top + $target.offsetHeight/2 < this.$el.offsetHeight/2){
							this.realPlacement = 'right-start'
						}
					}
				}
			},
			//点击他处关闭悬浮层
			hideLayer(event){
				if(this.layerShow && this.firstShow && this.closable){
					if($util.isContains(this.$el,event.target) || $util.isContains(this.getTargetEl(),event.target)){
						return;
					}
					this.$emit('update:show',false);
					this.$emit('model-change',false);
				}
			},
			//悬浮层显示前
			beforeEnter(el){
				this.$emit('show',el)
				if(typeof this.layerComponentWatch == 'function'){
					this.layerComponentWatch.apply(this,['show',el])
				}
			},
			//悬浮层显示时
			enter(el){
				this.$nextTick(()=>{
					//智能修改位置
					this.autoAdjust()
					//根据位置设置
					this.reset();
					//设置三角
					this.$nextTick(()=>{
						this.resetTriangle()
					})
					this.$emit('showing',el)
					if(typeof this.layerComponentWatch == 'function'){
						this.layerComponentWatch.apply(this,['showing',el])
					}
				})
			},
			//悬浮层显示后
			afterEnter(el){
				this.$emit('shown',el)
				if(typeof this.layerComponentWatch == 'function'){
					this.layerComponentWatch.apply(this,['shown',el])
				}
			},
			//悬浮层隐藏前
			beforeLeave(el){
				this.$emit('hide',el)
				if(typeof this.layerComponentWatch == 'function'){
					this.layerComponentWatch.apply(this,['hide',el])
				}
			},
			//悬浮层隐藏时
			leave(el){
				this.$emit('hidding',el)
				if(typeof this.layerComponentWatch == 'function'){
					this.layerComponentWatch.apply(this,['hidding',el])
				}
			},
			//悬浮层隐藏后
			afterLeave(el){
				this.$emit('hidden',el)
				if(typeof this.layerComponentWatch == 'function'){
					this.layerComponentWatch.apply(this,['hidden',el])
				}
			},
			//重置位置
			reset(){
				if(!$util.isElement(this.$el)){
					return;
				}
				//设置offset
				if(this.showTriangle){
					this.$el.style.padding = 0;
					if(this.realPlacement == 'bottom-start' || this.realPlacement == 'bottom' || this.realPlacement == 'bottom-end'){
						this.$el.style.paddingTop = `calc(${this.offset} + ${this.$refs.triangle.$el.offsetHeight}px)`;
					}else if(this.realPlacement == 'top-start' || this.realPlacement == 'top' || this.realPlacement == 'top-end'){
						this.$el.style.paddingBottom = `calc(${this.offset} + ${this.$refs.triangle.$el.offsetHeight}px)`;
					}else if(this.realPlacement == 'left-start' || this.realPlacement == 'left' || this.realPlacement == 'left-end'){
						this.$el.style.paddingRight = `calc(${this.offset} + ${this.$refs.triangle.$el.offsetWidth}px)`;
					}else if(this.realPlacement == 'right-start' || this.realPlacement == 'right' || this.realPlacement == 'right-end'){
						this.$el.style.paddingLeft = `calc(${this.offset} + ${this.$refs.triangle.$el.offsetWidth}px)`;
					}
				}
				let $target = this.getTargetEl();
				let $root = this.getRootEl();
				let pt = $util.getElementPoint($target,$root);
				if(this.fixed){
					if(this.fixedAuto){
						let flag = true;
						let element = $target.offsetParent;
						while (flag && element){
							if($util.getCssStyle(element,'transform') != 'none'){
								flag = false;
							}else {
								element = element.offsetParent;
							}
						}
						pt = $util.getElementPoint($target,element)
					}else {
						pt = $util.getElementBounding($target)
					}
				}
				if(this.realPlacement == 'bottom' || this.realPlacement == 'bottom-start' || this.realPlacement == 'bottom-end'){
					this.$el.style.top = (pt.top + $target.offsetHeight) + 'px';
					this.$el.style.bottom = 'auto';
					if(this.realPlacement == 'bottom'){
						this.$el.style.left = pt.left + $target.offsetWidth/2 - this.$el.offsetWidth/2 + 'px';
					}else if(this.realPlacement == 'bottom-start'){
						this.$el.style.left = pt.left + 'px';
					}else if(this.realPlacement == 'bottom-end'){
						this.$el.style.left = $target.offsetWidth + pt.left - this.$el.offsetWidth + 'px';
					}
					this.$el.style.right = 'auto';
				}else if(this.realPlacement == 'top' || this.realPlacement == 'top-start' || this.realPlacement == 'top-end'){
					this.$el.style.bottom = (this.fixed?(pt.bottom + $target.offsetHeight):($root.clientHeight - pt.top)) + 'px';
					this.$el.style.top = 'auto';
					if(this.realPlacement == 'top'){
						this.$el.style.left = pt.left + $target.offsetWidth/2 - this.$el.offsetWidth/2 + 'px';
					}else if(this.realPlacement == 'top-start'){
						this.$el.style.left = pt.left + 'px';
					}else if(this.realPlacement == 'top-end'){
						this.$el.style.left = $target.offsetWidth + pt.left - this.$el.offsetWidth + 'px';
					}
					this.$el.style.right = 'auto';
				}else if(this.realPlacement == 'left' || this.realPlacement == 'left-start' || this.realPlacement == 'left-end'){
					this.$el.style.right = (this.fixed?(pt.right + $target.offsetWidth):($root.clientWidth - pt.left)) + 'px';
					this.$el.style.left = 'auto';
					if(this.realPlacement == 'left'){
						this.$el.style.top = pt.top + $target.offsetHeight/2 - this.$el.offsetHeight/2 + 'px';
					}else if(this.realPlacement == 'left-start'){
						this.$el.style.top = pt.top + 'px';
					}else if(this.realPlacement == 'left-end'){
						this.$el.style.top = $target.offsetHeight + pt.top - this.$el.offsetHeight + 'px';
					}
					this.$el.style.bottom = 'auto';
				}else if(this.realPlacement == 'right' || this.realPlacement == 'right-start' || this.realPlacement == 'right-end'){
					this.$el.style.left = (pt.left + $target.offsetWidth) + 'px';
					this.$el.style.right = 'auto';
					if(this.realPlacement == 'right'){
						this.$el.style.top = pt.top + $target.offsetHeight/2 - this.$el.offsetHeight/2 + 'px';
					}else if(this.realPlacement == 'right-start'){
						this.$el.style.top = pt.top + 'px';
					}else if(this.realPlacement == 'right-end'){
						this.$el.style.top = $target.offsetHeight + pt.top - this.$el.offsetHeight + 'px';
					}
					this.$el.style.bottom = 'auto';
				}
			},
			//重置三角位置
			resetTriangle(){
				if(!this.showTriangle || (this.$refs.triangle && !this.$refs.triangle.$el)){
					return;
				}
				if(this.realPlacement == 'bottom-start'){
					this.$refs.triangle.$el.style.top = 'auto';
					this.$refs.triangle.$el.style.right = 'auto';
					this.$refs.triangle.$el.style.bottom = '100%';
					this.$refs.triangle.$el.style.left = (this.getTargetEl().offsetWidth / 2 - this.$refs.triangle.$el.offsetWidth /2)+ 'px';
				}else if(this.realPlacement == 'bottom'){
					this.$refs.triangle.$el.style.top = 'auto';
					this.$refs.triangle.$el.style.right = 'auto';
					this.$refs.triangle.$el.style.bottom = '100%';
					this.$refs.triangle.$el.style.left = (this.$el.offsetWidth/2 - this.$refs.triangle.$el.offsetWidth /2) + 'px';
				}else if(this.realPlacement == 'bottom-end'){
					this.$refs.triangle.$el.style.top = 'auto';
					this.$refs.triangle.$el.style.left = 'auto';
					this.$refs.triangle.$el.style.bottom = '100%';
					this.$refs.triangle.$el.style.right = (this.getTargetEl().offsetWidth / 2 - this.$refs.triangle.$el.offsetWidth /2) + 'px';
				}else if(this.realPlacement == 'top-start'){
					this.$refs.triangle.$el.style.top = '100%';
					this.$refs.triangle.$el.style.right = 'auto';
					this.$refs.triangle.$el.style.bottom = 'auto';
					this.$refs.triangle.$el.style.left = (this.getTargetEl().offsetWidth / 2 - this.$refs.triangle.$el.offsetWidth /2)+ 'px';
				}else if(this.realPlacement == 'top'){
					this.$refs.triangle.$el.style.top = '100%';
					this.$refs.triangle.$el.style.right = 'auto';
					this.$refs.triangle.$el.style.bottom = 'auto';
					this.$refs.triangle.$el.style.left = (this.$el.offsetWidth/2 - this.$refs.triangle.$el.offsetWidth /2) + 'px';
				}else if(this.realPlacement == 'top-end'){
					this.$refs.triangle.$el.style.top = '100%';
					this.$refs.triangle.$el.style.left = 'auto';
					this.$refs.triangle.$el.style.bottom = 'auto';
					this.$refs.triangle.$el.style.right = (this.getTargetEl().offsetWidth / 2 - this.$refs.triangle.$el.offsetWidth /2) + 'px';
				}else if(this.realPlacement == 'left-start'){
					this.$refs.triangle.$el.style.left = '100%';
					this.$refs.triangle.$el.style.right = 'auto';
					this.$refs.triangle.$el.style.top = (this.getTargetEl().offsetHeight / 2 - this.$refs.triangle.$el.offsetHeight /2) + 'px';
					this.$refs.triangle.$el.style.bottom = 'auto';
				}else if(this.realPlacement == 'left'){
					this.$refs.triangle.$el.style.left = '100%';
					this.$refs.triangle.$el.style.right = 'auto';
					this.$refs.triangle.$el.style.top = (this.$el.offsetHeight/2 - this.$refs.triangle.$el.offsetHeight /2) + 'px';
					this.$refs.triangle.$el.style.bottom = 'auto';
				}else if(this.realPlacement == 'left-end'){
					this.$refs.triangle.$el.style.left = '100%';
					this.$refs.triangle.$el.style.right = 'auto';
					this.$refs.triangle.$el.style.bottom = (this.getTargetEl().offsetHeight / 2 - this.$refs.triangle.$el.offsetHeight /2) + 'px';
					this.$refs.triangle.$el.style.top = 'auto';
				}else if(this.realPlacement == 'right-start'){
					this.$refs.triangle.$el.style.right = '100%';
					this.$refs.triangle.$el.style.left = 'auto';
					this.$refs.triangle.$el.style.top = (this.getTargetEl().offsetHeight / 2 - this.$refs.triangle.$el.offsetHeight /2) + 'px';
					this.$refs.triangle.$el.style.bottom = 'auto';
				}else if(this.realPlacement == 'right'){
					this.$refs.triangle.$el.style.right = '100%';
					this.$refs.triangle.$el.style.left = 'auto';
					this.$refs.triangle.$el.style.top = (this.$el.offsetHeight/2 - this.$refs.triangle.$el.offsetHeight /2) + 'px';
					this.$refs.triangle.$el.style.bottom = 'auto';
				}else if(this.realPlacement == 'right-end'){
					this.$refs.triangle.$el.style.right = '100%';
					this.$refs.triangle.$el.style.left = 'auto';
					this.$refs.triangle.$el.style.bottom = (this.getTargetEl().offsetHeight / 2 - this.$refs.triangle.$el.offsetHeight /2) + 'px';
					this.$refs.triangle.$el.style.top = 'auto';
				}
			},
			//获取绑定的元素
			getTargetEl(){
				if(this.target){
					return document.body.querySelector(this.target) || document.body;
				}else{
					return document.body;
				}
			},
			//获取根元素
			getRootEl(){
				if(this.fixed){
					return document.body;
				}
				if(this.root){
					return document.body.querySelector(this.root) || document.body;
				}else{
					return document.body;
				}
			}
		},
		beforeDestroy() {
			window.off(`resize.layer_${this._uid} click.layer_${this._uid}`)
		}
	}
</script>

<style scoped lang="less">
	@import "../../css/mvi-basic.less";
	.mvi-layer{
		position: absolute;
		z-index: 400;
		
		&.mvi-layer-fixed{
			position: fixed;
		}
		
		.mvi-layer-wrapper{
			display: block;
			position: relative;
			width: 100%;
			color: @font-color-default;
			border-radius: @radius-default;	
			
			&.mvi-layer-shadow{
				box-shadow:@boxshadow;
			}
			
			&.mvi-layer-border{
				border: 1px solid #eee;
			}
			
			.mvi-layer-triangle{
				position: absolute;
			}
		}
	}

	.mvi-layer-enter,.mvi-layer-leave-to{
		opacity: 0;
		transform: translateY(.6rem);
	}
</style>
