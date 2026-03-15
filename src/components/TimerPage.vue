<template>
  <div class="timer-page" :class="[backgroundColor, { 'fullscreen-mode': isFullscreen }]">
    <header class="header">
      <h1>🎤 Toastmaster Timer</h1>
      <div class="header-actions">
        <button class="btn btn-secondary" @click="toggleFullscreen">
          {{ isFullscreen ? '❌ Exit Fullscreen' : '🖥️ Fullscreen' }}
        </button>
        <button class="btn btn-secondary" @click="showConfigPresets = true">
          📋 Presets
        </button>
      </div>
    </header>

    <main class="main-content">
      <div class="top-bar">
        <div class="speaker-input">
          <input
            type="text"
            v-model="speakerName"
            placeholder="Enter speaker name..."
            class="speaker-name-input"
            autocomplete="off"
            autocorrect="off"
            autocapitalize="off"
            spellcheck="false"
          />
        </div>
        <div class="secondary-times">
          <div class="time-panel time-green">
            <span class="time-label">Green Time</span>
            <span class="time-text time-colored">{{ config.greenTime }}s</span>
          </div>
          <div class="time-panel time-yellow">
            <span class="time-label">Yellow Time</span>
            <span class="time-text time-colored">{{ config.yellowTime }}s</span>
          </div>
          <div class="time-panel time-red">
            <span class="time-label">Red Time</span>
            <span class="time-text time-colored">{{ config.redTime }}s</span>
          </div>
          <div class="time-panel">
            <span class="time-label">Total Time</span>
            <span class="time-text time-secondary">{{ formattedTotalTime }}</span>
          </div>
          <div class="time-panel">
            <span class="time-label">Elapsed Time</span>
            <span class="time-text time-secondary">{{ formattedTime }}</span>
          </div>
        </div>
      </div>

      <div class="timer-display">
        <div class="time-panel time-primary">
          <span class="time-label">Remaining Time</span>
          <span class="time-text time-remaining" :class="{ 'time-overtime': isOvertime }">{{ formattedRemainingTime }}</span>
        </div>
      </div>

      <div class="controls">
        <button class="btn btn-large" :class="isRunning ? 'btn-warning' : 'btn-success'" @click="toggleTimer">
          {{ isRunning ? '⏸️ Pause' : '▶️ Start' }}
        </button>
        <button class="btn btn-large btn-secondary" @click="resetTimer">
          🔄 Reset
        </button>
        <button class="btn btn-large btn-info" @click="saveRecord" :disabled="!canSaveRecord">
          💾 Save Record
        </button>
      </div>
    </main>

    <footer class="footer">
      <button class="btn btn-secondary" @click="showRecords = true">
        📋 View Records
      </button>
    </footer>

    <!-- Config Modal -->
    <div v-if="showConfig" class="modal-overlay" @click.self="showConfig = false">
      <div class="modal">
        <h2>⏱️ Timer Settings</h2>
        <div class="form-group">
          <label>Total Time (seconds)</label>
          <input type="number" v-model.number="config.totalTime" min="0" />
        </div>
        <div class="form-group">
          <label>Green Time (seconds)</label>
          <input type="number" v-model.number="config.greenTime" min="0" />
        </div>
        <div class="form-group">
          <label>Yellow Time (seconds)</label>
          <input type="number" v-model.number="config.yellowTime" min="0" />
        </div>
        <div class="form-group">
          <label>Red Time (seconds)</label>
          <input type="number" v-model.number="config.redTime" min="0" />
        </div>
        <div class="config-actions">
          <button class="btn btn-info" @click="showConfigPresets = true">
            📋 Saved Presets
          </button>
          <button class="btn btn-primary" @click="showSavePreset = true">
            💾 Save Preset
          </button>
        </div>
        <div class="modal-actions">
          <button class="btn btn-secondary" @click="showConfig = false">Cancel</button>
          <button class="btn btn-primary" @click="saveConfig">Apply Settings</button>
        </div>
      </div>
    </div>

    <!-- Save Preset Modal -->
    <div v-if="showSavePreset" class="modal-overlay" @click.self="showSavePreset = false">
      <div class="modal">
        <h2>💾 Add New Configuration Preset</h2>
        <div class="form-group">
          <label>Preset Name</label>
          <input 
            type="text" 
            v-model="presetName" 
            placeholder="e.g., 5-min Speech, Table Topics..." 
            class="preset-name-input"
          />
        </div>
        <div class="form-group">
          <label>Total Time (seconds)</label>
          <input type="number" v-model.number="newPreset.totalTime" min="0" />
        </div>
        <div class="form-group">
          <label>Green Time (seconds)</label>
          <input type="number" v-model.number="newPreset.greenTime" min="0" />
        </div>
        <div class="form-group">
          <label>Yellow Time (seconds)</label>
          <input type="number" v-model.number="newPreset.yellowTime" min="0" />
        </div>
        <div class="form-group">
          <label>Red Time (seconds)</label>
          <input type="number" v-model.number="newPreset.redTime" min="0" />
        </div>
        <div class="modal-actions">
          <button class="btn btn-secondary" @click="showSavePreset = false">Cancel</button>
          <button class="btn btn-primary" @click="savePreset" :disabled="!presetName.trim()">Save Preset</button>
        </div>
      </div>
    </div>

    <!-- Config Presets Modal -->
    <div v-if="showConfigPresets" class="modal-overlay" @click.self="showConfigPresets = false">
      <div class="modal modal-large">
        <h2>📋 Saved Configuration Presets</h2>
        <div class="presets-table-container">
          <table class="presets-table">
            <thead>
              <tr>
                <th>Name</th>
                <th>Total</th>
                <th>Green</th>
                <th>Yellow</th>
                <th>Red</th>
                <th>Actions</th>
              </tr>
            </thead>
            <tbody>
              <tr v-for="(preset, index) in configPresets" :key="preset.id">
                <td>{{ preset.name }}</td>
                <td>{{ preset.totalTime }}s</td>
                <td>{{ preset.greenTime }}s</td>
                <td>{{ preset.yellowTime }}s</td>
                <td>{{ preset.redTime }}s</td>
                <td>
                  <button class="btn btn-small btn-info" @click="editPreset(preset)" title="Edit">
                    ✏️
                  </button>
                  <button class="btn btn-small btn-success" @click="loadPreset(preset)" title="Load">
                    📥
                  </button>
                  <button class="btn btn-small btn-danger" @click="deletePreset(preset.id)" title="Delete">
                    🗑️
                  </button>
                </td>
              </tr>
              <tr v-if="configPresets.length === 0">
                <td colspan="6" class="no-records">No saved presets</td>
              </tr>
            </tbody>
          </table>
        </div>
        <div class="modal-actions">
          <button class="btn btn-primary" @click="openAddPreset">➕ Add New Preset</button>
          <button class="btn btn-secondary" @click="showConfigPresets = false">Close</button>
        </div>
      </div>
    </div>

    <!-- Edit Preset Modal -->
    <div v-if="showEditPreset" class="modal-overlay" @click.self="showEditPreset = false">
      <div class="modal">
        <h2>✏️ Edit Configuration Preset</h2>
        <div class="form-group">
          <label>Preset Name</label>
          <input type="text" v-model="editingPreset.name" class="preset-name-input" />
        </div>
        <div class="form-group">
          <label>Total Time (seconds)</label>
          <input type="number" v-model.number="editingPreset.totalTime" min="0" />
        </div>
        <div class="form-group">
          <label>Green Time (seconds)</label>
          <input type="number" v-model.number="editingPreset.greenTime" min="0" />
        </div>
        <div class="form-group">
          <label>Yellow Time (seconds)</label>
          <input type="number" v-model.number="editingPreset.yellowTime" min="0" />
        </div>
        <div class="form-group">
          <label>Red Time (seconds)</label>
          <input type="number" v-model.number="editingPreset.redTime" min="0" />
        </div>
        <div class="modal-actions">
          <button class="btn btn-secondary" @click="showEditPreset = false">Cancel</button>
          <button class="btn btn-primary" @click="updatePreset">Update Preset</button>
        </div>
      </div>
    </div>

    <!-- Records Modal -->
    <div v-if="showRecords" class="modal-overlay" @click.self="showRecords = false">
      <div class="modal modal-large">
        <h2>📋 Speech Records</h2>
        <div class="records-table-container">
          <table class="records-table">
            <thead>
              <tr>
                <th>#</th>
                <th>Date</th>
                <th>Speaker</th>
                <th>Duration</th>
                <th>Total Time</th>
                <th>Status</th>
                <th>Actions</th>
              </tr>
            </thead>
            <tbody>
              <tr v-for="(record, index) in records" :key="record.id">
                <td>{{ index + 1 }}</td>
                <td>{{ formatDate(record.date) }}</td>
                <td>{{ record.speaker || 'N/A' }}</td>
                <td>{{ formatDuration(record.duration) }}</td>
                <td>{{ formatDuration(record.totalTime) }}</td>
                <td>
                  <span class="status-badge" :class="getStatusClass(record.duration)">
                    {{ getStatusText(record.duration) }}
                  </span>
                </td>
                <td>
                  <button class="btn btn-small btn-danger" @click="deleteRecord(record.id)">
                    🗑️
                  </button>
                </td>
              </tr>
              <tr v-if="records.length === 0">
                <td colspan="7" class="no-records">No records yet</td>
              </tr>
            </tbody>
          </table>
        </div>
        <div class="modal-actions">
          <button class="btn btn-danger" @click="confirmClear = true" :disabled="records.length === 0">
            Clear All
          </button>
          <button class="btn btn-secondary" @click="showRecords = false">Close</button>
        </div>
      </div>
    </div>

    <!-- Clear Confirmation Modal -->
    <div v-if="confirmClear" class="modal-overlay" @click.self="confirmClear = false">
      <div class="modal">
        <h2>⚠️ Confirm Clear</h2>
        <p>Are you sure you want to clear all records? This action cannot be undone.</p>
        <div class="modal-actions">
          <button class="btn btn-secondary" @click="confirmClear = false">Cancel</button>
          <button class="btn btn-danger" @click="clearAllRecords">Clear All Records</button>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, computed, onMounted, onUnmounted } from 'vue'

