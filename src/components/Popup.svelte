<script>
    export let message = 'Default message';
    export let data = {};
    import * as d3 from 'd3';
    import { onMount } from 'svelte';

    const equipmentEquivalent = {
        "APC": 152,
        "MRL": 127,
        "tank": 141,
        "field artillery": 190,
        "vehicles and fuel tanks": 108,
        "aircraft": 95,
        "anti-aircraft warfare": 152,
        "cruise missiles": 254,
        "drone": 762,
        "helicopter": 119,
        "naval ship": 12,
        "special equipment": 95,
        "submarines": 38
    };

    Object.keys(data).forEach(key => {
        if (data[key] === null) {
            data[key] = 0;
        }
    });

    let selectedValue = "APC";
    let buildingCount, radius;

    $: buildingCount = parseFloat((data[selectedValue] / equipmentEquivalent[selectedValue]).toFixed(2));
    $: radius = 385 / equipmentEquivalent[selectedValue] / 2;

</script>

<main>
    <div class="container">
        <h1 class="title">Comparing Military Equipment Heights to the Empire State Building</h1>
        <div class="subtitle_container">
            <div class="select_container">
                <select bind:value={selectedValue}>
                    <optgroup label="Land">
                        <option value="APC" selected>APC</option>
                        <option value="MRL">MRL</option>
                        <option value="tank">Tank</option>
                        <option value="field artillery">Field Artillery</option>
                        <option value="vehicles and fuel tanks">Vehicles and Fuel Tanks</option>
                    </optgroup>
                    <optgroup label="Air">
                        <option value="aircraft">Aircraft</option>
                        <option value="anti-aircraft warfare">Anti-Aircraft Warfare</option>
                        <option value="cruise missiles">Cruise Missiles</option>
                        <option value="drone">Drone</option>
                        <option value="helicopter">Helicopter</option>
                    </optgroup>
                    <optgroup label="Sea">
                        <option value="naval ship">Naval Ship</option>
                        <option value="special equipment">Special Equipment</option>
                        <option value="submarines">Submarines</option>
                    </optgroup>
                </select>
            </div>
            <h5 class="subtitle_empire">Empire State Building</h5>
        </div>
        <div class="chart">
            <svg id="graph">
            {#each Array.from({ length: data[selectedValue] }) as _, index}
                <circle
                cx={(radius * 2 > 4 ? radius : 4) + (385 / equipmentEquivalent[selectedValue]) * Math.floor((index) / equipmentEquivalent[selectedValue])}
                cy={385 - (radius * 2 > 4 ? radius : 0) - (radius * 2 * ((index) % equipmentEquivalent[selectedValue]))}
                r={radius}
                fill="steelblue"
                stroke="#000000"
                />
            {/each}
            </svg>
            <svg viewBox="-100 -100 200 200" class="empire_state">
                <g>
                    <path
                    d="
                        M -40 90
                        L -40 100
                        L 40 100
                        L 40 90
                        L 35 90
                        L 35 70
                        L 30 70
                        L 30 60
                        L 25 60
                        L 25 -10
                        L 23 -10
                        L 23 -25
                        L 21 -25
                        L 21 -35
                        L 20 -35
                        L 20 -50
                        L 17 -50
                        L 17 -60
                        L 8 -70
                        L 8 -80
                        L 5 -80
                        L 5 -83
                        L 3 -83
                        Q 0 -80 0 -100
                        Q 0 -80 0 -100
                        L -3 -83
                        L -5 -83
                        L -5 -80
                        L -8 -80
                        L -8 -70
                        L -17 -60
                        L -17 -50
                        L -20 -50
                        L -20 -35
                        L -21 -35
                        L -21 -25
                        L -23 -25
                        L -23 -10
                        L -25 -10
                        L -25 60
                        L -30 60
                        L -30 70
                        L -35 70
                        L -35 90
                        Z"
                    fill="#FFDD00"
                    />
                </g>
            </svg>
        </div>
        <div class="info">
            <p>The height of the Empire State Building is about <span class="important_info">380 meters</span>.</p>
            <p><span class="important_info equipment_name">{selectedValue} Count</span>: {data[selectedValue]}</p>
            <p>{data[selectedValue]} {selectedValue} equals the height of <span class="important_info">{buildingCount}</span> Empire State Buildings.</p>
        </div>
    </div>
</main>

<style>
    .container {
        height: 600px;
        overflow: hidden;
        font-family: "PT Serif", serif;
    }

    .title {
        font-size: 32px;
        font-weight: 700; 
        color: #000; 
        text-transform: capitalize;
        text-align: center;
    }

    .subtitle_container {
        display: flex;
        justify-content: space-between;
        align-items: center;
        margin-bottom: 5px
    }
    .select_container {
        width: 50%;
    }
    .subtitle_empire {
        margin: 0;
        flex-grow: 1;
        text-align: center;
    }
    .chart {
        height: 350px;
        display: flex;
    }
    svg {
        height: 100%;
        width: 50%;
        border: 3px solid #000;
        padding: 0px;
    }
    .empire_state {
        border: 3px solid #000;
    }
    .info {
        line-height: 8px;
    }
    .important_info {
        font-weight: 700;
    }
    .equipment_name {
        text-transform: capitalize;
    }
    p {
        font-size: 16px;
    }
</style>