<script lang="ts">
    import { fade } from "svelte/transition";
    import CopyToClipboard from "svelte-copy-to-clipboard";

    let url = "";
    let lastUrl = "";
    let resultUrl = [];
    let btnDisabled = false;
    let infoText = "Enter a URL to shorten";
    let infoStatus = "info";

    function handleClick() {
        if (url === "") {
            infoText = "Please enter a URL";
            infoStatus = "error";
            return;
        }
        if (url === lastUrl) {
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
            headers: { "Content-Type": "application/json" },
            body: JSON.stringify({ url: url }),
        };
        fetch("https://2d.rocks/api/create", requestOptions)
            .then((res) => res.json())
            .then((res) => {
                resultUrl = [res.prefix + res.pseudo_id];
                infoStatus = "success";
                infoText = "Shortened URL";
                for (const [, value] of Object.entries(res.extra)) {
                    resultUrl = [...resultUrl, res.prefix + value];
                }
            })
            .catch(() => {
                infoText = "Failed to shorten URL";
                infoStatus = "error";
            });
        setTimeout(() => {
            btnDisabled = false;
        }, 5000);
    }

    function onCopySuccess() {
        infoStatus = "success";
        infoText = "Copied to clipboard";
    }

    function onCopyFail() {
        infoStatus = "error";
        infoText = "Failed to copy";
    }
</script>

<svelte:head>
    <title>Smol-URL</title>
</svelte:head>

<main>
    <div class="main">
        <h1>2d.rocks url-shortener</h1>
        <input
            class="input-url"
            type="url"
            name="url"
            placeholder="Enter a url to shorten"
            bind:value={url}
        />
        <p class={infoStatus}>Status: {infoText}</p>
        <button
            class="btn-shorten"
            on:click={handleClick}
            disabled={btnDisabled}>Submit</button
        >
    </div>
    {#if resultUrl.length > 0}
        <div class="result" transition:fade={{ duration: 500 }}>
            <h2>Your shortened url is:</h2>
            <h4>Hint: Click the link to copy</h4>
            {#each resultUrl as url}
                <CopyToClipboard
                    text={url}
                    on:copy={onCopySuccess}
                    on:fail={onCopyFail}
                    let:copy
                >
                    <p class="url" on:click={copy}>{url}</p>
                </CopyToClipboard>
            {/each}
        </div>
    {/if}
</main>

<style>
    .url {
        cursor: pointer;
    }

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
        display: inline-block;
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
        background-image: url("/images/bg2.svg");
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
