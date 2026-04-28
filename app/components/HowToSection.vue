<script setup lang="ts">
interface HowToStep {
  title: string
  description: string
}

interface Props {
  title: string
  description?: string
  items: HowToStep[]
}

withDefaults(defineProps<Props>(), {
  items: () => []
})
</script>

<template>
  <section v-if="items.length > 0" class="pt-2">
    <h2 class="text-2xl font-bold text-base-content mb-6 flex items-center gap-2">
      <Icon name="heroicons:play-circle-20-solid" class="text-primary w-6 h-6" aria-hidden="true" />
      {{ title }}
    </h2>
    <p v-if="description" class="mb-6 text-base-content/80 leading-relaxed">
      {{ description }}
    </p>
    <div class="grid md:grid-cols-3 gap-5">
      <div 
        v-for="(item, index) in items" 
        :key="index"
        class="flex flex-col gap-2 bg-base-200/40 p-5 rounded-2xl border border-primary/20"
      >
        <span class="text-4xl font-black text-primary/25 leading-none">{{ index + 1 }}</span>
        <h3 class="font-bold text-base-content">{{ item.title }}</h3>
        <p class="text-sm text-base-content/70 leading-relaxed">
          {{ item.description }}
        </p>
      </div>
    </div>
  </section>
</template>
