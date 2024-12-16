<script>
    import { dndzone } from 'svelte-dnd-action';
    let {field = $bindable(), positionClass} = $props();
    let items = $state([]);
    let dropFromOthersDisabled = $state(false);

    function handleDndConsider(e) {
        items = e.detail.items;
    }
    function handleDndFinalize(e) {
        items = e.detail.items;
        if (items.length == 1) {
            dropFromOthersDisabled = true;
            field = e.detail.items[0].id;
        }
        else {
            dropFromOthersDisabled = false;
            field = null;
        }
    }
</script>

<div use:dndzone="{{items, dropFromOthersDisabled}}" onconsider="{handleDndConsider}" onfinalize="{handleDndFinalize}" class="absolute bg-gray-800 rounded-lg block w-16 h-16 border-2 border-gray-200 {positionClass}">
    {#each items as item(item.id)}
        <p class="attribute text-white m-4 block" animate:flip={{duration: 300}}>{item.value}</p>
    {/each}
</div>