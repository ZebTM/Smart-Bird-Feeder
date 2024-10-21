<script>
    import { spring } from 'svelte/motion';
    import { fade } from 'svelte/transition';

    export let foodLevel = 100;
    export let birdDetected = false;
    export let weatherCondition = 'Sunny';
    export let currentBird = null;
    export let isCleaning = false;
    export let isRecharging = false;

    let birdPosition = spring({ x: 150, y: 150 }, { stiffness: 0.1, damping: 0.25 });
    let raindrops = [];
    let foodParticles = [];
    let sparkles = []; // Initialize sparkles array

    // Use a spring for smooth animation of food level
    let animatedFoodLevel = spring(foodLevel, { stiffness: 0.1, damping: 0.7 });

    $: {
        animatedFoodLevel.set(foodLevel);
    }

    $: {
        if (birdDetected) {
            birdPosition.set({ x: 100, y: 200 });
        } else {
            birdPosition.set({ x: 150, y: 150 });
        }
    }

    $: {
        if (weatherCondition === 'Rainy') {
            raindrops = Array(10).fill().map((_, i) => ({
                x: 140 + i * 4,
                y: Math.random() * 40 + 60,
                speed: Math.random() * 2 + 1
            }));
        } else {
            raindrops = [];
        }
    }

    $: {
        if (isCleaning) {
            sparkles = Array(10).fill().map(() => ({
                id: Math.random(),
                x: Math.random() * 100 + 50,
                y: Math.random() * 200 + 50,
                size: Math.random() * 3 + 1,
                duration: Math.random() * 1000 + 500
            }));
            setTimeout(() => {
                sparkles = [];
            }, 1500);
        }
    }

    $: {
        if (isRecharging) {
            // Remove the setTimeout that sets isRecharging to false
            // The parent component will handle this now
        }
    }

    export function createFoodParticles() {
        const newParticles = Array(8).fill().map(() => ({
            id: Math.random(),
            x: Math.random() * 60 + 70,
            y: 250 - $animatedFoodLevel * 1.8,
            size: Math.random() * 2 + 1,
        }));
        foodParticles = [...foodParticles, ...newParticles];
        setTimeout(() => {
            foodParticles = foodParticles.filter(p => !newParticles.includes(p));
        }, 500);
    }

    function getBirdShape(bird) {
        if (!bird) return '';

        const birdName = typeof bird === 'string' ? bird : bird.name;

        switch (birdName) {
            case 'Sparrow':
                return 'M-10,0 Q0,-15 10,0 Q0,15 -10,0 Z M12,-2 Q17,-7 22,-2 Q17,3 12,-2 Z';
            case 'Robin':
                return 'M-10,0 Q0,-20 10,0 T-10,0 Z M12,-5 Q17,-10 22,-5 Q17,0 12,-5 Z';
            case 'Blue Jay':
                return 'M-15,0 Q0,-25 15,0 Q0,15 -15,0 Z M17,-7 Q22,-12 27,-7 Q22,-2 17,-7 Z';
            case 'Cardinal':
                return 'M-12,0 Q0,-22 12,0 Q6,10 0,5 Q-6,10 -12,0 Z M14,-7 Q19,-12 24,-7 Q19,-2 14,-7 Z';
            case 'Finch':
                return 'M-8,0 Q0,-12 8,0 Q4,8 0,4 Q-4,8 -8,0 Z M10,-3 Q15,-8 20,-3 Q15,2 10,-3 Z';
            default:
                return 'M-10,0 Q0,-15 10,0 Q0,15 -10,0 Z M12,-2 Q17,-7 22,-2 Q17,3 12,-2 Z';
        }
    }

    function getBirdColor(bird) {
        if (!bird) return '#000000';

        if (typeof bird === 'string') {
            // Default colors for backwards compatibility
            const defaultColors = {
                'Sparrow': '#A52A2A',
                'Robin': '#FF6347',
                'Blue Jay': '#4169E1',
                'Cardinal': '#DC143C',
                'Finch': '#DAA520'  // Changed from '#FFD700' to '#DAA520'
            };
            return defaultColors[bird] || '#000000';
        }

        return bird.color;
    }
</script>

