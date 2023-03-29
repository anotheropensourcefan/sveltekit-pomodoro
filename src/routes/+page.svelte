<style>
	:root {
	  --font-size: 24;
	}
	* {
	  font-family: GlassAntiqua, Verdana, Courier Prime, Courier New, monospace;
	  font-size: 12px;
	}

	h1 {
	  text-align: center;
	  font-size: 150%;
	}

	progress {
	  border-radius: 5px;
	  width: 100%;
	}

	p,
	input,
	button {
	  text-align: center;
	}

	input {
	  width: calc(var(--font-size) * 5px);
	}

	#name-input {
	  width: 100%;
	  background: none !important;
	  border: none;
	}

	.sub {
	  font-size: 50%;
	}

	button {
	  cursor: pointer;
	  border: none;
	  background: none;
	  color: green;
	}

	button:hover {
	  color: inherit;
	}

	.params-grid {
	  width: 100%;
	  display: flex;
	  flex-direction: row;
	  flex-wrap: wrap;
	  padding-bottom: 10px;
	  justify-content: space-evenly;
	  flex-wrap: wrap;
	}
	.params-grid > span {
	  flex: 0 0 auto;
	}
	.left {
	  text-align: right;
	  width: 35%;
	  margin-right: 5%;
	}
	.right {
	  text-align: left;
	  width: 35%;
	  margin-left: 5%;
	}
	.time-wrapper {
	  width: 20%;
	  text-align: center;
	}
	.preset {
	  width: 15%;
	  margin: auto;
	}
	.start-pause {
	  width: 25%;
	  margin: auto;
	  font-size : --font-size;
	}

	.time {
	  padding-bottom: 0;
	  padding-top: 0;
	  margin: 0;
	}
</style>

