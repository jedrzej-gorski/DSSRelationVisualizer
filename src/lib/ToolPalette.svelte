<script>
    import {flip} from 'svelte/animate';
    import AxisField from './AxisField.svelte';
    import AttributeControl from './AttributeControl.svelte';
    import { dndzone } from 'svelte-dnd-action';
    let {attributes = [], metadata = [], fieldX = $bindable(), fieldY = $bindable(), fieldZ = $bindable(), constantDimensions = $bindable()} = $props();
    let counter = 0;
    let items = $state([]);

    function handleDndConsider(e) {
        items = e.detail.items;
    }
    function handleDndFinalize(e) {
        items = e.detail.items;
        if (fieldX != null && fieldY != null && fieldZ != null) {
            constantDimensions = {};
            e.detail.items.forEach((item) => {
                constantDimensions[item.id] = metadata[item.id];
            })
        }
        else {
            constantDimensions = null;
        }
    }

    $effect(() => {
        if (attributes.length != 0) {
            items = attributes.map(attr => ({id: counter++, value: attr}));
        }
        
    });

</script>


<div class="absolute flex flex-col items-center gap-5 w-[25%] top-[10%] left-[2.5%] h-[70%] min-w-[250px] bg-gray-800 opacity-75 rounded-lg p-3  border-2 border-gray-700">
    <div class="w-full h-[30%] min-h-[200px] border-2 border-gray-700 relative">
        <AxisField bind:field={fieldX} positionClass="bottom-0 left-0" />
        <AxisField bind:field={fieldY} positionClass="top-0 left-[40%]" />
        <AxisField bind:field={fieldZ} positionClass="bottom-0 right-0" />
    </div>
    <div use:dndzone="{{items}}" onconsider="{handleDndConsider}" onfinalize="{handleDndFinalize}" id="attribute-list" class="w-full h-[65%] flex flex-col items-start">
        {#each items as item(item.id)}
            <div animate:flip={{duration: 300}}>
                {#if constantDimensions != null}
                    <AttributeControl {item} columnMetadata={metadata[item.id]} bind:value={constantDimensions[item.id]}></AttributeControl>
                {:else}
                    <AttributeControl {item} columnMetadata={metadata[item.id]}></AttributeControl>
                {/if}
            </div>
        {/each}
    </div>
</div>

<style>
    .attribute {
        font-family: "Raleway", sans-serif;
    }
</style>