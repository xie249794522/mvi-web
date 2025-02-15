<template>
	<div :disabled="disabled" :class="['mvi-input-container','mvi-input-container-'+size,border?'mvi-input-border':'',(showWordLimit && maxlength>0)?'mvi-input-container-words':'',required?'mvi-input-required':'']" :data-type="(type=='textarea'?'textarea':'input')">
		<!-- 左侧图标 -->
		<div @click="leftClick" class="mvi-input-left-icon" v-if="$slots.left || leftIconUrl || leftIconType">
			<slot name="left" v-if="$slots.left"></slot>
			<m-icon v-else-if="leftIconUrl || leftIconType" :type="leftIconType" :url="leftIconUrl" :spin="leftIconSpin"
			:size="leftIconSize" :color="leftIconColor"/>
		</div>
		<!-- 左侧文本 -->
		<div :class="['mvi-input-label',labelClass?labelClass:'']" v-if="label" :style="labelStyle">
			<span v-text="label"></span>
		</div>
		<!-- 文本域 -->
		<textarea v-if="type=='textarea'" :placeholder="placeholder" :maxlength="maxlength" :disabled="disabled" :readonly="readonly"
		 :autofocus="autofocus" class="mvi-textarea" v-on="listeners" v-model="realValue" ref="textarea" :rows="rowsFilter"
		 :name="name" :style="textareaStyle" @focus="getFocus" @blur="getBlur" autocomplete="off"></textarea>
		<!-- 输入框 -->
		<input v-else :type="inputType" :inputmode="computedInputMode" :placeholder="placeholder" :disabled="disabled"
		 :readonly="readonly || isDatePicker" :autofocus="autofocus" class="mvi-input" v-on="listeners" v-model="realValue"
		 @click="callDate" ref="input" :name="name" :style="inputStyle" @focus="getFocus" @blur="getBlur" autocomplete="off" :maxlength="(isDatePicker?-1:maxlength)">
		<!-- 清除图标 -->
		<div @click="doClearValue" class="mvi-input-clear-icon" v-if="clearable" v-show="showClear">
			<m-icon type="times-o" />
		</div>
		<!-- 右侧图标 -->
		<div @click="rightClick" class="mvi-input-right-icon" v-if="$slots.right || (rightIconUrl || rightIconType)">
			<slot name="right" v-if="$slots.right"></slot>
			<m-icon v-else-if="rightIconUrl || rightIconType" ref="rightIcon" :type="rightIconType" :url="rightIconUrl" :spin="rightIconSpin"
			:size="rightIconSize" :color="rightIconColor" />
		</div>
		<!-- 显示文字长度限制 -->
		<div v-if="showWordLimit && maxlength>0" class="mvi-input-words">{{realValue.length}}/{{maxlength}}</div>
		<!-- 日期 -->
		<m-date-native-picker v-if="isDatePicker" ref="datepicker" :type="dateType" v-model="realDate"></m-date-native-picker>
	</div>
</template>

