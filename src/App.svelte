<script lang="ts">
    import { fade } from "svelte/transition";
    import { writable } from "svelte/store";
    import Shortened from "./components/Shortened.svelte";

    type Entry = {
        url: string;
        pseudo_id: number;
        extra: { [key: string]: string };
    }

    const storedShortened = localStorage.getItem("shortened");
    let parsed: Array<Entry> = JSON.parse(storedShortened);
    export const shortened = writable(parsed === null ? [] : parsed);

    shortened.subscribe((value) => {
        localStorage.setItem("shortened", JSON.stringify(value));
        parsed = JSON.parse(localStorage.getItem("shortened"));
    });


    function addShortened(data) {
        parsed = [data, ...parsed];
        localStorage.setItem("shortened", JSON.stringify(parsed));


    }


    let url = "";
    let lastUrl = "";
    let btnDisabled = false;
    let infoText = "Enter a URL to shorten";
    let infoStatus = "info";

    function handleClick() {
        if (url == "") {
            infoText = "Please enter a URL";
            infoStatus = "error";
            return;
        }
        if (url == lastUrl) {
            infoText = "You have to enter a new URL";
            infoStatus = "error";
            return;
        }
        try {
            new URL(url);
        } catch (e) {
            infoText = "Please enter a valid URL";
            infoStatus = "error";
            return;
        }

        lastUrl = url;
        btnDisabled = true;
        infoText = "Loading...";
        infoStatus = "info";
        const requestOptions = {
            method: "PUT",
            headers: {"Content-Type": "application/json"},
            body: JSON.stringify({url: url}),
        };
        fetch("/api/create", requestOptions)
            .then(async res => {
                if (res.status !== 200) {
                    throw new Error(res.statusText);
                }
                let data: Entry = await res.json();
                addShortened(data);

                infoStatus = "success";
                infoText = "Shortened URL";
            })
            .catch(() => {
                infoText = "Failed to shorten URL";
                infoStatus = "error";
            });
        setTimeout(() => {
            btnDisabled = false;
        }, 5000);
    }

</script>

<svelte:head>
    <title>Smol-URL</title>
    <meta property="og:title" content="Smol-URL">
    <meta property="og:description" content="A simple URL shortener">
    <meta property="og:url" content="https://s.2d.rocks/">
    <meta property="og:image" content="https://s.2d.rocks/images/logo.png">
    <meta property="og:type" content="website">
    <meta property="og:site_name" content="Smol-URL">
    <meta name="theme-color" content="#6600ff">
</svelte:head>

<main>

    <div class="main">

        <h1>Smol-URL</h1>
        <input
                bind:value={url}
                class="input-url"
                name="url"
                placeholder="Enter a url to shorten"
                type="url"
        />
        <p class={infoStatus}>Status: {infoText}</p>
        <button
                class="btn-shorten"
                disabled={btnDisabled}
                on:click={handleClick}
        >Submit
        </button>
    </div>
    {#if true}
        <div class="result" transition:fade={{ duration: 500 }}>
            <h2>History</h2>
            {#each parsed as entry}
                <Shortened originalUrl="{entry.url}" id={entry.pseudo_id} extraData={entry.extra}/>
            {/each}
        </div>
    {/if}
</main>

<style>

    .error {
        color: red;
    }

    .success {
        color: green;
    }

    .input-url {
        width: 100%;
        padding: 12px 20px;
        margin: 8px 0;
        /*display: inline-block;*/
        border: 1px solid #ccc;
        border-radius: 4px;
        box-sizing: border-box;
    }

    .btn-shorten {
        background-color: #4caf50;
        color: white;
        padding: 12px 20px;
        margin: 8px 0;
        border: none;
        border-radius: 4px;
        cursor: pointer;
    }

    .btn-shorten:disabled {
        background-color: #ccc;
        cursor: wait;
    }

    :global(body) {
        background-color: #333333;
        background-size: cover;
        background-position: center;
        background-image: url("images/bg2.svg");
    }

    main {
        text-align: center;
        padding: 3em;
        margin: 1em auto;
        width: 50%;
        background-color: #fff;
        border-radius: 4px;
        box-shadow: 0 4px 8px 0 rgb(0, 201, 153), 0 6px 20px 0 rgb(87, 106, 232);
    }
</style>
