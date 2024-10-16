<script>
    import { onMount } from 'svelte';
    import { fade, fly, scale } from 'svelte/transition';
    import BirdFeederSVG from './BirdFeederSVG.svelte';

    let foodLevel = 100;
    let birdDetected = false;
    let recording = false;
    let currentBird = '';
    let dailyVisits = 0;
    let lastCleanedDate = new Date().toLocaleDateString();
    let weatherCondition = 'Sunny';
    let temperature = 72; // Fahrenheit

    // Simulated bird species
    const birdSpecies = [
        { name: 'Sparrow', color: '#A52A2A' },
        { name: 'Robin', color: '#FF6347' },
        { name: 'Blue Jay', color: '#4169E1' },
        { name: 'Cardinal', color: '#DC143C' },
        { name: 'Finch', color: '#DAA520' }  // Changed from '#FFD700' to '#DAA520' (Goldenrod)
    ];
    let recentBirds = [];

    // Weather conditions
    const weatherConditions = ['Sunny', 'Cloudy', 'Rainy', 'Windy'];

    let notifications = [];
    let notificationId = 0;

    let isRefilling = false;

    let feederCleanliness = 100; // New variable for feeder cleanliness

    let batteryLevel = 100;
    let batteryDrainInterval;

    let isCleaning = false;

    let isRecharging = false;

    // New variables for bird statistics
    let showBirdStats = false;
    let birdStats = {};

    // New variables for sorting
    $: sortColumn = 'name';
    $: sortDirection = 'asc';

    function addNotification(message) {
        notificationId++;
        const newNotification = { id: notificationId, message };
        notifications = [...notifications, newNotification];
        setTimeout(() => {
            notifications = notifications.filter(n => n.id !== newNotification.id);
        }, 3000);
    }

    onMount(() => {
        dailyVisits = Math.floor(Math.random() * 10);
        temperature = Math.floor(Math.random() * 30) + 60;
        simulateBatteryDrain();
    });

    const simulateBirdDetection = () => {
        if (batteryLevel === 0) {
            addNotification("Cannot detect birds. Battery depleted. Please recharge.");
            return;
        }

        if (foodLevel === 0 && !birdDetected) {
            addNotification("Cannot detect birds. Feeder is empty! Please refill.");
            return;
        }

        birdDetected = !birdDetected;
        recording = birdDetected;
        if (birdDetected) {
            currentBird = birdSpecies[Math.floor(Math.random() * birdSpecies.length)];
            dailyVisits++;
            recentBirds = [{ ...currentBird, timestamp: new Date() }, ...recentBirds.slice(0, 9)];
            
            const oldLevel = foodLevel;
            simulateFoodLevelChange();
            const foodConsumed = oldLevel - foodLevel;

            // Update bird stats
            if (!birdStats[currentBird.name]) {
                birdStats[currentBird.name] = { dailyVisits: 0, totalVisits: 0, totalFoodConsumed: 0 };
            }
            birdStats[currentBird.name].dailyVisits++;
            birdStats[currentBird.name].totalVisits++;
            birdStats[currentBird.name].totalFoodConsumed += foodConsumed;
            
            addNotification(`${currentBird.name} detected! Food level decreased from ${oldLevel}% to ${foodLevel}%`);
            
            if (birdFeederSVG) birdFeederSVG.createFoodParticles();
            
            // Decrease cleanliness slightly with each bird visit
            feederCleanliness = Math.max(0, feederCleanliness - 2);
            decreaseBattery(0.5); // Decrease battery by 0.5% for each bird detection
        } else {
            addNotification("Bird detection stopped.");
            currentBird = null; // Change this from empty string to null
        }
    };

    const simulateFoodLevelChange = () => {
        const consumption = Math.floor(Math.random() * 10 + 5);
        foodLevel = Math.max(0, foodLevel - consumption);
        if (foodLevel === 0) {
            addNotification("Feeder is empty! Please refill.");
            birdDetected = false;
            currentBird = '';
        }
    };

    const refillFood = () => {
        const startLevel = foodLevel;
        const refillInterval = setInterval(() => {
            foodLevel += 1;
            if (foodLevel >= 100) {
                clearInterval(refillInterval);
                foodLevel = 100;
                addNotification("Feeder has been refilled to 100%");
            }
        }, 20);
    };

    const cleanFeeder = () => {
        if (batteryLevel === 0) {
            addNotification("Cannot clean feeder. Battery depleted. Please recharge.");
            return;
        }
        isCleaning = true;
        lastCleanedDate = new Date().toLocaleDateString();
        feederCleanliness = 100;
        addNotification("Feeder is being cleaned...");
        setTimeout(() => {
            addNotification("Feeder has been cleaned. Cleanliness restored to 100%");
            isCleaning = false;
        }, 1500);
        decreaseBattery(2);
    };

    const changeWeather = () => {
        if (batteryLevel === 0) {
            addNotification("Cannot change weather. Battery depleted. Please recharge.");
            return;
        }
        const oldWeather = weatherCondition;
        const availableWeather = weatherConditions.filter(w => w !== oldWeather);
        weatherCondition = availableWeather[Math.floor(Math.random() * availableWeather.length)];
        temperature = Math.floor(Math.random() * 30) + 60;
        addNotification(`Weather changed from ${oldWeather} to ${weatherCondition}`);
        decreaseBattery(0.2); // Decrease battery by 0.2% for changing weather
    };

    $: daysSinceCleaning = Math.floor((new Date() - new Date(lastCleanedDate)) / (1000 * 60 * 60 * 24));

    // Function to format the timestamp
    function formatTimestamp(date) {
        return date.toLocaleTimeString([], { hour: '2-digit', minute: '2-digit' });
    }

    let birdFeederSVG;

    $: isRecentBirdsScrollable = recentBirds.length > 5;

    // Function to get cleanliness status text
    function getCleanlinessStatus(cleanliness) {
        if (cleanliness > 80) return "Excellent";
        if (cleanliness > 60) return "Good";
        if (cleanliness > 40) return "Fair";
        if (cleanliness > 20) return "Poor";
        return "Critical";
    }

    $: cleanlinessStatus = getCleanlinessStatus(feederCleanliness);

    function simulateBatteryDrain() {
        batteryDrainInterval = setInterval(() => {
            batteryLevel = Math.max(0, parseFloat((batteryLevel - 0.1).toFixed(1))); // Round to 1 decimal
            if (batteryLevel < 20 && Math.floor(batteryLevel) % 5 === 0) {
                addNotification(`Low battery: ${batteryLevel}%`);
            }
            if (batteryLevel === 0) {
                clearInterval(batteryDrainInterval);
                addNotification("Battery depleted. Please recharge.");
            }
        }, 6000); // Run every 6 seconds
    }

    function decreaseBattery(amount) {
        batteryLevel = Math.max(0, parseFloat((batteryLevel - amount).toFixed(1))); // Round to 1 decimal
        if (batteryLevel === 0) {
            clearInterval(batteryDrainInterval);
            addNotification("Battery depleted. Please recharge.");
        }
    }

    // Add a function to recharge the battery
    const rechargeBattery = () => {
        if (batteryLevel >= 100) {
            addNotification("Battery is already fully charged.");
            return;
        }
        isRecharging = true;
        const rechargeInterval = setInterval(() => {
            batteryLevel = Math.min(100, parseFloat((batteryLevel + 0.5).toFixed(1)));
            if (batteryLevel >= 100) {
                clearInterval(rechargeInterval);
                isRecharging = false;
                addNotification("Battery recharged to 100%");
            }
        }, 50);
        if (!batteryDrainInterval) {
            simulateBatteryDrain();
        }
    };

    // Function to toggle bird stats modal
    const toggleBirdStats = () => {
        showBirdStats = !showBirdStats;
    };

    // Function to get total visits
    const getTotalVisits = () => {
        return Object.values(birdStats).reduce((total, bird) => total + bird.totalVisits, 0);
    };

    // Function to get most frequent visitor
    const getMostFrequentVisitor = () => {
        return Object.entries(birdStats).reduce((max, [name, stats]) => 
            stats.totalVisits > (max.stats?.totalVisits || 0) ? {name, stats} : max
        , {}).name || 'N/A';
    };

    // Function to calculate average food consumption per visit for a bird species
    function getAverageFoodConsumption(stats) {
        return stats.totalVisits > 0 ? (stats.totalFoodConsumed / stats.totalVisits).toFixed(2) : '0.00';
    }

    // Function to toggle sort
    function toggleSort(column) {
        if (sortColumn === column) {
            sortDirection = sortDirection === 'asc' ? 'desc' : 'asc';
        } else {
            sortColumn = column;
            sortDirection = 'asc';
        }
    }

    // Function to sort bird stats
    $: sortedBirdStats = Object.entries(birdStats).sort((a, b) => {
        const [nameA, statsA] = a;
        const [nameB, statsB] = b;
        let comparison = 0;

        switch (sortColumn) {
            case 'name':
                comparison = nameA.localeCompare(nameB);
                break;
            case 'dailyVisits':
                comparison = statsA.dailyVisits - statsB.dailyVisits;
                break;
            case 'totalVisits':
                comparison = statsA.totalVisits - statsB.totalVisits;
                break;
            case 'avgFoodConsumption':
                const avgA = parseFloat(getAverageFoodConsumption(statsA));
                const avgB = parseFloat(getAverageFoodConsumption(statsB));
                comparison = avgA - avgB;
                break;
        }

        return sortDirection === 'asc' ? comparison : -comparison;
    });