// Timer state
const isRunning = ref(false)
const elapsedTime = ref(0)
const timerInterval = ref(null)
const startTime = ref(0)
const savedElapsed = ref(0)
const speakerName = ref('')

// Fullscreen state
const isFullscreen = ref(false)

// Config state
const showConfig = ref(false)
const showConfigPresets = ref(false)
const showEditPreset = ref(false)
const showSavePreset = ref(false)
const presetName = ref('')
const editingPreset = ref({ id: null, name: '', totalTime: 0, greenTime: 0, yellowTime: 0, redTime: 0 })
const newPreset = ref({ totalTime: 120, greenTime: 60, yellowTime: 30, redTime: 30 })
const showRecords = ref(false)
const config = ref({
  totalTime: 120,
  greenTime: 60,
  yellowTime: 30,
  redTime: 30
})

// Config presets state
const configPresets = ref([])

// Records state
const records = ref([])
const confirmClear = ref(false)

// Load data from localStorage
onMounted(() => {
  const savedConfig = localStorage.getItem('timerConfig')
  if (savedConfig) {
    config.value = JSON.parse(savedConfig)
  }
  const savedRecords = localStorage.getItem('speechRecords')
  if (savedRecords) {
    records.value = JSON.parse(savedRecords)
  }
  const savedPresets = localStorage.getItem('configPresets')
  if (savedPresets) {
    configPresets.value = JSON.parse(savedPresets)
  }
  // Start the animation frame loop
  requestAnimationFrame(updateTimer)

  // Listen for fullscreen change events
  document.addEventListener('fullscreenchange', handleFullscreenChange)
  // Prevent backspace from navigating back in Safari
  document.addEventListener('keydown', handleKeydown, { capture: true })
})

