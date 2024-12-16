<script>
    import RangeSlider from 'svelte-range-slider-pips';
    let {item, columnMetadata=[], value = $bindable(null)} = $props()
</script>

<div class="flex flex-row">
    <p class="attribute text-white m-4 w-fit">{item.value}</p>
    {#if value != null}
        <RangeSlider bind:value min={columnMetadata[0]} max={columnMetadata[1]}></RangeSlider>
    {/if}
</div>
