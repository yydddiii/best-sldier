<template>
    <div style="width: 600px; margin: 0 auto">
        <slider
            draggable="base"
            :slides-per-view="3"
            :space-between="10"
            loop
            show-arrows
            centered-slides
            :current-slide-style="{
                width: '500px',
            }"
        >
            <div
                v-for="i in 10"
                style=""
                :style="{
                    background: colors[i - 1],
                }"
                class="test-slide"
            >
                {{ i - 1 }}
            </div>
        </slider>
    </div>
</template>

<script setup>
const real_index = ref(0);

const colors = [
    "#FF6B6B",
    "#4ECDC4",
    "#45B7D1",
    "#96CEB4",
    "#FFEAA7",
    "#DDA0DD",
    "#98D8C8",
    "#F7DC6F",
    "#BB8FCE",
    "#85C1E9",
];
</script>

<style>
.test-slide {
    display: flex;
    justify-content: center;
    align-items: center;
    height: 400px;
    width: 100%;
}
</style>
