<template>
	<input @blur="selectDateForIOS" @change="selectDateForAndroid" v-on="listeners" class="mvi-date-native-picker" :type="dateType"/>
</template>

<script>
	import $util from "../../util/util"
	export default {
		name:"m-date-native-picker",
		model:{
			prop:'value',
			event:'model-change'
		},
		props:{
			type:{//选择的类型
				type:String,
				default:'date',
				validator(value){
					return ['date','datetime','month','time'].includes(value)
				}
			},
			value:{
				type:Date,
				default:null
			},
			max:{
				type:Date,
				default:null
			},
			min:{
				type:Date,
				default:null
			}
		},
		computed:{
			listeners(){
				return Object.assign({},this.$listeners)
			},
			dateType(){
				if(this.type == 'datetime'){
					return 'datetime-local';
				}else{
					return this.type;
				}
			}
		},
		methods:{
			//触发日期选择弹窗
			trigger(){
				if($util.judgeAccessTerminalBrowser('ios')){
					this.$el.focus();
				}else{
					this.$el.click();
				}
			},
			//IOS下选择日期
			selectDateForIOS(){
				if($util.judgeAccessTerminalBrowser('ios')){
					let date = this.dateParse(this.$el.value);
					if(date){
						if(this.min){
							let minTime = this.min.getTime();
							if(date.getTime() < minTime){
								this.$emit('error','选择的时间小于限定最小时间');
								return;
							}
						}
						if(this.max){
							let maxTime = this.max.getTime();
							if(date.getTime() > maxTime){
								this.$emit('error','选择的时间大于限定最大时间')
								return;
							}
						}
						
					}
					this.$emit('update:value',date);
					this.$emit('model-change',date);
				}
			},
			//安卓系统下选择日期
			selectDateForAndroid(){
				if(!$util.judgeAccessTerminalBrowser('ios')){
					let date = this.dateParse(this.$el.value);
					if(date){
						if(this.min){
							let minTime = this.min.getTime();
							if(date.getTime() < minTime){
								this.$emit('error','选择的时间小于限定最小时间');
								return;
							}
						}
						if(this.max){
							let maxTime = this.max.getTime();
							if(date.getTime() > maxTime){
								this.$emit('error','选择的时间大于限定最大时间')
								return;
							}
						}
					}
					this.$emit('update:value',date);
					this.$emit('model-change',date);
				}
			},
			//解析方法
			dateParse(value){
				if(value){
					if(this.type == 'date'){
						let obj = this.dateParseDate(value);
						let d = new Date();
						d.setFullYear(obj.year);
						d.setMonth(obj.month-1);
						d.setDate(obj.date);
						return d;
					}else if(this.type == 'datetime'){
						let dateArray = value.split('T');
						let dateObj = this.dateParseDate(dateArray[0]);
						let timeObj = this.dateParseTime(dateArray[1]);
						let d = new Date();
						d.setFullYear(dateObj.year);
						d.setMonth(dateObj.month - 1);
						d.setDate(dateObj.date);
						d.setHours(timeObj.hour);
						d.setMinutes(timeObj.min);
						return d;
					}else if(this.type == 'month'){
						let obj = this.dateParseDate(value);
						let d = new Date();
						d.setFullYear(obj.year);
						d.setMonth(obj.month-1);
						return d;
					}else if(this.type == 'time'){
						let obj = this.dateParseTime(value);
						let d = new Date();
						d.setHours(obj.hour);
						d.setMinutes(obj.min);
						return d;
					}
				}else{
					return null;
				}
				
			},
			//解析时间(格式如22:22)
			dateParseTime(value){
				let dateArray = value.split(':');
				let hour = dateArray[0];
				let min = dateArray[1];
				return {hour,min};
			},
			//解析日期(格式如2019-01-20)
			dateParseDate(value){
				let dateArray = value.split('-');
				let year = Number(dateArray[0]);//年份
				let month = Number(dateArray[1]);//月份
				let date = Number(dateArray[2]);//日期
				return {year,month,date};
			}
		}
	}
</script>

<style lang="less" scoped>
	@import "../../css/mvi-basic.less";
	
	.mvi-date-native-picker{
		width: 0;
		height: 0;
		opacity: 0;
		border: none;
		min-width: 0;
		max-width: 0;
		position: relative;
	}
</style>
