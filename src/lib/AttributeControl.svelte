<script>
    import { setContext } from 'svelte';
    import RangeSlider from 'svelte-range-slider-pips';
    let {item, columnMetadata=[], value = $bindable(), fieldX = $bindable(), fieldY = $bindable(), fieldZ = $bindable()} = $props()

    function setXAxis() {
        if (fieldX == item.id) {
            fieldX = null;
        } else {
            fieldX = item.id;
        }
    }
    function setYAxis() {
        if (fieldY == item.id) {
            fieldY = null;
        } else {
            fieldY = item.id;
        }
    }
    function setZAxis() {
        if (fieldZ == item.id) {
            fieldZ = null;
        } else {
            fieldZ = item.id;
        }
    }

</script>

<div class="flex flex-row w-full min-h-[50px] border-gray-700 border-2 rounded-lg">
    <p class="attribute text-white m-4 w-fit h-100%">{item.value}</p>
    <div class="flex flex-col w-full">
        <div class="flex flex-row w-full">
            <button class:activeX={fieldX == item.id} class="bg-gray-800 rounded-lg block w-6 h-6 border-2 border-red-800" onclick={setXAxis}>
                <p class="text-red-800">X</p>
            </button>
            <button class:activeY={fieldY == item.id} class="bg-gray-800 rounded-lg block w-6 h-6 border-2 border-green-800" onclick={setYAxis}>
                <p class="text-green-800">Y</p>
            </button>
            <button class:activeZ={fieldZ == item.id} class="bg-gray-800 rounded-lg block w-6 h-6 border-2 border-blue-800" onclick={setZAxis}>
                <p class="text-blue-800">Z</p>
            </button>
        </div>
        {#if value != null && fieldX != item.id && fieldY != item.id && fieldZ != item.id && fieldX != null && fieldY != null && fieldZ != null}
        <div class="flex flex-row w-full h-[24px]">
            <RangeSlider bind:value step={0.0025} min={columnMetadata[0]} max={columnMetadata[1]} springValues={{damping: 1, stiffness: 1}} first={'label'} last={'label'}></RangeSlider>
            <p class="attribute text-white m-4 w-[5%] relative top-[-15px]">{value}</p>
        </div>
    {/if}
    </div>
</div>

<style>
    .activeX {
        @apply bg-red-500 ;
    }
    .activeY {
        @apply bg-green-500;
    }
    .activeZ {
        @apply bg-blue-500;
    }
</style>