<svg viewBox="0 0 200 300" xmlns="http://www.w3.org/2000/svg">
    <!-- Sky background -->
    <rect x="0" y="0" width="200" height="300" fill={weatherCondition === 'Sunny' ? '#87CEEB' : '#708090'} />

    <!-- Sun or cloud -->
    {#if weatherCondition === 'Sunny'}
        <circle cx="160" cy="40" r="20" fill="#FFD700">
            <animate attributeName="r" values="18;22;18" dur="3s" repeatCount="indefinite" />
        </circle>
    {:else if weatherCondition === 'Cloudy' || weatherCondition === 'Rainy'}
        <g transform="translate(160, 40)">
            <animateTransform attributeName="transform" type="translate" values="160,40;165,40;160,40" dur="5s" repeatCount="indefinite" />
            <ellipse cx="0" cy="0" rx="30" ry="20" fill="#FFFFFF" />
        </g>
    {/if}

    <!-- Rain (if applicable) -->
    {#if weatherCondition === 'Rainy'}
        {#each raindrops as raindrop, i}
            <line 
                x1={raindrop.x} 
                y1={raindrop.y} 
                x2={raindrop.x - 5} 
                y2={raindrop.y + 20} 
                stroke="#4169E1" 
                stroke-width="2"
                opacity="0.7"
            >
                <animate 
                    attributeName="y1" 
                    from={raindrop.y} 
                    to={raindrop.y + 240} 
                    dur={`${raindrop.speed}s`} 
                    repeatCount="indefinite" 
                />
                <animate 
                    attributeName="y2" 
                    from={raindrop.y + 20} 
                    to={raindrop.y + 260} 
                    dur={`${raindrop.speed}s`} 
                    repeatCount="indefinite" 
                />
            </line>
        {/each}
    {/if}

    <!-- Feeder body -->
    <rect x="50" y="50" width="100" height="200" fill="#8B4513" />
    
    <!-- Feeder roof -->
    <polygon points="25,50 100,10 175,50" fill="#A52A2A" />
    

    <g transform="translate(113, 72) scale(-0.5)">
        <svg xmlns="http://www.w3.org/2000/svg" fill="#000000" width="50px" height="50px" viewBox="0 0 24 24">
            <path d="M6.34999 6.34893L4.92999 4.92893C6.80527 3.05422 9.34835 2.00107 12 2.00107C14.6516 2.00107 17.1947 3.05422 19.07 4.92893L17.65 6.34893C16.1503 4.85283 14.1184 4.01263 12 4.01263C9.88162 4.01263 7.84972 4.85283 6.34999 6.34893Z"/>
            <path d="M9.17001 9.16893L7.76001 7.75893C8.88501 6.63533 10.41 6.00421 12 6.00421C13.59 6.00421 15.115 6.63533 16.24 7.75893L14.82 9.17893C14.0733 8.42776 13.0592 8.0034 12 7.99893C11.4746 7.99852 10.9542 8.10163 10.4686 8.30239C9.98303 8.50314 9.54176 8.79759 9.17001 9.16893Z"/>
            <path fill-rule="evenodd" clip-rule="evenodd" d="M18 10.9989H14C14 11.5294 13.7893 12.0381 13.4142 12.4131C13.0391 12.7882 12.5304 12.9989 12 12.9989C11.4696 12.9989 10.9609 12.7882 10.5858 12.4131C10.2107 12.0381 10 11.5294 10 10.9989H6C5.46957 10.9989 4.96086 11.2096 4.58579 11.5847C4.21071 11.9598 4 12.4685 4 12.9989V21.9989H20V12.9989C20 12.4685 19.7893 11.9598 19.4142 11.5847C19.0391 11.2096 18.5304 10.9989 18 10.9989ZM6 17.9989V14.9989H18V17.9989H6Z"/>
            <path d="M13 10.9989C13 11.5512 12.5523 11.9989 12 11.9989C11.4477 11.9989 11 11.5512 11 10.9989C11 10.4466 11.4477 9.99893 12 9.99893C12.5523 9.99893 13 10.4466 13 10.9989Z"/>
        </svg>
    </g>

    <!-- Food level -->
    <rect 
        x="60" 
        y={250 - $animatedFoodLevel * 1.8} 
        width="80" 
        height={$animatedFoodLevel * 1.8} 
        fill="#FFD700" 
    />
    
    <!-- Falling food particles -->
    {#each foodParticles as particle (particle.id)}
        <circle
            cx={particle.x}
            cy={particle.y}
            r={particle.size}
            fill="#FFD700"
            in:fade="{{ duration: 100 }}"
            out:fade="{{ duration: 400 }}"
        />
    {/each}
    
    <!-- Perch -->
    <rect x="30" y="220" width="140" height="10" fill="#8B4513" />
    
    <!-- Bird (if detected) -->
    {#if birdDetected && currentBird}
        <g transform="translate({$birdPosition.x}, {$birdPosition.y}) scale(0.8)" in:fade="{{ duration: 300 }}" out:fade="{{ duration: 300 }}">
            <path d={getBirdShape(currentBird)} fill={getBirdColor(currentBird)} />
            <circle cx="5" cy="-5" r="2" fill="black" />
        </g>
    {/if}

    <!-- Cleaning sparkles -->
    {#each sparkles as sparkle (sparkle.id)}
        <circle
            cx={sparkle.x}
            cy={sparkle.y}
            r={sparkle.size}
            fill="#FFFFFF"
            opacity="0.8"
            in:fade="{{ duration: sparkle.duration }}"
            out:fade="{{ duration: sparkle.duration }}"
        >
            <animate
                attributeName="r"
                values="{sparkle.size};{sparkle.size * 2};{sparkle.size}"
                dur="{sparkle.duration}ms"
                repeatCount="indefinite"
            />
        </circle>
    {/each}

    <!-- Recharging animation -->
    {#if isRecharging}
        <g transform="translate(150, 30) scale(0.5)">
            <rect x="0" y="0" width="60" height="30" rx="3" ry="3" fill="#4CAF50" />
            <rect x="60" y="10" width="6" height="10" fill="#4CAF50" />
            <path d="M20,8 L40,22 H30 L35,8 H25 L20,22 H30 Z" fill="#FFD700">
                <animate attributeName="opacity" values="1;0;1" dur="1s" repeatCount="indefinite" />
            </path>
        </g>
    {/if}
</svg>

<style>
    svg {
        width: 100%;
        height: auto;
    }
</style>
