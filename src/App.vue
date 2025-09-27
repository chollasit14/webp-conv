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
                    indeterminate
                    color="primary"
                    size="60"
                    class="mb-4"
                  />
                  <h3 class="text-h5">กำลังแปลงไฟล์...</h3>
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
                    แปลงไฟล์สำเร็จ {{ images.length }} ไฟล์
                  </p>
                </div>
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
                      :class="{ 'converted': img.converted }"
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
                          :model-value="img.converted"
                          class="align-center justify-center"
                          contained
                        >
                          <v-icon size="24" color="success">
                            mdi-check-circle
                          </v-icon>
                        </v-overlay>
                      </div>
                      
                      <!-- File Info -->
                      <v-card-text class="pa-2">
                        <div class="text-body-1 font-weight-medium mb-1 text-truncate">
                          {{ img.name }}
                        </div>
                        <div class="text-body-2 text-grey-darken-1 mb-2">
                          {{ (img.size / 1024).toFixed(1) }} KB
                        </div>
                        
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
                :disabled="!convertedAll"
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
                        แปลงไฟล์ได้อย่างรวดเร็วใน
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
}

const files = ref<File[]>([])
const images = ref<ImageItem[]>([])
const isDragging = ref(false)
const isProcessing = ref(false)
const fileInput = ref<InstanceType<typeof VFileInput>>()
const additionalFileInput = ref<HTMLInputElement>()

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
  for (const file of fileList) {
    if (file.type.startsWith('image/')) {
      const reader = new FileReader()
      reader.onload = (e) => {
        images.value.push({
          file,
          name: file.name,
          size: file.size,
          converted: false,
          url: null,
          preview: e.target?.result as string
        })
      }
      reader.readAsDataURL(file)
    }
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
  }
}

// ฟังก์ชันแปลงไฟล์เป็น WebP
const convertToWebp = (file: File): Promise<{ blob: Blob; url: string }> =>
  new Promise((resolve) => {
    const reader = new FileReader()
    reader.onload = (e) => {
      const img = new Image()
      img.src = e.target?.result as string
      img.onload = () => {
        const canvas = document.createElement('canvas')
        canvas.width = img.width
        canvas.height = img.height
        const ctx = canvas.getContext('2d')
        if (!ctx) return
        ctx.drawImage(img, 0, 0)
        canvas.toBlob(
          (blob) => {
            if (!blob) return
            const url = URL.createObjectURL(blob)
            resolve({ blob, url })
          },
          'image/webp',
          0.8
        )
      }
    }
    reader.readAsDataURL(file)
  })

// แปลงทั้งหมด
const convertAll = async () => {
  isProcessing.value = true
  for (const img of images.value) {
    if (!img.converted) {
      const { url } = await convertToWebp(img.file)
      img.converted = true
      img.url = url
    }
  }
  isProcessing.value = false
}

const convertedAll = computed(() =>
  images.value.length > 0 && images.value.every((i) => i.converted)
)

// ดาวน์โหลดทั้งหมด (zip)
const downloadAll = async () => {
  const zip = new JSZip()
  for (const img of images.value) {
    if (img.converted && img.url) {
      const res = await fetch(img.url)
      const blob = await res.blob()
      zip.file(img.name.replace(/\.[^.]+$/, '.webp'), blob)
    }
  }
  const content = await zip.generateAsync({ type: 'blob' })
  const link = document.createElement('a')
  link.href = URL.createObjectURL(content)
  link.download = 'images-webp.zip'
  link.click()
}

// ลบรูป
const removeImage = (index: number) => {
  const image = images.value[index]
  if (image && image.url) {
    URL.revokeObjectURL(image.url)
  }
  images.value.splice(index, 1)
}

// รีเซ็ตใหม่
const resetAll = () => {
  // Clean up object URLs
  images.value.forEach(img => {
    if (img.url) URL.revokeObjectURL(img.url)
  })
  files.value = []
  images.value = []
  isProcessing.value = false
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

.image-card:hover .preview-img-small {
  transform: scale(1.05);
}

.gap-2 > * + * {
  margin-top: 8px;
}

.ga-2 {
  gap: 8px;
}

.image-card.converted {
  border: 2px solid rgb(var(--v-theme-success));
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