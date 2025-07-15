<script>
    export let colorScale;
    export let hoveredContinent;

</script>

<div class='legend' on:mouseleave={() => (hoveredContinent = null)}>
    {#each colorScale.domain() as continent}
    <!-- svelte-ignore a11y-mouse-events-have-key-events -->
    <p
    on:mouseover={() => (hoveredContinent = continent)}
    class:unhovered={hoveredContinent && hoveredContinent !== continent}
    >
        <span style="background-color: {colorScale(continent)};"></span>
        {continent}
    </p>
    {/each}
</div>

<style>
    .legend {
        display: flex;
        flex-direction: row;
        flex-wrap: wrap;
        margin-bottom: 0.5rem;
    }

    .legend p {
        display: flex;
        align-items: center;
        margin: 0.25rem 0.75rem 0.25rem 0;
        transition: opacity 300ms ease;
        cursor: pointer;
    }

    .legend span {
        width: 12px;
        height: 12px;
        border-radius: 3px;
        margin-right: 0.5rem;
        border: 1px solid rgba(0, 0, 0, 0.25);
    }

    .unhovered {
        opacity: 0.3;
    }
</style>