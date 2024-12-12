<script>
    import { onMount } from 'svelte';
    import * as d3 from 'd3';    

    let numOfTopSongs = 100
    let maxSongs = 0
    let artistName = null
    let loadingTime = null
    let songsList = []
  
    const fetchData = async () => {
        const startTime = performance.now();

        const data = await d3.csv('/dataset/MostStreamedSpotifySongs2024.csv', d => ({
            track: d.Track,
            artist: d.Artist,
            spotifyStreams: +d['Spotify Streams']
         }));
        
        maxSongs = data.length;

        songsList = data.sort((a, b) => b.spotifyStreams - a.spotifyStreams).slice(0, numOfTopSongs);
        
        const endTime = performance.now();
        loadingTime = (endTime - startTime).toFixed(2);
    }
    const handleSubmit = (event) => {
        event.preventDefault();
        fetchData();
    };
    onMount(() => {
        fetchData(); 
    });
</script>
<div>
    <form on:submit={handleSubmit} class="my-4 flex gap-8">
        <label class="block mb-2">
        Top:
            <input
                type="number"
                bind:value={numOfTopSongs}
                min="1"
                max={maxSongs}
                class="border p-2 rounded text-zinc-900"
            />
        </label>

        <label for="">
            Filter Artist:
            <input 
                type="text"
                bind:value={artistName}
                class="border p-2 rounded text-zinc-900"
                placeholder="work in progres"
            />
        </label>
        <button type="submit" class="bg-lime-900 text-white px-4 py-2 rounded hover:bg-lime-700">
        Show Songs
        </button>
    </form>

    {#if loadingTime}
        <p class="text-gray-700">Data loaded in: {loadingTime} ms</p>
    {/if}

    {#if songsList.length > 0}
        <ol class="mt-4 space-y-2">
    {#each songsList as songsList, i}
        <li class="flex items-center space-x-4">
        <span class="font-bold">{i + 1}.</span>
        <span>{songsList.track} - {songsList.artist}</span>
        </li>
    {/each}
        </ol>
    {:else}
        <p>Loading in songs...</p>
    {/if}
</div>
