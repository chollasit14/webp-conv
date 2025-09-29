<template>
  <v-app>
    <v-main class="bg-grey-lighten-4">
      <v-container class="py-8">
        <!-- Header -->
        <div class="text-center mb-8">
          <h1 class="text-h3 font-weight-bold text-primary mb-2">
            <v-icon size="40" class="mr-3">mdi-image-multiple</v-icon>
            WebP Converter
          </h1>
          <p class="text-h6 text-grey-darken-1">แปลงรูปภาพให้เป็น WebP อย่างง่ายดาย</p>
        </div>

        <!-- Upload Card -->
        <v-row justify="center">
          <v-col cols="12" md="8" lg="6">
            <v-card 
              class="mb-6 elevation-4"
              :class="{ 'upload-hover': isDragging }"
              @dragover.prevent="isDragging = true"
              @dragleave.prevent="isDragging = false"
              @drop.prevent="handleDrop"
            >
              <v-card-text class="text-center py-8">
                <div v-if="!images.length">
                  <v-icon 
                    size="80" 
                    :color="isDragging ? 'primary' : 'grey-lighten-1'"
                    class="mb-4"
                  >
                    mdi-cloud-upload
                  </v-icon>
                  <h3 class="text-h5 mb-3">อัปโหลดรูปภาพของคุณ</h3>
                  <p class="text-grey-darken-1 mb-4">
                    ลากและปล่อยไฟล์ หรือคลิกเพื่อเลือกไฟล์
                  </p>
                  <p class="text-caption text-grey-darken-1 mb-4">
                    (ขนาดไฟล์สูงสุด 10 MB ต่อไฟล์)
                  </p>
                  <v-file-input
                    v-model="files"
                    multiple
                    accept="image/*"
                    hide-details
                    style="display: none"
                    ref="fileInput"
                    @change="handleUpload"
                  />
                  <v-btn
                    color="primary"
                    size="large"
                    variant="flat"
                    @click="triggerFileUpload"
                    class="px-8"
                  >
                    <v-icon left>mdi-upload</v-icon>
                    เลือกไฟล์
                  </v-btn>
                </div>
                
                <!-- Processing State -->
                <div v-else-if="isProcessing">
                  <v-progress-circular
                    :model-value="conversionProgress"
                    :size="80"
                    :width="8"
                    color="primary"
                    class="mb-4"
                  >
                    <span class="text-h6">{{ conversionProgress }}%</span>
                  </v-progress-circular>
                  <h3 class="text-h5 mb-2">กำลังแปลงไฟล์...</h3>
                  <p class="text-grey-darken-1">
                    {{ convertedCount }} / {{ images.length }} ไฟล์
                  </p>
                </div>

                <!-- Uploaded State -->
                <div v-else-if="!convertedAll">
                  <v-icon size="80" color="info" class="mb-4">
                    mdi-image-check
                  </v-icon>
                  <h3 class="text-h5 text-info mb-3">อัปโหลดรูปเรียบร้อย!</h3>
                  <p class="text-grey-darken-1">
                    อัปโหลดไฟล์สำเร็จ {{ images.length }} ไฟล์
                  </p>
                </div>

                <!-- Completed State -->
                <div v-else>
                  <v-icon size="80" color="success" class="mb-4">
                    mdi-check-circle
                  </v-icon>
                  <h3 class="text-h5 text-success mb-3">แปลงเสร็จเรียบร้อย!</h3>
                  <p class="text-grey-darken-1">
                    แปลงไฟล์สำเร็จ {{ successCount }} ไฟล์
                    <span v-if="errorCount > 0" class="text-error">
                      (ล้มเหลว {{ errorCount }} ไฟล์)
                    </span>
                  </p>
                </div>
              </v-card-text>
            </v-card>
          </v-col>
        </v-row>

        <!-- Quality Settings -->
        <v-row v-if="images.length && !convertedAll" justify="center">
          <v-col cols="12" md="8" lg="6">
            <v-card class="mb-6 elevation-2">
              <v-card-title class="d-flex align-center">
                <v-icon class="mr-2">mdi-tune</v-icon>
                ตั้งค่าคุณภาพ
              </v-card-title>
              <v-card-text>
                <div class="d-flex align-center">
                  <v-slider
                    v-model="quality"
                    :min="0.1"
                    :max="1"
                    :step="0.1"
                    thumb-label
                    :disabled="isProcessing"
                    class="mr-4"
                  >
                    <template v-slot:thumb-label="{ modelValue }">
                      {{ Math.round(modelValue * 100) }}%
                    </template>
                  </v-slider>
                  <v-chip color="primary" variant="flat" class="ml-2">
                    {{ Math.round(quality * 100) }}%
                  </v-chip>
                </div>
                <p class="text-caption text-grey-darken-1 mt-2">
                  คุณภาพสูง = ขนาดไฟล์ใหญ่ / คุณภาพต่ำ = ขนาดไฟล์เล็ก
                </p>
              </v-card-text>
            </v-card>
          </v-col>
        </v-row>

        <!-- Images Grid -->
        <v-row v-if="images.length">
          <v-col cols="12">
            <v-card class="mb-6 elevation-4">
              <v-card-title class="d-flex align-center">
                <v-icon class="mr-2">mdi-image-multiple</v-icon>
                ไฟล์ที่อัปโหลด ({{ images.length }} ไฟล์)
              </v-card-title>
              
              <v-card-text>
                <v-row>
                  <v-col 
                    v-for="(img, index) in images" 
                    :key="index"
                    cols="12" sm="6" md="4" lg="3" xl="2"
                  >
                    <v-card 
                      class="image-card"
                      :class="{ 
                        'converted': img.converted,
                        'error': img.error
                      }"
                      elevation="2"
                    >
                      <!-- Image Preview -->
                      <div class="image-preview-compact">
                        <img 
                          :src="img.preview" 
                          :alt="img.name"
                          class="preview-img-compact"
                        />
                        <v-overlay 
                          :model-value="img.converted || img.error"
                          class="align-center justify-center"
                          contained
                        >
                          <v-icon 
                            v-if="img.converted" 
                            size="24" 
                            color="success"
                          >
                            mdi-check-circle
                          </v-icon>
                          <v-icon 
                            v-if="img.error" 
                            size="24" 
                            color="error"
                          >
                            mdi-alert-circle
                          </v-icon>
                        </v-overlay>
                      </div>
                      
                      <!-- File Info -->
                      <v-card-text class="pa-2">
                        <div class="text-body-2 font-weight-medium mb-1 text-truncate" :title="img.name">
                          {{ img.name }}
                        </div>
                        <div class="text-caption text-grey-darken-1 mb-2">
                          {{ formatFileSize(img.size) }}
                          <span v-if="img.converted && img.webpSize" class="text-success">
                            → {{ formatFileSize(img.webpSize) }}
                          </span>
                        </div>
                        
                        <!-- Error Message -->
                        <v-alert
                          v-if="img.error"
                          type="error"
                          density="compact"
                          class="text-caption mb-2"
                        >
                          {{ img.errorMessage }}
                        </v-alert>
                        
                        <!-- Action Buttons -->
                        <div class="d-flex flex-column ga-1">
                          <v-btn
                            v-if="img.converted && img.url"
                            :href="img.url"
                            :download="img.name.replace(/\.[^.]+$/, '.webp')"
                            color="success"
                            variant="flat"
                            size="small"
                            block
                          >
                            <v-icon size="16" class="mr-1">mdi-download</v-icon>
                            ดาวน์โหลด
                          </v-btn>
                          
                          <v-btn
                            color="error"
                            variant="flat"
                            size="small"
                            block
                            @click="removeImage(index)"
                          >
                            <v-icon size="16" class="mr-1">mdi-delete</v-icon>
                            ลบไฟล์
                          </v-btn>
                        </div>
                      </v-card-text>
                    </v-card>
                  </v-col>
                </v-row>
              </v-card-text>
            </v-card>
          </v-col>
        </v-row>

        <!-- Action Buttons -->
        <v-row v-if="images.length" justify="center">
          <v-col cols="12" class="text-center">
            <div class="action-buttons">
              <v-btn
                color="primary"
                size="large"
                variant="flat"
                :disabled="convertedAll || isProcessing"
                :loading="isProcessing"
                @click="convertAll"
                class="mx-2 mb-4"
              >
                <v-icon class="mr-2">mdi-auto-fix</v-icon>
                แปลงเป็น WebP
              </v-btn>

              <v-btn
                color="success"
                size="large"
                variant="flat"
                :disabled="successCount === 0"
                @click="downloadAll"
                class="mx-2 mb-4"
              >
                <v-icon class="mr-2">mdi-download-multiple</v-icon>
                ดาวน์โหลดทั้งหมด
              </v-btn>

              <v-btn
                color="secondary"
                size="large"
                variant="outlined"
                @click="addMoreFiles"
                :disabled="isProcessing"
                class="mx-2 mb-4"
              >
                <v-icon class="mr-2">mdi-plus</v-icon>
                เพิ่มไฟล์
              </v-btn>

              <v-btn
                color="grey-darken-1"
                size="large"
                variant="outlined"
                @click="resetAll"
                :disabled="isProcessing"
                class="mx-2 mb-4"
              >
                <v-icon class="mr-2">mdi-refresh</v-icon>
                เริ่มต้นใหม่
              </v-btn>
            </div>
          </v-col>
        </v-row>

        <!-- Info Card -->
        <v-row justify="center">
          <v-col cols="12" md="10" lg="8">
            <v-card class="elevation-2">
              <v-card-title class="d-flex align-center mb-5">
                <v-icon class="mr-2">mdi-information</v-icon>
                เกี่ยวกับ WebP Converter
              </v-card-title>
              
              <v-card-text>
                <v-row>
                  <v-col cols="12" md="4">
                    <div class="text-center mb-4">
                      <v-icon size="48" color="primary" class="mb-2">
                        mdi-lightning-bolt
                      </v-icon>
                      <h4 class="text-h6 mb-2">รวดเร็ว</h4>
                      <p class="text-body-2">
                        แปลงไฟล์ได้อย่างรวดเร็วใน browser ของคุณ
                      </p>
                    </div>
                  </v-col>
                  
                  <v-col cols="12" md="4">
                    <div class="text-center mb-4">
                      <v-icon size="48" color="success" class="mb-2">
                        mdi-shield-check
                      </v-icon>
                      <h4 class="text-h6 mb-2">ปลอดภัย</h4>
                      <p class="text-body-2">
                        ไฟล์ของคุณจะไม่ถูกส่งไปยังเซิร์ฟเวอร์
                      </p>
                    </div>
                  </v-col>
                  
                  <v-col cols="12" md="4">
                    <div class="text-center mb-4">
                      <v-icon size="48" color="info" class="mb-2">
                        mdi-resize
                      </v-icon>
                      <h4 class="text-h6 mb-2">คุณภาพสูง</h4>
                      <p class="text-body-2">
                        ลดขนาดไฟล์โดยคงคุณภาพของภาพ
                      </p>
                    </div>
                  </v-col>
                </v-row>
              </v-card-text>
            </v-card>
          </v-col>
        </v-row>

        <!-- Hidden file input for adding more files -->
        <input 
          type="file" 
          ref="additionalFileInput"
          multiple
          accept="image/*"
          style="display: none"
          @change="handleAdditionalUpload"
        />
      </v-container>

      <!-- Snackbar for notifications -->
      <v-snackbar
        v-model="snackbar.show"
        :color="snackbar.color"
        :timeout="3000"
        location="top"
      >
        {{ snackbar.message }}
        <template v-slot:actions>
          <v-btn
            variant="text"
            @click="snackbar.show = false"
          >
            ปิด
          </v-btn>
        </template>
      </v-snackbar>

      <!-- Footer -->
      <v-footer class="bg-grey-darken-4 text-center py-6 mt-8">
        <v-container>
          <v-row>
            <v-col cols="12">
              <div class="d-flex flex-column align-center">
                <v-divider class="my-3 bg-grey-darken-2" style="max-width: 200px;"></v-divider>

                <div class="text-grey-lighten-2 text-body-2">
                  <v-icon size="16" class="mr-1">mdi-copyright</v-icon>
                  {{ new Date().getFullYear() }} สร้างโดย 
                  <span class="text-primary font-weight-bold mx-1">S.Chollasit</span>
                </div>

                <div class="text-grey-darken-1 text-caption mt-2">
                  Made with <v-icon size="12" color="red" class="mx-1">mdi-heart</v-icon> using Vue.js & Vuetify
                </div>
              </div>
            </v-col>
          </v-row>
        </v-container>
      </v-footer>
    </v-main>
  </v-app>
