<script setup lang="ts">
interface Props {
  title: string
  items: string[]
}

withDefaults(defineProps<Props>(), {
  items: () => []
})
</script>

<template>
  <section v-if="items.length > 0" class="pt-2">
    <h2 class="text-2xl font-bold text-base-content mb-6 flex items-center gap-2">
      <Icon name="heroicons:sparkles-20-solid" class="text-primary w-6 h-6" aria-hidden="true" />
      {{ title }}
    </h2>
    <div class="grid sm:grid-cols-2 gap-4">
      <div 
        v-for="(item, index) in items" 
        :key="index"
        class="bg-base-200/40 p-5 rounded-2xl border border-base-content/10 flex items-center gap-4"
      >
        <div class="shrink-0">
          <Icon 
            name="heroicons:check-circle-20-solid" 
            class="text-success w-5 h-5" 
          />
        </div>
        <span class="text-base-content font-medium leading-tight">
          {{ item }}
        </span>
      </div>
    </div>
  </section>
</template>
