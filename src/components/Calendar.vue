<template>
<div class="calendar">
	<div class="calendar-header">
	  <ul class="calendar-header-list">
		<li class="listitem-small calendar-header-prev" v-show="showPrevMonthButton"><a href="#" class="calendar-header-link" @click="getPrevMonth()"><</a></li>
		<li><span class="calendar-header-year">{{localizedYear}}</span><span class="calendar-header-month">{{localizedMonth}}</span></li>
		<li class="listitem-small calendar-header-next" v-show="showNextMonthButton"><a href="#" class="calendar-header-link" @click="getNextMonth()">></a></li>
	  </ul>
	</div>

	<div class="calendar-month">
	  <span v-for="day(value, index) in this.state.localizedWeekdays" class="calendar-month-header">{{value}}</span>
	  <span v-for="day(value, index) in this.daysInWeekFromPastMonth" class="calendar-month-cell inactive"></span>
	  <span v-for="day(value, index) in this.numberOfDaysInCurrentMonth" class="calendar-month-cell" :class="[{'today': isToday(value)},{'selected from-date': isFromDate(value)},{'selected to-date': isToDate(value)}, {'inrange': isInRange(value)}, {'inaccessible':isInThePast(value)}]" @click="setSelectedDates(value)">
		  {{value}}
	  </span>
	</div>
</div>
</template>

<script>

export default {
  name: 'Calendar',
  props: {
	  state: {
		  type: Object,
		  required: false,
	  },
	  calendarType: {
		  type: String,
		  required: true,
	  },
  },
  computed: {
	daysInWeekFromPastMonth(){
		return new Date(this.currentDate.getFullYear(), this.currentDate.getMonth(), 1).getDay();
	},
	showPrevMonthButton(){
		return ((this.state.bindCalendars && this.calendarType === "primary") || !this.state.bindCalendars) &&
		(this.state.allowPastDates || this.currentDate.getMonth() > new Date().getMonth());
	},
	showNextMonthButton(){
		return (this.state.bindCalendars && this.calendarType === "secondary") || !this.state.bindCalendars;
	},
	numberOfDaysInCurrentMonth(){
		return new Date(this.currentDate.getFullYear(), this.currentDate.getMonth()+1, 0).getDate();
	},
	currentDate(){
		return this.calendarType === "primary" ? this.state.primaryCalendarMonth : this.state.secondaryCalendarMonth;
	},
	today(){
		return this.currentDate.getDay();
	},
	localizedMonth(){
  		return this.currentDate.toLocaleString(this.state.propLocale, { month: "long" });
    },
  	localizedYear(){
  		return this.currentDate.toLocaleString(this.state.propLocale, { year: "numeric" });
    },
  },
  methods: {
  	isToday(value) {
		return this.currentDate.getMonth() === new Date().getMonth() && this.currentDate.getFullYear() === new Date().getFullYear() &&	this.today === value;
	},
	isFromDate(value) {
		const fromDate = this.state.selectedDates.fromDate !== null ? new Date(this.state.selectedDates.fromDate): null;
  		return (fromDate !== null && fromDate.getFullYear() === this.currentDate.getFullYear() && fromDate.getMonth() === this.currentDate.getMonth()) ? fromDate.getDate() === value: false;
  	},
	isToDate(value) {
		const toDate = this.state.selectedDates.toDate !== null ? new Date(this.state.selectedDates.toDate): null;
  		return (toDate !== null && toDate.getFullYear() === this.currentDate.getFullYear() && toDate.getMonth() === this.currentDate.getMonth()) ? toDate.getDate() === value: false;
  	},
	setSelectedDates(value) {
		if (this.allowPastDates || !this.isInThePast(value)) {
			const selectedDate = new Date(this.currentDate.getFullYear(), this.currentDate.getMonth(), value);
			const payload = {
				date: this.getDateInfo(selectedDate),
				dateType: this.getDateType(selectedDate),
			};
			this.$emit("dateSelected", payload);
		}
	},
	getDateTime(date) {
		return date !== null ? date.getTime() : null;
	},
	getDateInfo(selectedDate){
		const selectedDateTime = this.getDateTime(selectedDate);
		const fromDateTime = this.getDateTime(this.state.selectedDates.fromDate);
		const toDateTime = this.getDateTime(this.state.selectedDates.toDate);
		const dateInfo = (selectedDate !== null && (selectedDateTime === fromDateTime || selectedDateTime === toDateTime)) ? null : selectedDate;
		return dateInfo;
	},
	getDateType(selectedDate){
		const selectedDateTime = this.getDateTime(selectedDate);
		const fromDate = this.state.selectedDates.fromDate;
		const toDate = this.state.selectedDates.toDate;
		switch (selectedDateTime) {
			case this.getDateTime(fromDate):
				return "fromDate";
			case this.getDateTime(toDate):
				return "toDate";
			default:
				return ((fromDate === null && toDate === null) || (fromDate !== null && toDate !== null))  ? "fromDate": "toDate";
		}
	},
	getNextMonth(){
		let eventName = this.state.bindCalendars ? "getNextMonth" : "getNextMonthForCalendar";
		this.$emit(eventName, {calendarType: this.calendarType});
	},
	getPrevMonth(){
		let eventName = this.state.bindCalendars ? "getPrevMonth" : "getPrevMonthForCalendar";
		this.$emit(eventName, {calendarType: this.calendarType});
	},
	isInRange(dayIn) {
		if (this.state.selectedDates.toDate === null || this.state.selectedDates.fromDate === null) {
			return false;
		} else {
			const checkTime = new Date(this.currentDate.getFullYear(), this.currentDate.getMonth(), dayIn).getTime();
			const value = (checkTime <= this.state.selectedDates.toDate.getTime() && checkTime >= this.state.selectedDates.fromDate.getTime()) ? true: false;
			return value;
		}
	},
	isInThePast(dayIn) {
		const checkTime = new Date(this.currentDate.getFullYear(), this.currentDate.getMonth(), dayIn).getTime();
		const todaysTime = new Date().getTime();
		return checkTime < todaysTime ? true: false;
	}
  },
}
</script>

