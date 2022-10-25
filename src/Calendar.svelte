<!-- Credit: https://svelte.dev/repl/5aa7d011c2104b0c8906248f66da22ea?version=3.12.1 -->
<div class="calendar">
	{#each headers as header}
	  <span class="day-name" on:click={()=>dispatch('headerClick',header)}>{header}</span>
	{/each}

	{#each days as day}
		{#if day.enabled}
			<span class="day" on:click={()=>dispatch('dayClick',day)}>{day.name}</span>
		{:else}
			<span class="day day-disabled" on:click={()=>dispatch('dayClick',day)}>{day.name}</span>
		{/if}
	{/each}

	{#each groupedItemsArray as itemsArray}
    <div
        class="task"
        style="grid-column: {itemsArray[0].startCol} / span {1};      
        grid-row: {itemsArray[0].startRow};"
        >
            {#each itemsArray as item}
              {#if item.allDay}
                <div class="event-wrapper">
                  <a on:click={()=>dispatch('itemClick',item)} 
                    class="event all-day-event" 
                    style="background-color: {styling.eventHighlightColor}; border: 4px solid {styling.eventHighlightColor};"
                    onMouseOver="this.style.filter = 'brightness(85%)'"
                    onMouseOut="this.style.filter = 'brightness(100%)'"
                    >
                    <div class="event-title">
                      {item.title}
                    </div>
                  </a>
                </div>
              {:else}
                <div class="event-wrapper">
                  <a on:click={()=>dispatch('itemClick',item)} 
                    class="event event-time"
                    style="background-color: white"
                    onMouseOver="this.style.filter = 'brightness(85%)'"
                    onMouseOut="this.style.filter = 'brightness(100%)'"
                    >
                      <div class="event-dot" style="border: calc(8px/2) solid {styling.eventHighlightColor};"></div>
                      <div class="event-time">
                        {item.time}
                      </div>
                      <div class="event-title">
                        {item.title}
                      </div>
                  </a>
                </div>
              {/if}
            {/each}
        </div>
    {/each}
</div>

<script>
	import {createEventDispatcher, onMount} from 'svelte';

    let groupBy = function(xs, key) {
        return xs.reduce(function(rv, x) {
            (rv[x[key]] = rv[x[key]] || []).push(x);
            return rv;
        }, {});
    };

	export var headers = [];
	export let days = [];
	export let items = [];
  export let styling = {};

  let groupedItems = groupBy(items, 'combo');
  let keys = Object.keys(groupedItems);
  let groupedItemsArray = keys.map(key => groupedItems[key].sort((a, b) => {
    if(a.allDay && b.alLDay) 
      return a.title.localeCompare(b.title);
    else if(a.allDay && !b.allDay) {
      return -5;
    } else if(!a.allDay && b.allDay) {
      return 5;
    } else {
      if(a.date.getTime() == b.date.getTime()) {
        return a.title.localeCompare(b.title) * 101;
      } else if (a.date.getTime() > b.date.getTime()) {
        return -100;
      } else {
        return 100;
      }
    }
  }));

	let dispatch = createEventDispatcher();
	
</script>

<style>
.calendar {
  display: grid;
  width: 100%;
  grid-template-columns: repeat(7, minmax(120px, 1fr));
  grid-template-rows: 50px;
  grid-auto-rows: unset;
  overflow: auto;
}
.event-wrapper {
  clear: both;
  content: "";
  display: table;
  margin-top: 0px;
  width: 100%;
}
.event {
  color: #3e4042;
  cursor: pointer;
  align-items: center;
  display: flex;
  text-decoration: none;
  cursor: pointer;
  margin-left: 2px;
  margin-right: 2px;
  margin-top: 1px;
  z-index: 6;
  border-radius: 3px;
  font-size: .85em;
  position: relative;
  white-space: nowrap;
  box-sizing: border-box;
}
/* .time-event {

} */
/* .time-event:hover {
  background: rgba(0,0,0,.1);
} */
.event-time {
  margin-right: 3px;
  padding: 2px 0;
  margin-bottom: 1px;
}
.event-title {
  flex-grow: 1;
  flex-shrink: 1;
  font-weight: 600;
  min-width: 0;
  overflow: hidden;
  margin-bottom: 1px;
}
.all-day-event {
  text-align: center;
  color: white;
  border-radius: 8px;
  font-size: .85em;
  line-height: 10px;
}
.event-dot {
  border-radius: 4px;
  border-radius: calc(8px/2);
  box-sizing: content-box;
  height: 0;
  margin: 0 4px;
  width: 0;
  }
.day {
  border-bottom: 1px solid rgba(166, 168, 179, 0.12);
  border-right: 1px solid rgba(166, 168, 179, 0.12);
  text-align: right;
  padding: 10px 16px;
  letter-spacing: 1px;
  font-size: 14px;
  box-sizing: border-box;
  color: #98a0a6;
  position: relative;
  z-index: 1;
}
.day:nth-of-type(7n + 7) {
  border-right: 0;
}
.day:nth-of-type(n + 1):nth-of-type(-n + 7) {
  grid-row: 1;
}
.day:nth-of-type(n + 8):nth-of-type(-n + 14) {
  grid-row: 2;
}
.day:nth-of-type(n + 15):nth-of-type(-n + 21) {
  grid-row: 3;
}
.day:nth-of-type(n + 22):nth-of-type(-n + 28) {
  grid-row: 4;
}
.day:nth-of-type(n + 29):nth-of-type(-n + 35) {
  grid-row: 5;
}
.day:nth-of-type(n + 36):nth-of-type(-n + 42) {
  grid-row: 6;
}
.day:nth-of-type(7n + 1) {
  grid-column: 1/1;
}
.day:nth-of-type(7n + 2) {
  grid-column: 2/2;
}
.day:nth-of-type(7n + 3) {
  grid-column: 3/3;
}
.day:nth-of-type(7n + 4) {
  grid-column: 4/4;
}
.day:nth-of-type(7n + 5) {
  grid-column: 5/5;
}
.day:nth-of-type(7n + 6) {
  grid-column: 6/6;
}
.day:nth-of-type(7n + 7) {
  grid-column: 7/7;
}
.day-name {
  opacity: 0.75;
  color: #3e4042;
  font-size: var(--spectrum-global-dimension-font-size-200);
  font-weight: 600;
  /* font-size: 12px; */
  text-transform: uppercase;
  /* color: #e9a1a7; */
  text-align: center;
  border-bottom: 1px solid rgba(166, 168, 179, 0.12);
  line-height: 50px;
}
.day-disabled {
  color: rgba(152, 160, 166, 0.75);
  background-color: #ffffff;
  background-image: url("data:image/svg+xml,%3Csvg width='40' height='40' viewBox='0 0 40 40' xmlns='http://www.w3.org/2000/svg'%3E%3Cg fill='%23fdf9ff' fill-opacity='1' fill-rule='evenodd'%3E%3Cpath d='M0 40L40 0H20L0 20M40 40V20L20 40'/%3E%3C/g%3E%3C/svg%3E");
  cursor: not-allowed;
}
.task {
  font-size: 14px;
  position: relative;
  align-self: center;
	z-index:2;
	border-radius: 15px;
}
.event-wrapper:first-of-type {
    margin-top: 30px;
}
.task--warning {
  border-left-color: #fdb44d;
  background: #fef0db;
  color: #fc9b10;
  margin-top: -5px;
}
.task--danger {
  border-left-color: #fa607e;
  grid-column: 2 / span 3;
  grid-row: 3;
  margin-top: 15px;
  background: rgba(253, 197, 208, 0.7);
  color: #f8254e;
}
.task--info {
  border-left-color: #4786ff;
  margin-top: 15px;
  background: rgba(218, 231, 255, 0.7);
  color: #0a5eff;
}
.task--primary {
  background: #4786ff;
  border: 0;
  border-radius: 14px;
  color: #f00;
  box-shadow: 0 10px 14px rgba(71, 134, 255, 0.4);
}
.task-detail {
  position: absolute;
  left: 0;
  top: calc(100% + 8px);
  background: #efe;
  border: 1px solid rgba(166, 168, 179, 0.2);
  color: #100;
  padding: 20px;
  box-sizing: border-box;
  border-radius: 14px;
  box-shadow: 0 10px 40px rgba(0, 0, 0, 0.08);
  z-index: 2;
}
.task-detail h2 {
  font-size: 15px;
  margin: 0;
  color: #91565d;
}
.task-detail p {
  margin-top: 4px;
  font-size: 12px;
  margin-bottom: 0;
  font-weight: 500;
  color: rgba(81, 86, 93, 0.7);
}

</style>


