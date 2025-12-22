<script>

    // objects and auto-sets
    let loopEntity;
    let startTime;
    let timer;
    let beat = 1;
    let bar = 1;

    // configs
    let updateResolution = 1

    // setup
    let bpm = 180
    let beatsPerBar = 4


    function start() {
        init()
        loopEntity = setInterval(() => {loop()}, updateResolution);
    }

    function stop() {
        clearInterval(loopEntity);
    }

    function init() {
        startTime = Date.now()
        timer = Date.now()
    }

    function loop() {
        let timeElapsed = Date.now() - timer
        if (timeElapsed >= (60/bpm*1000)) {
            beat++;

            if (beat > beatsPerBar) { beat -= beatsPerBar; bar++; }
            timer = Date.now()
        }
    }
</script>

<div class="w-screen h-screen bg-zinc-950">
<div
        class="w-screen h-[56.25vw] max-h-screen
         absolute top-1/2 left-1/2 transform -translate-x-1/2 -translate-y-1/2
         p-[4vw]
         inter num-style bg-zinc-900 text-zinc-50">
    <button id="play-control" on:click={start}>Start</button>
    <button id="play-control" on:click={stop}>Stop</button>
    <div
            class="flex flex-col w-fit
            absolute top-1/2 transform -translate-y-1/2
            gap-3">
        <span id="bar" class="font-regular text-4xl text-zinc-200">Bar <span class="font-medium text-zinc-50">{bar}</span></span>
        <span id="beat" class="font-semibold text-9xl">{beat}<span class="font-regular text-4xl text-zinc-200">&nbsp;/&nbsp;4</span></span>
    </div>

    <div
            class="flex flex-col">
        <span>This</span>
    </div>

    <div></div>
</div>
</div>
