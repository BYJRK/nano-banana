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
                        <img :src="img" alt="生成的艺术作品" class="w-full max-h-[400px] rounded-lg border-2 border-black shadow-lg object-contain cursor-pointer" @click="previewImage = img" @load="e => onImageLoad(e, img)" />
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

    <!-- Fullscreen Preview Overlay -->
    <Teleport to="body">
        <Transition name="fade" @after-leave="resetZoom">
            <div v-if="previewImage" class="fixed inset-0 z-50 bg-black/80 backdrop-blur-sm" @click.self="previewImage = null" @wheel.prevent="onWheel">
                <button @click="previewImage = null" class="absolute top-4 right-4 text-white text-3xl font-bold hover:text-yellow-300 transition-colors z-10 leading-none">&times;</button>
                <!-- Zoom hint -->
                <div class="absolute top-4 left-4 text-white/70 text-lg font-medium z-10 select-none pointer-events-none">
                    {{ Math.round(scale * 100) }}%
                </div>
                <!-- Image container -->
                <div ref="containerRef" class="w-full h-full overflow-hidden flex items-center justify-center"
                    @mousedown.prevent="onDragStart" @mousemove="onDragMove" @mouseup="onDragEnd" @mouseleave="onDragEnd"
                    :style="{ cursor: scale > 1 ? (dragging ? 'grabbing' : 'grab') : 'default' }">
                    <img :src="previewImage" alt="全屏预览" ref="previewImgRef"
                        class="max-w-[90vw] max-h-[90vh] object-contain rounded-lg shadow-2xl select-none"
                        :style="imageTransformStyle" draggable="false" @load="onPreviewImgLoad" />
                </div>
                <!-- Minimap -->
                <Transition name="fade">
                    <div v-if="scale > 1 && previewImgNatural.w" ref="minimapRef"
                        class="absolute bottom-4 right-4 border border-white/40 rounded overflow-hidden shadow-lg z-10 bg-black/60 select-none"
                        :style="minimapContainerStyle"
                        @mousedown.prevent.stop="onMinimapDragStart"
                        @mousemove.stop="onMinimapDragMove"
                        @mouseup.stop="onMinimapDragEnd"
                        @mouseleave.stop="onMinimapDragEnd">
                        <img :src="previewImage" class="block w-full h-full object-contain pointer-events-none" draggable="false" />
                        <div class="absolute border-2 border-yellow-300 bg-yellow-300/20 rounded-sm shadow-[0_0_4px_rgba(253,224,71,0.5)]" :style="minimapViewportStyle" />
                    </div>
                </Transition>
            </div>
        </Transition>
    </Teleport>
</template>

<script setup lang="ts">
import { ref, computed, watch, onMounted, onUnmounted, nextTick } from 'vue'

const props = defineProps<{
    results: string[]
    loading: boolean
    error: string | null
    canPush: boolean
    showCancel: boolean
}>()

const imageSizes = ref<Record<string, string>>({})
const previewImage = ref<string | null>(null)

// Zoom & pan state
const scale = ref(1)
const panX = ref(0)
const panY = ref(0)
const dragging = ref(false)
const dragStart = ref({ x: 0, y: 0 })
const panStart = ref({ x: 0, y: 0 })
const containerRef = ref<HTMLElement | null>(null)
const previewImgRef = ref<HTMLImageElement | null>(null)
const previewImgNatural = ref({ w: 0, h: 0 })

const MIN_SCALE = 0.5
const MAX_SCALE = 10
const MINIMAP_MAX = 160

const resetZoom = () => {
    scale.value = 1
    panX.value = 0
    panY.value = 0
    previewImgNatural.value = { w: 0, h: 0 }
}

const onPreviewImgLoad = () => {
    const img = previewImgRef.value
    if (img) {
        previewImgNatural.value = { w: img.naturalWidth, h: img.naturalHeight }
    }
}

// Clamp pan so image doesn't fly off screen
const clampPan = (s: number, px: number, py: number) => {
    if (s <= 1) return { x: 0, y: 0 }
    const el = previewImgRef.value
    if (!el) return { x: px, y: py }
    const rect = el.getBoundingClientRect()
    const baseW = rect.width / scale.value
    const baseH = rect.height / scale.value
    const maxPanX = (baseW * (s - 1)) / 2
    const maxPanY = (baseH * (s - 1)) / 2
    return {
        x: Math.max(-maxPanX, Math.min(maxPanX, px)),
        y: Math.max(-maxPanY, Math.min(maxPanY, py)),
    }
}

const onWheel = (e: WheelEvent) => {
    const factor = e.deltaY < 0 ? 1.15 : 1 / 1.15
    const newScale = Math.min(MAX_SCALE, Math.max(MIN_SCALE, scale.value * factor))
    // Zoom toward cursor position
    const el = previewImgRef.value
    if (el) {
        const rect = el.getBoundingClientRect()
        const cx = rect.left + rect.width / 2
        const cy = rect.top + rect.height / 2
        const dx = e.clientX - cx
        const dy = e.clientY - cy
        const ratio = 1 - newScale / scale.value
        const newPanX = panX.value + dx * ratio
        const newPanY = panY.value + dy * ratio
        const clamped = clampPan(newScale, newPanX, newPanY)
        panX.value = clamped.x
        panY.value = clamped.y
    }
    scale.value = newScale
}