<script>
	// audio
	let cuckoo;

	let firstVisit = true;
	let running = false;
	// to start or pause
	let session = false;
	let pause = false;
	let sessionMinutes;
	let pauseMinutes;
	function toDefault() {
	  sessionMinutes = 25;
	  pauseMinutes = 5;
	}
	toDefault();
	let dateNow = Date.now();
	let newDate = new Date();
	let minutes;
	let seconds;
	let interval;

	let name = "";
	let focusMode = false;

	let morningStart = 7;
	let lunchStart = 12;
	let lunchEnd = 13;
	let meal = "Lunch";
	let eveningEnd = 19;

	if (lunchStart <= 9 && lunchEnd <= 10 && lunchEnd - lunchStart <= 3) {
	  meal = "Breakfast";
	} else if (lunchStart >= 18 && lunchEnd >= 24 && lunchEnd - lunchStart <= 3) {
	  meal = "Dinner";
	} else if (lunchEnd - lunchStart >= 4) {
	  meal = "Major break";
	} else {
	  meal = "Lunch";
	}

	let random = 0;
	function setRandom(number = 1) {
	  random = Math.random();
	  random = Math.round(random * number);
	}

	let messageTime = "";

	const arrayMessageFirstVisit = [
	  "Welcome to Pomo d'oro !",
	  "Discover Pomo d'oro, your companion for time tracking",
	  "Never miss a break with Pomo d'oro",
	  "Say hi to productivity ! Say hi to Pomo d'oro !",
	  "Pomo d'oro will get you both productive and happy.",
	  "Start your Pomo d'oro experience now !",
	  "Organize yourself with Pomo d'oro. It's free like FCC."
	];
	let messageFirstVisit = arrayMessageFirstVisit[random];

	// setRandom(Math.min(arrayMessageOpeningNoName.length,arrayMessageFirstVisit.length))

	setRandom(1);
	// normalement, setRandom(arrayMessageFirstVisit.length) puis setRandom à chaque changement session/break

	const arrayMessageNotRunning = [
	  "Come back when it feels right.",
	  "Pomo d'oro will get you both productive and happy.",
	  "Pomo d'oro, your companion for time tracking.",
	  "Never ever miss a break with Pomo d'oro."
	];
	let messageNotRunning = arrayMessageNotRunning[random];

	const arrayMessagePause = [
	  "Time to relax",
	  "Make a break and you will work faster. So you will do more breaks"
	];
	let messagePause = arrayMessagePause[random];

	const arrayMessageSession = [
	  {
	    author: "Flaubert",
	    source: "Correspondances",
	    quote:
	      "Je vais me mettre à travailler furieusement, à peine rentré; je l'espère du moins. La vie n'est tolérable qu'avec une marotte, un travail quelconque. Dès qu'on abandonne sa chimère, on meurt de tristesse. Il faut se cramponner dessus et souhaiter qu'elle nous emporte."
	  },
	  {
	    author: "Voltaire",
	    source: "Candide",
	    quote:
	      "Travaillons sans raisonner, c'est le seul moyen de rendre la vie supportable."
	  },
	  {
	    author: "Voltaire",
	    source: "Candide",
	    quote:
	      "Le travail éloigne de nous trois grands maux: l'ennui, le vice, et le besoin."
	  },
	  {
	    author: "Antoine de Saint-Exupéry",
	    source: "Le petit Prince",
	    quote:
	      "Peut-être bien que cet homme est absurde. Cependant il est moins absurde que le roi, que le vaniteux, que le businessman et que le buveur. Au moins son travail a-t-il un sens. Quand il allume son réverbère, c'est comme s'il faisait naître une étoile de plus, ou une fleur. Quand il éteint son réverbère, ça endort la fleur ou l'étoile. C'est une occupation très jolie. C'est véritablement utile puisque c'est joli."
	  }
	];
	let messageSession = arrayMessageSession[random];

	const history = [
	  {
	    date: new Date().toLocaleString("en", {
	      dateStyle: "short"
	    }),
	    sessions: 0,
	    breaks: 0
	  }
	];

	let number = 1;

	// found on github/AndersDJohnson/setIntervalSynchronous.js
	var setIntervalSynchronous = function(func, delay) {
	  var intervalFunction, timeoutId, clear;
	  // Call to clear the interval.
	  clear = function() {
	    clearTimeout(timeoutId);
	  };
	  intervalFunction = function() {
	    func();
	    timeoutId = setTimeout(intervalFunction, delay);
	  };
	  // Delay start.
	  timeoutId = setTimeout(intervalFunction, delay);
	  // You should capture the returned function for clearing.
	  return clear;
	};
	setIntervalSynchronous(() => {
	  newDate = new Date();
	  dateNow = Math.round(1679723994 / 60 - Date.now() / 60000);
	}, 1000);
	// chaque seconde, je recalcule tout : soit j'ai la valeur calculée mathématiquement, soit j'ai l'intervalle entre Date.now() et dateNowChange
	// OU je fais confiance à l'objet Date. La seule valeur mise à jour (state) est le timestamp en millisecondes. Celui-ci est utilisé pour créer une valeur de fin, et une intervalle. à partir de cette intervalle, je calcule les variables d'état "minutes" et "secondes"

	let dateNowChange = Date.now();
	let minutesDate;
	let secondsDate;

	function resetCounter() {
	  let targetMinutes;
	  session ? (targetMinutes = sessionMinutes) : (targetMinutes = pauseMinutes);
	  dateNowChange = Date.now() + targetMinutes * 60000;
	  setIntervalToChange();
	  updateCounter();
	}

	let intervalToChange;
	function setIntervalToChange() {
	  //intervalToChange est mis à jour chaque seconde avec updateCounter()
	  intervalToChange = dateNowChange - Date.now();
	  // minutes:seconds to display are calculated each second, based on the interval
	  minutesDate = Math.ceil(intervalToChange / 60000);
	  secondsDate = Math.round(intervalToChange / 1000) - (minutesDate - 1) * 60;
	}
	let intervalSeconds;

	function updateCounter() {
	  intervalSeconds = setIntervalSynchronous(() => {
	    if (intervalToChange > 0) {
	      setIntervalToChange();
	    } else {
	      // a custom message according to the timetable the user defined
	      let actualTime = new Date().getHours();
	      if (actualTime < morningStart) {
		// I was thinking about "Early start today !" but the problem is that the message only displays after the first session is complete, so I think it's too late to display that
		messageTime = "";
	      } else if (actualTime > eveningEnd) {
		messageTime = "Your day is finished. Great job !";
	      } else if (lunchStart <= actualTime < lunchEnd) {
		messageTime = "Bon appétit ! Consider having a break now.";
	      }

	      // adds a new day on history
	      if (
		history[history.length - 1].date !==
		new Date().toLocaleString("en", {
		  dateStyle: "short"
		})
	      ) {
		history.push({
		  date: new Date().toLocaleString("en", {
		    dateStyle: "short"
		  }),
		  sessions: 0,
		  breaks: 0
		});
	      }
	      // updates today
	      if (session) {
		history[history.length - 1].sessions += 1;
	      } else {
		history[history.length - 1].breaks += 1;
	      }

	      // toggles between session and pause and plays a sound
	      session = !session;
	      pause = !pause;
	      if (running) cuckoo.play();

	      // reset the counter, then updates it by calling updateCounter()
	      resetCounter();
	      /* let targetMinutes;
																																session ? (targetMinutes = sessionMinutes) : (targetMinutes = pauseMinutes);
																																dateNowChange = Date.now() + targetMinutes * 60000;
																																setIntervalToChange();
																																updateCounter(); */
	      //runningFunc();
	    }
	  }, 1000);
	}

	/* function runningFunc() {
																												  let targetMinutes;
																												  session ? (targetMinutes = sessionMinutes) : (targetMinutes = pauseMinutes);
																												  dateNowChange = Date.now() + targetMinutes * 60000;
																												  setIntervalToChange();
																												  updateCounter();
																												} */

	function handleClick() {
	  running = !running;
	  if (!running) {
	    intervalSeconds();
	    console.log("paused : intervalSeconds() has been cleared");
	  } else {
	    //randomizing the messages
	    setRandom(number);
	    // if never launched
	    if (!session && !pause) {
	      firstVisit = false;
	      session = true;
	      //runningFunc() toggles the session/pause and defines dateNowChange, then calls updateCounter() which updates each second
	      resetCounter();
	    } else {
	      // dateNowChange, the timestamp (in ms) that will toggle session/pause, gets updated
	      dateNowChange = Date.now() + intervalToChange;
	    }
	    updateCounter();
	  }
	}