</script>

<div class="page-container">
    <header class="project-info">
        <h1>Smart Bird Feeder</h1>
        <p>By: Bartosz K, Grant G, Zach B</p>
        <a href="your-project-link" target="_blank">Project Write-Up</a>
        <button on:click={() => alert("This simulation allows you to test bird detection, food levels, weather changes, and feeder maintenance. Use the controls to interact with the smart bird feeder.")}>
            Info
        </button>
    </header>

    <div class="content-container">
        <div class="feeder-ui">
            <div class="bird-feeder-svg">
                <BirdFeederSVG 
                    bind:this={birdFeederSVG}
                    {foodLevel} 
                    {birdDetected} 
                    {weatherCondition} 
                    {currentBird} 
                    {isRefilling}
                    {isCleaning}
                    {isRecharging}
                />
            </div>

            <div class="controls-and-status">
                <div class="panel status-panel">
                    <h2>Feeder Status</h2>
                    <div class="status-grid">
                        <div class="status-item">
                            <div class="icon">🐦</div>
                            <div>
                                <p class="status-label">Bird Detected</p>
                                <p class="status-value">{birdDetected ? "Yes" : "No"}</p>
                            </div>
                        </div>
                        <div class="status-item">
                            <div class="icon">📹</div>
                            <div>
                                <p class="status-label">Recording</p>
                                <p class="status-value">{recording ? "Yes" : "No"}</p>
                            </div>
                        </div>
                        <div class="status-item">
                            <div class="icon">📅</div>
                            <div>
                                <p class="status-label">Daily Visits</p>
                                <p class="status-value">{dailyVisits}</p>
                            </div>
                        </div>
                        <div class="status-item">
                            <div class="icon">🧹</div>
                            <div>
                                <p class="status-label">Days Since Cleaning</p>
                                <p class="status-value">{daysSinceCleaning}</p>
                            </div>
                        </div>
                        <div class="status-item">
                            <div class="icon">☀️</div>
                            <div>
                                <p class="status-label">Weather</p>
                                <p class="status-value">{weatherCondition}</p>
                            </div>
                        </div>
                        <div class="status-item">
                            <div class="icon">🌡️</div>
                            <div>
                                <p class="status-label">Temperature</p>
                                <p class="status-value">{temperature}°F</p>
                            </div>
                        </div>
                        <div class="status-item">
                            <div class="icon">🧼</div>
                            <div>
                                <p class="status-label">Cleanliness</p>
                                <p class="status-value">{cleanlinessStatus} ({feederCleanliness}%)</p>
                            </div>
                        </div>
                        <div class="status-item">
                            <div class="icon">🔋</div>
                            <div>
                                <p class="status-label">Battery Level</p>
                                <p class="status-value">{batteryLevel.toFixed(1)}%</p>
                            </div>
                        </div>
                    </div>
                    <div class="food-level">
                        <p class="status-label">Food Level</p>
                        <div class="food-bar-container">
                            <div class="food-bar" style="width: {foodLevel}%;"></div>
                        </div>
                        <p class="status-value">{foodLevel}%</p>
                    </div>
                </div>

                <div class="panel recent-birds">
                    <h2>Recent Birds</h2>
                    <div class="bird-list-container" class:scrollable={isRecentBirdsScrollable}>
                        <ul class="bird-list">
                            {#each recentBirds as bird}
                                <li class="bird-item" transition:fade>
                                    <span class="bird-icon" style="background-color: {bird.color};"></span>
                                    <span class="bird-name">{bird.name}</span>
                                    <span class="bird-time">{formatTimestamp(bird.timestamp)}</span>
                                </li>
                            {/each}
                        </ul>
                    </div>
                    <button on:click={toggleBirdStats} class="more-info-button">More Info</button>
                </div>

                <div class="panel controls">
                    <h2>Simulation Controls</h2>
                    <button on:click={simulateBirdDetection} class="control-button">
                        <span class="icon">🐦</span>
                        <span class="button-text">{birdDetected ? "Stop Bird Detection" : "Simulate Bird Detection"}</span>
                    </button>
                    <button on:click={refillFood} class="control-button">
                        <span class="icon">🥜</span>
                        <span class="button-text">Refill Food</span>
                    </button>
                    <button on:click={cleanFeeder} class="control-button">
                        <span class="icon">🧹</span>
                        <span class="button-text">Clean Feeder ({feederCleanliness}%)</span>
                    </button>
                    <button on:click={changeWeather} class="control-button">
                        <span class="icon">🌤️</span>
                        <span class="button-text">Change Weather</span>
                    </button>
                    <button on:click={rechargeBattery} class="control-button">
                        <span class="icon">🔋</span>
                        <span class="button-text">Recharge Battery ({batteryLevel.toFixed(1)}%)</span>
                    </button>
                </div>
            </div>
        </div>
    </div>
</div>

<!-- Notifications -->
<div class="notifications-container">
    {#each notifications as notification (notification.id)}
        <div class="notification" transition:fly="{{ y: 50, duration: 300 }}">
            {notification.message}
        </div>
    {/each}
</div>

<!-- Bird Stats Modal -->
{#if showBirdStats}
    <div class="modal-backdrop" on:click={toggleBirdStats} transition:fade>
        <div class="modal-content" on:click|stopPropagation transition:scale>
            <h2>Bird Visit Statistics</h2>
            <table>
                <tr>
                    <th on:click={() => toggleSort('name')}>
                        Bird Species {sortColumn === 'name' ? (sortDirection === 'asc' ? '▲' : '▼') : ''}
                    </th>
                    <th on:click={() => toggleSort('dailyVisits')}>
                        Daily Visits {sortColumn === 'dailyVisits' ? (sortDirection === 'asc' ? '▲' : '▼') : ''}
                    </th>
                    <th on:click={() => toggleSort('totalVisits')}>
                        Total Visits {sortColumn === 'totalVisits' ? (sortDirection === 'asc' ? '▲' : '▼') : ''}
                    </th>
                    <th on:click={() => toggleSort('avgFoodConsumption')}>
                        Avg. Food Consumption (%) {sortColumn === 'avgFoodConsumption' ? (sortDirection === 'asc' ? '▲' : '▼') : ''}
                    </th>
                </tr>
                {#each sortedBirdStats as [name, stats]}
                    <tr>
                        <td>{name}</td>
                        <td>{stats.dailyVisits}</td>
                        <td>{stats.totalVisits}</td>
                        <td>{getAverageFoodConsumption(stats)}%</td>
                    </tr>
                {/each}
            </table>
            <p>Total Visits: {getTotalVisits()}</p>
            <p>Most Frequent Visitor: {getMostFrequentVisitor()}</p>
            <button on:click={toggleBirdStats}>Close</button>
        </div>
    </div>
{/if}

<style>
    :global(body) {
        background-color: #1e1e1e;
        color: #e0e0e0;
        font-family: Arial, sans-serif;
        margin: 0;
        padding: 0;
        overflow-y: auto;
    }

    .page-container {
        display: flex;
        flex-direction: column;
        min-height: 100vh;
    }

    .project-info {
        background-color: #2a2a2a;
        padding: 20px;
        text-align: center;
        box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
        z-index: 10;
        border-radius: 0 0 10px 10px;
        margin-bottom: 10px;
    }

    .content-container {
        flex: 1;
        display: flex;
        flex-direction: column;
        padding: 10px 20px;
    }

    .feeder-ui {
        display: flex;
        justify-content: space-between;
        gap: 40px;
        max-width: 1400px;
        width: 100%;
        margin: 0 auto;
    }

    .bird-feeder-svg {
        flex: 1;
        background-color: #2a2a2a;
        padding: 30px;
        border-radius: 10px;
        box-shadow: 0 4px 6px rgba(0, 0, 0, 0.1);
    }

    .controls-and-status {
        flex: 1;
        display: flex;
        flex-direction: column;
        gap: 25px;
        max-width: 45%;
    }

    .panel {
        background-color: #2a2a2a;
        padding: 20px;
        border-radius: 10px;
        box-shadow: 0 4px 6px rgba(0, 0, 0, 0.1);
    }

    .panel h2 {
        color: #4CAF50;
        margin-top: 0; /* Remove top margin */
        margin-bottom: 10px; /* Reduced from 15px */
    }

    .status-panel, .controls {
        flex-shrink: 0;
    }

    .recent-birds {
        flex: 1;
        display: flex;
        flex-direction: column;
    }

    .bird-list-container {
        flex: 1;
        overflow-y: hidden;
    }

    .bird-list-container.scrollable {
        overflow-y: auto;
        max-height: 250px; /* Adjust this value to fit 5 birds comfortably */
    }

    .bird-list {
        list-style-type: none;
        padding: 0;
        margin: 0;
    }

    .bird-item {
        display: flex;
        align-items: center;
        gap: 15px;
        margin: 12px 0;
        padding-right: 20px;
    }

    .bird-icon {
        width: 24px;
        height: 24px;
        border-radius: 50%;
        flex-shrink: 0;
    }

    .bird-name {
        flex-grow: 1;
        white-space: nowrap;
        overflow: hidden;
        text-overflow: ellipsis;
        font-size: 1.1em;
    }

    .bird-time {
        font-size: 0.9em;
        color: #888;
        flex-shrink: 0;
    }

    .status-grid {
        display: grid;
        grid-template-columns: repeat(2, 1fr);
        gap: 20px;
    }

    .status-item {
        display: flex;
        align-items: center;
        gap: 15px;
    }

    .status-item .icon {
        font-size: 1.5em;
        width: 30px; /* Fixed width for all icons */
        text-align: center;
        flex-shrink: 0;
    }

    .status-item div {
        display: flex;
        flex-direction: column;
        justify-content: center;
    }

    .status-label {
        font-size: 0.9em;
        color: #888;
        margin: 0;
        line-height: 1.2;
    }

    .status-value {
        font-size: 1.1em;
        font-weight: bold;
        margin: 0;
        line-height: 1.2;
    }

    .food-level {
        margin-top: 25px;
    }

    .food-bar-container {
        background-color: #444;
        height: 25px;
        border-radius: 12px;
        overflow: hidden;
        margin: 12px 0;
    }

    .food-bar {
        height: 100%;
        background-color: #4CAF50;
        border-radius: 10px;
        transition: width 0.5s ease;
    }

    .bird-list-container::-webkit-scrollbar {
        width: 10px;
    }

    .bird-list-container::-webkit-scrollbar-track {
        background: #2a2a2a;
    }

    .bird-list-container::-webkit-scrollbar-thumb {
        background-color: #4CAF50;
        border-radius: 5px;
        border: 2px solid #2a2a2a;
    }

    .bird-list-container::-webkit-scrollbar-thumb:hover {
        background-color: #45a049;
    }

    .bird-list-container {
        overflow-y: visible;
    }

    .bird-list {
        padding-right: 5px;
    }

    .bird-list::-webkit-scrollbar {
        width: 10px;
    }

    .bird-list::-webkit-scrollbar-track {
        background: #2a2a2a;
    }

    .bird-list::-webkit-scrollbar-thumb {
        background-color: #4CAF50;
        border-radius: 5px;
        border: 2px solid #2a2a2a;
    }

    .bird-list::-webkit-scrollbar-thumb:hover {
        background-color: #45a049;
    }

    .recent-birds {
        overflow: hidden;
    }

    .bird-item {
        display: flex;
        align-items: center;
        gap: 15px;
        margin: 12px 0;
        padding-right: 20px;
    }

    .bird-icon {
        width: 24px;
        height: 24px;
        border-radius: 50%;
        flex-shrink: 0;
    }

    .bird-name {
        flex-grow: 1;
        white-space: nowrap;
        overflow: hidden;
        text-overflow: ellipsis;
        font-size: 1.1em;
    }

    .bird-time {
        font-size: 0.9em;
        color: #888;
        flex-shrink: 0;
    }

    .control-button {
        display: flex;
        align-items: center;
        justify-content: flex-start;
        gap: 15px;
        margin: 12px 0;
        width: 100%;
        background-color: #3a3a3a;
        color: #e0e0e0;
        border: none;
        padding: 15px;
        border-radius: 6px;
        cursor: pointer;
        transition: background-color 0.3s;
        font-size: 1.1em;
    }

    .control-button:hover {
        background-color: #4a4a4a;
    }

    .icon {
        font-size: 1.7em;
        margin-right: 12px;
    }

    h1, h2 {
        color: #4CAF50;
        margin-bottom: 15px;
    }

    a {
        color: #4CAF50;
        text-decoration: none;
    }

    a:hover {
        text-decoration: underline;
    }

    .notifications-container {
        position: fixed;
        bottom: 20px;
        right: 20px;
        display: flex;
        flex-direction: column;
        align-items: flex-end;
        gap: 10px;
        z-index: 1000;
    }

    .notification {
        background-color: #4CAF50;
        color: white;
        padding: 12px 20px;
        border-radius: 5px;
        box-shadow: 0 2px 10px rgba(0, 0, 0, 0.2);
        max-width: 300px;
        word-wrap: break-word;
        font-size: 14px;
    }

    .page-container::after {
        content: none;
    }

    /* Styling for the controls-and-status scrollbar */
    .controls-and-status::-webkit-scrollbar {
        width: 10px;
    }

    .controls-and-status::-webkit-scrollbar-track {
        background: #1e1e1e;
        border-radius: 5px;
    }

    .controls-and-status::-webkit-scrollbar-thumb {
        background-color: #4CAF50;
        border-radius: 5px;
        border: 2px solid #1e1e1e;
    }

    .controls-and-status::-webkit-scrollbar-thumb:hover {
        background-color: #45a049;
    }

    /* For Firefox */
    .controls-and-status {
        scrollbar-width: thin;
        scrollbar-color: #4CAF50 #1e1e1e;
    }

    /* Remove any position: fixed styles */
    .feeder-ui, .bird-feeder-svg, .controls-and-status {
        position: static;
    }

    /* Ensure the page is scrollable */
    :global(body) {
        overflow-y: auto;
    }

    /* Scrollbar styles */
    .bird-list-container.scrollable::-webkit-scrollbar {
        width: 10px;
    }

    .bird-list-container.scrollable::-webkit-scrollbar-track {
        background: #2a2a2a;
    }

    .bird-list-container.scrollable::-webkit-scrollbar-thumb {
        background-color: #4CAF50;
        border-radius: 5px;
        border: 2px solid #2a2a2a;
    }

    .bird-list-container.scrollable::-webkit-scrollbar-thumb:hover {
        background-color: #45a049;
    }

    /* Add a style for the cleanliness bar if you want to visualize it */
    .cleanliness-bar-container {
        background-color: #444;
        height: 10px;
        border-radius: 5px;
        overflow: hidden;
        margin-top: 5px;
    }

    .cleanliness-bar {
        height: 100%;
        background-color: #4CAF50;
        border-radius: 5px;
        transition: width 0.3s ease;
    }

    /* Add a style for the battery level bar if you want to visualize it */
    .battery-bar-container {
        background-color: #444;
        height: 10px;
        border-radius: 5px;
        overflow: hidden;
        margin-top: 5px;
    }

    .battery-bar {
        height: 100%;
        background-color: #4CAF50;
        border-radius: 5px;
        transition: width 0.3s ease;
    }

    .more-info-button {
        margin-top: 10px;
        padding: 5px 10px;
        background-color: #4CAF50;
        color: white;
        border: none;
        border-radius: 4px;
        cursor: pointer;
    }

    .more-info-button:hover {
        background-color: #45a049;
    }

    .modal-backdrop {
        position: fixed;
        top: 0;
        left: 0;
        width: 100%;
        height: 100%;
        background-color: rgba(0, 0, 0, 0.5);
        display: flex;
        justify-content: center;
        align-items: center;
        z-index: 1000;
    }

    .modal-content {
        background-color: #2a2a2a;
        padding: 20px;
        border-radius: 10px;
        max-width: 80%;
        max-height: 80%;
        overflow-y: auto;
    }

    table {
        width: 100%;
        border-collapse: collapse;
        margin-bottom: 20px;
    }

    th, td {
        border: 1px solid #4CAF50;
        padding: 8px;
        text-align: left;
    }

    th {
        background-color: #4CAF50;
        color: white;
        cursor: pointer;
        user-select: none;
    }

    th:hover {
        background-color: #45a049;
    }
</style>