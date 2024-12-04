<script>
    import {flip} from 'svelte/animate';
    import AxisField from './AxisField.svelte';
    import { dndzone } from 'svelte-dnd-action';
    let {attributes = []} = $props();
    let counter = 0;
    let items = $state([]);
    let axesDict = {};
    let fieldX = $state(null);
    let fieldY = $state(null);
    let fieldZ = $state(null);
    $inspect(attributes);

    function handleDndConsider(e) {
        items = e.detail.items;
    }
    function handleDndFinalize(e) {
        items = e.detail.items;
    }

    $effect(() => {
        items = attributes.map(attr => ({id: counter++, value: attr}));
    });

</script>


<div class="absolute flex flex-col items-center gap-5 w-[25%] top-[10%] left-[2.5%] h-[70%] min-w-[250px] bg-gray-800 opacity-75 rounded-lg p-3  border-2 border-gray-700">
    <div class="w-full h-[30%] min-h-[200px] border-2 border-gray-700 relative">
        <AxisField bind:field="{fieldX}" positionClass="bottom-0 left-0" />
        <AxisField bind:field="{fieldY}" positionClass="top-0 left-[40%]" />
        <AxisField bind:field="{fieldZ}" positionClass="bottom-0 right-0" />
    </div>
    <div use:dndzone="{{items}}" onconsider="{handleDndConsider}" onfinalize="{handleDndFinalize}" id="attribute-list" class="w-full h-[65%] flex flex-col items-start">
        {#each items as item(item.id)}
        <p class="attribute text-white m-4 w-fit" animate:flip={{duration: 300}}>{item.value}</p>
    {/each}
    </div>
</div>

<style>
    .attribute {
        font-family: "Raleway", sans-serif;
    }
</style>