</script>

{#if name !== ""}
{#if !focusMode}
<h1>
		{#if firstVisit || !running}
		Hi {name} ! {messageFirstVisit}
		{:else}
			{#if session}
			{messageSession.quote}
			<span class="sub">{messageSession.source}, {messageSession.author}</span> 
			{:else}
			A break is like breathing : vital
			{/if}
		{/if}
</h1>

{#if minutesDate !== undefined}
<p id='timer-label' >
		{#if secondsDate >= 0 && minutesDate >= 0}
		<span id='time-left' >{secondsDate === 60 ? minutesDate : minutesDate - 1}:{secondsDate < 10 ? "0" + secondsDate : secondsDate === 60 ? "00" : secondsDate}</span> before next {session ? "break" : "session"}<br>
		{/if}
	{messageTime}

</p>
{/if}

<div class="params-grid">
	
	{#if !firstVisit}
	<button class='start-pause' id='start_stop' type='text' on:click={handleClick}>
			{running ? ": pause :" : "> resume >"}
	</button>
	{/if}

	<button class='start-pause' type='text' on:click={() => {
		if (firstVisit) firstVisit = false
		session = true
		pause = false
		running = true
		resetCounter()
		}}>
			{!firstVisit ? "< restart <" : "> start >"}
	</button>

	<button class='start-pause' type='text' on:click={() => {
		if (firstVisit) firstVisit = false
		session = true
		pause = false
		running = true
		toDefault()
		resetCounter()
		}}>
			{"< new life <"} 
	</button>

	{#if running}
	<button class='start-pause' on:click={() => focusMode = !focusMode}>
		~ focus ~
	</button>
	{/if}

</div>

{:else}

{#if running}
<p>
	<label><progress max="{session ? sessionMinutes : pauseMinutes}" value="{intervalToChange/60000}"></progress></label><br>
{#if pause}
<i>On a break</i>
{/if}
<br>
	<button on:click={() => focusMode = !focusMode}>
		x
	</button>
</p>
{/if}

{/if}


{#if !running}

<section id="params">
	<div class="params-grid">
		<span class='left' id='session-label' >My session is</span>
		<span class='time-wrapper'>
			<button class='time' id='session-increment' on:click={() => {
				if(sessionMinutes < 60) sessionMinutes += 1}}>
				+
			</button>
			<button class='time' id='session-decrement' on:click={() => {if (sessionMinutes > 10) {
				sessionMinutes -= 1
				if (sessionMinutes < minutes) minutes = sessionMinutes
				}}}>
				-
			</button>
		</span>
		<span class='right'><strong id='session-length'>{sessionMinutes}</strong> minutes long</span>
</div>
<br>
<div class="params-grid">
		<span class='left' id='break-label' >My break is</span>
		<span class='time-wrapper'>
			<button class='time' id='break-increment' on:click={() => {if(pauseMinutes<60) pauseMinutes += 1}}>
			+
			</button>
			<button class='time' id='break-decrement' on:click={() => {if(pauseMinutes>1) pauseMinutes -= 1}}>
				-
			</button>
		</span>
			<span class='right'><strong id='break-length'>{pauseMinutes}</strong> {pauseMinutes !== 1 ? "minutes" : "minute"} long</span>
	</div>
<br>
<div class="params-grid">
<span class='preset'>
		<button on:click={() =>
			{
			sessionMinutes = 15
			pauseMinutes = 3
			}
			}>
			15/3
		</button>
	</span>
	<span class='preset'>
		<button on:click={() =>
			{
			sessionMinutes = 25
			pauseMinutes = 5
			}
			}
			id='reset'>
			25/5
		</button>
	</span>
	<span class='preset'>
		<button on:click={() =>
			{
			sessionMinutes = 40
			pauseMinutes = 10
			}
			}>
			40/10
		</button>
	</span>
	<span class='preset'>
		<button on:click={() =>
				{
				sessionMinutes = 60
				pauseMinutes = 15
				}
				}>
				60/15
			</button>
	</span>
		
</div>
<div class="params-grid">
	<span class="left">My day starts at</span>
		<span class='time-wrapper'>
			<button class='time' on:click={() => morningStart += 1}>
					+
				</button>
				<button class='time' on:click={() => {if (morningStart > 0) morningStart -= 1}}>
					-
			</button>
		</span>
		
		<span class="right"><strong>{morningStart}</strong> o'clock</span>
</div>
<br>
<div class="params-grid">
<span class='left'>
	My day ends at
</span>
<span class='time-wrapper'>
	<button class='time' on:click={() => {if (eveningEnd < 24) eveningEnd += 1}}>
		+
	</button>
	<button class='time' on:click={() => eveningEnd -= 1}>
		-
	</button>
</span>
<span class='right'><strong>{eveningEnd}</strong> o'clock{eveningEnd < morningStart ? " tomorrow" : ""}</span>
</div>
<br>
<div class="params-grid">
<span class='left'>
	{lunchStart <= 9 && lunchEnd <=10 && lunchEnd-lunchStart <= 3 ? "My breakfast" : "My lunch"} starts at
</span>
<span class='time-wrapper'>
	<button class='time' on:click={() => {
		if (lunchStart >= lunchEnd-1) lunchEnd += 1
		lunchStart += 1}}>
		+
	</button>
	<button class='time' on:click={() => lunchStart -= 1}>
		-
	</button>
</span>
<span class='right'><strong>{lunchStart}</strong> o'clock</span>
</div>
<br>
<div class="params-grid">
<span class='left'>
	{lunchStart <= 9 && lunchEnd <=10 && lunchEnd-lunchStart <= 3 ? "My breakfast" : "My lunch"} ends at
</span>
<span class='time-wrapper'>
	<button class='time' on:click={() => lunchEnd += 1}>
		+
	</button>
	<button class='time' on:click={() => {if (lunchEnd <= lunchStart+1) {
																																		lunchStart -=1
																																		}
																																		lunchEnd -= 1
																																		}}>
		-
	</button>
</span>
<span class='right'><strong>{lunchEnd}</strong> o'clock</span>
</div>

</section>

<p class='paragraph'>
	Control time rather than being controlled by it !<br>
	My name is Eole, from Earth.<br>
	I designed this lovely app as part of my <a href="https://www.freecodecamp.org">FreeCodeCamp</a> learning path. This is my first attempt writing Javascript with a framework. So far, Svelte has been amazing.<br>
</p>
{/if}

	
{:else}
<h1 id='messageFirstVisit-ID'>
	{messageFirstVisit}
</h1>
{/if}

{#if !running}
<p>
	{#if name !== ""}
	What is your name ?
	{/if}
	<input type='text' placeholder="Enter your name" bind:value={name} id="name-input">
</p>
<p>
	{#if name !== ""}
	{#each history as day}
	{day.date} : {day.sessions === 0 ? "no" : day.sessions} session{day.sessions !== 1 ? "s" : ""} ; {day.breaks === 0 ? "no" : day.breaks} break{day.breaks !== 1 ? "s" : ""}
	{/each}
	{/if}
</p>
{/if}


<audio id='beep' bind:this={cuckoo} >
<source src='./cuckoo_1.mp3' />
<source src='./cuckoo_1.wav' />
</audio>
