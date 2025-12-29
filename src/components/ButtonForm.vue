<template>
  <div class="form-overlay" @click.self="close">
    <div class="form-container">
      <div class="form-header">
        <h2 class="form-title">{{ editingButton ? 'Edit Button' : 'Add New Button' }}</h2>
        <button class="close-button" @click="close">×</button>
      </div>
      
      <form @submit.prevent="handleSubmit" class="form-content">
        <div class="form-group">
          <label class="form-label">Label</label>
          <input
            v-model="formData.label"
            type="text"
            class="form-input"
            placeholder="Button label"
            required
          />
        </div>

        <div class="form-group">
          <label class="form-label">Link Type</label>
          <select v-model="linkType" class="form-input">
            <option value="url">URL</option>
            <option value="file">Local File</option>
          </select>
        </div>

        <div v-if="linkType === 'url'" class="form-group">
          <label class="form-label">URL</label>
          <input
            v-model="formData.url"
            type="url"
            class="form-input"
            placeholder="https://example.com"
            :required="linkType === 'url'"
          />
        </div>

        <div v-if="linkType === 'file'" class="form-group">
          <label class="form-label">File</label>
          <input
            @change="handleFileSelect"
            type="file"
            class="form-input"
            :required="linkType === 'file'"
            ref="fileInput"
          />
          <div v-if="formData.fileName" class="file-info">
            Selected: {{ formData.fileName }}
            <button type="button" @click="clearFile" class="clear-file-btn">×</button>
          </div>
        </div>

        <div class="form-group">
          <label class="form-label">Color</label>
          <div class="preset-colors">
            <button
              v-for="presetColor in presetColors"
              :key="presetColor.value"
              type="button"
              @click="formData.color = presetColor.value"
              class="preset-color-button"
              :class="{ active: formData.color === presetColor.value }"
              :style="{ backgroundColor: presetColor.value }"
              :title="presetColor.name"
            >
              <span v-if="formData.color === presetColor.value" class="checkmark">✓</span>
            </button>
            <button
              type="button"
              @click="showCustomColor = !showCustomColor"
              class="preset-color-button custom-color-toggle"
              :class="{ active: showCustomColor }"
              title="Custom Color"
            >
              🎨
            </button>
          </div>
          <div v-if="showCustomColor" class="custom-color-picker">
            <div class="color-picker-container">
              <input
                v-model="formData.color"
                type="color"
                class="color-input"
              />
              <input
                v-model="formData.color"
                type="text"
                class="form-input color-text"
                placeholder="#000000"
              />
            </div>
          </div>
        </div>

        <div class="form-actions">
          <button type="button" class="btn btn-secondary" @click="close">Cancel</button>
          <button type="submit" class="btn btn-primary">{{ editingButton ? 'Update' : 'Add' }}</button>
        </div>
      </form>
    </div>
  </div>
</template>

<script setup>
import { ref, watch } from 'vue'
import { saveFile } from '../composables/fileStorage.js'

const props = defineProps({
  editingButton: {
    type: Object,
    default: null
  }
})

const emit = defineEmits(['submit', 'close'])

const linkType = ref('url')
const fileInput = ref(null)
const showCustomColor = ref(false)

const presetColors = [
  { name: 'White', value: '#FFFFFF' },
  { name: 'Black', value: '#000000' },
  { name: 'Purple', value: '#9C27B0' },
  { name: 'Yellow', value: '#FFE66D' },
  { name: 'Pink', value: '#FF6B9D' },
  { name: 'Blue', value: '#4ECDC4' },
  { name: 'Green', value: '#95E1D3' },
  { name: 'Orange', value: '#FFA07A' },
  { name: 'Red', value: '#FF6B6B' }
]

const formData = ref({
  label: '',
  url: '',
  color: '#FFFFFF',
  fileId: null,
  fileName: null,
  fileType: null
})

watch(() => props.editingButton, (newButton) => {
  if (newButton) {
    linkType.value = newButton.fileId ? 'file' : 'url'
    const color = newButton.color || '#FFFFFF'
    // Check if color is a preset, otherwise show custom picker
    showCustomColor.value = !presetColors.some(p => p.value === color)
    formData.value = {
      label: newButton.label || '',
      url: newButton.url || '',
      color: color,
      fileId: newButton.fileId || null,
      fileName: newButton.fileName || null,
      fileType: newButton.fileType || null
    }
  } else {
    linkType.value = 'url'
    showCustomColor.value = false
    formData.value = {
      label: '',
      url: '',
      color: '#FFFFFF',
      fileId: null,
      fileName: null,
      fileType: null
    }
    if (fileInput.value) {
      fileInput.value.value = ''
    }
  }
}, { immediate: true })

