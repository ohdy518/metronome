<script>
    import {Play, Square, RotateCcw, SlidersVertical} from '@lucide/svelte'
    import {Pane} from 'tweakpane';
    import {browser} from "$app/environment";

    // objects and auto-sets
    let loopEntity;
    let startTime;
    let timer;
    let beat = 1;
    let bar = 1;
    let started = false;

    const PARAMS = {
        beat: 1,
        bar: 1,
        started: false,
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

    // setup
    let bpm = 180
    let beatsPerBar = 4

    function start() {
        if (PARAMS.started) {return}
        PARAMS.started = true;
        init()
        startTime = Date.now()
        timer = Date.now()
        loopEntity = setInterval(() => {loop()}, updateResolution);
    }

    function stop() {
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
    }

    function loop() {
        let timeElapsed = Date.now() - timer
        if (timeElapsed >= (60/PARAMS.bpm*1000)) {
            PARAMS.beat++;

            if (PARAMS.beat > PARAMS.beatsPerBar) { PARAMS.beat -= PARAMS.beatsPerBar; PARAMS.bar++; }
            timer = Date.now()
        }
    }

    function setting() {
        pane.hidden = !(pane.hidden)
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
        <button id="start-button" class="group border-2 border-transparent hover:border-zinc-300 p-1.5" on:click={start}><Play class="group-hover:text-emerald-400"/></button>
        <button id="stop-button" class="group border-2 border-transparent hover:border-zinc-300 p-1.5" on:click={stop}><Square class="group-hover:text-rose-400"/></button>
        <button id="stop-button" class="group border-2 border-transparent hover:border-zinc-300 p-1.5" on:click={reset}><RotateCcw class="group-hover:text-amber-400"/></button>
        <button id="stop-button" class="group border-2 border-transparent hover:border-zinc-300 p-1.5" on:click={setting}><SlidersVertical class="group-hover:text-sky-400"/></button>
    </div>

    <div id="config-container"
         class="absolute top-1/4 right-1/4 jbm">

    </div>

    <div
            class="flex flex-col w-fit
            absolute top-1/2 transform -translate-y-1/2
            gap-3">
        <span id="bar" class="font-regular text-4xl text-zinc-300">Bar <span class="font-medium text-zinc-50">{PARAMS.bar}</span></span>
        <span id="beat" class="font-semibold text-9xl">{PARAMS.beat}<span class="font-regular text-4xl text-zinc-300">&nbsp;/&nbsp;{PARAMS.beatsPerBar}</span></span>
    </div>

    <div
            class="flex flex-col w-fit
            absolute top-3/4">
        <span class="font-regular text-3xl text-zinc-300"><span class="font-medium text-zinc-50">{PARAMS.bpm}</span> BPM &middot; <span class="font-medium text-zinc-50">A&flat;m</span></span>
    </div>

    <div></div>
</div>
</div>
