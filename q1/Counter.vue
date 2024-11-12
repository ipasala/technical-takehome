<template>
    <div>
        <h1>Counter: {{ count }}</h1>
        <button @click="count++">Increment</button>
        <button @click="count--">Decrement</button>
        <button @click="resetCount()">Reset</button>
    </div>
</template>

<script setup>
import { ref } from 'vue';

const defaultCount = 0;
const count = ref(defaultCount);

const resetCount = () => {
    count.value = defaultCount;
}
</script>

<style scoped>
</style>