<script>
	import $util from "../../util/util"
	import mIcon from "../icon/icon"
	import mDateNativePicker from "../dateNativePicker/dateNativePicker"
	export default {
		name: "m-input",
		data(){
			return {
				focus:false,//输入框或者文本域是否获取焦点
			}
		},
		model: {
			prop: 'value',
			event: 'model-change'
		},
		props: {
			label: { //输入框左侧文本
				type: String,
				default: null
			},
			labelClass: { //左侧文本额外类名
				type: String,
				default: null
			},
			labelWidth: { //左侧文本宽度，默认值2rem
				type: String,
				default: null
			},
			labelAlign: { //左侧文本对齐方式
				type: String,
				default: 'left',
				validator(value) {
					return ['left', 'center', 'right'].includes(value)
				}
			},
			labelOffset: { //文本与输入框的间距
				type: String,
				default: null
			},
			value: { //输入框的值
				type: [String, Number],
				default: ""
			},
			placeholder: { //占位符
				type: String,
				default: ""
			},
			type: { //输入框类型
				type: String,
				default: 'text'
			},
			name: { //原生name属性
				type: String,
				default: null
			},
			maxlength: { //最大字数限制
				type: Number,
				default: -1
			},
			size: { //输入框尺寸
				type: String,
				default: 'medium',
				validator(value) {
					return ['medium', 'large'].includes(value)
				}
			},
			border: { //是否显示下边框
				type: Boolean,
				default: false
			},
			disabled: { //是否禁用
				type: Boolean,
				default: false
			},
			readonly: { //是否只读
				type: Boolean,
				default: false
			},
			required: { //是否显示表单必填星号
				type: Boolean,
				default: false
			},
			clearable: { //是否启用清除图标
				type: Boolean,
				default: false
			},
			autofocus: { //是否自动聚焦
				type: Boolean,
				default: false
			},
			showWordLimit: { //是否显示字数统计，需要设置maxlength属性
				type: Boolean,
				default: false
			},
			leftIcon: { //文本左侧图标
				type: [String, Object],
				default: null
			},
			rightIcon: { //右侧图标
				type: [String, Object],
				default: null
			},
			date: { //日期选择的默认日期
				type: Date,
				default: null
			},
			format: { //输入框中显示的日期格式
				type: String,
				default: null
			},
			rows: { //type=textarea时的行数
				type: Number,
				default: 1
			},
			autosize: { //自动调整高度
				type: [Boolean, Object],
				default: false,
				validator(value) {
					if ($util.isObject(value)) {
						if ($util.isNumber(value.minRows) && $util.isNumber(value.maxRows)) {
							if (value.minRows > value.maxRows) {
								return false;
							}
						}
					}
					return true;
				}
			},
			inputAlign: {//文本对齐方式
				type: String,
				default: 'left',
				validator(value) {
					return ['left', 'center', 'right'].includes(value)
				}
			},
			inputMode:{//输入框调起移动端键盘类型
				type:[String,Boolean],
				default:false,
				validator(value){
					return [false,'none','text','decimal','numeric','tel','search','email','url'].includes(value)
				}
			}
		},
		computed: {
			listeners() {
				return Object.assign({}, this.$listeners)
			},
			showClear() {
				if(this.disabled || this.readonly){
					return false;
				}
				if(this.focus){
					if (this.realValue === '') {
						return false;
					} else {
						return true;
					}
				}else{
					return false;
				}
			},
			leftIconType() {
				let type = null;
				if ($util.isObject(this.leftIcon)) {
					if (typeof this.leftIcon.type == "string") {
						type = this.leftIcon.type;
					}
				} else if (typeof this.leftIcon == "string") {
					type = this.leftIcon;
				}
				return type;
			},
			leftIconUrl() {
				let url = null;
				if ($util.isObject(this.leftIcon)) {
					if (typeof this.leftIcon.url == "string") {
						url = this.leftIcon.url;
					}
				}
				return url;
			},
			leftIconSpin() {
				let spin = false;
				if ($util.isObject(this.leftIcon)) {
					if (typeof this.leftIcon.spin == "boolean") {
						spin = this.leftIcon.spin;
					}
				}
				return spin;
			},
			leftIconSize(){
				let size = null;
				if ($util.isObject(this.leftIcon)) {
					if (typeof this.leftIcon.size == "string") {
						size = this.leftIcon.size;
					}
				}
				return size;
			},
			leftIconColor(){
				let color = null;
				if ($util.isObject(this.leftIcon)) {
					if (typeof this.leftIcon.color == "string") {
						color = this.leftIcon.color;
					}
				}
				return color;
			},
			rightIconType() {
				let type = null;
				if ($util.isObject(this.rightIcon)) {
					if (typeof this.rightIcon.type == "string") {
						type = this.rightIcon.type;
					}
				} else if (typeof this.rightIcon == "string") {
					type = this.rightIcon;
				}
				return type;
			},
			rightIconUrl() {
				let url = null;
				if ($util.isObject(this.rightIcon)) {
					if (typeof this.rightIcon.url == "string") {
						url = this.rightIcon.url;
					}
				}
				return url;
			},
			rightIconSpin() {
				let spin = false;
				if ($util.isObject(this.rightIcon)) {
					if (typeof this.rightIcon.spin == "boolean") {
						spin = this.rightIcon.spin;
					}
				}
				return spin;
			},
			rightIconSize(){
				let size = null;
				if ($util.isObject(this.rightIcon)) {
					if (typeof this.rightIcon.size == "string") {
						size = this.rightIcon.size;
					}
				}
				return size;
			},
			rightIconColor(){
				let color = null;
				if ($util.isObject(this.rightIcon)) {
					if (typeof this.rightIcon.color == "string") {
						color = this.rightIcon.color;
					}
				}
				return color;
			},
			//原生日期选择的值
			realDate:{
				set(value){
					this.$emit('update:date',value);
				},
				get(){
					return this.date;
				}
			},
			//输入框的值
			realValue:{
				set(value){
					if(this.value !== value){
						this.$emit('model-change',value);
						this.$emit('update:value',value);
					}
				},
				get(){
					let value = this.value === null ? '':this.value.toString();
					if(this.isDatePicker){
						value = this.getDateValue();
					}else {
						//数字类型会过滤非数字字符
						if(this.type == 'number'){
							value = value.replace(/\D/g, '');
						}
						//如果设置了maxlength，则进行字符串截取
						if (this.maxlength > 0 && value.length > this.maxlength) {
							value = value.substr(0, this.maxlength);
						}
					}
					if(this.value !== value){
						this.$emit('model-change',value);
						this.$emit('update:value',value);
					}
					return value;
				}
			},
			//输入框的type值
			inputType() {
				let type = 'text';
				if (this.isDatePicker || this.type == 'number') {
					type = 'text';
				} else {
					type = this.type;
				}
				return type;
			},
			//键盘模式
			computedInputMode(){
				let mode = false;
				if(typeof this.inputMode == 'string'){
					mode = this.inputMode
				}
				return mode
			},
			//datepicker组件的type值
			dateType() {
				let type = "date";
				if (this.isDatePicker) {
					type = this.type;
				}
				return type;
			},
			//判断是否日期选择
			isDatePicker() {
				if (['date', 'datetime', 'month', 'time'].includes(this.type)) {
					return true;
				} else {
					return false;
				}
			},
			//文本域的rows
			rowsFilter() {
				let rows = this.rows;
				if ($util.isObject(this.autosize)) {
					if ($util.isNumber(this.autosize.minRows)) {
						if (this.rows < this.autosize.minRows) {
							rows = this.autosize.minRows;
						}
					}
					if ($util.isNumber(this.autosize.maxRows)) {
						if (this.rows > this.autosize.maxRows) {
							rows = this.autosize.maxRows;
						}
					}
				}
				return rows;
			},
			//label的样式
			labelStyle() {
				let style = {};
				if (this.labelWidth) {
					style.width = this.labelWidth;
				}
				if (this.labelAlign) {
					if(this.labelAlign == 'left'){
						style.justifyContent = 'flex-start';
					}else if(this.labelAlign == 'right'){
						style.justifyContent = 'flex-end';
					}else{
						style.justifyContent = this.labelAlign;
					}
				}
				if (this.labelOffset) {
					style.marginRight = this.labelOffset;
				}
				return style;
			},
			//输入框样式
			inputStyle() {
				let style = {};
				if (this.inputAlign) {
					style.textAlign = this.inputAlign;
				}
				return style;
			},
			//文本域样式
			textareaStyle() {
				let style = {};
				if (this.inputAlign) {
					style.textAlign = this.inputAlign;
				}
				return style;
			}
		},
		components:{
			mIcon,mDateNativePicker
		},
		mounted() {
			if (this.$refs.textarea) {
				this.setMaxMinHeight();
				if (this.autosize == true || $util.isObject(this.autosize)) {
					this.autosizeSet();
				}
			}
		},
		watch: {
			realValue(newValue) {
				if (this.$refs.textarea && (this.autosize == true || $util.isObject(this.autosize))) {
					this.autosizeSet();
				}
			},
			rows(newValue) {
				if (this.$refs.textarea) {
					this.setMaxMinHeight();
				}
			},
			autosize(newValue) {
				if (this.$refs.textarea) {
					this.setMaxMinHeight();
				}
			},
			showClear(newValue){
				this.$nextTick(()=>{
					if (this.$refs.textarea && (this.autosize == true || $util.isObject(this.autosize))) {
						this.autosizeSet();
					}
				})
			}
		},
		methods: {
			//输入框或者文本域获取焦点
			getFocus(){
				if(this.disabled){
					return;
				}
				setTimeout(()=>{
					this.focus = true;
				},200)
			},
			//输入框或者文本域失去焦点
			getBlur(){
				if(this.disabled){
					return;
				}
				setTimeout(()=>{
					this.focus = false;
				},200)
			},
			//左侧图标点击
			leftClick() {
				if(this.disabled){
					return;
				}
				this.$emit('left-click', this.realValue);
			},
			//右侧图标点击
			rightClick() {
				if(this.disabled){
					return;
				}
				this.$emit('right-click', this.realValue);
			},
			//清除输入框
			doClearValue() {
				if(this.disabled){
					return;
				}
				this.realValue = '';
				if (this.type == 'textarea') {
					this.$refs.textarea.focus();
				} else if (this.isDatePicker) {
					this.realDate = null;
				} else {
					this.$refs.input.focus();
				}
				this.$emit('clear','')
			},
			//高度自适应设置
			autosizeSet() {
				this.$refs.textarea.style.height = 'auto';
				this.$refs.textarea.scrollTop = 0;
				this.$refs.textarea.style.height = $util.getScrollHeight(this.$refs.textarea) + 'px';
			},
			//行数转为高度
			rows2Height(rows) {
				let lineHeight = Math.floor(parseFloat($util.getCssStyle(this.$refs.textarea, 'line-height')));
				return rows * lineHeight;
			},
			//设置最大高度和最小高度
			setMaxMinHeight() {
				if ($util.isObject(this.autosize)) {
					if ($util.isNumber(this.autosize.maxRows)) {
						let maxHeight = this.rows2Height(this.autosize.maxRows);
						this.$refs.textarea.style.maxHeight = maxHeight + "px";
					}
					if ($util.isNumber(this.autosize.minRows)) {
						let minHeight = this.rows2Height(this.autosize.minRows);
						this.$refs.textarea.style.minHeight = minHeight + "px";
					}
				} else {
					this.$refs.textarea.style.maxHeight = "";
					this.$refs.textarea.style.minHeight = "";
				}
			},
			//调起日历
			callDate() {
				//如果是日历输入框
				if (this.isDatePicker && !this.disabled && !this.readonly) {
					this.$refs.datepicker.trigger();
				}
			},
			//选择日期后转换成输入框的value值
			getDateValue() {
				if (this.date instanceof Date) {
					if (this.dateType == 'date') {
						let year = this.date.getFullYear();
						let month = (this.date.getMonth() + 1) < 10 ? '0' + (this.date.getMonth() + 1) : this.date.getMonth() + 1;
						let date = this.date.getDate() < 10 ? '0' + this.date.getDate() : this.date.getDate();
						if (this.format == 'yyyy年mm月dd日') {
							return year + "年" + month + "月" + date + "日";
						} else if (this.format == 'yyyy/mm/dd') {
							return year + "/" + month + "/" + date;
						} else if (this.format == 'yyyy.mm.dd') {
							return year + "." + month + "." + date;
						} else {
							return year + "-" + month + "-" + date;
						}
					} else if (this.dateType == 'datetime') {
						let year = this.date.getFullYear();
						let month = (this.date.getMonth() + 1) < 10 ? '0' + (this.date.getMonth() + 1) : this.date.getMonth() + 1;
						let date = this.date.getDate() < 10 ? '0' + this.date.getDate() : this.date.getDate();
						let hour = this.date.getHours() < 10 ? '0' + this.date.getHours() : this.date.getHours();
						let minutes = this.date.getMinutes() < 10 ? '0' + this.date.getMinutes() : this.date.getMinutes();
						if (this.format == 'yyyy年MM月dd日hh时mm分') {
							return year + "年" + month + "月" + date + "日" + hour + "时" + minutes + "分";
						} else if (this.format == 'yyyy年MM月dd日 hh:mm') {
							return year + "年" + month + "月" + date + "日" + " " +  hour + ":" + minutes;
						} else if (this.format == 'yyyy/MM/dd hh:mm') {
							return year + "/" + month + "/" + date + " " + hour + ":" + minutes;
						} else if (this.format == 'yyyy.MM.dd hh:mm') {
							return year + "." + month + "." + date + " " + hour + ":" + minutes;
						} else {
							return year + "-" + month + "-" + date + " " + hour + ":" + minutes;
						}
					} else if (this.dateType == 'month') {
						let year = this.date.getFullYear();
						let month = (this.date.getMonth() + 1) < 10 ? '0' + (this.date.getMonth() + 1) : this.date.getMonth() + 1;
						if (this.format == 'yyyy年mm月') {
							return year + "年" + month + "月";
						} else if (this.format == 'yyyy/mm') {
							return year + "/" + month;
						} else if (this.format == 'yyyy.mm') {
							return year + "." + month;
						} else {
							return year + "-" + month;
						}
					} else if (this.dateType == 'time') {
						let hour = this.date.getHours() < 10 ? '0' + this.date.getHours() : this.date.getHours();
						let minutes = this.date.getMinutes() < 10 ? '0' + this.date.getMinutes() : this.date.getMinutes();
						if (this.format == 'hh时mm分') {
							return hour + "时" + minutes + "分";
						}else {
							return hour + ":" + minutes;
						}
					}
				} else {
					return "";
				}

			}
		},
	}