</template>

<script setup lang="ts">
import { ref, computed } from 'vue'
import JSZip from 'jszip'
import { VFileInput } from 'vuetify/components'

interface ImageItem {
  file: File
  name: string
  size: number
  converted: boolean
  url: string | null
  preview: string
  error: boolean
  errorMessage: string
  webpSize: number | null
}

interface Snackbar {
  show: boolean
  message: string
  color: string
}

const MAX_FILE_SIZE = 10 * 1024 * 1024 // 10 MB

const files = ref<File[]>([])
const images = ref<ImageItem[]>([])
const isDragging = ref(false)
const isProcessing = ref(false)
const quality = ref(0.8)
const convertedCount = ref(0)
const fileInput = ref<InstanceType<typeof VFileInput>>()
const additionalFileInput = ref<HTMLInputElement>()
const snackbar = ref<Snackbar>({
  show: false,
  message: '',
  color: 'info'
})

// Computed properties
const conversionProgress = computed(() => {
  if (images.value.length === 0) return 0
  return Math.round((convertedCount.value / images.value.length) * 100)
})

const convertedAll = computed(() =>
  images.value.length > 0 && images.value.every((i) => i.converted || i.error)
)

const successCount = computed(() => 
  images.value.filter(i => i.converted && !i.error).length
)

