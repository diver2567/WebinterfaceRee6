<script lang="ts">
    import { createBubbler } from 'svelte/legacy';

    const bubble = createBubbler();
    import { onMount } from "svelte";
    import DataPopup from "../data_popup/dataPopup.svelte";
    import { jsonIntoModel, type DataType, type ConfigurableDataType } from "../data_popup/popup";
    import { model2JSON } from "../data_popup/popup";
    import LoadingIndicator from "../loadingIndicator.svelte";
    import { get_js, post_js } from "$lib/scripts/constants";


    interface Props {
        icon: string;
        title: string;
        endpoint: string;
        description: string;
        primaryIcon: string;
        isEnabled: (json: any) => boolean;
        model: ConfigurableDataType<any>[];
        render: (json: any) => string;
    }

    let {
        icon,
        title,
        endpoint,
        description,
        primaryIcon,
        isEnabled,
        model,
        render
    }: Props = $props();

    let loading = $state(true);
    let error = $state(false);
    let errorMessage = $state("Something went wrong.")

    let enabled = $state(false);
    let picking = $state(false);
    let config = $state(false);
    let object: any = $state({});

    onMount(() => reload())

    async function reload() {
        object = {};
        loading = true;
        const json = await get_js(endpoint)
        console.log(json)

        if(!json.success) {
            error = true;
            errorMessage = json.message ?? "Something went wrong."
            return;
        }

        object = json.object;
        enabled = isEnabled(object);

        loading = false
    }

</script>

{#if picking}
<DataPopup title="{config ? "Configure" : "Setup"} {title.toLowerCase()}." builder={() => {
    return config ? jsonIntoModel(structuredClone(model), object) : structuredClone(model)
}} close={() => picking = false} 

action1={config ? "Save" : "Create"}
action1Handler={async (data) => {

    picking = false;
    loading = true;
    console.log(model2JSON(data))
    const json = await post_js(endpoint + "/add", model2JSON(data))

    if(!json.success) {
        console.log(json)
        error = true;
        errorMessage = json.message ?? "Something went wrong."
        setTimeout(() => {
            loading = false;
            error = false;
        }, 2000);
        return;
    }

    loading = false;
    reload();
}}

action2="Cancel"
action2Handler={() => picking = false}

/>
{/if}

<div class="box">
    <div class="box-title">
        <div class="content">
            <div class="title">
                <span class="material-icons icon-primary icon-small">{icon}</span>
                <h1 class="text-medium">{title}</h1>
            </div>
            <p class="text-bg">{description}</p>
        </div>

        {#if loading}
        <div class="loading">
            <LoadingIndicator error={error} message={errorMessage} size="45" />
        </div>

        {:else}

        <div class="button-bar">
            {#if !enabled}

            <div onclick={() => {
                picking = true
                config = false
            }} onkeydown={bubble('keydown')} class="button">
                <span class="material-icons icon-small icon-primary">add</span>
                <p class="text-small">Setup</p>
            </div>

            {:else}

            <div onclick={() => {
                picking = true
                config = true
            }} onkeydown={bubble('keydown')} class="button">
                <span class="material-icons icon-small icon-primary">settings</span>
                <p class="text-small">Settings</p>
            </div>

            <div onclick={async () => {
                
                loading = true;
                const json = await post_js(endpoint + "/remove", "{}")

                if(!json.success) {
                    error = true;
                    errorMessage = json.message ?? "Something went wrong."
                    setTimeout(() => {
                        loading = false;
                        error = false;
                    }, 2000);
                    return;
                }

                reload()

            }} onkeydown={bubble('keydown')} class="button">
                <span class="material-icons icon-small icon-primary">close</span>
                <p class="text-small">Disable</p>
            </div>

            {/if}
        </div>

        {/if}
    </div>

    {#if !loading && enabled}
    <div class="content default-margin">
        <div class="model">
            <div class="title">
                <span class="material-icons icon-primary icon-small">{primaryIcon}</span>
                <p class="text-small">{render(object)}</p>
            </div>
        </div>
    </div>
    {/if}
</div>


<style lang="scss">
    @import '$lib/default.scss';
    @import '$lib/styles/box.scss';

    .model {
        display: flex;
        align-items: center;
        justify-content: space-between;
        border-radius: 0.5rem;
        background-color: var(--onyx);
        padding: 0.5rem;

        .title {
            display: flex;
            align-items: center;
            gap: 0.5rem;
        }
    }
</style>