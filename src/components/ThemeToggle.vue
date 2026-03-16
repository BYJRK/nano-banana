<template>
    <button
        @click="cycleTheme"
        :title="themeLabel"
        class="flex items-center gap-1.5 px-3 py-1.5 rounded-lg border-2 border-black/30 bg-white/20 hover:bg-white/35 transition-all text-sm font-bold text-white shadow-sm select-none"
    >
        <span class="text-base leading-none">{{ themeIcon }}</span>
        <span class="hidden sm:inline">{{ themeLabel }}</span>
    </button>
</template>

<script setup lang="ts">
import { ref, computed, onMounted, onUnmounted } from 'vue'

type Theme = 'system' | 'light' | 'dark'

const STORAGE_KEY = 'nano-banana-theme'
const ORDER: Theme[] = ['system', 'light', 'dark']

const theme = ref<Theme>('system')

const themeIcon = computed(() => {
    if (theme.value === 'system') return '🌐'
    if (theme.value === 'light') return '☀️'
    return '🌙'
})

const themeLabel = computed(() => {
    if (theme.value === 'system') return '跟随系统'
    if (theme.value === 'light') return '浅色'
    return '深色'
})

const applyTheme = (t: Theme) => {
    const html = document.documentElement
    if (t === 'dark') {
        html.classList.add('dark')
    } else if (t === 'light') {
        html.classList.remove('dark')
    } else {
        const prefersDark = window.matchMedia('(prefers-color-scheme: dark)').matches
        html.classList.toggle('dark', prefersDark)
    }
}

const cycleTheme = () => {
    const idx = ORDER.indexOf(theme.value)
    theme.value = ORDER[(idx + 1) % ORDER.length]
    localStorage.setItem(STORAGE_KEY, theme.value)
    applyTheme(theme.value)
}

let mediaQuery: MediaQueryList | null = null

const handleSystemChange = () => {
    if (theme.value === 'system') applyTheme('system')
}

onMounted(() => {
    const saved = localStorage.getItem(STORAGE_KEY) as Theme | null
    theme.value = ORDER.includes(saved as Theme) ? (saved as Theme) : 'system'
    applyTheme(theme.value)

    mediaQuery = window.matchMedia('(prefers-color-scheme: dark)')
    mediaQuery.addEventListener('change', handleSystemChange)
})

onUnmounted(() => {
    mediaQuery?.removeEventListener('change', handleSystemChange)
})
</script>
