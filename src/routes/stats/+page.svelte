<script>
    import { onMount } from 'svelte';
    import * as d3 from "d3";

    let startRank = 1;
    let numOfSongs = 10;
    let filterOption = "all";
    let artistFilter = "";
    let el;
    let chartData = [];
    const tooltip = { show: false, x: 0, y: 0, content: "" };

    const fetchData = async () => {
    console.time("fetching");
    const data = await d3.csv('/dataset/MostStreamedSpotifySongs2024.csv', d => ({
        track: d.Track,
        artist: d.Artist,
        spotifyStreams: +d['Spotify Streams'].replace(/,/g, ""), 
        explicit: d['Explicit Track'] === "1"
    }));

    // Here is to multiply the the array to have more data, if you want to run you own test just change this code
    const multipliedData = Array(40).fill(data).flat();

    const rankedData = multipliedData
        .sort((a, b) => b.spotifyStreams - a.spotifyStreams)
        .map((d, i) => ({ ...d, rank: i + 1 }));
    return rankedData;
};

    const formatNumber = (number) => number.toLocaleString();

    const validateInputs = () => {
        if (startRank < 1 || startRank > 4600) {
            startRank = 1;
        }
    };

    const updateChart = async () => {
        validateInputs();

        const inData = await fetchData();

        let filteredData = inData.filter(d => {
            if (filterOption === "explicit") return d.explicit;
            if (filterOption === "non-explicit") return !d.explicit;
            return true;
        });

        if (artistFilter.trim()) {
            const artists = artistFilter.split(",").map(a => a.trim().toLowerCase());
            filteredData = filteredData.filter(d => artists.includes(d.artist.toLowerCase()));
        }

        chartData = filteredData.slice(startRank - 1, startRank - 1 + numOfSongs);
        renderChart();
        console.timeEnd("fetching")
    };

    const renderChart = () => {
        const maxStreams = Math.max(...chartData.map(d => d.spotifyStreams));
        const chart = d3.select(el);
        chart.selectAll("*").remove();
        chart.selectAll("div")
            .data(chartData)
            .enter()
            .append("div")
            .style("width", d => `${(d.spotifyStreams / maxStreams) * 50}rem`)
            .style("height", "3rem")
            .style("margin", ".2rem")
            .style("background-color", d => d.explicit ? "orange" : "limegreen")
            .style("color", "white")
            .style("font-size", "1rem")
            .style("display", "flex")
            .style("align-items", "center")
            .style("padding", ".5rem")
            .style("position", "relative")
            .style("overflow", "hidden")
            .html(d => {
                const width = (d.spotifyStreams / maxStreams) * 50;
                if (width < 1.5) {
                    return "";
                } else if (width < 5) {
                    return `<span style="font-weight: bold;">${d.rank}.</span>`;
                } else {
                    return `

                        <span style="font-weight: bold; margin-right: 1rem;">${d.rank}.</span>
                        <span>${d.track} - 
                            <span 
                                style="cursor: pointer; font-style: italic; text-decoration: none;"
                            >
                                ${d.artist}
                            </span>
                        </span>
                        <span style="margin-left: auto;">(${formatNumber(d.spotifyStreams)})</span>
                    `;
                }
            })
            .on("mouseover", function (event, d) {
                const width = this.offsetWidth / 16;
                if (width < 20) {
                    tooltip.show = true;
                    tooltip.x = event.pageX;
                    tooltip.y = event.pageY;
                    tooltip.content = `${d.rank}. ${d.track} - ${d.artist} (${formatNumber(d.spotifyStreams)})`;
                }
            })
            .on("mousemove", function (event) {
                tooltip.x = event.pageX;
                tooltip.y = event.pageY;
            })
            .on("mouseout", () => {
                tooltip.show = false;
            })
            .on("click", (event, d) => {
                const artist = d.artist;
                const artists = artistFilter.split(",").map(a => a.trim());
                if (!artists.includes(artist)) {
                    artistFilter = artists.concat(artist).filter(Boolean).join(", ");
                }
            });
    };

    const handleSubmit = () => {
        updateChart();
    };

    onMount(() => {
        updateChart();
    });
</script>

<h2 class="text-3xl font-serif">Spotify Song List</h2>

<div class="controls">
    <label class="block">
        Start Rank: 
        <input 
            type="number" 
            bind:value={startRank} 
            min="1" 
            max="4600"
            class="border p-2 text-black" 
        />
    </label>
    <label class="block">
        Number of Songs: 
        <input 
            type="number" 
            bind:value={numOfSongs} 
            min="1" 
            class="border p-2 text-black" 
        />
    </label>
    <label class="block">
        Filter: 
        <select 
            bind:value={filterOption} 
            class="border p-2 text-black"
        >
            <option value="all">All</option>
            <option value="non-explicit">Without Explicit</option>
            <option value="explicit">Only Explicit</option>
        </select>
    </label>
    <label class="block">
        Artist Filter (comma-separated): 
        <input 
            type="text" 
            bind:value={artistFilter} 
            placeholder="e.g., Artist1, Artist2"
            class="border p-2 text-black"
        />
    </label>
    <button 
        on:click={handleSubmit} 
        class="bg-blue-500 text-white p-2 rounded"
    >
        Submit
    </button>
</div>

<div bind:this={el} class="chart"></div>

{#if tooltip.show}
    <div 
        class="tooltip" 
        style="position: absolute; top: {tooltip.y}px; left: {tooltip.x}px; background: black; color: white; padding: 0.5rem; border-radius: 0.25rem; pointer-events: none;"
    >
        {tooltip.content}
    </div>
{/if}

<style>
    .controls {
        margin-bottom: 1rem;
        display: flex;
        flex-wrap: wrap;
        gap: 1rem;
        align-items: center;
    }

    input[type="number"],
    input[type="text"],
    select {
        color: black;
    }

    button {
        cursor: pointer;
    }

    .tooltip {
        z-index: 10;
        white-space: nowrap;
    }

    .chart div {
        cursor: default;
    }

    .chart span {
        cursor: default;
    }
</style>