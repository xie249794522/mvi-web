<template>
	<m-modal
		v-on="listeners"
		v-model="show"
		:footer-padding="false"
		@hide="modalHide"
		@hidding="modalHidding"
		@hidden="modalHidden"
		:width="computedWidth"
		:z-index="computedZIndex"
		:radius="computedRadius"
		:local="computedLocal"
		:use-padding="computedUsePadding"
		:animation="computedAnimation"
		@show="modalShow"
		@showing="modalShowing"
		@shown="modalShown"
		:timeout="computedTimeout"
		:overlay-color="computedOverlayColor"
	>
		<template v-if="computedTitle" v-slot:title>
			<div v-html="computedTitle" class="mvi-dialog-title"></div>
			<m-icon class="mvi-dialog-close" v-if="computedShowTimes" @click="cancelFun" type="times"></m-icon>
		</template>
		<template v-slot:default v-if="contentShow">
			<div v-if="computedMessage" v-html="computedMessage" class="mvi-dialog-content"></div>
			<div v-if="type == 'Prompt'" :class="['mvi-dialog-input',computedMessage?'mvi-dialog-input-mt':'']">
				<input
					ref="input"
					:type="computedInput.type"
					:placeholder="computedInput.placeholder"
					:maxlength="computedInput.maxlength"
					:class="inputClass"
					:style="inputStyle"
					:autofocus="computedInput.autofocus"
					v-model.trim="computedValue"
					@focus="inputFocus"
					@blur="inputBlur"
				/>
				<m-icon v-if="computedInput.clearable" ref="icon" v-show="showClear" type="times-o" class="mvi-dialog-times" @click="doClear" />
			</div>
			<div class="mvi-dialog-footer">
				<m-button v-if="type!='Alert'" :type="computedBtns.cancel.type" :color="computedBtns.cancel.color" :sub-color="computedBtns.cancel.subColor" :plain="computedBtns.cancel.plain" class="mvi-dialog-cancel" @click="cancelFun">{{computedBtns.cancel.text}}</m-button>
				<m-button :type="computedBtns.ok.type" :color="computedBtns.ok.color" :sub-color="computedBtns.ok.subColor" :plain="computedBtns.ok.plain" class="mvi-dialog-ok" @click="okFun">{{computedBtns.ok.text}}</m-button>
			</div>
		</template>
	</m-modal>
</template>

