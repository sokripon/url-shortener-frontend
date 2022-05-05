
<script>
    import {fly, fade } from 'svelte/transition';
    let url = ""
    let lastUrl = "";
    let resultUrl = "";
    let tooltip = "Click to copy";
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
            method: 'PUT',
            headers: { 'Content-Type': 'application/json' },
            body: JSON.stringify({ url: url })
        };
        fetch('https://example.com/create', requestOptions)
            .then(res => res.json())
            .then(res => {
                resultUrl = res.url;
                infoStatus = "success";
                infoText = "Shortened URL";
                tooltip = "Click to copy";
            })
            .catch(err => {
                infoText = "Failed to shorten URL";
                infoStatus = "error";
            });
        let interval = setInterval(() => {
            btnDisabled = false;
            clearInterval(interval);
        }, 5000);

    }

    function copyShortURL() {
        navigator.clipboard.writeText(resultUrl);
        tooltip = "Copied!";
    }
</script>

<main>
    <div class="main">
        <h1>2d.rocks url-shortener</h1>
        <input class="input-url" type="url" name="url" placeholder="Enter a url to shorten" bind:value={url}/>
        <p class="{infoStatus}">Status: {infoText}</p>
        <button class="btn-shorten" on:click={handleClick} disabled="{btnDisabled}">Submit</button>

    </div>

    {#if resultUrl !== ""}
        <div class="result" transition:fade={{duration:500}}>
            <h2>Your shortened url is:</h2>
            <div class="tooltip" on:click={copyShortURL}>{resultUrl}
                <span class="tooltip-text">{tooltip}</span>
            </div>
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
        display: inline-block;
        border: 1px solid #ccc;
        border-radius: 4px;
        box-sizing: border-box;
    }

    /*Copied from https://www.w3schools.com/css/css_tooltip.asp ehe*/
    .tooltip {
        position: relative;
        display: inline-block;
        border-bottom: 1px dotted black;
        cursor: pointer;
    }

    .tooltip .tooltip-text {
        visibility: visible;
        width: 120px;
        background-color: black;
        color: #fff;
        text-align: center;
        border-radius: 6px;
        padding: 5px 0;
        position: absolute;
        z-index: 1;
        top: 150%;
        left: 50%;
        margin-left: -60px;
    }

    .tooltip .tooltip-text::after {
        content: "";
        position: absolute;
        bottom: 100%;
        left: 50%;
        margin-left: -5px;
        border-width: 5px;
        border-style: solid;
        border-color: transparent transparent black transparent;
    }

    .btn-shorten {
        background-color: #4CAF50;
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