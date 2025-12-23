<script>
    import {Play, Square, RotateCcw, SlidersVertical, Triangle} from '@lucide/svelte'
    import {Pane} from 'tweakpane';
    import {animate, easings, utils, waapi} from 'animejs'
    import {browser} from "$app/environment";

    // objects and auto-sets
    let loopEntity;
    let startTime;
	let timeElapsed = 0
	let globalTimeElapsed = 0
    let timer;
	// let keydownEvent;

    let barIndicator;
    let beatIndicator;
	let currentBeatIndicator;
	let controlIndicator

	let startButton;
	let armButton;
	let controlIndicatorColor = 'bg-zinc-50';

	let currentExtendAnimation;
	let currentShiftAnimation;
	let armAnimation;

    if (browser) {
        barIndicator = document.getElementById('bar-change-indicator');
        beatIndicator = document.getElementById('beat-change-indicator');
		currentBeatIndicator = document.getElementById('current-beat-indicator');
		controlIndicator = document.getElementById('control-change-indicator');

		startButton = document.getElementById('start-button');
		armButton = document.getElementById('arm-button');
    }

    const PARAMS = {
        beat: 1,
        bar: 1,
        started: false,
		armed: false,
        bpm: 180,
        beatsPerBar: 4,
    }

    let pane;

    if (browser) {
        pane = new Pane({
            title: "Configurations",
            container: document.getElementById('config-container'),
        })
        const f1 = pane.addFolder({
            title: 'Status',
        })
        f1.addBinding(PARAMS, 'beat', {min: 1, step: 1})
        f1.addBinding(PARAMS, 'bar', {min: 1, step: 1})
        f1.addBinding(PARAMS, 'started', {readonly: true})
		f1.addBinding(PARAMS, 'armed', {readonly: true})
        const f2 = pane.addFolder({
            title: 'Setup',
        })
        f2.addBinding(PARAMS, 'bpm', {min: 1})
        f2.addBinding(PARAMS, 'beatsPerBar', {min: 1, step: 1})

        // pane.style.fontFamily = "JetBrains Mono"
        pane.hidden = true
    }
    // configs
    let updateResolution = 1

	function disarmAndStart() {
		disarm()
		start()
	}

	function arm() {
		if (PARAMS.started) {return}
		PARAMS.armed = true;
		startButton.disabled = true;
		armButton.disabled = true;

		controlIndicator.classList.remove(controlIndicatorColor)
		controlIndicatorColor = 'bg-amber-400'
		controlIndicator.classList.add(controlIndicatorColor)
		armAnimation = blink(controlIndicator, 450, true)

		document.addEventListener('keydown', disarmAndStart, {once: true});
	}

	function disarm() {
		PARAMS.armed = false;
		startButton.disabled = false;
		armButton.disabled = false;

		document.removeEventListener('keydown', disarmAndStart);

		armAnimation.revert()
		controlIndicator.style.opacity = '0';
	}

    function start() {
        if (PARAMS.started) {return}
        PARAMS.started = true;
        init()
        startTime = Date.now()
        timer = Date.now()

		extend(beatIndicator, 60/PARAMS.bpm*PARAMS.beatsPerBar*1000)
		blink(barIndicator)
		// shift(currentBeatIndicator, 60/PARAMS.bpm*1000/4)

        loopEntity = setInterval(() => {loop()}, updateResolution);
    }

    function stop() {
		disarm()
		currentExtendAnimation.pause()
		currentShiftAnimation.pause()
        pane.refresh()
        PARAMS.started = false;
        clearInterval(loopEntity);
    }

    function reset() {
        stop()
        init()
    }

    function init() {
        PARAMS.beat = 1;
        PARAMS.bar = 1;
		timeElapsed = 0
		globalTimeElapsed = 0
		// beatIndicator.style.scale = "0 1";
		utils.set(beatIndicator, {scale: "0 1"})
		waapi.animate(beatIndicator, {
			scale: "0 1",
			duration: 0,
			ease: "linear",
		});
		utils.set(currentBeatIndicator, {x: 0})
    }

    function loop() {
        timeElapsed = Date.now() - timer
		globalTimeElapsed = Date.now() - startTime
        if (timeElapsed >= (60/PARAMS.bpm*1000)) {
            PARAMS.beat++;

            if (PARAMS.beat > PARAMS.beatsPerBar) {
                PARAMS.beat -= PARAMS.beatsPerBar;
                PARAMS.bar++;
                extend(beatIndicator, 60/PARAMS.bpm*PARAMS.beatsPerBar*1000)
                blink(barIndicator)
				shift(currentBeatIndicator, 60/PARAMS.bpm*1000/4, true)
            } else {
				shift(currentBeatIndicator, 60/PARAMS.bpm*1000/4)
			}
            timer = Date.now()
            pane.refresh()
        }
    }

    function blink(element, dur=-1, loop=false) {
        // console.log(element)
        element.style.opacity = '1';
        return animate(element, {opacity: 0, duration: ((dur!==-1)?dur:250), ease: 'OutQuint', loop: loop})
    }

    function extend(element, dur) {
        element.style.transformOrigin = "left";
        element.style.scale = "0 1";
        currentExtendAnimation = waapi.animate(element, {
            scale: ["0 1", "1 1"],
            duration: dur,
            ease: "linear",
        });
    }

	function shift(element, dur, back = false) {
		console.log(element, dur);
		let amount = 0;
		if (back) {
			amount = 12 - 12/PARAMS.beatsPerBar;
		} else {
			amount = 12 / PARAMS.beatsPerBar;
		}
		currentShiftAnimation = animate(element, {
			x: `${back?'-':'+'}=${amount}rem`,
			duration: 0,
			ease: "inOutQuint",
		})
	}

    function setting() {
        pane.hidden = !(pane.hidden)
    }

	function toMMSS(ms) {
		const totalSeconds = Math.max(0, Math.floor(ms / 1000));
		const minutes = Math.floor(totalSeconds / 60);
		const seconds = totalSeconds % 60;
		return `${String(minutes).padStart(2, '0')}:${String(seconds).padStart(2, '0')}`;
	}
