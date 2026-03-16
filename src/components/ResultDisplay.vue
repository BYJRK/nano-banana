<template>
    <div class="bg-white dark:bg-gray-800 border-4 border-black dark:border-zinc-600 border-t-0 rounded-b-lg p-4 shadow-lg min-h-[400px] flex flex-col">
        <div class="flex-1 bg-gray-50 dark:bg-gray-900 border-2 border-black dark:border-zinc-700 rounded-lg p-6 flex items-center justify-center">
            <!-- Loading State -->
            <div v-if="loading" class="text-center">
                <div class="w-12 h-12 border-4 border-yellow-300 border-t-orange-500 rounded-full animate-spin mx-auto mb-4" />
                <p class="font-bold text-base flex items-center justify-center gap-2 dark:text-gray-100">
                    🍌 正在创造魔法...
                    <button
                        v-if="showCancel"
                        @click="$emit('cancel')"
                        class="text-sm text-blue-600 underline hover:text-blue-700"
                    >
                        取消
                    </button>
                </p>
                <p class="text-gray-600 dark:text-gray-400">请稍等片刻</p>
            </div>

            <!-- Error State -->
            <div v-else-if="error" class="text-center">
                <div class="text-red-500 text-6xl mb-4">🍌💥</div>
                <p class="text-red-600 font-bold text-base mb-2">哎呀！出了点问题</p>
                <p class="text-gray-600 dark:text-gray-400 text-sm">{{ error }}</p>
            </div>

            <!-- Result Images -->
            <div v-else-if="results && results.length > 0" class="w-full">
                <div class="grid gap-4" :class="gridClass">
                    <div v-for="(img, index) in results" :key="`${img}-${index}`" class="relative group">
                        <img :src="img" alt="生成的艺术作品" class="w-full rounded-lg border-2 border-black shadow-lg object-contain" @load="e => onImageLoad(e, img)" />
                        <div v-if="imageSizes[img]" class="mt-1 text-center text-xs text-gray-500 dark:text-gray-400 font-mono">
                            {{ imageSizes[img] }}
                        </div>
                        <div class="absolute bottom-2 right-2 flex gap-2 opacity-0 group-hover:opacity-100 transition-opacity">
                            <button
                                v-if="canPush"
                                @click="$emit('push', img)"
                                class="px-3 py-1 bg-green-300 text-black font-bold border-2 border-black rounded-lg shadow-lg hover:bg-green-400 text-sm"
                            >
                                🎨 二次创作
                            </button>
                            <button
                                @click="$emit('download', img)"
                                class="px-3 py-1 bg-yellow-300 text-black font-bold border-2 border-black rounded-lg shadow-lg hover:bg-yellow-400 text-sm"
                            >
                                ⬇️ 下载
                            </button>
                        </div>
                    </div>
                </div>
            </div>

            <!-- Empty State -->
            <div v-else class="text-center">
                <div class="w-12 h-12 border-4 border-gray-300 rounded-lg mx-auto mb-4 flex items-center justify-center">
                    <span class="text-2xl">🍌</span>
                </div>
                <h3 class="font-bold text-base mb-2 flex items-center justify-center gap-2 dark:text-gray-100">🍌 等待魔法开始...</h3>
                <p class="text-gray-600 dark:text-gray-400">上传图片并选择风格开始创作</p>
            </div>
        </div>
    </div>
</template>

<script setup lang="ts">
import { ref, computed, watch } from 'vue'

const props = defineProps<{
    results: string[]
    loading: boolean
    error: string | null
    canPush: boolean
    showCancel: boolean
}>()

const imageSizes = ref<Record<string, string>>({})

watch(
    () => props.results,
    () => {
        imageSizes.value = {}
    },
    { deep: true }
)

const gridClass = computed(() => {
    const count = props.results.length
    if (count === 1) return 'grid-cols-1'
    if (count === 2) return 'grid-cols-2'
    if (count <= 4) return 'grid-cols-2'
    return 'grid-cols-3'
})

const onImageLoad = (event: Event, image: string) => {
    const img = event.currentTarget as HTMLImageElement | null
    if (img?.naturalWidth && img.naturalHeight) {
        imageSizes.value[image] = `${img.naturalWidth} × ${img.naturalHeight}`
    }
}

defineEmits<{
    download: [image: string]
    push: [image: string]
    cancel: []
}>()
</script>
