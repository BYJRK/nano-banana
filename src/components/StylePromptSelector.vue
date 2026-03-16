<template>
    <div class="bg-white dark:bg-gray-800 border-4 border-black border-t-0 rounded-b-lg p-4 shadow-lg h-full flex flex-col">
        <!-- 选择模式的标签页 -->
        <div class="flex mb-4 bg-gray-100 dark:bg-gray-700 rounded-lg p-1 border-2 border-black">
            <button
                @click="activeTab = 'style'"
                :class="[
                    'flex-1 py-2 px-3 rounded-md font-bold transition-all flex items-center justify-center gap-2',
                    activeTab === 'style' ? 'bg-yellow-300 text-black' : 'text-gray-600 dark:text-gray-400 hover:text-black dark:hover:text-white'
                ]"
            >
                🎨 预设风格
            </button>
            <button
                @click="activeTab = 'custom'"
                :class="[
                    'flex-1 py-2 px-3 rounded-md font-bold transition-all flex items-center justify-center gap-2',
                    activeTab === 'custom' ? 'bg-yellow-300 text-black' : 'text-gray-600 dark:text-gray-400 hover:text-black dark:hover:text-white'
                ]"
            >
                ✏️ 自定义提示词
            </button>
        </div>

        <!-- 预设风格选择 -->
        <div v-if="activeTab === 'style'" class="space-y-2 flex-1 overflow-y-auto">
            <div
                v-for="template in templates"
                :key="template.id"
                @click="selectStyle(template.id)"
                :class="[
                    'p-4 rounded-lg border-2 border-black cursor-pointer transition-all',
                    selectedStyle === template.id ? 'bg-yellow-300 border-orange-500' : 'bg-yellow-50 dark:bg-gray-700 hover:bg-yellow-100 dark:hover:bg-gray-600'
                ]"
            >
                <div class="flex items-start gap-3">
                    <!-- 缩略图 -->
                    <img v-if="template.image" :src="template.image" :alt="template.title" class="w-20 h-20 rounded border-2 border-black object-cover flex-shrink-0" />

                    <!-- 内容 -->
                    <div class="flex-1 min-w-0">
                        <div class="text-base font-bold mb-1 dark:text-gray-100">{{ template.title }}</div>
                        <p class="text-sm text-gray-600 dark:text-gray-400 mb-2">{{ template.description }}</p>

                        <!-- 可展开的提示词预览 -->
                        <details class="group">
                            <summary class="cursor-pointer text-xs text-gray-500 dark:text-gray-400 hover:text-gray-700 dark:hover:text-gray-200 flex items-center gap-1">
                                <span>查看完整提示词</span>
                                <svg class="w-3 h-3 transition-transform group-open:rotate-180" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                                    <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M19 9l-7 7-7-7" />
                                </svg>
                            </summary>
                            <div class="mt-2 p-2 bg-gray-100 dark:bg-gray-600 rounded text-xs text-gray-700 dark:text-gray-300 border dark:border-gray-500">
                                {{ template.prompt }}
                            </div>
                        </details>
                    </div>
                </div>
            </div>
        </div>

        <!-- 自定义提示词 -->
        <div v-else class="flex flex-col gap-3 flex-1">
            <label class="font-bold flex items-center gap-2 dark:text-gray-100">🍌 描述你的创意想法：</label>

            <textarea
                :value="customPrompt"
                @input="updateCustomPrompt(($event.target as HTMLTextAreaElement).value)"
                placeholder="例如：将图片转换为超现实主义风格，加入漂浮的香蕉和鲜艳的色彩..."
                class="w-full px-4 py-3 border-2 border-black dark:border-gray-600 rounded-lg resize-none focus:outline-none focus:ring-2 focus:ring-orange-500 focus:border-transparent flex-1 min-h-[160px] dark:bg-gray-700 dark:text-gray-100 dark:placeholder-gray-500"
            />

            <p class="text-sm text-gray-600 dark:text-gray-400 font-medium flex items-center gap-1">💡 描述越具体，效果越好！</p>
        </div>
    </div>
</template>

<script setup lang="ts">
import { ref, watch } from 'vue'
import type { StyleTemplate } from '../types'

const props = defineProps<{
    selectedStyle: string
    customPrompt: string
    templates: StyleTemplate[]
}>()

const emit = defineEmits<{
    'update:selectedStyle': [value: string]
    'update:customPrompt': [value: string]
}>()

const activeTab = ref<'style' | 'custom'>('style')

// 监听选择状态，自动切换标签页
watch(
    () => props.selectedStyle,
    newValue => {
        if (newValue && activeTab.value !== 'style') {
            activeTab.value = 'style'
        }
    }
)

watch(
    () => props.customPrompt,
    newValue => {
        if (newValue && activeTab.value !== 'custom') {
            activeTab.value = 'custom'
        }
    }
)

const selectStyle = (styleId: string) => {
    // 选择风格时清空自定义提示词
    emit('update:customPrompt', '')
    emit('update:selectedStyle', props.selectedStyle === styleId ? '' : styleId)
}

const updateCustomPrompt = (value: string) => {
    // 输入自定义提示词时清空风格选择
    if (value) {
        emit('update:selectedStyle', '')
    }
    emit('update:customPrompt', value)
}
</script>
