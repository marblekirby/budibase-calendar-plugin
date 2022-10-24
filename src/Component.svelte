<script>
	import Calendar from "./Calendar.svelte";
	import {getContext, createEventDispatcher, onMount} from 'svelte';

  const { styleable } = getContext("sdk")
  const component = getContext("component")

  export let dataProvider;
  export let eventDateMapping;
  export let eventTitleMapping;
  export let onEventClick = null;

	var dayNames = ["Sun", "Mon", "Tue", "Wed", "Thu", "Fri", "Sat"];
	let monthNames = ["January", "February", "March", "April", "May", "June", "July", "August", "September", "October", "November", "December"];

	let headers = [];
	let now = new Date();
	let year = now.getFullYear();		//	this is the month & year displayed
	let month = now.getMonth();
	// let eventText="Click an item or date";

  let dateKey = eventDateMapping;
  let titleKey = eventTitleMapping;

  let allItems = dataProvider?.rows?.map(x => { 
    return {
      title: x[titleKey],
      date: new Date(x[dateKey])
    }
  }) ?? [];

	var days = [];	//	The days to display in each box

	function randInt(max) {
		return Math.floor(Math.random()*max)+1;
	}
	
	//	The Calendar Component just displays stuff in a row & column. It has no knowledge of dates.
	//	The items[] below are placed (by you) in a specified row & column of the calendar.
	//	You need to call findRowCol() to calc the row/col based on each items start date. Each date box has a Date() property.
	var items = [];

	const initMonthItems = () => {
    items = [];
    
		let y = year - 1900;
		let m = month;

    let thisMonthItems = allItems.filter(x => {
      let xM = Number(x.date.getMonth())
      let xY = Number(x.date.getYear())
      return xM == m && xY == y;
    });

		//This is where you calc the row/col to put each dated item
		for (let i of thisMonthItems) {
			let rc = findRowCol(i.date);
			if (rc == null) {
				i.startCol = i.startRow = 0;
			} else {
				i.startCol = rc.col;
				i.startRow = rc.row;
        i.combo = `${rc.row},${rc.col}`

        items.push(i);
			}
		}
	}

	$: month,year,initContent();

	// choose what date/day gets displayed in each date box.
	function initContent() {
		headers = dayNames;
		initMonth();
		initMonthItems();
	}

	function initMonth() {
		days = [];
		let monthAbbrev = monthNames[month].slice(0,3);
		let nextMonthAbbrev = monthNames[(month+1)%12].slice(0,3);
		//	find the last Monday of the previous month
		var firstDay = new Date(year, month, 1).getDay();
		//console.log('fd='+firstDay+' '+dayNames[firstDay]);
		var daysInThisMonth = new Date(year, month+1, 0).getDate();
		var daysInLastMonth = new Date(year, month, 0).getDate();
		var prevMonth = month==0 ? 11 : month-1;
		
		//	show the days before the start of this month (disabled) - always less than 7
		for (let i=daysInLastMonth-firstDay;i<daysInLastMonth;i++) {
			let d = new Date(prevMonth==11?year-1:year,prevMonth,i+1);
			days.push({name:''+(i+1),enabled:false,date:d,});
		}
		//	show the days in this month (enabled) - always 28 - 31
		for (let i=0;i<daysInThisMonth;i++) {
			let d = new Date(year,month,i+1);
			if (i==0) days.push({name:monthAbbrev+' '+(i+1),enabled:true,date:d,});
			else days.push({name:''+(i+1),enabled:true,date:d,});
			//console.log('i='+i+'  dt is '+d+' date() is '+d.getDate());
		}
		//	show any days to fill up the last row (disabled) - always less than 7
		for (let i=0;days.length%7;i++) {
			let d = new Date((month==11?year+1:year),(month+1)%12,i+1);
			if (i==0) days.push({name:nextMonthAbbrev+' '+(i+1),enabled:false,date:d,});
			else days.push({name:''+(i+1),enabled:false,date:d,});
		}
	}

	function findRowCol(dt) {
		for (let i=0;i<days.length;i++) {
			let d = days[i].date;
			if (d.getYear() === dt.getYear()
				&& d.getMonth() === dt.getMonth()
				&& d.getDate() === dt.getDate())
				return {row:Math.floor(i/7)+2,col:i%7+1};
		}
		return null;	
	}

	function itemClick(e) {
    if(onEventClick)
		  onEventClick(e);
	}
	function dayClick(e) {
	}
	function headerClick(e) {
		
	}
	function nextMonth() {
		month++;
		if (month == 12) {
			year++;
			month=0;
		}
    initMonth();
		initMonthItems();
	}
  function nextYear() {
		year++;
    initMonth();
		initMonthItems();
	}
	function prevMonth() {
		if (month==0) {
			month=11;
			year--;
		} else {
			month--;
		}
    initMonth();
		initMonthItems();
	}
  function prevYear() {
		year--;
    initMonth();
		initMonthItems();
	}
	
</script>

<div class="calendar-container" use:styleable={$component.styles}>
  <div class="calendar-header">
    <h1>
      <button on:click={()=>year--}>&Lt;</button>
      <button on:click={()=>prevMonth()}>&lt;</button>
       {monthNames[month]} {year}
      <button on:click={()=>nextMonth()}>&gt;</button>
      <button on:click={()=>year++}>&Gt;</button>
    </h1>
		<!-- {eventText} -->
	</div>

	{#key items}
		<Calendar
			{headers}
			{days}
			{items}
		on:itemClick={(e)=>itemClick(e)}
			/>
	{/key}
</div>
	
<style>
.calendar-container {
  width: 100%;
  height: 100%;
  margin: auto;
  overflow: hidden;
  box-shadow: 0 2px 20px rgba(0, 0, 0, 0.1);
  border-radius: 10px;
  background: #fff;
}
.calendar-header {
  text-align: center;
  padding: 20px 0;
  background: #eef;
  border-bottom: 1px solid rgba(166, 168, 179, 0.12);
}
.calendar-header h1 {
  margin: 0;
  font-size: 18px;
}
.calendar-header button {
  background: #eef;
  border: 1px ;
  padding: 6;
  color: rgba(81, 86, 93, 0.7);
  cursor: pointer;
  outline: 0;
}
</style>