const errorCount = computed(() => 
  images.value.filter(i => i.error).length
)

// Helper function
const formatFileSize = (bytes: number): string => {
  if (bytes < 1024) return bytes + ' B'
  if (bytes < 1024 * 1024) return (bytes / 1024).toFixed(1) + ' KB'
  return (bytes / (1024 * 1024)).toFixed(1) + ' MB'
}

const showSnackbar = (message: string, color: string = 'info') => {
  snackbar.value = { show: true, message, color }
}

// เปิด file dialog
const triggerFileUpload = () => {
  fileInput.value?.$el.querySelector('input')?.click()
}

// อัปโหลดไฟล์
const handleUpload = () => {
  if (!files.value) return
  processFiles(files.value)
}

// จัดการ drag and drop
const handleDrop = (e: DragEvent) => {
  isDragging.value = false
  const droppedFiles = Array.from(e.dataTransfer?.files || [])
  processFiles(droppedFiles)
}

// ประมวลผลไฟล์
const processFiles = (fileList: File[]) => {
  let validFiles = 0
  let skippedFiles = 0

  for (const file of fileList) {
    // ตรวจสอบว่าเป็นไฟล์รูปภาพ
    if (!file.type.startsWith('image/')) {
      skippedFiles++
      continue
    }

    // ตรวจสอบขนาดไฟล์
    if (file.size > MAX_FILE_SIZE) {
      showSnackbar(`ไฟล์ ${file.name} มีขนาดใหญ่เกิน 10 MB`, 'error')
      skippedFiles++
      continue
    }

    validFiles++
    const reader = new FileReader()
    reader.onload = (e) => {
      images.value.push({
        file,
        name: file.name,
        size: file.size,
        converted: false,
        url: null,
        preview: e.target?.result as string,
        error: false,
        errorMessage: '',
        webpSize: null
      })
    }
    reader.onerror = () => {
      showSnackbar(`ไม่สามารถอ่านไฟล์ ${file.name}`, 'error')
    }
    reader.readAsDataURL(file)
  }

  if (validFiles > 0) {
    showSnackbar(`อัปโหลดไฟล์สำเร็จ ${validFiles} ไฟล์`, 'success')
  }
  if (skippedFiles > 0) {
    showSnackbar(`ข้าม ${skippedFiles} ไฟล์ (ไม่รองรับหรือขนาดใหญ่เกินไป)`, 'warning')
  }
}

