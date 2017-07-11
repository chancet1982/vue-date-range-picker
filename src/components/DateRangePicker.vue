<template>
	<div class="date-picker" v-click-outside="outsideClickHandler">
		<div v-show="selectedToDate" class="result" v-if="selectedFromDate !== null && selectedToDate !== null">
			<h1>{{fromDateText}} {{getLocalizedDate(selectedFromDate)}} {{toDateText}} {{getLocalizedDate(selectedToDate)}}</h1>
		</div>

		<div class="mq-desktop-only">
			<input class="date-picker-desktop-input" :placeholder="desktopInputPlaceholder" @focus="inputFocusHandler" v-show="showInput" />
			<button @click="toggleCalendarButtonHandler()" class="btn toggle-calendar-button"><span v-show="this.calendarContainerVisible">Collapse</span><span v-show="!this.calendarContainerVisible">Expand</span></button>
		</div>

		<div class="mq-mobile-only">
			<div class="date-picker-from">
				<label for="date-picker-input">{{fromDateText}}</label>
			    <input class="date-picker-from-input" type="date" placeholder="dd-mm-yyyy" v-model="selectedFromDate" />
			</div>
			<div class="date-picker-to">
				<label for="date-picker-input">{{toDateText}}</label>
			    <input class="date-picker-to-input" type="date" placeholder="dd-mm-yyyy" v-model="selectedToDate" />
			</div>
		</div>

		<div class="date-picker-calendar" v-show="calendarContainerVisible" >
			<div class="toggle-calendar-binding" @click="toggleCalendarsBinding()">
				<span v-show="!propBindCalendars"><svg xmlns="http://www.w3.org/2000/svg" width="12" height="12"><rect x="2" y="6" width="8" height="5" rx="1" fill="#eee"/><path fill="none" stroke="#eee" d="m3.5,4a2.5,2.5 0 0,1 5,0v2"/></svg></span>
				<span v-show="propBindCalendars"><svg xmlns="http://www.w3.org/2000/svg" width="12" height="12"><rect x="2" y="6" width="8" height="5" rx="1" fill="#eee"/><path fill="none" stroke="#eee" d="m3.5,6V4a2.5,2.5 0 0,1 5,0v2"/></svg></span>
			</div>
			<calendar @dateSelected="dateSelectedHandler" @getNextMonth="getNextMonthHandler" @getPrevMonth="getPrevMonthHandler" @getNextMonthForCalendar="getNextMonthForCalendar" @getPrevMonthForCalendar="getPrevMonthForCalendar" :state="state" calendar-type="primary"></calendar>
			<calendar @dateSelected="dateSelectedHandler" @getNextMonth="getNextMonthHandler" @getPrevMonth="getPrevMonthHandler" @getNextMonthForCalendar="getNextMonthForCalendar" @getPrevMonthForCalendar="getPrevMonthForCalendar" :state="state" calendar-type="secondary"></calendar>
			<div class="date-picker-calendar-footer" v-show="showButtons">
			  <a href="#" class="btn btn-50 btn-cancel" @click="closeAndDiscard">{{cancelButtonText}}</a>
			  <a href="#" class="btn btn-50 btn-confirm" @click="closeAndSave">{{confirmButtonText}}</a>
			</div>
		</div>
	</div>
</template>

<script>
import Calendar from './Calendar.vue'

