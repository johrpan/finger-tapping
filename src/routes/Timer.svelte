<script lang="ts">
    interface Props {
        countdownSeconds: number;
        duration: number;
        onFinished: (startTime: number) => void;
    }

    const { countdownSeconds, duration, onFinished }: Props = $props();

    let currentState = $state(0);
    const stateInactive = 0;
    const stateCountdown = 1;
    const stateRunning = 2;
    const stateFinished = 3;

    let progress: HTMLDivElement;

    let currentCountdownNumber = $state(countdownSeconds);
    let startTime: number;

    function nextCountdownNumber() {
        if (currentCountdownNumber > 1) {
            currentCountdownNumber--;
            setTimeout(nextCountdownNumber, 1000);
        } else {
            currentState = stateRunning;
            setTimeout(updateProgress, 100);

            // Record the actual start time stamp. This is the only piece of
            // information that influences which key presses get included in
            // the data output.
            startTime = performance.now();
        }
    }

    function updateProgress() {
        const elapsedTime = performance.now() - startTime;

        if (elapsedTime < duration) {
            progress.style = `--progress: ${elapsedTime / duration}`;
            setTimeout(updateProgress, 100);
        } else {
            currentState = stateFinished;
            progress.style = "--progress: 1";
            onFinished(startTime);
        }
    }

    /**
     * Starts the timer.
     */
    export function start() {
        currentCountdownNumber = countdownSeconds;
        progress.style = "--progress: 0";
        currentState = stateCountdown;
        setTimeout(nextCountdownNumber, 1000);
    }

    /**
     * Hides the timer.
     */
    export function hide() {
        currentState = stateInactive;
    }
</script>

<div class="container {currentState === stateInactive ? 'hidden' : ''}">
    <div
        id="progress"
        class="progress"
        style="--progress: 0"
        bind:this={progress}
    >
        <div
            class="progress-label {currentState !== stateCountdown
                ? 'highlight'
                : ''}"
        >
            {#if currentState === stateCountdown}
                {currentCountdownNumber}
            {:else if currentState === stateRunning}
                GO
            {:else}
                ✓
            {/if}
        </div>
    </div>
</div>

<style>
    .container {
        position: fixed;
        bottom: 0;
        left: 0;
        width: 100%;
        height: 100%;
        overflow: hidden;
        z-index: -1;
    }

    .progress {
        position: absolute;
        top: 50%;
        left: 50%;
        width: 33vh;
        height: 33vh;
        transform: translate(-50%, -50%);
        border-radius: 50%;
        background: conic-gradient(
            goldenrod calc(var(--progress) * 360deg),
            lightgrey 0deg
        );
        display: flex;
        align-items: center;
        justify-content: center;
        z-index: 0;
    }

    .progress::before {
        position: absolute;
        top: 50%;
        left: 50%;
        width: calc(33vh - 32px);
        height: calc(33vh - 32px);
        transform: translate(-50%, -50%);
        content: "";
        display: block;
        background: white;
        border-radius: 50%;
        z-index: 1;
    }

    .progress-label {
        position: relative;
        z-index: 2;
        font-size: 11vh;
        color: grey;
    }

    .progress-label.highlight {
        color: goldenrod;
    }
</style>