onUnmounted(() => {
  document.removeEventListener('fullscreenchange', handleFullscreenChange)
  document.removeEventListener('keydown', handleKeydown, { capture: true })
  if (timerInterval.value) {
    clearInterval(timerInterval.value)
  }
})

// Fullscreen toggle
const toggleFullscreen = async () => {
  try {
    if (!document.fullscreenElement) {
      await document.documentElement.requestFullscreen()
    } else {
      await document.exitFullscreen()
    }
  } catch (err) {
    console.error('Fullscreen error:', err)
  }
}

const handleFullscreenChange = () => {
  isFullscreen.value = !!document.fullscreenElement
}

// Prevent backspace/delete from navigating back in Safari
const handleKeydown = (e) => {
  if ((e.key === 'Backspace' || e.key === 'Delete') && isFullscreen.value) {
    const tagName = document.activeElement?.tagName
    if (tagName !== 'INPUT' && tagName !== 'TEXTAREA') {
      e.preventDefault()
      return false
    }
  }
}

// Save config to localStorage
const saveConfig = () => {
  localStorage.setItem('timerConfig', JSON.stringify(config.value))
  showConfig.value = false
}

// Timer logic
const toggleTimer = () => {
  if (isRunning.value) {
    pauseTimer()
  } else {
    startTimer()
  }
}