<script>
import $util from '../../util/util';
import mIcon from "../icon/icon"
import mButton from "../button/button"
import mModal from "../modal/modal"
export default {
	name: 'm-dialog-pc',
	data() {
		return {
			ok: false, //点击的是否是确定按钮
			show: true, //对话框是否打开，默认为true，即挂载就显示
			type: 'Alert', //弹窗类型
			title: null, //标题
			message: null, //描述
			btns:null,//弹窗确定按钮和取消按钮配置
			width: null, //模态框宽度
			callback: null, //回调函数
			zIndex: null, //遮罩z-index
			animation: null, //动画
			local: null, //是否局部
			usePadding: null, //局部显示时是否考虑滚动条影响
			radius: null, //圆角
			timeout: null, //自定义动画时间
			overlayColor: null, //遮罩层背景色
			showTimes:null,//是否显示关闭按钮
			input: {
				//输入框配置
				placeholder: null, //占位符
				type: null, //输入框类型
				autofocus: null,
				maxlength: null,
				clearable: null,
				value: null, //输入框的值
				align: null //输入框文本对齐方式
			},
			focus: false //输入框是否已经获得了焦点
		};
	},
	computed: {
		listeners() {
			return Object.assign({}, this.$listeners);
		},
		computedShowTimes(){
			if (typeof this.showTimes == 'boolean') {
				return this.showTimes;
			} else {
				return true;
			}
		},
		computedTitle() {
			if (typeof this.title == 'string') {
				return this.title;
			} else {
				return '提示';
			}
		},
		computedMessage() {
			if (typeof this.message == 'string') {
				return this.message;
			}else if ($util.isObject(this.message)) {
				return JSON.stringify(this.message);
			} else {
				return String(this.message)
			}
		},
		computedBtns() {
			let btns = {
				ok:{
					type:'primary',
					color:null,
					subColor:null,
					plain:false,
					text:'确定'
				},
				cancel:{
					type:'default',
					color:null,
					subColor:null,
					plain:false,
					text:'取消'
				}
			}
			if($util.isObject(this.btns)){
				if($util.isObject(this.btns.ok)){
					if(typeof this.btns.ok.type == 'string'){
						btns.ok.type = this.btns.ok.type;
					}
					if(typeof this.btns.ok.color == 'string'){
						btns.ok.color = this.btns.ok.color;
					}
					if(typeof this.btns.ok.subColor == 'string'){
						btns.ok.subColor = this.btns.ok.subColor;
					}
					if(typeof this.btns.ok.plain == 'boolean'){
						btns.ok.plain = this.btns.ok.plain;
					}
					if(typeof this.btns.ok.text == 'string'){
						btns.ok.text = this.btns.ok.text;
					}
				}
				if($util.isObject(this.btns.cancel)){
					if(typeof this.btns.cancel.type == 'string'){
						btns.cancel.type = this.btns.cancel.type;
					}
					if(typeof this.btns.cancel.color == 'string'){
						btns.cancel.color = this.btns.cancel.color;
					}
					if(typeof this.btns.cancel.subColor == 'string'){
						btns.cancel.subColor = this.btns.cancel.subColor;
					}
					if(typeof this.btns.cancel.plain == 'boolean'){
						btns.cancel.plain = this.btns.cancel.plain;
					}
					if(typeof this.btns.cancel.text == 'string'){
						btns.cancel.text = this.btns.cancel.text;
					}
				}
			}
			return btns
		},
		computedCallback() {
			if (typeof this.callback == 'function') {
				return this.callback;
			} else {
				return function() {};
			}
		},
		computedWidth() {
			if (typeof this.width == 'string' && this.width) {
				return this.width;
			} else {
				return '7.2rem';
			}
		},
		computedInput() {
			let input = {};
			if (typeof this.input.placeholder == 'string') {
				input.placeholder = this.input.placeholder;
			} else {
				input.placeholder = '';
			}
			if (typeof this.input.type == 'string') {
				if (this.input.type == 'number') {
					input.type = 'text';
				} else {
					input.type = this.input.type;
				}
			} else {
				input.type = 'text';
			}
			if (typeof this.input.autofocus == 'boolean') {
				input.autofocus = this.input.autofocus;
			} else {
				input.autofocus = true;
			}
			if ($util.isNumber(this.input.maxlength)) {
				input.maxlength = this.input.maxlength;
			} else {
				input.maxlength = -1;
			}
			if (typeof this.input.clearable == 'boolean') {
				input.clearable = this.input.clearable;
			} else {
				input.clearable = false;
			}
			if (typeof this.input.align == 'string') {
				input.align = this.input.align;
			} else {
				input.align = 'left';
			}
			return input;
		},
		computedValue: {
			get() {
				let value = '';
				if ((typeof this.input.value == 'string' && this.input.value) || typeof this.input.value == 'number') {
					value = this.input.value;
				}
				if (this.input.type == 'number') {
					value = value.replace(/\D/g, '');
				}
				if (this.computedInput.maxlength > 0 && value.length > this.computedInput.maxlength) {
					value = value.substr(0, this.computedInput.maxlength);
				}
				if(this.input.value !== value){
					this.$set(this.input,'value',value)
				}
				return value;
			},
			set(value) {
				if(this.input.value !== value){
					this.$set(this.input,'value',value)
				}
			}
		},
		computedZIndex() {
			if ($util.isNumber(this.zIndex)) {
				return this.zIndex;
			} else {
				return 1000;
			}
		},
		computedLocal() {
			if ((typeof this.local == 'string' && this.local) || $util.isElement(this.local)) {
				return true;
			} else {
				return false;
			}
		},
		computedUsePadding() {
			if (typeof this.usePadding == 'boolean') {
				return this.usePadding;
			} else {
				return false;
			}
		},
		computedAnimation() {
			if (typeof this.animation == 'string' && this.animation) {
				return this.animation;
			} else {
				return 'translate-top';
			}
		},
		computedRadius() {
			if (typeof this.radius == 'string' && this.radius) {
				return this.radius;
			} else {
				return '0.12rem';
			}
		},
		computedTimeout() {
			if ($util.isNumber(this.timeout)) {
				return this.timeout;
			} else {
				return 300;
			}
		},
		computedOverlayColor() {
			if (typeof this.overlayColor == 'string' && this.overlayColor) {
				return this.overlayColor;
			} else {
				return null;
			}
		},
		contentShow() {
			if (this.type == 'Alert' || this.type == 'Confirm') {
				if (this.computedMessage) {
					return true;
				} else {
					return false;
				}
			} else {
				return true;
			}
		},
		showClear() {
			if (this.focus && this.computedValue) {
				return true;
			} else {
				return false;
			}
		},
		inputClass() {
			let cls = [];
			if (this.showClear && this.computedInput.clearable) {
				cls.push('mvi-dialog-input-padding');
			}
			return cls;
		},
		inputStyle() {
			let style = {};
			if (['left', 'right', 'center'].includes(this.computedInput.align)) {
				style.textAlign = this.computedInput.align;
			}
			return style;
		}
	},
	components:{
		mIcon,mModal,mButton
	},
	methods: {
		//获取焦点
		inputFocus() {
			setTimeout(() => {
				this.focus = true;
			}, 200);
		},
		//失去焦点
		inputBlur(e) {
			setTimeout(() => {
				this.focus = false;
			}, 200);
		},
		//清除输入框的值
		doClear() {
			if(!this.computedInput.clearable){
				return;
			}
			this.computedValue = '';
			this.$refs.input.focus();
		},
		//确定
		okFun() {
			this.show = false;
			this.ok = true;
		},
		//取消
		cancelFun() {
			this.show = false;
			if(this.type == 'Alert'){
				this.ok = true;
			}else {
				this.ok = false;
			}
		},
		//模态框隐藏前
		modalHide(el){
			//触发全局的监听
			if(typeof this.dialogComponentWatch == 'function'){
				this.dialogComponentWatch.apply(this,['hide',this.type,el])
			}
		},
		//模态框隐藏时
		modalHidding(el){
			//触发全局的监听
			if(typeof this.dialogComponentWatch == 'function'){
				this.dialogComponentWatch.apply(this,['hidding',this.type,el])
			}
		},
		//模态框隐藏后
		modalHidden(el) {
			if (this.type == 'Alert') {
				this.computedCallback();
			} else if (this.type == 'Confirm') {
				this.computedCallback(this.ok);
			} else if (this.type == 'Prompt') {
				this.computedCallback(this.ok, this.computedValue);
			}
			//触发全局的监听
			if(typeof this.dialogComponentWatch == 'function'){
				this.dialogComponentWatch.apply(this,['hidden',this.type,el])
			}
			this.$el.remove();
			this.$destroy();
		},
		//模态框显示前
		modalShow(el){
			//触发全局的监听
			if(typeof this.dialogComponentWatch == 'function'){
				this.dialogComponentWatch.apply(this,['show',this.type,el])
			}
		},
		//模态框显示时
		modalShowing(el){
			//触发全局的监听
			if(typeof this.dialogComponentWatch == 'function'){
				this.dialogComponentWatch.apply(this,['showing',this.type,el])
			}
		},
		//模态框显示后
		modalShown(el) {
			//输入框获取焦点
			if (this.type == 'Prompt' && this.computedInput.autofocus) {
				this.$refs.input.focus();
			}
			//触发全局的监听
			if(typeof this.dialogComponentWatch == 'function'){
				this.dialogComponentWatch.apply(this,['shown',this.type,el])
			}
		}
	}
};
</script>