// เพิ่มไฟล์เพิ่มเติม
const addMoreFiles = () => {
  additionalFileInput.value?.click()
}

const handleAdditionalUpload = (e: Event) => {
  const target = e.target as HTMLInputElement
  if (target.files) {
    const newFiles = Array.from(target.files)
    processFiles(newFiles)
    target.value = '' // Reset input
  }
}

// ฟังก์ชันแปลงไฟล์เป็น WebP
const convertToWebp = (file: File, qualityValue: number): Promise<{ blob: Blob; url: string }> =>
  new Promise((resolve, reject) => {
    const reader = new FileReader()
    
    reader.onerror = () => reject(new Error('ไม่สามารถอ่านไฟล์'))
    
    reader.onload = (e) => {
      const img = new Image()
      img.onerror = () => reject(new Error('ไม่สามารถโหลดรูปภาพ'))
      
      img.src = e.target?.result as string
      img.onload = () => {
        try {
          const canvas = document.createElement('canvas')
          canvas.width = img.width
          canvas.height = img.height
          const ctx = canvas.getContext('2d')
          
          if (!ctx) {
            reject(new Error('ไม่สามารถสร้าง canvas context'))
            return
          }
          
          ctx.drawImage(img, 0, 0)
          canvas.toBlob(
            (blob) => {
              if (!blob) {
                reject(new Error('ไม่สามารถแปลงเป็น WebP'))
                return
              }
              const url = URL.createObjectURL(blob)
              resolve({ blob, url })
            },
            'image/webp',
            qualityValue
          )
        } catch (error) {
          reject(error)
        }
      }
    }
    reader.readAsDataURL(file)
  })

