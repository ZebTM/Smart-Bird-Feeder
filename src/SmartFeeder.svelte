<script>
    let foodLevel = 100;
    let birdDetected = false;
    let recording = false;

    // Functions to simulate behavior
    const simulateBirdDetection = () => {
        birdDetected = !birdDetected;
        recording = birdDetected; // Start recording if a bird is detected
    };

    const simulateFoodLevelChange = () => {
        foodLevel = foodLevel > 0 ? foodLevel - 10 : 0; // Decrease food level
    };
</script>

<div class="container">
    <!-- Project Info Section -->
    <div class="project-info">
        <h1>Smart Bird Feeder</h1>
        <p>By: Your Name</p>
        <a href="your-project-link" target="_blank">Project Write-Up</a>
        <button
            on:click={() =>
                alert("Info: This simulates bird detection and food levels.")}
        >
            Info
        </button>
    </div>

    <!-- Bird Feeder UI Section -->
    <div class="feeder-ui">
        <div class="bird-feeder-svg">
            <!-- Replace this with your SVG bird feeder -->
            <img src="src/assets/birdfeeder.svg" alt="Bird Feeder" />
        </div>

        <!-- Food Level Indicator -->
        <div class="food-level">
            <p>Food Level: {foodLevel}%</p>
            <div class="food-bar" style="width: {foodLevel}%;"></div>
        </div>

        <!-- Motion Sensor and Recording Status -->
        <div class="status">
            <p>Bird Detected: {birdDetected ? "Yes" : "No"}</p>
            <p>Recording: {recording ? "Yes" : "No"}</p>
        </div>

        <!-- Simulate Buttons -->
        <div class="controls">
            <button on:click={simulateBirdDetection}>
                {birdDetected ? "Stop Simulation" : "Simulate Bird Detection"}
            </button>
            <button on:click={simulateFoodLevelChange}
                >Simulate Food Level Change</button
            >
        </div>
    </div>
</div>

<style>
    @import "./app.css";
</style>