<style lang="scss" scoped>
.calendar {
	width: calc(7*50px+20px);
	float: left;
	.calendar-header {
		background-color: rgb(0, 142, 222);
		color: #fff;
		padding: 25px 10px;
		&:before,
		&:after {
			content: " "; /* 1 */
		    display: table; /* 2 */
		}
		&:after {
		    clear: both;
		}
		.calendar-header-year,
		.calendar-header-month {
			display: block;
			clear: both;
		}
		.calendar-header-year {
			font-size: 16px;
			line-height: 20px;
		}
		.calendar-header-month {
			font-size: 36px;
			line-height: 36px;
			margin-bottom: -20px;
		}

		.calendar-header-list {
			padding: 0;
			margin: 0;
			list-style-type: none;
			text-align: center;
			position: relative;
			li{
				display: inline-block;
				a{
				cursor: pointer;
					&:link,
					&:visited,
					&:hover {
						display: inline-block;
						text-decoration: none;
						color: #eee;
						line-height: 32px;
						height:32px;
						width: 32px;
						border-radius: 50%;
					}
					&:hover {
						background-color: rgb(0, 152, 232);
					}
				}
				&.calendar-header-prev,
				&.calendar-header-next {
					position:absolute;
					top: 50%;
					margin-top: -16px;
				}
				&.calendar-header-prev {
					left: 0;
				}
				&.calendar-header-next {
					right: 0;
				}
			}
		}
	}
	.calendar-month {
		padding: 10px;
		&:before,
		&:after {content: " "; display: table; }
		&:after {clear: both;}

		span {
			display: inline-block;
			width: 50px;
			height: 50px;
			line-height: 50px;
			text-align: center;
			float: left;
			cursor: pointer;
			&.today {
				background-color:#fff;
				border-radius: 50%;
				box-shadow: 0 2px 4px 0 rgba(0,0,0,0.3);
			}
			&.inactive {
				cursor: default;
			}
			&.selected {
				&.from-date{
					color: #fff;
					background-color: rgb(96, 170, 81);
					border-radius: 5px;
					&:after {
						content: '';
						position: absolute;
						width: 50px;
						height: 50px;
						display: block;
						left: 0;
						top: 0;
						background-color: rgb(96, 170, 81);
						border-radius: 5px 0 0 5px;
						z-index: -1;
					}
				}
				&.to-date{
					color: #fff;
					&:after {
						content: '';
						position: absolute;
						width: 50px;
						height: 50px;
						display: block;
						left: 0;
						top: 0;
						background-color: rgb(193, 81, 62);
						border-radius: 0 5px 5px 0;
						z-index: -1;
					}

				}
			}
			&.inrange{
				position: relative;
				&:before {
					content: '';
					position: absolute;
					width: 50px;
					height: 50px;
					display: block;
					left: 0;
					top:0;
					background-color:rgba(0, 0, 0, 0.05);
					z-index: -2;
				}
			}
		}
		span:nth-of-type(7n+1) {
			clear: left;
		}
	}
}
</style>
