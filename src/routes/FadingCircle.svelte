<script lang="ts">
    interface Props {
        position: string;
        onRemove: () => void;
    }

    let { position, onRemove }: Props = $props();
</script>

<div class="container">
    <div
        class="circle {position == 'left' ? 'left' : 'right'}"
        onanimationend={onRemove}
    ></div>
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

    .circle {
        position: absolute;
        bottom: 0;
        width: 50vw;
        height: 50vw;
        border-radius: 50%;
        background-color: goldenrod;
    }

    .circle.left {
        left: 0;
        transform: translate(-50%, 50%) scale(0);
        animation: growFadeLeft 1s ease-out;
    }

    .circle.right {
        right: 0;
        transform: translate(50%, 50%) scale(0);
        animation: growFadeRight 1s ease-out;
    }

    @keyframes growFadeLeft {
        to {
            transform: translate(-50%, 50%) scale(1);
            opacity: 0;
        }
    }

    @keyframes growFadeRight {
        to {
            transform: translate(50%, 50%) scale(1);
            opacity: 0;
        }
    }
</style>
