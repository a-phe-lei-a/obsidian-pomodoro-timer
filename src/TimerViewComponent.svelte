<script lang="ts">
import TasksComponent from 'TasksComponent.svelte'
import TimerSettingsComponent from 'TimerSettingsComponent.svelte'
import type Timer from 'Timer'
import type Tasks from 'Tasks'
import type TaskTracker from 'TaskTracker'

export let timer: Timer
export let tasks: Tasks
export let tracker: TaskTracker
export let render: (content: string, el: HTMLElement) => void

let extra: 'settings' | 'tasks' | 'close' = 'tasks'
const offset = 440

// Calculs réactifs
$: strokeOffset = $timer.remained.millis / $timer.count * offset
$: isBreakMode = $timer.mode === 'BREAK' || $timer.mode === 'LONG_BREAK'
$: cycleDisplay = $timer.longBreakInterval > 0 
    ? `${calculateCurrentCycle($timer.completedCycles, $timer.longBreakInterval, $timer.mode)}/${$timer.longBreakInterval}`
    : ''

// Fonction pour calculer correctement le cycle actuel
function calculateCurrentCycle(completedCycles: number, interval: number, mode: string): number {
    // Si on n'a pas encore de cycles complets, on est au premier cycle en mode WORK
    if (completedCycles === 0 && mode === 'WORK') return 1;
    
    // Si on est en mode BREAK après un cycle, on affiche le cycle qu'on vient de terminer
    if (mode === 'BREAK' || mode === 'LONG_BREAK') {
        // Si on est au dernier cycle (qui déclenche la pause longue), on affiche le numéro d'interval
        if (completedCycles % interval === 0) return interval;
        // Sinon, on affiche le numéro du cycle qu'on vient de terminer
        return completedCycles % interval;
    }
    
    // En mode WORK après au moins un cycle
    return (completedCycles % interval) + 1;
}

// Actions du minuteur
const toggleTimer = () => {
    $timer.running ? timer.pause() : timer.start()
}

const reset = () => {
    timer.reset()
}

const toggleMode = () => {
    timer.toggleMode()
}

// Gestion de l'interface
const toggleExtra = (value: 'settings' | 'tasks') => {
    extra = extra === value ? 'close' : value
}
</script>