const startTimer = () => {
  isRunning.value = true
  startTime.value = Date.now()
}

const pauseTimer = () => {
  isRunning.value = false
  savedElapsed.value = elapsedTime.value
}

const resetTimer = () => {
  pauseTimer()
  elapsedTime.value = 0
  savedElapsed.value = 0
}

// Update timer using requestAnimationFrame for smooth updates
const updateTimer = () => {
  if (isRunning.value) {
    const delta = Math.floor((Date.now() - startTime.value) / 1000)
    elapsedTime.value = savedElapsed.value + delta
  }
  requestAnimationFrame(updateTimer)
}

// Computed: Formatted total time display
const formattedTotalTime = computed(() => {
  const mins = Math.floor(config.value.totalTime / 60)
  const secs = config.value.totalTime % 60
  return `${mins.toString().padStart(2, '0')}:${secs.toString().padStart(2, '0')}`
})

// Computed: Formatted elapsed time display
const formattedTime = computed(() => {
  const mins = Math.floor(elapsedTime.value / 60)
  const secs = elapsedTime.value % 60
  return `${mins.toString().padStart(2, '0')}:${secs.toString().padStart(2, '0')}`
})

// Computed: Formatted remaining time display
const formattedRemainingTime = computed(() => {
  const remaining = Math.max(0, config.value.totalTime - elapsedTime.value)
  const mins = Math.floor(remaining / 60)
  const secs = remaining % 60
  return `${mins.toString().padStart(2, '0')}:${secs.toString().padStart(2, '0')}`
})

// Computed: Is overtime
const isOvertime = computed(() => {
  return elapsedTime.value >= config.value.totalTime
})

// Computed: Background color based on remaining time
const backgroundColor = computed(() => {
  // Show white when timer is stopped/reset (at starting position)
  if (!isRunning.value && elapsedTime.value === 0) {
    return 'bg-white'
  }

  const remaining = config.value.totalTime - elapsedTime.value

  if (remaining > config.value.greenTime) {
    return 'bg-white'
  } else if (remaining > config.value.yellowTime) {
    return 'bg-green'
  } else if (remaining > 0) {
    return 'bg-yellow'
  } else {
    return 'bg-red'
  }
})

// Computed: Can save record
const canSaveRecord = computed(() => {
  return elapsedTime.value > 0
})

// Save record
const saveRecord = () => {
  if (!canSaveRecord.value) return

  const newRecord = {
    id: Date.now(),
    date: new Date().toISOString(),
    speaker: speakerName.value.trim() || 'Unknown',
    duration: elapsedTime.value,
    totalTime: config.value.totalTime
  }

  records.value.unshift(newRecord)
  localStorage.setItem('speechRecords', JSON.stringify(records.value))
  speakerName.value = ''
  resetTimer()
}

// Delete record
const deleteRecord = (id) => {
  records.value = records.value.filter(r => r.id !== id)
  localStorage.setItem('speechRecords', JSON.stringify(records.value))
}

// Clear all records
const clearAllRecords = () => {
  records.value = []
  localStorage.setItem('speechRecords', JSON.stringify([]))
  confirmClear.value = false
}

// Save configuration preset
const savePreset = () => {
  if (!presetName.value.trim()) return

  const newPresetItem = {
    id: Date.now(),
    name: presetName.value.trim(),
    totalTime: newPreset.value.totalTime,
    greenTime: newPreset.value.greenTime,
    yellowTime: newPreset.value.yellowTime,
    redTime: newPreset.value.redTime
  }

  configPresets.value.unshift(newPresetItem)
  localStorage.setItem('configPresets', JSON.stringify(configPresets.value))
  presetName.value = ''
  newPreset.value = { totalTime: 120, greenTime: 60, yellowTime: 30, redTime: 30 }
  showSavePreset.value = false
}

