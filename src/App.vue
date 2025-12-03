<script setup>
import { ref } from 'vue';
import SnapdomDemo from './components/SnapdomDemo.vue';
import Html2CanvasDemo from './components/Html2CanvasDemo.vue';

const currentTab = ref('snapdom');
</script>

<template>
  <div class="app-container">
    <header class="app-header">
      <h1>海报生成方案对比</h1>
      <div class="tabs">
        <button 
          :class="{ active: currentTab === 'snapdom' }" 
          @click="currentTab = 'snapdom'"
        >
          Snapdom 演示
        </button>
        <button 
          :class="{ active: currentTab === 'html2canvas' }" 
          @click="currentTab = 'html2canvas'"
        >
          Html2Canvas 演示
        </button>
      </div>
    </header>

    <main>
      <SnapdomDemo v-if="currentTab === 'snapdom'" />
      <Html2CanvasDemo v-if="currentTab === 'html2canvas'" />
    </main>
  </div>
</template>

<style>
/* Global Reset */
* {
  box-sizing: border-box;
  margin: 0;
  padding: 0;
}

body {
  background-color: #f3f4f6;
  color: #1f2937;
  font-family: 'Inter', sans-serif;
}

.app-container {
  min-height: 100vh;
}

.app-header {
  background: white;
  padding: 1rem 2rem;
  box-shadow: 0 1px 3px 0 rgba(0, 0, 0, 0.1);
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 2rem;
}

h1 {
  font-size: 1.5rem;
  font-weight: 700;
  color: #111827;
}

.tabs {
  display: flex;
  gap: 1rem;
}

.tabs button {
  padding: 0.5rem 1rem;
  border: none;
  background: transparent;
  font-size: 1rem;
  font-weight: 500;
  color: #6b7280;
  cursor: pointer;
  border-radius: 6px;
  transition: all 0.2s;
}

.tabs button:hover {
  background: #f3f4f6;
  color: #374151;
}

.tabs button.active {
  background: #e0e7ff;
  color: #4f46e5;
}
</style>
