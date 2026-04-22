<template>
    <div class="dice-wrapper" :class="{ 'shake': props.rolling }">
        <div class="dice shadow-glow" :class="[shapeClass, { 'd6-dots': props.type === 6 }]">
            <template v-if="props.type === 6">
                <!-- Current d6 logic -->
                <div class="d6-inner" :class="dotsLayoutClass">
                    <template v-if="props.value <= 3">
                        <span v-for="face in props.value" class="dot" :key="face"></span>
                    </template>
                    <template v-else-if="props.value == 4">
                        <div class="column"><span class="dot"></span><span class="dot"></span></div>
                        <div class="column"><span class="dot"></span><span class="dot"></span></div>
                    </template>
                    <template v-else-if="props.value == 5">
                        <div class="column"><span class="dot"></span><span class="dot"></span></div>
                        <div class="column justify-center"><span class="dot"></span></div>
                        <div class="column"><span class="dot"></span><span class="dot"></span></div>
                    </template>
                    <template v-else-if="props.value == 6">
                        <div class="column"><span class="dot"></span><span class="dot"></span><span class="dot"></span></div>
                        <div class="column"><span class="dot"></span><span class="dot"></span><span class="dot"></span></div>
                    </template>
                </div>
            </template>
            <template v-else>
                <!-- For non-d6, just show the number in the middle -->
                <span class="dice-number" :class="numberPaddingClass">{{ props.value }}</span>
            </template>
        </div>
    </div>
</template>

<script setup lang="ts">
import { computed } from 'vue';

const props = defineProps<{ 
    value: number;
    type: number;
    rolling?: boolean;
}>();

const shapeClass = computed(() => `shape-d${props.type}`);

const dotsLayoutClass = computed(() => {
    if (props.type !== 6) return '';
    const layouts = ['', 'first', 'second', 'third', 'fourth', 'fifth', 'sixth'];
    return layouts[props.value] || '';
});

const numberPaddingClass = computed(() => {
    if (props.type === 4) return 'pt-6'; // move text down a bit for the triangle
    if (props.type === 10 || props.type === 100) return 'pb-4'; // move text up for kite
    return '';
});

</script>

<style lang="postcss" scoped>
.dice-wrapper {
    margin-right: 10px;
    margin-bottom: 10px;
    filter: drop-shadow(0px 0px 8px #d82091);
}

.shake {
    animation: shake 0.3s infinite;
}

@keyframes shake {
    0% { transform: translate(1px, 1px) rotate(0deg); }
    10% { transform: translate(-1px, -2px) rotate(-1deg); }
    20% { transform: translate(-3px, 0px) rotate(1deg); }
    30% { transform: translate(3px, 2px) rotate(0deg); }
    40% { transform: translate(1px, -1px) rotate(1deg); }
    50% { transform: translate(-1px, 2px) rotate(-1deg); }
    60% { transform: translate(-3px, 1px) rotate(0deg); }
    70% { transform: translate(3px, 1px) rotate(-1deg); }
    80% { transform: translate(-1px, -1px) rotate(1deg); }
    90% { transform: translate(1px, 2px) rotate(0deg); }
    100% { transform: translate(1px, -2px) rotate(-1deg); }
}

.dice {
    width: 104px;
    height: 104px;
    background-color: #fff;
    display: flex;
    justify-content: center;
    align-items: center;
    padding: 16px;
    position: relative;
    color: #4a1d96;
    transition: transform 0.2s ease-in-out;
}

/* D6 layout specifics */
.d6-dots {
    border-radius: 12px;
}
.d6-inner {
    width: 100%;
    height: 100%;
}

.first { display: flex; justify-content: center; align-items: center; }
.second { display: flex; justify-content: space-between; }
.second .dot:nth-of-type(2) { align-self: flex-end; }
.third { display: flex; justify-content: space-between; }
.third .dot:nth-of-type(1) { align-self: flex-end; }
.third .dot:nth-of-type(2) { align-self: center; }
.fourth, .sixth, .fifth { display: flex; justify-content: space-between; }
.fourth .column, .sixth .column, .fifth .column { display: flex; flex-direction: column; justify-content: space-between; }
.fifth .justify-center { justify-content: center; }

.dot {
    display: block;
    width: 18px;
    height: 18px;
    border-radius: 50%;
    background-color: #000;
}

.dice-number {
    font-size: 2.75rem;
    font-weight: 800;
    line-height: 1;
    z-index: 10;
}

/* Shapes using clip-path */
.shape-d4 { clip-path: polygon(50% 0%, 0% 100%, 100% 100%); }
.shape-d8 { clip-path: polygon(50% 0%, 100% 50%, 50% 100%, 0% 50%); }
.shape-d10 { clip-path: polygon(50% 0%, 100% 35%, 50% 100%, 0% 35%); }
.shape-d12 { clip-path: polygon(50% 0%, 100% 38%, 82% 100%, 18% 100%, 0% 38%); }
.shape-d20 { clip-path: polygon(50% 0%, 100% 25%, 100% 75%, 50% 100%, 0% 75%, 0% 25%); }
.shape-d100 { clip-path: circle(48% at 50% 50%); }

</style>