// Open add preset modal
const openAddPreset = () => {
  presetName.value = ''
  newPreset.value = { totalTime: 120, greenTime: 60, yellowTime: 30, redTime: 30 }
  showSavePreset.value = true
  showConfigPresets.value = false
}

// Load configuration preset
const loadPreset = (preset) => {
  // Reset timer when loading a new preset
  pauseTimer()
  elapsedTime.value = 0
  savedElapsed.value = 0

  config.value = {
    totalTime: preset.totalTime,
    greenTime: preset.greenTime,
    yellowTime: preset.yellowTime,
    redTime: preset.redTime
  }
  showConfigPresets.value = false
  showConfig.value = false
}

// Edit configuration preset
const editPreset = (preset) => {
  editingPreset.value = { ...preset }
  showEditPreset.value = true
}

// Update configuration preset
const updatePreset = () => {
  const index = configPresets.value.findIndex(p => p.id === editingPreset.value.id)
  if (index !== -1) {
    configPresets.value[index] = { ...editingPreset.value }
    localStorage.setItem('configPresets', JSON.stringify(configPresets.value))
    showEditPreset.value = false
  }
}

// Delete configuration preset
const deletePreset = (id) => {
  configPresets.value = configPresets.value.filter(p => p.id !== id)
  localStorage.setItem('configPresets', JSON.stringify(configPresets.value))
}

// Format date
const formatDate = (dateString) => {
  const date = new Date(dateString)
  return date.toLocaleString()
}

// Format duration
const formatDuration = (seconds) => {
  const mins = Math.floor(seconds / 60)
  const secs = seconds % 60
  return `${mins}m ${secs}s`
}

// Get status class
const getStatusClass = (duration) => {
  const remaining = config.value.totalTime - duration

  if (remaining > config.value.greenTime) {
    return 'status-white'
  } else if (remaining > config.value.yellowTime) {
    return 'status-green'
  } else if (remaining > 0) {
    return 'status-yellow'
  } else {
    return 'status-red'
  }
}

// Get status text
const getStatusText = (duration) => {
  const remaining = config.value.totalTime - duration

  if (remaining > config.value.greenTime) {
    return 'On Track'
  } else if (remaining > config.value.yellowTime) {
    return 'Within Time'
  } else if (remaining > 0) {
    return 'Overtime (Yellow)'
  } else {
    return 'Too Long (Red)'
  }
}
</script>

<style>
.timer-page {
  height: 100vh;
  width: 100%;
  display: flex;
  flex-direction: column;
  transition: background-color 0.3s ease;
}

.timer-page.fullscreen-mode .main-content {
  justify-content: space-between;
}

.timer-page.fullscreen-mode .time-panel.time-primary {
  flex: 1;
  max-width: none;
  padding: 2rem 4rem;
  background: transparent;
}

.timer-page.fullscreen-mode .time-remaining {
  font-size: 15vw !important;
}

.timer-page.fullscreen-mode .time-label {
  font-size: 1rem;
}

.timer-page.fullscreen-mode .time-text {
  font-size: 1.25rem;
}

.timer-page.fullscreen-mode .time-colored {
  font-size: 1rem;
}

.timer-page.fullscreen-mode .time-secondary {
  font-size: 1.25rem;
}

.bg-white {
  background-color: #6b7280;
}

.bg-green {
  background-color: #22c55e;
}

.bg-yellow {
  background-color: #eab308;
}

.bg-red {
  background-color: #ef4444;
}

.header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 1rem 2rem;
  background: rgba(0, 0, 0, 0.1);
}

.header-actions {
  display: flex;
  gap: 0.5rem;
  align-items: center;
}

.header h1 {
  color: white;
  font-size: 1.5rem;
  text-shadow: 0 2px 4px rgba(0, 0, 0, 0.2);
  margin: 0;
}

.main-content {
  flex: 1;
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
  gap: 2rem;
  padding: 2rem;
  width: 100%;
  max-width: 1400px;
  margin: 0 auto;
}

.top-bar {
  display: flex;
  justify-content: space-between;
  align-items: center;
  width: 100%;
  gap: 2rem;
}

.speaker-input {
  flex: 1;
  max-width: 400px;
}

.speaker-name-input {
  width: 100%;
  padding: 0.75rem 1.25rem;
  border: 2px solid rgba(255, 255, 255, 0.3);
  border-radius: 12px;
  font-size: 1rem;
  background: rgba(255, 255, 255, 0.15);
  color: white;
  backdrop-filter: blur(10px);
  transition: all 0.2s ease;
  box-sizing: border-box;
}

