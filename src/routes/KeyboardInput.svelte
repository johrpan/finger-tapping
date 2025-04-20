<script lang="ts">
    interface Props {
        position: string;
        keyCode: string;
    }

    let { position, keyCode }: Props = $props();

    import { onMount } from "svelte";
    import FadingCircle from "./FadingCircle.svelte";

    export interface KeyPress {
        startTime: number;
        endTime: number;
    }

    let keyPresses = $state<KeyPress[]>([]);
    let animations = $state<number[]>([]);
    let currentStartTime = 0;

    function handleKeyDown(event: KeyboardEvent) {
        if (!event.repeat && event.code === keyCode) {
            currentStartTime = event.timeStamp;
            animations.push(event.timeStamp);
        }
    }

    function handleKeyUp(event: KeyboardEvent) {
        if (event.code === keyCode) {
            keyPresses.push({
                startTime: currentStartTime,
                endTime: event.timeStamp,
            });
        }
    }

    function removeAnimation(id: number) {
        animations = animations.filter((animation) => animation != id);
    }

    /**
     * Returns all key presses recorded during the lifetime of the component.
     */
    export function getKeyPresses(): KeyPress[] {
        return keyPresses
    }

    onMount(() => {
        window.addEventListener("keydown", handleKeyDown);
        window.addEventListener("keyup", handleKeyUp);
    });
</script>

{#each animations as animation (animation)}
    <FadingCircle {position} onRemove={() => removeAnimation(animation)} />
{/each}
