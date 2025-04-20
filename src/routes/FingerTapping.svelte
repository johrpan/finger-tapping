<script lang="ts">
    import KeyboardInput, { type KeyPress } from "./KeyboardInput.svelte";
    import Timer from "./Timer.svelte";

    interface Props {
        leftKeyCode: string;
        rightKeyCode: string;
        duration: number;
        onContinue: () => void;
    }

    let { leftKeyCode, rightKeyCode, duration, onContinue }: Props = $props();

    let timer: Timer;
    let keyboardInputLeft: KeyboardInput;
    let keyboardInputRight: KeyboardInput;
    let keyPressesLeft: KeyPress[];
    let keyPressesRight: KeyPress[];

    function onFinished(startTime: number) {
        keyPressesLeft = keyboardInputLeft
            .getKeyPresses()
            .filter(
                (keyPress) =>
                    keyPress.startTime >= startTime &&
                    keyPress.endTime <= startTime + duration,
            )
            .map((keyPress) => {
                return {
                    startTime: keyPress.startTime - startTime,
                    endTime: keyPress.endTime - startTime,
                };
            });

        keyPressesRight = keyboardInputRight
            .getKeyPresses()
            .filter(
                (keyPress) =>
                    keyPress.startTime >= startTime &&
                    keyPress.endTime <= startTime + duration,
            )
            .map((keyPress) => {
                return {
                    startTime: keyPress.startTime - startTime,
                    endTime: keyPress.endTime - startTime,
                };
            });
        
        setTimeout(() => {timer.hide(); onContinue();}, 1500);
    }

    /**
     * Start the countdown.
     */
    export function start() {
        timer.start();
    }

    /**
     * Returns all key presses for the left hand.
     */
    export function getKeyPressesLeft(): KeyPress[] {
        return keyPressesLeft;
    }

    /**
     * Returns all key presses for the right hand.
     */
    export function getKeyPressesRight(): KeyPress[] {
        return keyPressesRight;
    }
</script>

<Timer countdownSeconds={3} {duration} {onFinished} bind:this={timer} />
<KeyboardInput
    position="left"
    keyCode={leftKeyCode}
    bind:this={keyboardInputLeft}
/>
<KeyboardInput
    position="right"
    keyCode={rightKeyCode}
    bind:this={keyboardInputRight}
/>