const handleFileSelect = async (event) => {
  const file = event.target.files[0]
  if (file) {
    // Convert file to base64 for storage in IndexedDB
    const reader = new FileReader()
    reader.onload = async (e) => {
      const fileId = formData.value.fileId || `file_${Date.now()}_${Math.random().toString(36).substr(2, 9)}`
      const fileData = e.target.result
      
      try {
        // Store file in IndexedDB
        await saveFile(fileId, fileData, file.name, file.type)
        formData.value.fileId = fileId
        formData.value.fileName = file.name
        formData.value.fileType = file.type
        formData.value.fileData = null
      } catch (error) {
        console.error('Error saving file:', error)
        alert('Error saving file. Please try again.')
      }
    }
    reader.readAsDataURL(file)
  }
}

const clearFile = async () => {
  if (formData.value.fileId) {
    try {
      const { deleteFile } = await import('../composables/fileStorage.js')
      await deleteFile(formData.value.fileId)
    } catch (error) {
      console.error('Error deleting file:', error)
    }
  }
  formData.value.fileId = null
  formData.value.fileName = null
  formData.value.fileType = null
  if (fileInput.value) {
    fileInput.value.value = ''
  }
}

const handleSubmit = () => {
  emit('submit', { ...formData.value })
  close()
}

const close = () => {
  emit('close')
}
</script>

<style scoped>
.form-overlay {
  position: fixed;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background: rgba(0, 0, 0, 0.5);
  display: flex;
  align-items: center;
  justify-content: center;
  z-index: 1000;
  padding: 1rem;
}

@media (max-width: 768px) {
  .form-overlay {
    padding: 0;
    align-items: stretch;
  }
}

.form-container {
  background: white;
  border: 4px solid #000;
  box-shadow: 12px 12px 0px 0px #000;
  max-width: 500px;
  width: 100%;
  padding: 0;
  max-height: 90vh;
  overflow: auto;
}

@media (max-width: 768px) {
  .form-container {
    max-width: 100%;
    max-height: 100vh;
    width: 100%;
    height: 100%;
    border-width: 0;
    box-shadow: none;
    display: flex;
    flex-direction: column;
  }
}

.form-header {
  background: #000;
  color: white;
  padding: 1.5rem;
  display: flex;
  justify-content: space-between;
  align-items: center;
}

@media (max-width: 768px) {
  .form-header {
    padding: 1rem;
  }
}

.form-title {
  margin: 0;
  font-size: 1.5rem;
  font-weight: 700;
  text-transform: uppercase;
}

@media (max-width: 768px) {
  .form-title {
    font-size: 1.25rem;
  }
}

.close-button {
  background: white;
  color: #000;
  border: 3px solid #000;
  width: 40px;
  height: 40px;
  font-size: 24px;
  font-weight: bold;
  cursor: pointer;
  display: flex;
  align-items: center;
  justify-content: center;
  box-shadow: 4px 4px 0px 0px #000;
  transition: all 0.1s ease;
}

@media (max-width: 768px) {
  .close-button {
    width: 36px;
    height: 36px;
    font-size: 20px;
    border-width: 2px;
    box-shadow: 3px 3px 0px 0px #000;
  }
}

.close-button:hover {
  transform: translate(2px, 2px);
  box-shadow: 2px 2px 0px 0px #000;
}

.form-content {
  padding: 2rem;
  overflow-y: auto;
  flex: 1;
}

@media (max-width: 768px) {
  .form-content {
    padding: 1.5rem 1rem;
  }
}

.form-group {
  margin-bottom: 1.5rem;
}

@media (max-width: 768px) {
  .form-group {
    margin-bottom: 1.25rem;
  }
}

.form-label {
  display: block;
  font-weight: 700;
  margin-bottom: 0.5rem;
  text-transform: uppercase;
  font-size: 0.875rem;
}

.form-input {
  width: 100%;
  padding: 0.75rem;
  border: 3px solid #000;
  font-size: 1rem;
  font-weight: 600;
  box-shadow: 4px 4px 0px 0px #000;
  transition: all 0.1s ease;
}

.form-input:focus {
  outline: none;
  transform: translate(2px, 2px);
  box-shadow: 2px 2px 0px 0px #000;
}