export default {
  name: 'DatePicker',
  props: {
	  date: {
		  type: Date,
		  required: false,
		  default: () => new Date(),
	  },
	  locale: {
		  type: String,
		  required: true,
		  default: "en-gb",
	  },
	  showCalendarArea: {
		  type: Boolean,
		  required: false,
		  default: false,
	  },
	  showButtons: {
		  type: Boolean,
		  required: false,
		  default: true,
	  },
	  showInput: {
		  type: Boolean,
		  required: false,
		  default: false,
	  },
	  cancelButtonText: {
		  type: String,
		  required: false,
		  default: "cancel",
	  },
	  confirmButtonText: {
		  type: String,
		  required: false,
		  default: "confirm",
	  },
	  inputFocusOpen: {
		type: Boolean,
		required: false,
		default: true,
	  },
	  outsideClickClose: {
		  type: Boolean,
		  required: false,
		  default: true,
	  },
	  allowPastDates: {
		type: Boolean,
		required: false,
		default: false,
	  },
	  buttonClickOpens: {
		  type: Boolean,
		  required: false,
		  default: true,
	  },
	  bindCalendars: {
		  type: Boolean,
		  required: false,
		  default: true,
	  },
  },
  components: {
	Calendar,
  },
  directives: {
    'click-outside': {
		bind (el, binding, vnode) {
		  el.event = (event) => {
			if (!(el == event.target || el.contains(event.target) )) {
			  vnode.context[binding.expression](event);
			}
		  };
		  document.body.addEventListener('click', el.event)
		},
		unbind (el) {
		  document.body.removeEventListener('click', el.event)
		},
    },
  },
  computed: {
	  state() {
		return {
		  selectedDates: {
			  fromDate: this.tempFromDate,
			  toDate: this.tempToDate,
		  },
		  propLocale: this.locale,
		  localizedWeekdays: this.localizedWeekdays,
		  primaryCalendarMonth: this.primaryCalendarMonth,
		  secondaryCalendarMonth: this.secondaryCalendarMonth,
		  allowPastDates: this.allowPastDates,
		  bindCalendars: this.propBindCalendars,
		}
	  },
	  localizedWeekdays() {
  		return [0,1,2,3,4,5,6].map((val) => {return new Date(this.date.getFullYear(), this.date.getMonth(), val).toLocaleString(this.locale, {weekday: 'short'})});
  	  },
	  autoPopulateDates () {
		return !this.showButtons;
	  },
	  isValid() {
		  return this.tempFromDate !== null && this.tempToDate !== null;
	  }
  },
  methods: {
	  outsideClickHandler(){
		if (this.outsideClickClose) {
			this.closeAndSave();
		}
	  },
	  getLocalizedDate(date){
  		const options = { weekday: 'long', year: 'numeric', month: 'short', day: 'numeric' };
    		return date.toLocaleString(this.state.propLocale, options);
      },
	  inputFocusHandler(){
		  if (this.inputFocusOpen) {
			  this.calendarContainerVisible = true ;
		  }
	  },
	  toggleCalendarButtonHandler(){
		if (this.buttonClickOpens) {
			this.calendarContainerVisible = !this.calendarContainerVisible ;
		}
	  },
	  closeAndDiscard(){
		  this.calendarContainerVisible = false;
		  this.tempFromDate = null;
		  this.tempFromDate = null;
		  this.selectedFromDate = null;
		  this.selectedToDate = null;
	  },
	  closeAndSave(){
		  this.calendarContainerVisible = false;
		  if (!this.autoPopulateDates) {
			  this.populateSelectedDates();
		  }
	  },
	  populateSelectedDates(){
		  this.selectedFromDate = this.tempFromDate;
		  this.selectedToDate = this.tempToDate;
		  const payload = {
			  selectedFromDate: this.selectedFromDate,
			  selectedToDate: this.selectedToDate,
		  };
		  this.$emit("selected-dates-change", payload);
	  },
	  toggleCalendarsBinding() {
		  this.bindCalendars = !this.bindCalendars;
	  },
	  getNextMonthForCalendar(payload){
		  switch (payload.calendarType) {
		  	case "primary":
				this.primaryCalendarMonth = this.getNextMonth(this.state.primaryCalendarMonth);
		  		break;
			case "secondary":
				this.secondaryCalendarMonth = this.getNextMonth(this.state.secondaryCalendarMonth);
		  		break;
		  	default:
		  		console.error("getNextMonthForCalendar(payload): UNKNOWN calendarType! payload: ", payload);
		  };
	  },
	  getPrevMonthForCalendar(payload){
		  switch (payload.calendarType) {
		  	case "primary":
				this.primaryCalendarMonth = this.getPrevMonth(this.state.primaryCalendarMonth);
		  		break;
			case "secondary":
				this.secondaryCalendarMonth = this.getPrevMonth(this.state.secondaryCalendarMonth);
		  		break;
		  	default:
		  		console.error("getNextMonthForCalendar(payload): UNKNOWN calendarType! payload: ", payload);
		  };
	  },
	  getNextMonthHandler(){
		  this.primaryCalendarMonth = this.getNextMonth(this.state.primaryCalendarMonth)
		  this.secondaryCalendarMonth = this.getNextMonth(this.state.secondaryCalendarMonth);
	  },
	  getPrevMonthHandler(){
  		  this.primaryCalendarMonth = this.getPrevMonth(this.state.primaryCalendarMonth)
		  this.secondaryCalendarMonth = this.getPrevMonth(this.state.secondaryCalendarMonth);
	  },
	  getNextMonth(inputDate){
		return inputDate.getMonth() < 11 ? new Date(inputDate.getFullYear(), inputDate.getMonth()+1, inputDate.getDate()): new Date(inputDate.getFullYear() +1, 0, inputDate.getDate());
	  },
	  getPrevMonth(inputDate){
		return inputDate.getMonth() > 0 ? new Date(inputDate.getFullYear(), inputDate.getMonth()-1, inputDate.getDate()) : new Date(inputDate.getFullYear() -1, 12, inputDate.getDate());
	  },
	  dateSelectedHandler(payload) {
		const payloadDate = payload.date !== null ? new Date(payload.date): null;
		if (payload.dateType === "fromDate") {
			this.tempFromDate = payloadDate;
			this.tempToDate = null;
		} else if (payload.dateType === "toDate" && this.isAfterFromDate(payloadDate)) {
			this.tempToDate = payloadDate;
		}
		if (this.autoPopulateDates) {
			this.populateSelectedDates();
		}
	  },
	  isAfterFromDate(payloadDate){
		  return payloadDate === null ? true: payloadDate.getTime() > this.tempFromDate.getTime()
	  }
  },
  data () {
    return {
	  fromDateText: '',
	  toDateText: ' - ',
	  desktopInputPlaceholder: 'Focus on input to expand datepicker',
	  calendarContainerVisible: this.showCalendarArea,
	  selectedFromDate: null,
	  selectedToDate: null,
	  tempFromDate: null,
	  tempToDate: null,
	  baseDate: this.date,
	  propBindCalendars: this.bindCalendars,
	  primaryCalendarMonth: this.date,
	  secondaryCalendarMonth: this.getNextMonth(this.date),
    }
  }
}
</script>

