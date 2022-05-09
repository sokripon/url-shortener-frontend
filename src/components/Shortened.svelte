<script lang="ts">
    import { copy } from 'svelte-copy';

    import { slide } from "svelte/transition";

    export let expanded = false;

    function toggleExpand() {
        expanded = !expanded;
    }

    export let id: number;
    export let originalUrl: string;
    export let extraData: { [key: string]: string };
    export let baseUrl: string = window.location.href;


</script>

<svelte:head>

    <link crossorigin="anonymous" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.1.1/css/all.min.css"
          integrity="sha512-KfkfwYDsLkIlwQp6LFnl8zNdLGxu9YAA1QvwINks4PhcElQSvqcyVLLD9aMhXd13uQjoXtEKNosOWaZqXgel0g=="
          referrerpolicy="no-referrer" rel="stylesheet"/>
</svelte:head>

<div class="flexbox flexbox-entry">
    <div class="flexbox flexbox-icon">
        <i class="fa-regular fa-copy pointer" use:copy={originalUrl}></i>
        <span class="span-long pointer" use:copy={originalUrl}>{originalUrl}</span>
        <i class="fa-solid fa-angle-{expanded ? 'up' : 'down'} pointer expand-icon" on:click={toggleExpand}></i>
    </div>
    <div class="flexbox flexbox-details">
        <div class="flexbox flexbox-icon">
            <i class="fa-regular fa-copy pointer" use:copy={baseUrl + id}></i>
            <span class="span-url pointer" title={baseUrl + id} use:copy={baseUrl + id}>{baseUrl + id}</span>
        </div>
        {#if expanded}
            <div class="flexbox flexbox-more" transition:slide={{ duration: 300}}>
                {#each Object.values(extraData) as value}
                    <div class="flexbox flexbox-icon">
                        <i class="fa-regular fa-copy pointer" use:copy={baseUrl + value}></i>
                        <span class="span-url pointer" title={baseUrl + value}
                              use:copy={baseUrl + value}>{baseUrl + value}</span>
                    </div>
                {/each}
            </div>
        {/if}
    </div>
</div>
<style>
    .flexbox {
        display: flex;
    }

    .flexbox-entry {
        flex-direction: column;
        align-items: center;
        padding: 10px;
        border-bottom: 1px solid #ccc;
    }

    .flexbox-details {
        justify-content: center;
        align-items: center;
        flex-direction: column;
    }

    .flexbox-more {
        flex-direction: column;
        justify-content: center;
        align-items: center;
    }

    .fa-copy {
        text-align: left;
        padding-right: 10px;
        padding-left: 5px;
    }


    .flexbox-icon {
        flex-direction: row;
        justify-content: center;
        align-items: center;
        width: 100%;
        margin: 1px;
    }

    .span-long {
        white-space: nowrap;
        overflow: hidden;
        text-overflow: ellipsis;
        max-width: 50%;
    }

    .pointer {
        cursor: pointer;
    }

    .expand-icon {
        text-align: right;
        padding-left: 20px;
        padding-right: 5px;
    }
</style>