</script>

<style lang="less" scoped>
	@import "../../css/mvi-basic.less";

	//输入框组件容器
	.mvi-input-container {
		display: flex;
		display: -webkit-flex;
		justify-content: flex-start;
		position: relative;
		background-color: #fff;
		color: @font-color-default;
		font-size: @font-size-default;
		margin: 0;
		vertical-align: middle;
		width: 100%;
		padding: @mp-sm @mp-md;
	}

	//是否显示边框
	.mvi-input-container.mvi-input-border::after {
		display: block;
		content: "";
		position: absolute;
		left: @mp-sm;
		bottom: 0;
		width: calc(~'100% - @{mp-xs}*2');
		height: 0;
		border-bottom: 1px solid @border-color;
		box-sizing: content-box;
	}

	//必填标识
	.mvi-input-container.mvi-input-required::before {
		position: absolute;
		left: @mp-xs;
		top: 50%;
		transform: translateY(-50%);
		display: flex;
		display: -webkit-flex;
		justify-content: center;
		align-items: center;
		content: '*';
		font-family: '宋体','SimSun';
		color: @error-normal;
	}
	//必填标志大小
	.mvi-input-container.mvi-input-container-medium.mvi-input-required{
		font-size: @font-size-default;
	}
	.mvi-input-container.mvi-input-container-large.mvi-input-required {
		font-size: @font-size-h6;
	}

	//禁用样式
	.mvi-input-container[disabled] {
		opacity: .6;
	}

	//label文字
	.mvi-input-container>.mvi-input-label {
		display: flex;
		display:-webkit-flex;
		justify-content:flex-start;
		align-items:center;
		padding: 0;
		margin: 0 @mp-sm 0 0;
		position: relative;
		width: 1rem;
		vertical-align: middle;
		
		span{
			display: block;
			text-overflow: ellipsis;
			white-space: nowrap;
			overflow: hidden;
			vertical-align: middle;
		}
	}

	//左侧图标样式
	.mvi-input-container>.mvi-input-left-icon {
		display: flex;
		display: -webkit-flex;
		justify-content: center;
		align-items: center;
		margin-right: @mp-sm;
		vertical-align: middle;
	}

	//右侧图标样式
	.mvi-input-container>.mvi-input-right-icon {
		display: flex;
		display: -webkit-flex;
		justify-content: center;
		align-items: center;
		vertical-align: middle;
		margin-left: @mp-sm;
	}

	//清除图标
	.mvi-input-container>.mvi-input-clear-icon {
		display: flex;
		display: -webkit-flex;
		justify-content: center;
		align-items: center;
		opacity: .5;
		margin-left: @mp-sm;
		width: 0.3rem;
		vertical-align: middle;
		cursor: pointer;
	}

	//中等输入框时label、左侧图标、右侧图标、清除图标高度以及字体大小
	.mvi-input-container.mvi-input-container-medium .mvi-input-label,
	.mvi-input-container.mvi-input-container-medium .mvi-input-left-icon,
	.mvi-input-container.mvi-input-container-medium .mvi-input-right-icon,
	.mvi-input-container.mvi-input-container-medium .mvi-input-clear-icon {
		font-size: @font-size-default;
		height: .48rem;
	}
	
	//较大输入框时label、左侧图标、右侧图标、清除图标高度以及字体大小
	.mvi-input-container.mvi-input-container-large .mvi-input-label,
	.mvi-input-container.mvi-input-container-large .mvi-input-left-icon,
	.mvi-input-container.mvi-input-container-large .mvi-input-right-icon,
	.mvi-input-container.mvi-input-container-large .mvi-input-clear-icon{
		font-size: @font-size-h6;
		height: .6rem;
	}

	/* 非文本域时的样式 */
	.mvi-input-container[data-type='input'] {
		align-items: center;
	}

	//输入框
	.mvi-input-container>.mvi-input {
		appearance: none;
		-webkit-appearance: none;
		display: block;
		width: 100%;
		padding: 0;
		border: none;
		border-radius: 0;
		vertical-align: middle;
		color: inherit;
		background-color: inherit;
		flex: 1;
		font-family: @font-family;
		background-image: none;
		margin: 0;

		&::-webkit-input-placeholder,&::placeholder,&::-ms-input-placeholder,&::-moz-placeholder{
			color: inherit;
			font-family: inherit;
			font-size: inherit;
			opacity: .5;
			vertical-align: middle;
		}
		
		
		&[disabled]{
			background-color: inherit;
			color: inherit;
			opacity: 1;
		}
		
	}

	//不同大小的输入框
	.mvi-input-container.mvi-input-container-medium .mvi-input {
		height: .48rem;
		font-size: @font-size-default;
	}

	.mvi-input-container.mvi-input-container-large .mvi-input {
		height: .6rem;
		font-size: @font-size-h6;
	}


	//文本域时的样式
	.mvi-input-container[data-type='textarea'] {
		align-items: flex-start;
	}

	//文本域
	.mvi-input-container>.mvi-textarea {
		appearance: none;
		-webkit-appearance: none;
		display: block;
		width: 100%;
		flex: 1;
		padding: 0;
		border: none;
		border-radius: 0;
		height: auto;
		resize: none;
		vertical-align: middle;
		color: inherit;
		background-color: inherit;
		font-family: @font-family;
		background-image: none;
		margin: 0;
		
		&::-webkit-input-placeholder,&::placeholder{
			color: inherit;
			opacity: .5;
			font-family: inherit;
			font-size: inherit;
			vertical-align: middle;
		}
	}

	//不同大小的文本域
	.mvi-input-container.mvi-input-container-medium .mvi-textarea {
		line-height: .48rem;
		font-size: @font-size-default;
	}

	.mvi-input-container.mvi-input-container-large .mvi-textarea {
		line-height: .6rem;
		font-size: @font-size-h6;
	}


	//文字长度限制
	.mvi-input-container.mvi-input-container-words {
		padding-bottom: @mp-lg+@mp-xs;
	}

	.mvi-input-container.mvi-input-container-words>.mvi-input-words {
		position: absolute;
		bottom: @mp-xs;
		right: @mp-sm;
		width: 100%;
		line-height: 1;
		text-align: right;
		color: inherit;
		opacity: .5;
		font-size: @font-size-small;
	}
</style>
