<template>
  <div id="app" class="h-screen flex flex-col">
    <div class="container mx-auto px-4 py-4 pb-8 flex flex-col flex-1 min-h-0">
      <header class="mt-4">
        <div class="header-content">
          <img src="/favicon.svg" alt="Launcher" class="header-icon" />
          <h1 class="app-title">Launcher</h1>
        </div>
      </header>

      <div class="top-section pb-4">
        <Timer />
        <RandomPicker />
      </div>

      <div class="buttons-container" :class="{ 'edit-mode-active': editMode }">
        <div class="toolbar">
          <button @click="showAddForm = true" class="add-button">
            + Add
          </button>
          <button 
            v-if="buttons.length > 0"
            @click="editMode = !editMode" 
            class="edit-mode-button"
            :class="{ active: editMode }"
          >
            {{ editMode ? 'Done' : 'Edit' }}
          </button>
        </div>


        <LauncherButton
          v-for="button in buttons"
          :key="button.id"
          :button="button"
          :edit-mode="editMode"
          :snap-to-grid="snapToGrid"
          @click="handleButtonClick"
          @edit="handleEdit"
          @delete="handleDelete"
          @update="handleButtonUpdate"
        />
      </div>

      <ButtonForm
        v-if="showForm"
        :editing-button="editingButton"
        @submit="handleFormSubmit"
        @close="closeForm"
      />
    </div>
  </div>
</template>

<script setup>
import { ref, computed } from 'vue'
import { useLauncher } from './composables/useLauncher'
import LauncherButton from './components/LauncherButton.vue'
import ButtonForm from './components/ButtonForm.vue'
import Timer from './components/Timer.vue'
import RandomPicker from './components/RandomPicker.vue'

const { buttons, addButton, updateButton, deleteButton, openUrl, snapToGrid } = useLauncher()

const showAddForm = ref(false)
const editMode = ref(false)
const editingButton = ref(null)

const showForm = computed(() => showAddForm.value || editingButton.value !== null)

const handleButtonClick = (button) => {
  // Only open URL if not in edit mode
  if (!editMode.value) {
    openUrl(button.url, button.fileId, button.fileType, button.fileName)
  }
}

const handleEdit = (button) => {
  editingButton.value = button
}

const handleDelete = (id) => {
  if (confirm('Are you sure you want to delete this button?')) {
    deleteButton(id)
  }
}

const handleButtonUpdate = (updates) => {
  if (updates.id) {
    const { id, ...buttonUpdates } = updates
    updateButton(id, buttonUpdates)
  }
}

const handleFormSubmit = (formData) => {
  if (editingButton.value) {
    updateButton(editingButton.value.id, formData)
    editingButton.value = null
  } else {
    addButton(formData)
    showAddForm.value = false
  }
}

const closeForm = () => {
  showAddForm.value = false
  editingButton.value = null
}
</script>

<style scoped>
.header-content {
  display: flex;
  align-items: center;
  gap: 1rem;
}

.header-icon {
  width: 48px;
  height: 48px;
  flex-shrink: 0;
}

.app-title {
  font-size: 3rem;
  font-weight: 900;
  text-transform: uppercase;
  margin: 0;
  color: #000;
  text-shadow: 4px 4px 0px rgba(0, 0, 0, 0.2);
  letter-spacing: -0.02em;
}

body { 
  margin: 0 !important; 
  padding: 0 !important;
}

.app-subtitle {
  font-size: 1.25rem;
  font-weight: 600;
  color: #333;
}

.buttons-container {
  position: relative;
  min-height: 300px;
  flex: 1;
  overflow: auto;
  margin-top: 2rem;
  background-image: 
    linear-gradient(to right, rgba(0,0,0,0.05) 1px, transparent 1px),
    linear-gradient(to bottom, rgba(0,0,0,0.05) 1px, transparent 1px);
  background-size: 20px 20px;
  border: 2px dashed rgba(0,0,0,0.1);
  border-radius: 4px;
  padding: 20px;
}

.buttons-container .toolbar {
  position: absolute;
  top: 20px;
  right: 20px;
  display: flex;
  gap: 0.75rem;
  flex-wrap: wrap;
  z-index: 10;
}


.add-button,
.edit-mode-button {
  padding: 0.5rem 1rem;
  font-size: 0.875rem;
  font-weight: 700;
  text-transform: uppercase;
  border: 3px solid #000;
  cursor: pointer;
  transition: all 0.1s ease;
  font-family: system-ui, -apple-system, sans-serif;
}

.add-button {
  background: #9C27B0;
  color: white;
  box-shadow: 4px 4px 0px 0px #000;
}

.add-button:hover {
  transform: translate(2px, 2px);
  box-shadow: 2px 2px 0px 0px #000;
}

.edit-mode-button {
  background: white;
  color: #000;
  box-shadow: 4px 4px 0px 0px #000;
}

.edit-mode-button:hover {
  transform: translate(2px, 2px);
  box-shadow: 2px 2px 0px 0px #000;
}

.edit-mode-button.active {
  background: #4CAF50;
  color: white;
}

.top-section {
  display: flex;
  gap: 2rem;
  margin-top: 2rem;
  flex-wrap: wrap;
}

.top-section > * {
  flex: 1;
  min-width: 300px;
}


@media (max-width: 640px) {
  .buttons-grid {
    grid-template-columns: 1fr;
  }
  
  .app-title {
    font-size: 2rem;
  }
}
</style>
