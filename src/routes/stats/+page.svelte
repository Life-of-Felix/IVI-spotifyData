<script>
    import { onMount } from 'svelte';
    import * as d3 from "d3";

    let numOfTopSongs = 10; 
    let el; 

    const fetchData = async () => {
        const data = await d3.csv('/dataset/MostStreamedSpotifySongs2024.csv', d => ({
            track: d.Track,
            artist: d.Artist,
            spotifyStreams: +d['Spotify Streams'].replace(/,/g, "") // Ta bort kommatecken och konvertera till nummer
        }));
        
        return data.sort((a, b) => b.spotifyStreams - a.spotifyStreams).slice(0, numOfTopSongs);
    };

    onMount(async () => {
        const inData = await fetchData(); 

        
        d3.select(el)
            .selectAll("div")
            .data(inData)
            .enter()
            .append("div")
            .style("width", d => `${d.spotifyStreams / 5e6}px`) 
            .style("height", "3rem")
            .style("margin", ".2rem")
            .style("background-color", "limegreen")
            .style("color", "white")
            .style("font-size", "1.2rem")
            .style("display", "flex")
            .style("align-items", "center")
            .style("padding",".5rem" )
            .text(d => `${d.track} - ${d.artist}>${d.spotifyStreams}  `);
    });
</script>

<h2 class="text-3xl font-serif">Top {numOfTopSongs} Songs by Spotify Streams</h2>

<!-- Div för graf -->
<div bind:this={el} class="chart"></div>