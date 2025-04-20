<script lang="ts">
    import { onMount } from "svelte";
    import FingerTapping from "./FingerTapping.svelte";
    import type { KeyPress } from "./KeyboardInput.svelte";

    const duration = 60_000;

    let currentState = $state(0);
    const stateInstructions = 0;
    const statePrepareLeftHand = 1;
    const stateRecordLeftHand = 2;
    const statePrepareRightHand = 3;
    const stateRecordRightHand = 4;
    const statePrepareBothHands = 5;
    const stateRecordBothHands = 6;
    const stateFinished = 7;

    let participantId = $state("");

    let downloadButton: HTMLAnchorElement;
    let fingerTapping: FingerTapping;

    let dataLeftHand: KeyPress[];
    let dataRightHand: KeyPress[];
    let dataBothHandsLeft: KeyPress[];
    let dataBothHandsRight: KeyPress[];

    function next() {
        if (currentState < stateFinished) {
            if (currentState === stateRecordLeftHand) {
                dataLeftHand = fingerTapping.getKeyPressesLeft();
            } else if (currentState === stateRecordRightHand) {
                dataRightHand = fingerTapping.getKeyPressesRight();
            } else if (currentState === stateRecordBothHands) {
                dataBothHandsLeft = fingerTapping.getKeyPressesLeft();
                dataBothHandsRight = fingerTapping.getKeyPressesRight();
            }

            currentState++;

            if (currentState === stateRecordLeftHand) {
                fingerTapping.start();
            } else if (currentState === stateRecordRightHand) {
                fingerTapping.start();
            } else if (currentState === stateRecordBothHands) {
                fingerTapping.start();
            } else if (currentState === stateFinished) {
                finished();
            }
        }
    }

    function reset() {
        currentState = 0;
    }

    function finished() {
        function toCsv(
            keyPresses: KeyPress[],
            experiment: string,
            hand: string,
        ) {
            return keyPresses
                .map(
                    (keyPress) =>
                        `${experiment},${hand},${keyPress.startTime},${keyPress.endTime}`,
                )
                .join("\n");
        }

        const csvContent = [
            "experiment,hand,start_time,end_time",
            toCsv(dataLeftHand, "left_hand_only", "left_hand"),
            toCsv(dataRightHand, "right_hand_only", "right_hand"),
            toCsv(dataBothHandsLeft, "both_hands", "left_hand"),
            toCsv(dataBothHandsRight, "both_hands", "right_hand"),
        ].join("\n");

        const blob = new Blob([csvContent], {
            type: "text/csv;charset=utf-8;",
        });
        const url = URL.createObjectURL(blob);

        downloadButton.href = url;
    }

    onMount(() =>
        window.addEventListener("keypress", (event) => {
            if (
                event.code == "Space" &&
                ![
                    stateRecordLeftHand,
                    stateRecordRightHand,
                    stateRecordBothHands,
                ].includes(currentState)
            ) {
                next();
            }
        }),
    );
</script>

<div
    class="content center {currentState === stateInstructions ? '' : 'hidden'}"
>
    <h1>Instructions</h1>
    <p>
        This application will record finger taps 1. for the left hand, 2. for
        the right hand and 3. for both hands simultaneously. Each test will take
        {duration / 1000} seconds.
    </p>
    <ul>
        <li>
            Put the third finger of your left hand on the
            <span class="key">Y</span> key.
        </li>
        <li>
            Put the third finger of your right hand on the
            <span class="key">.</span> key.
        </li>
        <li>
            Find a comfortable position for both hands with your wrists resting
            in front of the keyboard.
        </li>
        <li>
            Use the third finger of each hand for tapping while keeping the
            adjacent fingers in contact with the keyboard.
        </li>
        <li>You can practice tapping now.</li>
    </ul>
    <button onclick={next}>Continue <span class="key">Space</span></button>
</div>

<div
    class="content center {currentState === statePrepareLeftHand
        ? ''
        : 'hidden'}"
>
    <h1>Left Hand</h1>
    <p>
        Rest you hands until you feel comfortable to start. After continuing,
        there will be a countdown. When it reaches zero, finger tappings of your
        <em>left hand</em> will be recorded for {duration / 1000} seconds. Try to
        tap as quickly and as consistently as possible.
    </p>
    <button onclick={next}>Continue <span class="key">Space</span></button>
</div>

<div
    class="content center {currentState === statePrepareRightHand
        ? ''
        : 'hidden'}"
>
    <h1>Right Hand</h1>
    <p>
        Rest you hands until you feel comfortable to continue. After continuing,
        there will be a countdown. When it reaches zero, finger tappings of your
        <em>right hand</em> will be recorded for {duration / 1000} seconds. Try to
        tap as quickly and as consistently as possible.
    </p>
    <button onclick={next}>Continue <span class="key">Space</span></button>
</div>

<div
    class="content center {currentState === statePrepareBothHands
        ? ''
        : 'hidden'}"
>
    <h1>Both Hands</h1>
    <p>
        Rest your hands until you feel comfortable to continue. After
        continuing, there will be a countdown. When it reaches zero, finger
        tappings of <em>both hands</em> will be recorded for {duration / 1000} seconds.
        Try to tap as quickly and as consistently as possible.
    </p>
    <button onclick={next}>Continue <span class="key">Space</span></button>
</div>

<div class="content center {currentState === stateFinished ? '' : 'hidden'}">
    <h1>Finished</h1>
    <p>
        The tests are finished. Thank you very much for participating! Use the
        button below to download the results.
    </p>
    <p>
        Participant ID: <input bind:value={participantId} />
    </p>
    <div style="display: flex; gap: 0.5rem;">
        <!-- svelte-ignore a11y_missing_attribute -->
        <a
            class="button"
            download="key_presses_{participantId}.csv"
            bind:this={downloadButton}>Download CSV</a
        >
        <button onclick={reset}>Restart</button>
    </div>
</div>

<FingerTapping
    leftKeyCode="KeyZ"
    rightKeyCode="Period"
    {duration}
    onContinue={next}
    bind:this={fingerTapping}
/>