<!-- svelte-ignore a11y-click-events-have-key-events -->
<!-- svelte-ignore a11y-no-static-element-interactions -->
<div class="container">
    <div class="main">
        <div class="timer">
            <div class="timer-display">
                <!-- Affichage du mode -->
                <div class="status control" on:click={toggleMode}>
                    <span class="mode">
                        {#if $timer.mode === 'WORK'}
                            Work
                        {:else if $timer.mode === 'LONG_BREAK'}
                            Long Break
                        {:else}
                            Break
                        {/if}
                    </span>
                </div>
                
                <!-- Affichage du temps restant -->
                <div on:click={toggleTimer} class="control">
                    <span class="timer-text">
                        {$timer.remained.human}
                    </span>
                    {#if $timer.longBreakInterval > 0}
                        <span class="cycle-counter">{cycleDisplay}</span>
                    {/if}
                </div>
            </div>
            
            <!-- Cercle d'animation -->
            <svg class="timer" width="160" height="160" xmlns="http://www.w3.org/2000/svg">
                <g>
                    <circle
                        class="circle_timer"
                        r="69.85699"
                        cy="81"
                        cx="81"
                        stroke-width="2"
                        fill="none"
                    />
                    <circle
                        class="circle_animation {isBreakMode ? 'break_mode' : ''} {$timer.running ? 'pulsing' : ''}"
                        r="69.85699"
                        cy="81"
                        cx="81"
                        stroke-width="8"
                        fill="none"
                        style="stroke-dashoffset: {strokeOffset}"
                    />
                </g>
            </svg>
        </div>
        
        <!-- Boutons de contrôle -->
        <div class="btn-group">
            <!-- Bouton des tâches -->
            <span on:click={() => toggleExtra('tasks')} class="control">
                <svg xmlns="http://www.w3.org/2000/svg" width="16" height="16" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="lucide lucide-list-todo">
                    <rect x="3" y="5" width="6" height="6" rx="1" />
                    <path d="m3 17 2 2 4-4" />
                    <path d="M13 6h8" />
                    <path d="M13 12h8" />
                    <path d="M13 18h8" />
                </svg>
            </span>

            <!-- Bouton play/pause -->
            {#if !$timer.running}
                <span on:click={toggleTimer} class="control">
                    <svg xmlns="http://www.w3.org/2000/svg" width="16" height="16" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="lucide lucide-play">
                        <polygon points="5 3 19 12 5 21 5 3" />
                    </svg>
                </span>
            {:else}
                <span on:click={toggleTimer} class="control">
                    <svg xmlns="http://www.w3.org/2000/svg" width="16" height="16" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="lucide lucide-pause">
                        <rect width="4" height="16" x="6" y="4" />
                        <rect width="4" height="16" x="14" y="4" />
                    </svg>
                </span>
            {/if}
            
            <!-- Bouton reset -->
            <span on:click={reset} class="control">
                <svg xmlns="http://www.w3.org/2000/svg" width="16" height="16" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="lucide lucide-rotate-ccw">
                    <path d="M3 12a9 9 0 1 0 9-9 9.75 9.75 0 0 0-6.74 2.74L3 8" />
                    <path d="M3 3v5h5" />
                </svg>
            </span>
            
            <!-- Bouton paramètres -->
            <span on:click={() => toggleExtra('settings')} class="control">
                <svg xmlns="http://www.w3.org/2000/svg" width="16" height="16" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="lucide lucide-settings-2">
                    <path d="M20 7h-9" />
                    <path d="M14 17H5" />
                    <circle cx="17" cy="17" r="3" />
                    <circle cx="7" cy="7" r="3" />
                </svg>
            </span>
        </div>
    </div>

    <!-- Panneau d'options supplémentaires -->
    <div class="pomodoro-extra">
        {#if extra == 'tasks'}
            <TasksComponent {tasks} {tracker} {render} />
        {:else if extra == 'settings'}
            <TimerSettingsComponent />
        {/if}
    </div>
</div>

<style>
/* Structure de base */
.container {
    width: 100%;
    min-width: 200px;
    display: flex;
    flex-direction: column;
    height: 100%;
}

.main {
    width: 100%;
    display: flex;
    flex-direction: column;
    align-items: center;
    z-index: 2;
}

/* Timer */
.timer {
    position: relative;
    width: 160px;
    height: 160px;
}

.timer svg {
    transform: rotate(-90deg);
    z-index: 3;
}

.timer-display {
    position: absolute;
    width: 100%;
    height: 160px;
    display: flex;
    flex-direction: column;
    align-items: center;
    justify-content: center;
    text-align: center;
    z-index: 4;
    padding: 30px;
}

/* Texte et labels */
.timer-text {
    display: block;
    color: var(--pomodoro-timer-text-color);
    font-size: 1.1em;
    font-weight: bold;
    margin-block-start: 1rem;
    margin-block-end: 1.75rem;
}

.status {
    font-size: 0.7rem;
    display: flex;
    align-items: center;
}

.status span {
    display: inline-block;
}

.cycle-counter {
    display: block;
    font-size: 0.6em;
    color: var(--text-muted);
    margin-top: -1.5rem;
    margin-bottom: 1.5rem;
}

/* Cercles du timer */
.circle_timer {
    stroke: var(--pomodoro-timer-color);
}

.circle_animation {
    stroke-dasharray: 440;
    stroke-dashoffset: 440;
    stroke: var(--pomodoro-timer-elapsed-color);
}

.circle_animation.break_mode {
    stroke: var(--pomodoro-timer-break-color, #e57373);
}

.circle_animation.pulsing {
    animation: pulse 2s ease-in-out infinite;
}

/* Animation de pulsation */
@keyframes pulse {
    0%, 100% {
        stroke-width: 8px;
        opacity: 1;
    }
    50% {
        stroke-width: 10px;
        opacity: 0.8;
    }
}

/* Groupe de boutons et contrôles */
.btn-group {
    margin-top: 1rem;
    display: flex;
    justify-content: space-between;
    width: 160px;
}

.control {
    cursor: pointer;
}

.control:hover {
    opacity: 0.7;
}

.control svg:active {
    opacity: 0.5;
}

.pomodoro-extra {
    width: 100%;
    margin-top: 2rem;
}
</style>