// แปลงทั้งหมด (แบบ parallel)
const convertAll = async () => {
  isProcessing.value = true
  convertedCount.value = 0

  // แปลงแบบ parallel เพื่อความเร็ว
  const promises = images.value.map(async (img) => {
    if (!img.converted && !img.error) {
      try {
        const { blob, url } = await convertToWebp(img.file, quality.value)
        img.converted = true
        img.url = url
        img.webpSize = blob.size
        img.error = false
        img.errorMessage = ''
      } catch (error) {
        img.error = true
        img.errorMessage = error instanceof Error ? error.message : 'เกิดข้อผิดพลาด'
      } finally {
        convertedCount.value++
      }
    }
  })

  await Promise.all(promises)
  isProcessing.value = false

  if (errorCount.value > 0) {
    showSnackbar(`แปลงสำเร็จ ${successCount.value} ไฟล์, ล้มเหลว ${errorCount.value} ไฟล์`, 'warning')
  } else {
    showSnackbar(`แปลงสำเร็จทั้งหมด ${successCount.value} ไฟล์!`, 'success')
  }
}

// ดาวน์โหลดทั้งหมด (zip)
const downloadAll = async () => {
  try {
    const zip = new JSZip()
    let addedFiles = 0

    for (const img of images.value) {
      if (img.converted && img.url && !img.error) {
        const res = await fetch(img.url)
        const blob = await res.blob()
        zip.file(img.name.replace(/\.[^.]+$/, '.webp'), blob)
        addedFiles++
      }
    }

    if (addedFiles === 0) {
      showSnackbar('ไม่มีไฟล์ที่สามารถดาวน์โหลดได้', 'warning')
      return
    }

    const content = await zip.generateAsync({ type: 'blob' })
    const link = document.createElement('a')
    link.href = URL.createObjectURL(content)
    link.download = 'images-webp.zip'
    link.click()
    
    showSnackbar(`ดาวน์โหลด ${addedFiles} ไฟล์เรียบร้อย!`, 'success')
  } catch (error) {
    showSnackbar('เกิดข้อผิดพลาดในการสร้างไฟล์ ZIP', 'error')
  }
}

// ลบรูป
const removeImage = (index: number) => {
  const image = images.value[index]
  if (image && image.url) {
    URL.revokeObjectURL(image.url)
  }
  if (image && image.preview) {
    URL.revokeObjectURL(image.preview)
  }
  images.value.splice(index, 1)
  
  showSnackbar('ลบไฟล์เรียบร้อย', 'info')
}

// รีเซ็ตใหม่
const resetAll = () => {
  // Clean up object URLs
  images.value.forEach(img => {
    if (img.url) URL.revokeObjectURL(img.url)
    if (img.preview) URL.revokeObjectURL(img.preview)
  })
  files.value = []
  images.value = []
  isProcessing.value = false
  convertedCount.value = 0
  quality.value = 0.8
  
  showSnackbar('รีเซ็ตเรียบร้อย', 'info')
}
</script>

<style scoped>
.upload-hover {
  border: 2px dashed rgb(var(--v-theme-primary)) !important;
  background-color: rgba(var(--v-theme-primary), 0.05) !important;
}

.image-preview-compact {
  position: relative;
  height: 80px;
  overflow: hidden;
  border-radius: 4px 4px 0 0;
}

.preview-img-compact {
  width: 100%;
  height: 100%;
  object-fit: cover;
  transition: transform 0.3s ease;
}

.image-card {
  transition: all 0.3s ease;
  border-radius: 12px !important;
}

.image-card:hover {
  transform: translateY(-2px);
  box-shadow: 0 8px 25px rgba(0,0,0,0.15) !important;
}

.image-card:hover .preview-img-compact {
  transform: scale(1.05);
}

.image-card.converted {
  border: 2px solid rgb(var(--v-theme-success));
}

.image-card.error {
  border: 2px solid rgb(var(--v-theme-error));
}

.action-buttons {
  display: flex;
  flex-wrap: wrap;
  justify-content: center;
  gap: 8px;
}

@media (max-width: 600px) {
  .action-buttons .v-btn {
    flex: 1 1 auto;
    min-width: 140px;
  }
}
</style>