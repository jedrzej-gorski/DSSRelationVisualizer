<script>
    let {item} = $props();
    let isActive = $state(false);
    function toggleActive() {
        isActive = !isActive;
    }
</script>

<div class="flex flex-row w-full min-h-[50px] border-gray-700 border-2 rounded-lg">
    <button class:inactive={!isActive} class="attribute text-white m-4 w-fit h-100%" onclick={toggleActive}>{item.value}</button>
</div>
