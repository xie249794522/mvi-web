<template>
	<m-modal
		v-on="listeners"
		:show="show"
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
		:modal-color="computedIos?'rgba(255,255,255,.95)':''"
		:overlay-color="computedOverlayColor"
	>
		<template v-if="computedTitle || (computedIos && computedMessage)" v-slot:title>
			<div v-html="computedTitle" v-if="computedTitle" class="mvi-dialog-title"></div>
			<div v-if="computedMessage && computedIos" v-html="computedMessage" class="mvi-dialog-ios-content"></div>
		</template>
		<template v-slot:default v-if="contentShow">
			<div v-if="!computedIos && computedMessage" v-html="computedMessage" class="mvi-dialog-content"></div>
			<div v-if="type == 'prompt'" :class="['mvi-dialog-input',(!computedIos && computedMessage)?'mvi-dialog-input-mt':'']">
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
					:inputmode="computedInputMode"
				/>
				<m-icon v-if="computedInput.clearable" ref="icon" v-show="showClear" type="times-o" class="mvi-dialog-times" @click="doClear" />
			</div>
		</template>
		<template v-slot:footer>
			<div class="mvi-dialog-footer">
				<div
					v-if="type != 'alert'"
					class="mvi-dialog-cancel"
					v-text="computedBtnText[1]"
					@click="cancelFun"
					:style="'color:' + (computedBtnColor ? computedBtnColor[1] : '')"
				></div>
				<div
					class="mvi-dialog-ok"
					v-text="type == 'alert' ? computedBtnText : computedBtnText[0]"
					@click="okFun"
					:style="'color:' + (type == 'alert' ? (computedBtnColor ? computedBtnColor : '') : computedBtnColor ? computedBtnColor[0] : '')"
				></div>
			</div>
		</template>
	</m-modal>
</template>

