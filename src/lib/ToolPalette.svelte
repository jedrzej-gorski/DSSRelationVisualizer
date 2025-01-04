<script>
    import {flip} from 'svelte/animate';
    import AxisField from './AxisField.svelte';
    import AttributeControl from './AttributeControl.svelte';
    let {attributes = [], metadata = [], definedFieldCount = 0, fieldX = $bindable(), fieldY = $bindable(), fieldZ = $bindable(), savedDimensions = $bindable(), attributeActivity = $bindable()} = $props();
    let counter = 0;
    let items = $state([]);

    $effect(() => {
        if (attributes.length != 0) {
            items = attributes.map(attr => ({id: counter++, value: attr}));
        }
        
    });
</script>


<div class="flex flex-col items-center gap-5 min-w-[250px] bg-gray-800 opacity-75 rounded-lg p-3  border-2 border-gray-700">
        {#each items as item(item.id)}
            <AttributeControl {item} {definedFieldCount} bind:fieldX bind:fieldY bind:fieldZ columnMetadata={metadata[item.id]} bind:value={savedDimensions[item.id]} bind:isActive={attributeActivity[item.id]}></AttributeControl>
        {/each}
</div>

<style>
    .attribute {
        font-family: "Raleway", sans-serif;
    }
</style>