</script>

<div class="w-screen h-screen bg-zinc-950">
<div
        class="w-screen h-[56.25vw] max-h-screen
         absolute top-1/2 left-1/2 transform -translate-x-1/2 -translate-y-1/2
         p-[4vw]
         inter num-style bg-zinc-900 text-zinc-50">
    <div
            class="flex flex-row w-fit
            absolute top-1/4 transform -translate-y-[100%]
            gap-2 text-zinc-50">
		<div id="control-change-indicator" class="bg-zinc-50 w-1.5 opacity-0 mr-2"></div>
        <button id="start-button" class="group border-2 border-transparent hover:border-zinc-300 p-1.5 disabled:text-emerald-400" on:click={start}><Play class="group-hover:text-emerald-400"/></button>
        <button id="stop-button" class="group border-2 border-transparent hover:border-zinc-300 p-1.5" on:click={stop}><Square class="group-hover:text-rose-400"/></button>
        <button id="stop-button" class="group border-2 border-transparent hover:border-zinc-300 p-1.5" on:click={reset}><RotateCcw class="group-hover:text-rose-400"/></button>
        <button id="arm-button" class="group border-2 border-transparent hover:border-zinc-300 p-1.5 disabled:text-amber-400" on:click={arm}><Triangle class="group-hover:text-amber-400"/></button>
		<button id="stop-button" class="group border-2 border-transparent hover:border-zinc-300 p-1.5" on:click={setting}><SlidersVertical class="group-hover:text-sky-400"/></button>
	</div>

    <div id="config-container"
         class="absolute top-1/4 right-1/4 jbm">

    </div>

    <div
            class="flex flex-col w-fit
            absolute top-1/2 transform -translate-y-1/2
            gap-3">
        <div class="flex flex-row gap-4">
            <div id="bar-change-indicator" class="bg-zinc-50 w-1.5 opacity-0"></div>
            <span id="bar" class="font-regular text-4xl text-zinc-300">Bar <span class="font-medium text-zinc-50">{utils.padStart(3, '0')(PARAMS.bar)}</span>
<!--				&middot; {toMMSS(globalTimeElapsed)}-->
			</span>
        </div>
			<div class="flex flex-row gap-4">
            <div id="beat-change-indicator-dep" class="bg-zinc-50 w-1.5 scale-y-0"></div>
            <span id="beat" class="font-semibold text-9xl">{PARAMS.beat}<span class="font-regular text-4xl text-zinc-300">&nbsp;/&nbsp;{PARAMS.beatsPerBar}</span></span>
        </div>
		<div class="pl-4 mt-2 flex flex-row">
			<div id="beat-change-indicator-separator" class="bg-zinc-300 h-3 w-1"></div>
			<div>
				<div id="current-beat-indicator" class="bg-zinc-50 h-1" style="width: {12/PARAMS.beatsPerBar}rem"></div>

				<div id="beat-change-indicator" class="bg-zinc-300 h-2 w-48 scale-x-0"></div>
			</div>
			<div id="beat-change-indicator-separator" class="bg-zinc-300 h-3 w-1"></div>
		</div>
    </div>

	<div class="flex flex-row gap-4 absolute top-3/4">
		<div id="bpm-key-change-indicator" class="bg-zinc-50 w-1.5 opacity-0"></div>
		<div
				class="flex flex-col w-fit">
			<span class="font-regular text-3xl text-zinc-300"><span class="font-medium text-zinc-50">{PARAMS.bpm}</span> BPM &middot; <span class="font-medium text-zinc-50">A&sharp;</span></span>
		</div>
	</div>
    <div></div>
</div>
</div>
