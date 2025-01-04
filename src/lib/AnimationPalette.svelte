<script>
    import * as THREE from "three";
    let {attributes=[], metadata=[], definedFieldCount = 0, fieldX = $bindable(), fieldY = $bindable(), fieldZ = $bindable(), savedDimensions = $bindable(), attributeActivity = $bindable()} = $props()
    let items = $state([]);
    let counter = 0;

    function play() {
        console.log("play");
    }
    function pause() {
        console.log("pause");
    }
    function stop() {
        console.log("stop");
    }

    $effect(() => {
        if (attributes.length != 0) {
            items = attributes.map(attr => ({id: counter++, value: attr}));
        }
        
    });
</script>

<div class="flex flex-col items-center gap-5 w-full min-w-[250px] bg-gray-800 opacity-75 rounded-lg p-3  border-2 border-gray-700">
    {#each items as item(item.id)}
        {#if fieldX != item.id && fieldY != item.id && fieldZ != item.id && definedFieldCount > 1 && attributeActivity[item.id]}
            <AnimationControl {item}></AnimationControl>
        {/if}
    {/each}
</div>