.preset-colors {
  display: grid;
  grid-template-columns: repeat(5, 1fr);
  gap: 0.75rem;
  margin-bottom: 1rem;
}

@media (max-width: 768px) {
  .preset-colors {
    grid-template-columns: repeat(4, 1fr);
    gap: 0.5rem;
  }
}

.preset-color-button {
  width: 100%;
  aspect-ratio: 1;
  border: 3px solid #000;
  cursor: pointer;
  box-shadow: 4px 4px 0px 0px #000;
  transition: all 0.1s ease;
  display: flex;
  align-items: center;
  justify-content: center;
  font-size: 1.25rem;
  position: relative;
  font-weight: bold;
}

.preset-color-button:hover {
  transform: translate(2px, 2px);
  box-shadow: 2px 2px 0px 0px #000;
}

.preset-color-button.active {
  transform: translate(3px, 3px);
  box-shadow: 1px 1px 0px 0px #000;
  border-width: 4px;
}

.preset-color-button .checkmark {
  color: white;
  text-shadow: 2px 2px 0px #000;
  font-size: 1.5rem;
}

.custom-color-toggle {
  background: linear-gradient(45deg, #ff6b9d, #4ecdc4, #ffe66d);
  color: #000;
}

.custom-color-picker {
  margin-top: 1rem;
  padding: 1rem;
  background: #f0f0f0;
  border: 3px solid #000;
  box-shadow: 4px 4px 0px 0px #000;
}

.color-picker-container {
  display: flex;
  gap: 0.5rem;
  align-items: center;
}

.color-input {
  width: 80px;
  height: 50px;
  border: 3px solid #000;
  cursor: pointer;
  box-shadow: 4px 4px 0px 0px #000;
}

.color-text {
  flex: 1;
}

.color-preview {
  margin-top: 0.5rem;
  height: 60px;
  border: 3px solid #000;
  box-shadow: 4px 4px 0px 0px #000;
}

.file-info {
  margin-top: 0.5rem;
  padding: 0.5rem;
  background: #f0f0f0;
  border: 2px solid #000;
  font-size: 0.875rem;
  font-weight: 600;
  display: flex;
  justify-content: space-between;
  align-items: center;
}

.clear-file-btn {
  background: #ff0000;
  color: white;
  border: 2px solid #000;
  width: 24px;
  height: 24px;
  font-size: 18px;
  font-weight: bold;
  cursor: pointer;
  display: flex;
  align-items: center;
  justify-content: center;
  box-shadow: 2px 2px 0px 0px #000;
  transition: all 0.1s ease;
}

.clear-file-btn:hover {
  transform: translate(1px, 1px);
  box-shadow: 1px 1px 0px 0px #000;
}

.form-actions {
  display: flex;
  gap: 1rem;
  margin-top: 2rem;
}

@media (max-width: 768px) {
  .form-actions {
    gap: 0.75rem;
    margin-top: 1.5rem;
    padding-top: 1rem;
    border-top: 2px solid #000;
  }
}

.btn {
  flex: 1;
  padding: 1rem;
  font-size: 1rem;
  font-weight: 700;
  text-transform: uppercase;
  border: 4px solid #000;
  cursor: pointer;
  transition: all 0.1s ease;
  font-family: system-ui, -apple-system, sans-serif;
}

@media (max-width: 768px) {
  .btn {
    padding: 0.75rem;
    font-size: 0.875rem;
    border-width: 3px;
  }
}

.btn-primary {
  background: #9C27B0;
  color: white;
  box-shadow: 6px 6px 0px 0px #000;
}

.btn-primary:hover {
  transform: translate(3px, 3px);
  box-shadow: 3px 3px 0px 0px #000;
}

@media (max-width: 768px) {
  .btn-primary {
    box-shadow: 4px 4px 0px 0px #000;
  }
  
  .btn-primary:hover {
    box-shadow: 2px 2px 0px 0px #000;
  }
}

.btn-secondary {
  background: white;
  color: #000;
  box-shadow: 6px 6px 0px 0px #000;
}

.btn-secondary:hover {
  transform: translate(3px, 3px);
  box-shadow: 3px 3px 0px 0px #000;
}

@media (max-width: 768px) {
  .btn-secondary {
    box-shadow: 4px 4px 0px 0px #000;
  }
  
  .btn-secondary:hover {
    box-shadow: 2px 2px 0px 0px #000;
  }
}
</style>