const onDragStart = (e: MouseEvent) => {
    if (scale.value <= 1) return
    dragging.value = true
    dragStart.value = { x: e.clientX, y: e.clientY }
    panStart.value = { x: panX.value, y: panY.value }
}
const onDragMove = (e: MouseEvent) => {
    if (!dragging.value) return
    const dx = e.clientX - dragStart.value.x
    const dy = e.clientY - dragStart.value.y
    const clamped = clampPan(scale.value, panStart.value.x + dx, panStart.value.y + dy)
    panX.value = clamped.x
    panY.value = clamped.y
}
const onDragEnd = () => {
    dragging.value = false
}

const imageTransformStyle = computed(() => ({
    transform: `translate(${panX.value}px, ${panY.value}px) scale(${scale.value})`,
    transformOrigin: 'center center',
    transition: dragging.value ? 'none' : 'transform 0.1s ease-out',
}))

// Minimap computations
const minimapContainerStyle = computed(() => {
    const { w, h } = previewImgNatural.value
    if (!w || !h) return { width: '0px', height: '0px' }
    const ratio = w / h
    let mw: number, mh: number
    if (ratio >= 1) {
        mw = MINIMAP_MAX
        mh = MINIMAP_MAX / ratio
    } else {
        mh = MINIMAP_MAX
        mw = MINIMAP_MAX * ratio
    }
    return { width: `${mw}px`, height: `${mh}px` }
})

const minimapViewportStyle = computed(() => {
    const el = previewImgRef.value
    const { w: natW, h: natH } = previewImgNatural.value
    if (!el || !natW || !natH) return {}
    const rect = el.getBoundingClientRect()
    const baseW = rect.width / scale.value
    const baseH = rect.height / scale.value
    // Viewport size relative to full image
    const vpW = baseW / scale.value
    const vpH = baseH / scale.value
    // Viewport center offset in image coords
    const vpCx = baseW / 2 - panX.value / scale.value
    const vpCy = baseH / 2 - panY.value / scale.value
    // Convert to minimap %
    const ratio = natW / natH
    const mw = ratio >= 1 ? MINIMAP_MAX : MINIMAP_MAX * ratio
    const mh = ratio >= 1 ? MINIMAP_MAX / ratio : MINIMAP_MAX
    const scaleToMinimap = mw / baseW
    const left = (vpCx - vpW / 2) * scaleToMinimap
    const top = (vpCy - vpH / 2) * scaleToMinimap
    const width = vpW * scaleToMinimap
    const height = vpH * scaleToMinimap
    return {
        left: `${Math.max(0, left)}px`,
        top: `${Math.max(0, top)}px`,
        width: `${Math.min(mw, width)}px`,
        height: `${Math.min(mh, height)}px`,
    }
})

// Minimap drag-to-pan
const minimapRef = ref<HTMLElement | null>(null)
const minimapDragging = ref(false)

const onMinimapDragStart = (e: MouseEvent) => {
    minimapDragging.value = true
    minimapMouseToPan(e)
}
const onMinimapDragMove = (e: MouseEvent) => {
    if (!minimapDragging.value) return
    minimapMouseToPan(e)
}
const onMinimapDragEnd = () => {
    minimapDragging.value = false
}

const minimapMouseToPan = (e: MouseEvent) => {
    const mm = minimapRef.value
    const el = previewImgRef.value
    if (!mm || !el) return
    const mmRect = mm.getBoundingClientRect()
    const imgRect = el.getBoundingClientRect()
    const baseW = imgRect.width / scale.value
    const baseH = imgRect.height / scale.value
    const { w: natW, h: natH } = previewImgNatural.value
    const ratio = natW / natH
    const mw = ratio >= 1 ? MINIMAP_MAX : MINIMAP_MAX * ratio
    const scaleToMinimap = mw / baseW
    const mx = e.clientX - mmRect.left
    const my = e.clientY - mmRect.top
    const imgX = mx / scaleToMinimap - baseW / 2
    const imgY = my / scaleToMinimap - baseH / 2
    const newPanX = -imgX * scale.value
    const newPanY = -imgY * scale.value
    const clamped = clampPan(scale.value, newPanX, newPanY)
    panX.value = clamped.x
    panY.value = clamped.y
}

const onKeydown = (e: KeyboardEvent) => {
    if (e.key === 'Escape') previewImage.value = null
}
onMounted(() => window.addEventListener('keydown', onKeydown))
onUnmounted(() => window.removeEventListener('keydown', onKeydown))

watch(previewImage, (v) => {
    if (v) resetZoom()
})

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

<style scoped>
.fade-enter-active,
.fade-leave-active {
    transition: opacity 0.2s ease;
}
.fade-enter-from,
.fade-leave-to {
    opacity: 0;
}
</style>
