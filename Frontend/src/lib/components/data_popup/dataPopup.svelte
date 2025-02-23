<script lang="ts">
    import { createBubbler } from 'svelte/legacy';

    const bubble = createBubbler();
    import { fade, scale } from "svelte/transition";
    import type { DataType } from "./popup";
    import { onMount } from "svelte";
    import ChannelValue from "./channelValue.svelte";
    import RoleValue from "./roleValue.svelte";
    import StringValue from "./stringValue.svelte";
    import IntValue from "./intValue.svelte";
    import SelectorValue from "./selectorValue.svelte";

    
    let content: DataType<any>[] = $state([]);


    interface Props {
        title: string;
        zIndex?: number;
        builder: () => DataType<any>[];
        action1?: string | undefined;
        action1Handler?: ((data: DataType<any>[]) => void) | undefined;
        action2?: string | undefined;
        action2Handler?: ((data: DataType<any>[]) => void) | undefined;
        close: () => void;
    }

    let {
        title,
        zIndex = 50,
        builder,
        action1 = undefined,
        action1Handler = undefined,
        action2 = undefined,
        action2Handler = undefined,
        close
    }: Props = $props();

    onMount(() => {
        content = builder();
    })

</script>

<div out:fade={{duration: 250}} in:fade={{duration: 250}} class="dialog-outer" style="z-index: {zIndex};">
    <div out:scale={{start: 0.8, duration: 250}} in:scale={{start: 0.8, duration: 250}} class="dialog">

        <div class="header">
            <h2>{title}</h2>
            <span onclick={close} onkeydown={bubble('keydown')} class="material-icons icon-medium clickable hover-primary">close</span>
        </div>

        <div class="content">

            {#each content as item}
            {#if item.visible}
            <div class="item">
                <p class="text-medium">{item.name}</p>

                {#if item.type == "string"}
                <StringValue current={item.value} callback={(string) => item.value = string} unit={item.unit} />
                {:else if item.type == "selector"}
                <SelectorValue current={item.value} strings={item.unit.split(",")} callback={(string) => item.value = string} />
                {:else if item.type == "int"}
                <IntValue current={item.value} callback={(int) => item.value = int} unit={item.unit} />
                {:else if item.type == "channel"}
                <ChannelValue current={item.value} callback={(channel) => item.value = channel} />
                {:else if item.type == "role"}
                <RoleValue current={item.value} callback={(role) => item.value = role}/>
                {/if}

            </div>
            {/if}
            {/each}

            <div class="buttons default-margin">
                {#if action1 != undefined}
                <button class="text-medium" onclick={() => (action1Handler ?? (() => {}))(content)} onkeydown={bubble('keydown')}>{action1}</button>
                {/if}
                {#if action2 != undefined}
                <button class="text-medium" onclick={() => (action2Handler ?? (() => {}))(content)} onkeydown={bubble('keydown')}>{action2}</button>
                {/if}
            </div>
        </div>
    </div>
</div>

<style lang="scss">
    @import '$lib/default.scss';
    @import '$lib/styles/dialog.scss';

    .item {
        display: flex;
        align-items: center;
        justify-content: space-between;
        padding: 0.2rem 0.5rem;
    }

    .buttons {
        display: flex;
        align-items: center;
        justify-content: space-between;
        gap: 0.5rem;

        button {
            display: flex;
            align-items: center;
            gap: var(--button-gap);
            padding: var(--button-padding);
            color: var(--text-color);
            background-color: var(--onyx);
            border-radius: 1rem;
            border: none;
            transition: 250ms all ease;
            cursor: pointer;

            &:hover {
                background-color: var(--outer-space);
            }
        }
    }

</style>