<script>
import $util from '../../util/util';
import mModal from "../modal/modal"
import mIcon from "../icon/icon"
export default {
	name: 'm-dialog',
	data() {
		return {
			ok: false, //点击的是否是确定按钮
			show: true, //对话框是否打开，默认为true，即挂载就显示
			type: 'alert', //弹窗类型
			title: null, //标题
			message: null, //描述
			btnText: null, //按钮文本
			btnColor: null, //按钮文字颜色
			width: null, //模态框宽度
			callback: null, //回调函数
			zIndex: null, //遮罩z-index
			animation: null, //动画
			local: null, //是否局部
			usePadding: null, //局部显示时是否考虑滚动条影响
			radius: null, //圆角
			timeout: null, //自定义动画时间
			overlayColor: null, //遮罩层背景色
			ios:null,//是否ios类型
			input: {
				//输入框配置
				placeholder: null, //占位符
				type: null, //输入框类型
				autofocus: null,
				maxlength: null,
				clearable: null,
				value: null, //输入框的值
				mode: null, //输入框键盘类型
				align: null //输入框文本对齐方式
			},
			focus: false //输入框是否已经获得了焦点
		};
	},
	computed: {
		listeners() {
			return Object.assign({}, this.$listeners);
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
		computedBtnText() {
			let bt = null;
			if (this.type == 'alert') {
				if (typeof this.btnText == 'string') {
					bt = this.btnText;
				} else {
					bt = '确定';
				}
			} else {
				bt = [];
				if (this.btnText instanceof Array) {
					if (typeof this.btnText[0] == 'string') {
						bt[0] = this.btnText[0];
					} else {
						bt[0] = '确定';
					}
					if (typeof this.btnText[1] == 'string') {
						bt[1] = this.btnText[1];
					} else {
						bt[1] = '取消';
					}
				} else {
					bt = ['确定', '取消'];
				}
			}
			return bt;
		},
		computedBtnColor() {
			let bt = null;
			if (this.type == 'alert') {
				if (typeof this.btnColor == 'string') {
					bt = this.btnColor;
				} else {
					bt = null;
				}
			} else {
				bt = [];
				if (this.btnColor instanceof Array) {
					if (typeof this.btnColor[0] == 'string') {
						bt[0] = this.btnColor[0];
					} else {
						bt[0] = null;
					}
					if (typeof this.btnColor[1] == 'string') {
						bt[1] = this.btnColor[1];
					} else {
						bt[1] = null;
					}
				} else {
					bt = [null, null];
				}
			}
			return bt;
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
				return '5.6rem';
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
			if (typeof this.input.mode == 'string') {
				input.mode = this.input.mode;
			} else {
				input.mode = false;
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
					value = this.input.value.toString();
				}
				if (this.input.type == 'number') {
					value = value.replace(/\D/g, '');
				}
				if (this.computedInput.maxlength > 0 && value.length > this.computedInput.maxlength) {
					value = value.substr(0, this.computedInput.maxlength);
				}
				if(this.input.value !== value){
					this.$set(this.input,'value',value);
				}
				return value;
			},
			set(value) {
				if(this.input.value !== value){
					this.$set(this.input,'value',value);
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
				return 'narrow';
			}
		},
		computedRadius() {
			if (typeof this.radius == 'string' && this.radius) {
				return this.radius;
			} else {
				return '0.2rem';
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
				if(this.computedIos){
					return 'rgba(0,10,20,.3)'
				}
				return null;
			}
		},
		computedIos(){
			if (typeof this.ios == 'boolean') {
				return this.ios;
			} else {
				return false;
			}
		},
		contentShow() {
			if (this.type == 'alert' || this.type == 'confirm') {
				if (this.computedMessage && !this.computedIos) {
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
				cls.push('mvi-dialog-input-padding')
			}
			return cls;
		},
		computedInputMode() {
			let mode = false;
			if ([false, 'none', 'text', 'decimal', 'numeric', 'tel', 'search', 'email', 'url'].includes(this.computedInput.mode)) {
				mode = this.computedInput.mode;
			}
			return mode;
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
		mIcon,mModal
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
			this.ok = false;
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
			if (this.type == 'alert') {
				this.computedCallback();
			} else if (this.type == 'confirm') {
				this.computedCallback(this.ok);
			} else if (this.type == 'prompt') {
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
			if (this.type == 'prompt' && this.computedInput.autofocus) {
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
	text-align: center;
	font-size: @font-size-h6;
	color: @font-color-default;
	font-weight: bold;
	line-height: 1.5;
}

.mvi-dialog-ios-content{
	display: block;
	width: 100%;
	font-size: .24rem;
	font-weight: normal;
	text-align: center;
	color: @font-color-default;
	line-height: 1.5;
	margin-top: @mp-xs;
}

.mvi-dialog-content {
	text-align: center;
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
	width: 90%;
	height: @small-height;
	line-height: 1.5;
	border-radius: @radius-default;
	border: 1px solid @border-color;
	color: @font-color-default;
	font-size: @font-size-default;
	padding: 0 @mp-sm 0 @mp-sm;
	background-color: #fff;
	vertical-align: middle;
	margin-left: 5%;

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
	right: ~'calc(5% + @{mp-sm})';
	transform: translateY(-50%);
	-webkit-transform: translateY(-50%);
	-moz-transform: translateY(-50%);
	-ms-transform: translateY(-50%);
	color: @font-color-mute;
	cursor: pointer;
}

.mvi-dialog-footer {
	height: 0.88rem;
	width: 100%;
	display: flex;
	display: -webkit-flex;
	justify-content: space-between;
}

.mvi-dialog-ok {
	position: relative;
	display: flex;
	display: -webkit-flex;
	align-items: center;
	justify-content: center;
	height: 100%;
	font-size: @font-size-h6;
	color: @primary-normal;
	flex: 1;
	cursor: pointer;
	font-weight: bold;
	user-select: none;
}

.mvi-dialog-cancel {
	position: relative;
	display: flex;
	display: -webkit-flex;
	align-items: center;
	justify-content: center;
	height: 100%;
	font-size: @font-size-h6;
	width: 50%;
	border-right: 1px solid @border-color;
	color: @primary-normal;
	cursor: pointer;
	user-select: none;
}

.mvi-dialog-ok:active::before,
.mvi-dialog-cancel:active::before {
	.mvi-active();
}
</style>
