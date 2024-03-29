<script>
  import Scroller from "@sveltejs/svelte-scroller";
  import { onMount } from 'svelte';
  import * as d3 from 'd3';
  import { news_data } from '../constants.js';
  
  import Modal from './Modal.svelte';
	import Popup from './Popup.svelte';
  import Button from './Button.svelte';
  import { modal } from './stores.js';
  import Hero from './Hero.svelte';

  let data = [];
  const dataKeep = ["date", "APC", "MRL", "aircraft", "anti-aircraft warfare", "cruise missiles", "drone", "field artillery", "helicopter", "naval ship", "special equipment", "submarines", "tank", "vehicles and fuel tanks"];
  const dataKey = dataKeep.slice(1, dataKeep.length - 1);
  let svg, x, y;
  let initialized = false;

  onMount(async () => {
    const res = await fetch('russia_losses_equipment.csv');
    const csv = await res.text();
    data = d3.csvParse(csv, d3.autoType);
    data = data.filter(entry => new Date(entry.date).getDate() === 1);
    console.log(data)
    data = filterObjectsInArray(data, dataKeep);

    renderGraph();
    initialized = true;
  });

  let count, index = 0, offset, progress;

  function filterObjectsInArray(arrayOfObjects, keysToKeep) {
    return arrayOfObjects.map(obj => keysToKeep.reduce((acc, key) => {
      if (obj.hasOwnProperty(key)) acc[key] = obj[key];
      return acc;
    }, {}));
  }

  function renderGraph() {
    const margin = {top: 20, right: 30, bottom: 40, left: 90},
          width = 700 - margin.left - margin.right,
          height = 400 - margin.top - margin.bottom;

    svg = d3.select("#my_dataviz")
      .append("svg")
      .attr("width", width + margin.left + margin.right)
      .attr("height", height + margin.top + margin.bottom)
      .append("g")
      .attr("transform", `translate(${margin.left},${margin.top})`);

    x = d3.scaleLinear()
      .domain([0, d3.max(dataKey, key => d3.max(data, d => d[key]))]) // Adjust domain based on data
      .range([0, width]);

    y = d3.scaleBand()
      .domain(dataKey)
      .range([0, height])
      .padding(0.3);

    svg.append('g')
      .call(d3.axisLeft(y));
  }

  function graphing(index) {
    const bars = svg.selectAll(".myRect").data(dataKey);
    const barsEnter = bars.enter().append("rect")
                        .attr("class", "myRect")
                        .attr("x", x(0))
                        .attr("y", key => y(key))
                        .attr("height", y.bandwidth())
                        .attr("fill", "#0057B7")
                        .attr("width", 0);
    barsEnter.merge(bars)
             .transition()
             .duration(750)
             .ease(d3.easeCubicInOut)
             .attr("x", x(0))
             .attr("y", key => y(key))
             .attr("width", key => x(data[index][key]))
             .attr("height", y.bandwidth())
             .attr("fill", "#0057B7");
    bars.exit()
        .transition()
        .duration(750)
        .ease(d3.easeCubicInOut)
        .attr("width", 0)
        .remove();
  }

  function formatDate(dateString) {
    const date = new Date(dateString);
    const month = date.getMonth() + 1; // getMonth() returns a zero-based index
    const day = date.getDate();
    const year = date.getFullYear();

    // Ensuring two digits for month and day
    const formattedMonth = month < 10 ? `0${month}` : month;
    const formattedDay = day < 10 ? `0${day}` : day;

    return `${formattedMonth}-${formattedDay}-${year}`;
  }

  // Reactive statement to redraw bars when index changes
  $: if (initialized && data.length > 0) {
    graphing(index);
  }

  $: if (data && data.length > index) {
    dataKey.forEach(key => {
      console.log(data[index][key]);
    });
  }

</script>

