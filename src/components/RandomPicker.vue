<template>
  <div class="picker-section">
    <h2 class="picker-title">Random Picker</h2>
    
    <div class="picker-display">
      <div v-if="isPicking" class="picking-animation">
        <div class="spinner">🎲</div>
        <div>Picking...</div>
      </div>
      <div v-else-if="selectedItem" class="selected-item">
        {{ selectedItem }}
      </div>
      <div v-else class="placeholder">
        <span v-if="presetType === 'custom'">Name your items and click "Pick Random" to select one at random.</span>
        <span v-else>Choose a type and click "Pick Random" to generate a random result.</span>
      </div>
    </div>

    <div class="preset-controls">
      <div class="preset-options">
        <div class="preset-option">
          <label class="preset-label">Type:</label>
          <select v-model="presetType" class="preset-select">
            <option value="number">Random Number</option>
            <option value="custom">Custom Items</option>
            <option value="coin">Coin Flip</option>
            <option value="dice">Dice Roll</option>
          </select>
        </div>

        <!-- Custom Items -->
        <div v-if="presetType === 'custom'" class="picker-controls">
          <div class="items-list" ref="itemsListRef">
            <div
              v-for="(item, index) in items"
              :key="index"
              class="item-row"
            >
              <input
                v-model="items[index]"
                type="text"
                class="item-input"
                :placeholder="`Item ${index + 1}`"
              />
              <button
                @click="removeItem(index)"
                class="remove-button"
              >
                ×
              </button>
            </div>
          </div>
        </div>

        <!-- Number Range -->
        <div v-if="presetType === 'number'" class="preset-config">
          <div class="range-inputs">
            <div class="range-input-group">
              <label class="range-label">Min:</label>
              <input
                v-model.number="numberMin"
                type="number"
                class="range-input"
                placeholder="0"
              />
            </div>
            <div class="range-input-group">
              <label class="range-label">Max:</label>
              <input
                v-model.number="numberMax"
                type="number"
                class="range-input"
                placeholder="100"
              />
            </div>
          </div>
        </div>

        <!-- Coin Flip Info -->
        <div v-if="presetType === 'coin'" class="preset-info">
          <p>Heads or Tails</p>
        </div>

        <!-- Dice Roll Info -->
        <div v-if="presetType === 'dice'" class="preset-info">
          <p>Roll a 6-sided die (1-6)</p>
        </div>
      </div>

      <div class="picker-actions">
        <button
          v-if="presetType === 'custom'"
          @click="addItem"
          class="action-button add-item-button"
        >
          + Add Item
        </button>
        <button
          @click="pickRandom"
          :disabled="!canPick"
          class="action-button pick-button"
        >
          Pick Random
        </button>
        <button
          @click="clearSelection"
          class="action-button clear-button"
        >
          Clear
        </button>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, nextTick, computed } from 'vue'

const items = ref([''])
const selectedItem = ref('')
const itemsListRef = ref(null)
const presetType = ref('number')
const numberMin = ref(1)
const numberMax = ref(100)
const isPicking = ref(false)

const canPick = computed(() => {
  if (presetType.value === 'custom') {
    return items.value.length > 0 && items.value.some(i => i.trim())
  }
  if (presetType.value === 'number') {
    return numberMin.value !== null && numberMax.value !== null && 
           numberMin.value < numberMax.value
  }
  return true // coin and dice are always valid
})

const addItem = async () => {
  items.value.push('')
  // Wait for DOM to update, then scroll to bottom
  await nextTick()
  if (itemsListRef.value) {
    itemsListRef.value.scrollTop = itemsListRef.value.scrollHeight
  }
}

const removeItem = (index) => {
  if (items.value.length > 1) {
    items.value.splice(index, 1)
  } else {
    items.value[0] = ''
  }
  selectedItem.value = ''
}