<style lang="scss" scoped>
$desktop-width: 1024px;

@mixin mobile-devices {
  @media (max-width: #{$desktop-width - 1px}) {
    @content;
  }
}

@mixin desktop {
  @media (min-width: #{$desktop-width}) {
    @content;
  }
}

@include mobile-devices {
	.mq-desktop-only {
		display: none;
	}
}

@include desktop {
	.mq-mobile-only {
		display: none;
	}
}

.date-picker-from,
.date-picker-to,
.date-picker-desktop-input {
	position: relative;
}

.date-picker-calendar {
	width: calc(14*50px+40px);
	position: absolute;
	left: 50%;
	margin-left: calc(-7*50px);
	background-color: #fff;
	box-shadow: 0 10px 20px 0 rgba(0,0,0,0.3);
	z-index: 999999;
	.date-picker-calendar-footer {
		clear: both;
		.btn{
			display: inline-block;
			cursor: pointer;
			text-align: center;
			padding: 10px;
			color: #fff;
			text-decoration: none;
			border-radius: 5px;
			&.btn-50 {
				width: calc(7*50px+20px);
				margin-bottom: 10px;
			}
			&.btn-cancel {
				background-color: rgb(180, 70, 100);
				&:hover {
					background-color: rgb(190, 80, 110);
				}
			}
			&.btn-confirm {
				background-color: rgb(110, 190, 70);
				&:hover {
					background-color: rgb(120, 200, 80);
				}
			}
		}
	}
	.toggle-calendar-binding {
		position:absolute;
		top:0;
		bottom:0;
		width:12px;
		left: 50%;
		margin-left: -6px;
		span {
			display: block;
			height: 32px;
			width: 32px;
			border-radius: 50%;
			cursor: pointer;
			position: relative;
			top: 54px;
			left:50%;
			margin-left: -16px;
			margin-top: -16px;
			&:hover {
				background-color:  rgb(0, 152, 232);
			}
			svg {
				position: absolute;
				top: 50%;
				left:50%;
				margin-top: -6px;
				margin-left: -6px;
			}
		}
	}
}

</style>

//TODO Add presents (phase 2.0)
//TODO add cell shifting based on locale. (phase 2.0)
//TODO add awesome handling to when both primary and secondary calendar are showing the same month (phase 2.0)
//TODO add global date handling.
//TODO add nicer style for buttons in general.
//TODO add close dialog button
