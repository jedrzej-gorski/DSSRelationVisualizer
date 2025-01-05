<script>
    let {attributeName, animationStartValue = $bindable(), animationEndValue = $bindable(), animationStep = $bindable(), isActive = $bindable(), columnMetadata} = $props();
    function toggleActive() {
        isActive = !isActive;
    }
    function onchangeStart(event) {
        if (animationStartValue == null) {
            animationStartValue = 0;
            return;
        }
        if (event.target.value > columnMetadata[1] || event.target.value < columnMetadata[0]) {
            animationStartValue = columnMetadata[0];
            return;
        }
        if (event.target.value > animationEndValue) {
            animationStartValue = animationEndValue;
            return;
        }
        animationStartValue = event.target.value;
    }
    function onchangeEnd(event) {
        if (animationEndValue == null) {
            animationEndValue = columnMetadata[1];
            return;
        }
        if (event.target.value > columnMetadata[1] || event.target.value < columnMetadata[0]) {
            animationEndValue = columnMetadata[1];
            return;
        }
        if (event.target.value < animationStartValue) {
            animationEndValue = animationStartValue;
            return;
        }
        animationEndValue = event.target.value;
    }
    function onchangeStep(event) {
        if (event.target.value > columnMetadata[1] || event.target.value < 0) {
            animationStep = 0.0025;
            return;
        }
        animationStep = event.target.value;
    }
</script>

<div class="flex flex-row w-full min-h-[50px] border-gray-700 border-2 rounded-lg items-center">
    <button class:inactive={!isActive} class="attribute text-white m-4 w-fit h-100%" onclick={toggleActive}>{
        attributeName}</button>
    <p class="text-white w-fit h-100%">from</p>
    <input onchange={onchangeStart} maxlength="6" type="number" min={columnMetadata[0]} step={0.0025} max={columnMetadata[1]} class="attribute text-white m-4 w-[10%] min-w-[65px] h-full bg-gray-800" bind:value={animationStartValue}>
    <p class="text-white w-fit h-100%">to</p>
    <input onchange={onchangeEnd} maxlength="6" type="number" min={columnMetadata[0]} step={0.0025} max={columnMetadata[1]} class="attribute text-white m-4 w-[10%] min-w-[65px] h-full bg-gray-800" bind:value={animationEndValue}>
    <p class="text-white w-fit h-100%">with step of</p>
    <input onchange={onchangeStep} maxlength="6" type="number" min={columnMetadata[0]} step={0.0025} max={columnMetadata[1]} class="attribute text-white m-4 w-[10%] min-w-[65px] h-full bg-gray-800" bind:value={animationStep}>
</div>

<style>
    .inactive {
        @apply opacity-35;
    }
</style>