const pickRandom = async () => {
  if (!canPick.value) return
  
  // Show picking animation
  isPicking.value = true
  selectedItem.value = ''
  
  // Brief delay to show animation
  await new Promise(resolve => setTimeout(resolve, 500))
  
  // Handle different preset types
  if (presetType.value === 'custom') {
    const validItems = items.value.filter(item => item.trim())
    if (validItems.length > 0) {
      const randomIndex = Math.floor(Math.random() * validItems.length)
      selectedItem.value = validItems[randomIndex]
    }
  } else if (presetType.value === 'number') {
    const min = Math.min(numberMin.value, numberMax.value)
    const max = Math.max(numberMin.value, numberMax.value)
    const randomNum = Math.floor(Math.random() * (max - min + 1)) + min
    selectedItem.value = randomNum.toString()
  } else if (presetType.value === 'coin') {
    const result = Math.random() < 0.5 ? 'Heads' : 'Tails'
    selectedItem.value = result
  } else if (presetType.value === 'dice') {
    const roll = Math.floor(Math.random() * 6) + 1
    selectedItem.value = roll.toString()
  }
  
  isPicking.value = false
}

const clearSelection = () => {
  selectedItem.value = ''
}
</script>

<style scoped>
.picker-section {
  background: white;
  border: 3px solid #000;
  box-shadow: 6px 6px 0px 0px #000;
  padding: 1rem;
  flex: 1;
  min-width: 300px;
}

@media (min-width: 768px) and (max-width: 1024px) {
  .picker-section {
    padding: 0.875rem;
    min-width: 280px;
  }
  
  .picker-title {
    font-size: 1.125rem;
    margin-bottom: 0.625rem;
  }
  
  .selected-item {
    font-size: 1.375rem;
    padding: 0.625rem 0.875rem;
  }
  
  .preset-options {
    gap: 0.75rem;
  }
  
  .picker-display {
    margin-bottom: 0.875rem;
    min-height: 35px;
  }
}

.picker-title {
  font-size: 1.25rem;
  font-weight: 900;
  text-transform: uppercase;
  margin-bottom: 0.75rem;
  color: #000;
  text-align: center;
}

.picker-display {
  display: flex;
  justify-content: center;
  margin-bottom: 1rem;
  min-height: 40px;
}

.selected-item {
  font-size: 1.5rem;
  font-weight: 900;
  color: #000;
  background: #ffff00;
  border: 3px solid #000;
  padding: 0.75rem 1rem;
  box-shadow: 4px 4px 0px 0px #000;
  text-align: center;
  word-break: break-word;
  animation: highlight 0.8s ease;
}

.picking-animation {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  gap: 0.5rem;
  font-size: 1rem;
  font-weight: 700;
  color: #9C27B0;
  text-transform: uppercase;
}

.spinner {
  font-size: 3rem;
  animation: spin 0.5s linear infinite;
}

.placeholder {
  font-size: 0.875rem;
  font-weight: 600;
  color: #999;
  display: flex;
  align-items: center;
  justify-content: center;
  text-align: center;
  padding: 0.25rem;
}

@keyframes highlight {
  0% {
    transform: scale(0.8) rotate(-5deg);
    opacity: 0;
  }
  50% {
    transform: scale(1.15) rotate(5deg);
  }
  100% {
    transform: scale(1) rotate(0deg);
    opacity: 1;
  }
}

@keyframes spin {
  0% {
    transform: rotate(0deg) scale(1);
  }
  50% {
    transform: rotate(180deg) scale(1.2);
  }
  100% {
    transform: rotate(360deg) scale(1);
  }
}

.picker-controls {
  display: flex;
  flex-direction: column;
  gap: 0.75rem;
}

.items-list {
  display: flex;
  flex-direction: column;
  gap: 0.5rem;
  height: 150px;
  overflow-y: auto;
  padding: 0.25rem;
  flex-shrink: 0;
}

@media (min-width: 768px) and (max-width: 1024px) {
  .items-list {
    height: 120px;
    max-height: 180px;
  }
}

.item-row {
  display: flex;
  gap: 0.5rem;
  align-items: center;
}

.item-input {
  flex: 1;
  padding: 0.5rem;
  border: 3px solid #000;
  font-size: 0.875rem;
  font-weight: 600;
  box-shadow: 3px 3px 0px 0px #000;
  transition: all 0.1s ease;
}

.item-input:focus {
  outline: none;
  transform: translate(2px, 2px);
  box-shadow: 2px 2px 0px 0px #000;
}

.remove-button {
  width: 28px;
  height: 28px;
  background: #ff0000;
  color: white;
  border: 3px solid #000;
  font-size: 16px;
  font-weight: bold;
  cursor: pointer;
  box-shadow: 3px 3px 0px 0px #000;
  transition: all 0.1s ease;
  display: flex;
  align-items: center;
  justify-content: center;
  flex-shrink: 0;
}

