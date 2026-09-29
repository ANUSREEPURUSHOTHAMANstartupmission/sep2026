<script>
	import {onMount} from 'svelte';

	export let date;

	let countDate = new Date(date).getTime();
	let interval = null;

	let dayText = 0, hourText = 0, minText = 0, secText = 0;

	onMount(() => {
		console.log(countDate);
		interval = setInterval(moveTimer, 1000)
	})

	function moveTimer(){
		const now = new Date().getTime();
		const gap = countDate - now;

		const second = 1000;
    const minute = second * 60;
    const hour = minute * 60;
    const day = hour * 24;

		if(gap < 0){
      clearInterval(interval);
			return;
    }

		const textDay = Math.floor(gap / day);
    const textHour = Math.floor((gap % day) / hour);
    const textMin = Math.floor((gap % hour) / minute);
    const textSec = Math.floor((gap % minute) / second);

		dayText = textDay.toLocaleString('en-US', {minimumIntegerDigits: 2, useGrouping:false});
    hourText = textHour.toLocaleString('en-US', {minimumIntegerDigits: 2, useGrouping:false});
    minText = textMin.toLocaleString('en-US', {minimumIntegerDigits: 2, useGrouping:false});
    secText = textSec.toLocaleString('en-US', {minimumIntegerDigits: 2, useGrouping:false});
	}

</script>

<div class="flex  flex-wrap space-x-4 gap-4 mb-2">
	<div class="counter rounded-2xl text-black  font-bold   text-xl    flex flex-col ">
		{dayText}
		<div class="text-xs font-light font-sans">Days</div>
	</div>
	<div class=" counter rounded-2xl text-black  font-bold   text-xl   flex flex-col ">
		{hourText}
		<div class="text-xs font-light font-sans">Hours</div>
	</div>
	<div class="counter rounded-2xl text-black  font-bold   text-xl   flex flex-col ">
		{minText}
		<div class="text-xs font-light font-sans">Mins</div>
	</div>
	<div class="counter rounded-2xl text-black  font-bold   text-xl   flex flex-col ">
		{secText}
		<div class="text-xs font-light font-sans">Secs</div>
	</div>
</div>