<main>
    <Hero/>
    <Scroller
    top={0.0}
    bottom={1}
    threshold={0.5}
    bind:count
    bind:index
    bind:offset
    bind:progress
    >
      <div class="background" slot="background">
          <div class="progress-bars">
            <progress class="page_progress" value={progress || 0} />
            <div class="progress_date"><span>Current date: {data[index] ? formatDate(data[index]['date']) : 'undefined'}</span></div>

            <div id="my_dataviz"></div>
            <div class="count_container">
              <div id="counter">
                {#each dataKey as key}
                  <div>
                    <span class="counter_key">{key}</span>: {data[index] ? data[index][key] === null ? 0 : data[index][key] : 'undefined'}
                  </div>
                {/each}
              </div>
            </div>
          </div>
      </div>

      <div class="foreground" slot="foreground">
          {#each news_data as news}
            <section>
              <div class="news-title">{news.title}</div>
              <p class="news-date">{data[index] ? formatDate(data[index]['date']) : 'undefined'}</p>
              <div>
                <h3 class="news-summary-title">News Summary:</h3>
                <ul class="news-summary">
                  {#each news.events as event}
                    <li>{event}</li>
                  {/each}
                </ul>
              </div>
              <Modal show={$modal}>
                <Button data={data[index]}/>
              </Modal>
              {#if index === 22}
                <div class="takeaway-container">
                  <h2 class="takeaway-title">Key Takeaway</h2>
                  <p class="takeaway-text">
                    The key takeaway from our visualization is crystal clear: war exacts a heavy toll, both in terms of resources which include various kinds of military equipment. By allowing users to quantify this toll with our dynamic bar graphs and visualize it against a familiar landmark, we empower them to grasp the enormity of the conflict effortlessly. Our visualization and content underscore the sobering reality that war is not just a distant event but a costly and resource-intensive endeavor that demands our attention and reflection.
                  </p>
                </div>
              {/if}
            </section>
          {/each}
      </div>

    </Scroller>

</main>

<style>

  :global(body) { 
    margin: 0;
    margin-left: 8px;
  }
  .background {
    width: 100%;
    height: 100vh;
    position: relative;
    margin: 0;
    padding: 0;
  }

  .foreground {
    width: 50%;
    margin: 0 0 0 auto;
    height: auto;
    position: relative;
  }

  section {
    height: 80vh;
    background-color: #FFF;
    outline: #000 solid 3px;
    text-align: left;
    color: black;
    padding: 1em;
    margin: 0 3px 0 0;
  }

  .progress-bars {
    position: absolute;
    background: #FAF9F6;
    visibility: visible;
    height: 100%;
    width: 50%;
  }

  .progress_date {
    display: flex;
    font-family: 'Arial', 'Helvetica Neue', Helvetica, sans-serif;
    font-size: 15px;
    font-weight: 600;
    line-height: 1.5;
    letter-spacing: 0.5px;
    color: #333;
    text-align: left;
    overflow-wrap: break-word;
  }

  .progress_date > span {
    margin: 0 auto;
  }

  .page_progress {
    width: 99%;
    color: #000;
  }

  #tooltip {
    position: absolute;
    visibility: hidden;
    pointer-events: none;
    background-color: #0057B7;
    padding: 10px;
    padding-top: 0px;    
    padding-bottom: 0px; 
    border-radius: 5px;
    font-family: Arial, Helvetica, sans-serif;
  }

  #my_dataviz {
    padding-left: 10px
  }

  .count_container {
    display: flex;
    flex-grow: 0;
  }

  #counter {
    width: 40%;
    margin-left: 20px;
    font-family: 'Arial', 'Helvetica Neue', Helvetica, sans-serif;
    font-size: 13px;
    font-weight: 400;
    line-height: 1.5;
    letter-spacing: 0.5px;
    color: #333;
    text-align: left;
    overflow-wrap: break-word;
    background-color: white;
    border-radius: 10px;
    box-shadow: 0 4px 8px rgba(0, 0, 0, 0.3);
    padding: 15px;
    margin-right: 20px;
  }

  .visualize_button {
    height: 30px;
  }

  .counter_key {
    font-weight: 600;
  }

  .news-title {
    font-family: "PT Serif", serif;
    font-size: 32px;
    font-weight: 700; 
    color: #000; 
    margin-bottom: 0px; 
    padding-bottom: 0px;
    text-transform: capitalize;
  }

  .news-date {
    font-family: "PT Serif", serif;
    font-size: 14px;
    font-weight: 600; 
    color: rgba(0, 0, 0, 0.4); 
    margin-bottom: 10px; 
    text-transform: capitalize;
  }

  .news-summary-title {
    font-family: "PT Serif", serif;
    font-size: 18px;
    font-weight: 600; 
    color: rgba(0, 0, 0, 1); 
    margin-bottom: 0px; 
    text-transform: capitalize;
  }

  .news-summary {
    font-family: "PT Serif", serif;
    font-size: 16px;
    font-weight: 400; 
    color: rgba(0, 0, 0, 1); 
    margin-bottom: 0px; 
    list-style-type: circle;
    line-height: 1.5;
  }
  .takeaway-container {
  background-color: #f2f2f2; 
  border-left: 5px solid #0057B7; 
  padding: 20px;
  margin: 20px 0;
  border-radius: 5px;
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1); 
  transition: transform 0.3s ease-in-out; 
}

.takeaway-container:hover {
  transform: translateY(-5px);
}

.takeaway-title {
  font-family: "PT Serif", serif;
  color: #0057B7;
  margin-bottom: 10px;
}

.takeaway-text {
  font-family: "PT Serif", serif;
  line-height: 1.6;
  color: #333;
}
</style>