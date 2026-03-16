<template>
    <div class="bg-white dark:bg-gray-800 border-4 border-black border-t-0 rounded-b-lg p-4 shadow-lg h-full flex flex-col">
        <!-- Upload Area -->
        <div
            ref="uploadArea"
            @click="fileInput?.click()"
            @dragenter.prevent="handleDragEnter"
            @dragover.prevent="handleDragOver"
            @dragleave.prevent="handleDragLeave"
            @drop.prevent="handleDrop"
            :class="[
                'border-4 border-dashed rounded-lg p-6 text-center cursor-pointer transition-all duration-300 flex-1 flex flex-col justify-center',
                isDragOver ? 'border-pink-400 bg-pink-50 dark:bg-pink-950' : 'border-gray-300 dark:border-gray-600 bg-gray-50 dark:bg-gray-700 hover:border-pink-400 hover:bg-pink-50 dark:hover:bg-pink-950'
            ]"
        >
            <input ref="fileInput" type="file" accept="image/*" multiple class="hidden" @change="handleFileSelect" />

            <!-- Upload Icon -->
            <div class="mb-4">
                <div class="w-12 h-12 bg-black rounded-lg mx-auto flex items-center justify-center">
                    <svg class="w-6 h-6 text-white" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                        <path
                            stroke-linecap="round"
                            stroke-linejoin="round"
                            stroke-width="2"
                            d="M7 16a4 4 0 01-.88-7.903A5 5 0 1115.9 6L16 6a5 5 0 011 9.9M15 13l-3-3m0 0l-3 3m3-3v12"
                        />
                    </svg>
                </div>
            </div>

            <h3 class="text-lg font-bold mb-2 flex items-center justify-center gap-2 dark:text-gray-100">🍌 拖拽上传</h3>
            <p class="text-gray-600 dark:text-gray-400 mb-1">或点击浏览文件</p>
            <p class="text-sm text-gray-500 dark:text-gray-400">支持多张图片 JPG, PNG, GIF 格式 (最大 5MB)</p>
        </div>

        <!-- Thumbnails -->
        <div v-if="thumbnails.length > 0" class="grid grid-cols-4 gap-3 mt-4">
            <div
                v-for="(thumbnail, index) in thumbnails"
                :key="index"
                class="relative aspect-square bg-gray-100 dark:bg-gray-700 rounded-lg overflow-hidden group border-2 transition-all duration-150 cursor-grab active:cursor-grabbing"
                :class="[
                    dragOverIndex === index && dragItemIndex !== index
                        ? 'border-pink-400 scale-105 shadow-lg'
                        : 'border-black',
                    dragItemIndex === index ? 'opacity-40' : ''
                ]"
                draggable="true"
                @dragstart="onThumbnailDragStart(index, $event)"
                @dragover.prevent.stop="onThumbnailDragOver(index)"
                @drop.prevent.stop="onThumbnailDrop(index)"
                @dragend="onThumbnailDragEnd"
            >
                <img :src="thumbnail" :alt="`Image ${index + 1}`" class="w-full h-full object-cover pointer-events-none" />
                <!-- drag handle indicator -->
                <div class="absolute bottom-1 left-1/2 -translate-x-1/2 text-white text-xs bg-black/50 rounded px-1 opacity-0 group-hover:opacity-100 transition-opacity select-none pointer-events-none">
                    ⠿
                </div>
                <button
                    @click="removeThumbnail(index)"
                    class="absolute top-1 right-1 w-6 h-6 bg-red-500 text-white rounded-full flex items-center justify-center text-sm font-bold opacity-0 group-hover:opacity-100 transition-opacity hover:bg-red-600"
                >
                    ×
                </button>
            </div>
        </div>
    </div>
</template>

<script setup lang="ts">
import { ref, computed } from 'vue'

const props = defineProps<{
    modelValue: string[]
}>()

const emit = defineEmits<{
    'update:modelValue': [value: string[]]
}>()

const fileInput = ref<HTMLInputElement>()
const uploadArea = ref<HTMLElement>()
const isDragOver = ref(false)
const dragItemIndex = ref<number | null>(null)
const dragOverIndex = ref<number | null>(null)

const thumbnails = computed(() => props.modelValue)

const handleFileSelect = (event: Event) => {
    const target = event.target as HTMLInputElement
    if (target.files) {
        handleFiles(Array.from(target.files))
        // 重置input的value，允许重新上传相同的文件
        target.value = ''
    }
}

const handleDragEnter = () => {
    isDragOver.value = true
}

const handleDragOver = () => {
    isDragOver.value = true
}

const handleDragLeave = (event: DragEvent) => {
    if (!uploadArea.value?.contains(event.relatedTarget as Node)) {
        isDragOver.value = false
    }
}

const handleDrop = (event: DragEvent) => {
    isDragOver.value = false
    if (event.dataTransfer?.files) {
        handleFiles(Array.from(event.dataTransfer.files))
    }
}

const handleFiles = (files: File[]) => {
    const imageFiles = files.filter(file => file.type.startsWith('image/'))

    imageFiles.forEach(file => {
        const reader = new FileReader()
        reader.onload = e => {
            if (e.target?.result) {
                const newImages = [...props.modelValue, e.target.result as string]
                emit('update:modelValue', newImages)
            }
        }
        reader.readAsDataURL(file)
    })
}

const removeThumbnail = (index: number) => {
    const newImages = props.modelValue.filter((_, i) => i !== index)
    emit('update:modelValue', newImages)
}

const onThumbnailDragStart = (index: number, event: DragEvent) => {
    dragItemIndex.value = index
    if (event.dataTransfer) {
        event.dataTransfer.effectAllowed = 'move'
        event.dataTransfer.setData('text/plain', String(index))
    }
}

const onThumbnailDragOver = (index: number) => {
    dragOverIndex.value = index
}

const onThumbnailDrop = (index: number) => {
    if (dragItemIndex.value === null || dragItemIndex.value === index) return
    const newImages = [...props.modelValue]
    const [moved] = newImages.splice(dragItemIndex.value, 1)
    newImages.splice(index, 0, moved)
    emit('update:modelValue', newImages)
    dragItemIndex.value = null
    dragOverIndex.value = null
}

const onThumbnailDragEnd = () => {
    dragItemIndex.value = null
    dragOverIndex.value = null
}
</script>