<style scoped lang="less">
@import '../../css/mvi-basic.less';

.mvi-dialog-title {
	display: block;
	width: 100%;
	font-size: @font-size-h6;
	color: @font-color-default;
	font-weight: bold;
	line-height: 1.5;
}

.mvi-dialog-close{
	position: absolute;
	right: .2rem;
	top: .3rem;
	opacity: .5;
	font-size: .28rem;
	font-weight: normal;
	
	&:hover{
		cursor: pointer;
		opacity: .8;
	}
}

.mvi-dialog-content {
	color: @font-color-default;
	line-height: 1.5;
	font-size: @font-size-default;
}

.mvi-dialog-input {
	display: block;
	width: 100%;
	position: relative;
	
	&.mvi-dialog-input-mt{
		margin-top: @mp-md;
	}
}

.mvi-dialog-input > input {
	display: block;
	appearance: none;
	-moz-appearance: none;
	-webkit-appearance: none;
	width: 100%;
	height: @medium-height;
	line-height: 1.5;
	border-radius: @radius-default;
	border: 1px solid @border-color;
	color: @font-color-default;
	font-size: @font-size-default;
	background-color: #fff;
	vertical-align: middle;
	padding: 0 @mp-sm;

	&::placeholder,
	&::-webkit-input-placeholder,
	&:-moz-placeholder,
	&::-moz-placeholder,
	&:-ms-input-placeholder {
		opacity: 0.5;
		font-family: inherit;
		font-size: inherit;
		color: inherit;
		vertical-align: middle;
	}

	&.mvi-dialog-input-padding {
		padding-right: @mp-sm * 3;
	}
}

.mvi-dialog-times {
	position: absolute;
	top: 50%;
	right: @mp-sm;
	transform: translateY(-50%);
	-webkit-transform: translateY(-50%);
	-moz-transform: translateY(-50%);
	-ms-transform: translateY(-50%);
	color: @font-color-mute;
	cursor: pointer;
}

.mvi-dialog-footer {
	width: 100%;
	display: flex;
	display: -webkit-flex;
	justify-content: flex-end;
	align-items: center;
	padding:0 @mp-sm;
	margin-top: @mp-lg;
	
	.mvi-dialog-cancel{
		margin-right: @mp-md;
	}
	
	.mvi-dialog-ok{
		
	}
}
</style>
