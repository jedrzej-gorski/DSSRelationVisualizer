<script>
    import * as THREE from "three";
    import AnimationControl from "./AnimationControl.svelte";

    let {
        attributes = [],
        metadata = [],
        definedFieldCount = 0,
        fieldX = $bindable(),
        fieldY = $bindable(),
        fieldZ = $bindable(),
        attributeActivity = $bindable(),
    } = $props();
    let animationDimensions = $derived.by(() => {
        let dimensions = [];
        if (definedFieldCount > 1) {
            attributes.forEach((attribute, index) => {
                if (
                    index != fieldX &&
                    index != fieldY &&
                    index != fieldZ &&
                    attributeActivity[index]
                ) {
                    dimensions.push({
                        id: index,
                        name: attributes[index],
                        animationStartValue: metadata[index][0],
                        animationEndValue: metadata[index][1],
                        animationStep: 0.0025,
                        isActive: true,
                    });
                }
            });
        }
        return dimensions;
    });
    let animationItems = $state([]);

    $effect(() => {
        animationItems = animationDimensions.map((dimension) => ({ 
            id: counter++, 
            value: dimension 
        }));
    });

    let counter = 0;
    let animationPeriod = $state(10);

    function play() {
        console.log("play");
    }
    function pause() {
        console.log("pause");
    }
    function stop() {
        console.log("stop");
    }

    $inspect(animationDimensions);
</script>

<div
    class="flex flex-col items-center gap-5 w-full min-w-[250px] bg-gray-800 opacity-75 rounded-lg p-3 border-2 border-gray-700"
>
    <div class="flex flex-row gap-2 items-center">
        <p class="text-white w-fit h-100%">Increment values every</p>
        <input
            maxlength="6"
            type="number"
            min={10}
            step={5}
            max={10000}
            class="attribute text-white m-4 w-[10%] min-w-[65px] h-full bg-gray-800"
            bind:value={animationPeriod}
        />
        <p class="text-white w-fit h-100%">ms</p>
    </div>
    {#each animationItems as item(item.id)}
        <AnimationControl 
            attributeName={item.value.name}
            bind:animationStartValue={item.value.animationStartValue} 
            bind:animationEndValue={item.value.animationEndValue} 
            bind:animationStep={item.value.animationStep} 
            bind:isActive={item.value.isActive} 
            columnMetadata={metadata[item.value.id]}
        />
    {/each}
    <div class="flex flex-row gap-2">
        <button class="bg-gray-700 text-white rounded-lg p-2" onclick={play}
            >Play</button
        >
        <button class="bg-gray-700 text-white rounded-lg p-2" onclick={pause}
            >Pause</button
        >
        <button class="bg-gray-700 text-white rounded-lg p-2" onclick={stop}
            >Stop</button
        >
    </div>
</div>
