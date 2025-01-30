<script>
    import { setContext } from 'svelte';
    import RangeSlider from 'svelte-range-slider-pips';
    let {item, columnMetadata=[], definedFieldCount = 0, isAnimating, value = $bindable(), fieldX = $bindable(), fieldY = $bindable(), fieldZ = $bindable(), isActive = $bindable()} = $props()
    let parsedValue = $derived.by(() => {
        if (value == null) {
            return null;
        }
        return value.toFixed(6);
    })

    function setXAxis() {
        if (!isActive) {
            return;
        }
        if (fieldX == item.id) {
            fieldX = null;
        } else {
            fieldX = item.id;
        }
    }
    function setYAxis() {
        if (!isActive) {
            return;
        }
        if (fieldY == item.id) {
            fieldY = null;
        } else {
            fieldY = item.id;
        }
    }
    function setZAxis() {
        if (!isActive) {
            return;
        }
        if (fieldZ == item.id) {
            fieldZ = null;
        } else {
            fieldZ = item.id;
        }
    }
    function toggleActive() {
        isActive = !isActive;
        if (fieldX == item.id) {
            fieldX = null;
        }
        if (fieldY == item.id) {
            fieldY = null;
        }
        if (fieldZ == item.id) {
            fieldZ = null;
        }
    }
    function onchange(event) {
        if (event.target.value == '') {
            event.target.value = 0;
        }
        if (event.target.value < columnMetadata[0]) {
            event.target.value = columnMetadata[0];
        }
        else if (event.target.value > columnMetadata[1]) {
            event.target.value = columnMetadata[1];
        }
        else {
            value = parseFloat(event.target.value) || 0;
        }
    }
    $inspect(parsedValue)
</script>

<div class="flex flex-row w-full min-h-[50px] border-gray-700 border-2 rounded-lg">
    <button class:inactive={!isActive} disabled={isAnimating || null} class="attribute text-white m-4 w-fit h-100%" onclick={toggleActive}>{item.value}</button>
    <div class="flex flex-col w-full">
        <div class="flex flex-row w-full">
            <button class:inactive={!isActive} class:activeX={fieldX == item.id} class="bg-gray-800 rounded-lg block w-6 h-6 border-2 border-red-800" onclick={setXAxis}>
                <p class="text-red-800">X</p>
            </button>
            <button class:inactive={!isActive} class:activeY={fieldY == item.id} class="bg-gray-800 rounded-lg block w-6 h-6 border-2 border-green-800" onclick={setYAxis}>
                <p class="text-green-800">Y</p>
            </button>
            <button class:inactive={!isActive} class:activeZ={fieldZ == item.id} class="bg-gray-800 rounded-lg block w-6 h-6 border-2 border-blue-800" onclick={setZAxis}>
                <p class="text-blue-800">Z</p>
            </button>
        </div>
        {#if value != null && fieldX != item.id && fieldY != item.id && fieldZ != item.id && definedFieldCount > 1 && isActive}
        <div class="flex flex-row w-full h-[24px]">
            <RangeSlider disabled={isAnimating} bind:value step={0.0025} min={columnMetadata[0]} max={columnMetadata[1]} springValues={{damping: 1, stiffness: 1}} first={'label'} last={'label'}></RangeSlider>
            <input {onchange} disabled={isAnimating || null} maxlength="6" type="number" value={parsedValue} class="attribute text-white m-4 w-[10%] min-w-[65px] h-full relative top-[-15px] bg-gray-800">
        </div>
    {/if}
    </div>
</div>

<style>
    .inactive {
        @apply opacity-35;
    }
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