.speaker-name-input::placeholder {
  color: rgba(255, 255, 255, 0.6);
}

.speaker-name-input:focus {
  outline: none;
  border-color: rgba(255, 255, 255, 0.8);
  background: rgba(255, 255, 255, 0.25);
}

.speaker-name-input:hover {
  background: rgba(255, 255, 255, 0.2);
}

.secondary-times {
  display: flex;
  gap: 1.5rem;
  align-items: center;
  justify-content: flex-end;
}

.time-panel {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 0.5rem;
  padding: 0.75rem 1.25rem;
  border-radius: 12px;
  background: rgba(255, 255, 255, 0.1);
}

.time-panel.time-green {
  background: rgba(34, 197, 94, 0.3);
}

.time-panel.time-yellow {
  background: rgba(234, 179, 8, 0.3);
}

.time-panel.time-red {
  background: rgba(239, 68, 68, 0.3);
}

.time-panel.time-primary {
  background: rgba(255, 255, 255, 0.15);
  border-radius: 16px;
  padding: 3rem 6rem;
}

.timer-page.fullscreen-mode .time-panel.time-primary {
  max-width: none;
  padding: 2rem 4rem;
}

.timer-display {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  width: 100%;
}

.time-label {
  font-size: 0.75rem;
  color: rgba(255, 255, 255, 0.7);
  text-transform: uppercase;
  letter-spacing: 1px;
  margin: 0;
}

.time-text {
  font-size: 2rem;
  font-weight: bold;
  color: white;
  text-shadow: 0 4px 8px rgba(0, 0, 0, 0.3);
  font-variant-numeric: tabular-nums;
  margin: 0;
  line-height: 1;
}

.time-colored {
  font-size: 1.5rem;
  font-weight: 600;
}

.time-secondary {
  font-size: 2rem;
  opacity: 0.7;
  font-weight: 500;
}

.time-remaining {
  font-size: 10rem;
  font-weight: bold;
  line-height: 1;
}

.time-overtime {
  color: #fef08a;
  animation: pulse 1s infinite;
}

@keyframes pulse {
  0%, 100% { opacity: 1; }
  50% { opacity: 0.5; }
}

.controls {
  display: flex;
  gap: 1rem;
  flex-wrap: wrap;
  justify-content: center;
}

.footer {
  padding: 1rem 2rem;
  background: rgba(0, 0, 0, 0.1);
  display: flex;
  justify-content: center;
}

.btn {
  padding: 0.75rem 1.5rem;
  border: none;
  border-radius: 8px;
  font-size: 1rem;
  font-weight: 600;
  cursor: pointer;
  transition: all 0.2s ease;
  display: inline-flex;
  align-items: center;
  gap: 0.5rem;
  white-space: nowrap;
}

.btn:hover:not(:disabled) {
  transform: translateY(-2px);
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.2);
}

.btn:disabled {
  opacity: 0.5;
  cursor: not-allowed;
}

.btn-large {
  padding: 1rem 2rem;
  font-size: 1.1rem;
}

.btn-success {
  background-color: #16a34a;
  color: white;
}

.btn-warning {
  background-color: #ca8a04;
  color: white;
}

.btn-secondary {
  background-color: rgba(255, 255, 255, 0.3);
  color: white;
}

.btn-primary {
  background-color: #3b82f6;
  color: white;
}

.btn-info {
  background-color: #0ea5e9;
  color: white;
}

.btn-danger {
  background-color: #dc2626;
  color: white;
}

.btn-small {
  padding: 0.4rem 0.8rem;
  font-size: 0.875rem;
}

.modal-overlay {
  position: fixed;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background: rgba(0, 0, 0, 0.6);
  display: flex;
  justify-content: center;
  align-items: center;
  z-index: 1000;
  padding: 1rem;
}

.modal {
  background: white;
  border-radius: 16px;
  padding: 2rem;
  max-width: 400px;
  width: 100%;
  box-shadow: 0 20px 60px rgba(0, 0, 0, 0.3);
  max-height: 90vh;
  overflow-y: auto;
}

.modal-large {
  max-width: 800px;
}

