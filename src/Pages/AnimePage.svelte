<script>
    import { onMount } from "svelte";
    import { Player, DefaultUi } from "@vime/svelte";
    import {
        Accordion,
        Button,
        AccordionItem,
        Badge,
        Spinner,
    } from "sveltestrap";
    import { apiUrl } from "../config";

    export let currentRoute;
    let animeId = currentRoute.namedParams.id;

    let duration = -1;
    let currentTime = 0;
    let paused = true;

    let animeInfo = {};
    let series = [];
    let isLoaded = false;
    let selectedSeria = {};

    onMount(async () => {
        animeInfo = await getAnimeInfo(animeId);
        series = await getAnimePlaylist(animeId);
        selectedSeria = series[0] || [];
        isLoaded = true;
    });

    async function getAnimeInfo(id) {
        const response = await fetch(`${apiUrl}/info`, {
            method: "POST", // *GET, POST, PUT, DELETE, etc.
            mode: "cors", // no-cors, *cors, same-origin
            cache: "no-cache", // *default, no-cache, reload, force-cache, only-if-cached
            credentials: "same-origin", // include, *same-origin, omit
            headers: {
                // "Content-Type": "application/json",
                "Content-Type": "application/x-www-form-urlencoded",
            },
            redirect: "follow", // manual, *follow, error
            referrerPolicy: "no-referrer", // no-referrer, *client
            body: `id=${id}`, // body data type must match "Content-Type" header
        });
        const json = await response.json();
        return json.data[0];
    }

    async function getAnimePlaylist(id) {
        const response = await fetch(`${apiUrl}/playlist`, {
            method: "POST", // *GET, POST, PUT, DELETE, etc.
            mode: "cors", // no-cors, *cors, same-origin
            cache: "no-cache", // *default, no-cache, reload, force-cache, only-if-cached
            credentials: "same-origin", // include, *same-origin, omit
            headers: {
                // "Content-Type": "application/json",
                "Content-Type": "application/x-www-form-urlencoded",
            },
            redirect: "follow", // manual, *follow, error
            referrerPolicy: "no-referrer", // no-referrer, *client
            body: `id=${id}`, // body data type must match "Content-Type" header
        });
        const json = await response.json();
        json.sort((a, b) => {
            return Number(a.name.split(" ")[0]) - Number(b.name.split(" ")[0]);
        });
        return json;
    }

    function onTimeUpdate(event) {
        currentTime = event.detail;
    }

    function keydownHandler(event) {
        event.preventDefault();
        switch (event.code) {
            case "ArrowLeft":
                currentTime = currentTime - 10;
                break;
            case "ArrowRight":
                currentTime = currentTime + 10;
                break;
            case "Space":
                paused = !paused;
                break;
        }
    }

    function seriesHandler(seria) {
        selectedSeria = seria;
    }
</script>

<svelte:window on:keydown={keydownHandler} />

{#if isLoaded}
    <h4>
        {animeInfo.title}
        <Badge class="me-2 mb-1 mt-1"
            >{selectedSeria.name || "Нет серий, Анонс?"}</Badge
        >
    </h4>
    <div style="max-width:800px" class="mx-auto">
        {#if series.length > 0}
            <div style="max-width:800px">
                <Player
                    on:vmCurrentTimeChange={onTimeUpdate}
                    {currentTime}
                    {duration}
                    {paused}
                    volume={70}
                >
                    <vm-video poster={selectedSeria.preview}>
                        <source data-src={selectedSeria.std} type="video/mp4" />
                        <track kind="captions" />
                    </vm-video>
                    <DefaultUi noDblClickFullscreen />
                </Player>
            </div>
        {/if}
    </div>
    <Accordion flush stayOpen class="mt-3">
        {#if series.length > 0}
            <AccordionItem active header="Серии">
                {#each series as seria (seria.name)}
                    {#if seria.name === selectedSeria.name}
                        <!-- если эта серия выбрана -->
                        <Button
                            class="me-2 mb-1 mt-1"
                            primary
                            on:click={() => seriesHandler(seria)}
                        >
                            {seria.name}
                        </Button>
                    {:else}
                        <Button
                            class="me-2 mb-1 mt-1"
                            outline
                            secondary
                            on:click={() => seriesHandler(seria)}
                        >
                            {seria.name}
                        </Button>
                    {/if}
                {/each}
            </AccordionItem>
        {/if}
        <AccordionItem active header="Год выпуска">
            {animeInfo.year}
        </AccordionItem>
        <AccordionItem active header="Жанр">
            <h5>
                {#each animeInfo.genre.split(",") as gen}
                    <Badge class="me-2 mb-1 mt-1" primary>{gen.trim()}</Badge>
                {/each}
            </h5>
        </AccordionItem>
        {#if animeInfo.director}
            <AccordionItem active header="Создатель">
                {animeInfo.director}
            </AccordionItem>
        {/if}
        <AccordionItem active header="Описание">
            {@html animeInfo.description}
        </AccordionItem>
    </Accordion>
{:else}
    <h2>loading <Spinner color="primary" /></h2>
{/if}