.remove-button:hover {
  transform: translate(2px, 2px);
  box-shadow: 2px 2px 0px 0px #000;
}

.picker-actions {
  display: flex;
  flex-direction: column;
  gap: 0.75rem;
}

@media (min-width: 768px) and (max-width: 1024px) {
  .picker-actions {
    flex-direction: row;
    flex-wrap: wrap;
  }
  
  .action-button {
    flex: 1;
    min-width: 120px;
  }
}

.action-button {
  padding: 0.5rem 1rem;
  font-size: 0.875rem;
  font-weight: 700;
  text-transform: uppercase;
  border: 3px solid #000;
  cursor: pointer;
  box-shadow: 4px 4px 0px 0px #000;
  transition: all 0.1s ease;
  font-family: system-ui, -apple-system, sans-serif;
}

.action-button:disabled {
  opacity: 0.5;
  cursor: not-allowed;
}

.action-button:not(:disabled):hover {
  transform: translate(2px, 2px);
  box-shadow: 2px 2px 0px 0px #000;
}

.add-item-button {
  background: white;
  color: #000;
}

.pick-button {
  background: #9C27B0;
  color: white;
}

.clear-button {
  background: #f0f0f0;
  color: #000;
}

.preset-controls {
  display: flex;
  flex-direction: column;
  gap: 0.75rem;
}

@media (min-width: 768px) and (max-width: 1024px) {
  .preset-controls {
    gap: 0.5rem;
  }
}

.preset-options {
  display: flex;
  flex-direction: column;
  gap: 1rem;
}

.preset-option {
  display: flex;
  flex-direction: column;
  gap: 0.5rem;
}

.preset-label {
  font-size: 0.875rem;
  font-weight: 700;
  text-transform: uppercase;
  color: #000;
}

.preset-select {
  padding: 0.5rem;
  border: 3px solid #000;
  font-size: 0.875rem;
  font-weight: 600;
  box-shadow: 3px 3px 0px 0px #000;
  transition: all 0.1s ease;
  background: white;
  color: #000;
  cursor: pointer;
  font-family: system-ui, -apple-system, sans-serif;
}

.preset-select:focus {
  outline: none;
  transform: translate(2px, 2px);
  box-shadow: 2px 2px 0px 0px #000;
}

.preset-config {
  display: flex;
  flex-direction: column;
  gap: 0.75rem;
}

@media (min-width: 768px) and (max-width: 1024px) {
  .preset-config {
    gap: 0.5rem;
  }
}

.range-inputs {
  display: flex;
  gap: 1rem;
  align-items: center;
}

@media (min-width: 768px) and (max-width: 1024px) {
  .range-inputs {
    gap: 0.5rem;
  }
  
  .range-input-group {
    flex: 1;
    min-width: 0;
  }
  
  .range-label {
    font-size: 0.6875rem;
  }
  
  .range-input {
    padding: 0.375rem 0.5rem;
    font-size: 0.8125rem;
    width: 100%;
    min-width: 0;
  }
}

.range-input-group {
  display: flex;
  flex-direction: column;
  gap: 0.25rem;
  flex: 1;
}

.range-label {
  font-size: 0.75rem;
  font-weight: 700;
  text-transform: uppercase;
  color: #000;
}

.range-input {
  padding: 0.5rem;
  border: 3px solid #000;
  font-size: 0.875rem;
  font-weight: 600;
  box-shadow: 3px 3px 0px 0px #000;
  transition: all 0.1s ease;
  background: white;
  color: #000;
  width: 100%;
}

.range-input:focus {
  outline: none;
  transform: translate(2px, 2px);
  box-shadow: 2px 2px 0px 0px #000;
}

.preset-info {
  padding: 0.75rem;
  background: transparent;
  border: none;
  box-shadow: none;
  font-size: 0.875rem;
  font-weight: 600;
  color: #666;
  text-align: center;
}

@media (min-width: 768px) and (max-width: 1024px) {
  .preset-option {
    gap: 0.375rem;
  }
}

@media (max-width: 640px) {
  .selected-item {
    font-size: 1.5rem;
    padding: 1rem 1.5rem;
  }
  
  .picker-title {
    font-size: 1.5rem;
  }
}
</style>

