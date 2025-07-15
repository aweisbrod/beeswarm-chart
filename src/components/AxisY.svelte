<script>
    export let yScale;
    export let groupByContinent;

    const ticks = yScale.domain();

    import { fade } from "svelte/transition";
    
</script>

<g class='axis y'>
    {#each ticks as tick, index}
    {#if groupByContinent}
        <g class="tick" transition:fade={{duration: 200}}>
            <rect y={yScale(tick) - 12} x={0} width={tick.toString().length * 7 + 2} height="16" fill="white" fill-opacity="0.75"/>
            <text y={yScale(tick)} x={0}>{tick}</text>
        </g>
    {/if}
    {/each}
</g>