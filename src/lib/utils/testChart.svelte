<script>
    import { onMount } from 'svelte';
    import * as d3 from 'd3';
  
    let data = [10, 20, 30, 40, 50,55];
  
    onMount(() => {
      const svg = d3.select('svg')
        .attr('width', 300)
        .attr('height', 300);
  
      svg.selectAll('rect')
        .data(data)
        .enter()
        .append('rect')
        .attr('x', (d, i) => i * 50)
        .attr('y', d => 300 - d * 5)
        .attr('width', 40)
        .attr('height', d => d * 5)
        .attr('fill', 'steelblue');
    });
  </script>
  
  <svg></svg>
  
  <style>
    svg {
      border: .2rem solid green;
      background-color: antiquewhite;
    }
  </style>