.modal h2 {
  margin: 0 0 1.5rem 0;
  color: #1f2937;
  font-size: 1.5rem;
}

.form-group {
  margin-bottom: 1rem;
}

.form-group label {
  display: block;
  margin-bottom: 0.5rem;
  color: #4b5563;
  font-weight: 500;
  font-size: 0.875rem;
}

.form-group input {
  width: 100%;
  padding: 0.75rem;
  border: 2px solid #e5e7eb;
  border-radius: 8px;
  font-size: 1rem;
  box-sizing: border-box;
}

.form-group input:focus {
  outline: none;
  border-color: #3b82f6;
}

.modal-actions {
  display: flex;
  gap: 0.5rem;
  justify-content: flex-end;
  margin-top: 1.5rem;
  flex-wrap: wrap;
}

.config-actions {
  display: flex;
  gap: 0.5rem;
  justify-content: space-between;
  margin-top: 1rem;
  margin-bottom: 1rem;
  flex-wrap: wrap;
}

.preset-preview {
  background: #f3f4f6;
  padding: 1rem;
  border-radius: 8px;
  margin-bottom: 1rem;
}

.preset-preview p {
  margin: 0.25rem 0;
  color: #4b5563;
  font-size: 0.875rem;
}

.preset-name-input {
  width: 100%;
  padding: 0.75rem;
  border: 2px solid #e5e7eb;
  border-radius: 8px;
  font-size: 1rem;
  box-sizing: border-box;
}

.preset-name-input:focus {
  outline: none;
  border-color: #3b82f6;
}

.records-table-container,
.presets-table-container {
  max-height: 400px;
  overflow-y: auto;
  margin-bottom: 1rem;
}

.records-table,
.presets-table {
  width: 100%;
  border-collapse: collapse;
  font-size: 0.875rem;
}

.records-table th,
.records-table td,
.presets-table th,
.presets-table td {
  padding: 0.75rem;
  text-align: left;
  border-bottom: 1px solid #e5e7eb;
}

.records-table th,
.presets-table th {
  background: #f9fafb;
  font-weight: 600;
  color: #374151;
  position: sticky;
  top: 0;
  z-index: 1;
}

.records-table tr:hover,
.presets-table tr:hover {
  background: #f9fafb;
}

.no-records {
  text-align: center;
  color: #9ca3af;
  padding: 2rem;
}

.status-badge {
  padding: 0.25rem 0.75rem;
  border-radius: 9999px;
  font-size: 0.75rem;
  font-weight: 600;
  display: inline-block;
}

.status-white {
  background: #e5e7eb;
  color: #374151;
}

.status-green {
  background: #dcfce7;
  color: #166534;
}

.status-yellow {
  background: #fef9c3;
  color: #854d0e;
}

.status-red {
  background: #fee2e2;
  color: #991b1b;
}

@media (max-width: 1024px) {
  .time-remaining {
    font-size: 8rem;
  }
  
  .time-panel.time-primary {
    padding: 2rem 4rem;
  }
  
  .timer-page.fullscreen-mode .time-remaining {
    font-size: 12rem;
  }
  
  .timer-page.fullscreen-mode .time-panel.time-primary {
    padding: 2rem 4rem;
  }
}

@media (max-width: 768px) {
  .time-text {
    font-size: 2.5rem;
  }

  .time-secondary {
    font-size: 1.5rem;
  }

  .time-remaining {
    font-size: 5rem;
  }

  .time-panel.time-primary {
    padding: 1.5rem 2rem;
  }
  
  .timer-page.fullscreen-mode .time-remaining {
    font-size: 8rem;
  }
  
  .timer-page.fullscreen-mode .time-panel.time-primary {
    padding: 1rem 2rem;
  }

  .top-bar {
    flex-direction: column;
    align-items: stretch;
  }

  .speaker-input {
    max-width: none;
  }

  .secondary-times {
    justify-content: center;
    gap: 1rem;
    flex-wrap: wrap;
  }

  .controls {
    flex-direction: column;
    align-items: stretch;
  }

  .btn-large {
    width: 100%;
    justify-content: center;
  }

  .header {
    flex-direction: column;
    gap: 1rem;
    text-align: center;
  }
  
  .header-actions {
    flex-wrap: wrap;
    justify-content: center;
  }
  
  .main-content {
    padding: 1rem;
    gap: 1.5rem;
  }
}
</style>
