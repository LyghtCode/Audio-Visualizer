<script lang="ts">
    import BottomBar from './BottomBar.svelte';
    import CanvasSpectrum from './CanvasSpectrum.svelte';

    let volumes: Uint8Array = new Uint8Array();
    let data: Uint8Array = new Uint8Array();

    let animation: number;

    let audioElement: HTMLAudioElement;

    let currentTime = 0;
    let duration = 0;
    let paused = true;
    let analyser: AnalyserNode;
    let render: () => void;

    $: volumes = data.filter((vol) => vol > 0);

    function startAnalyzer() {
        let audioCtx = new AudioContext();

        analyser = audioCtx.createAnalyser();

        let source = audioCtx.createMediaElementSource(audioElement);

        source.connect(analyser);

        source.connect(audioCtx.destination);

        data = new Uint8Array(analyser.frequencyBinCount);

        analyser.fftSize = 4096;

        analyser.getByteFrequencyData(data);

        if (data.length) data = new Uint8Array(analyser.frequencyBinCount);
    }

    function handleChange() {
        let file = new Audio('https://download.pariyatti.org/free/_moIbLs95/along_the_path_audio/streaming/Great_Compassion.mp3');

        if (!file) return;

        if (!analyser) startAnalyzer();

        audioElement.src = file.src;

        paused = true;

        if (animation) {
            cancelAnimationFrame(animation);
            animation = undefined;
        }

        function check() {
            if (audioElement.paused) {
                animation = requestAnimationFrame(check);
                return;
            }

            render();

            analyser.getByteFrequencyData(data);

            data = data;

            animation = requestAnimationFrame(check);
        }

        check();
    }
    handleChange();
</script>

<main>
    <!-- <input type="url" accept=".mp3" on:input={handleChange} /> -->
    <br />
    <!-- svelte-ignore a11y-media-has-caption -->
    <audio bind:this={audioElement} bind:currentTime bind:duration bind:paused />
    <CanvasSpectrum {volumes} bind:render />
    <BottomBar bind:currentTime bind:paused {duration} />
</main>
