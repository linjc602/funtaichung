<script setup>
import { ref } from 'vue';
import HelloWorld from './components/HelloWorld.vue'
</script>

<template>
  <!-- 主容器 -->
  <div class="min-h-screen flex flex-col items-center justify-center px-6 py-12">
    
    <!-- 標題區域 -->
    <header class="text-center mb-16 animate-fade-in">
      <h1 class="text-4xl md:text-6xl font-bold mb-4 tracking-wide">
        Hello welcome Taichung！
      </h1>
      <p class="text-xl md:text-2xl opacity-90">
        選擇你想先去的類型
      </p>
    </header>

    <!-- 類型選擇區域 (響應式網格) -->
    <main class="grid grid-cols-1 md:grid-cols-3 gap-8 w-full max-w-5xl">
      <div 
        v-for="category in categories" 
        :key="category.id"
        class="hover-scale border border-[#004D4D] border-opacity-20 rounded-xl p-8 text-center cursor-pointer bg-white bg-opacity-30 backdrop-blur-sm shadow-sm hover:shadow-md"
        @click="selectCategory(category.name)"
      >
        <div class="text-4xl mb-4">{{ category.icon }}</div>
        <h3 class="text-2xl font-semibold mb-2">{{ category.name }}</h3>
        <p class="text-sm opacity-70">{{ category.description }}</p>
      </div>
    </main>

    <!-- 頁尾裝飾 -->
    <footer class="mt-20 text-sm opacity-50 italic">
      Enjoy your slow travel in Taichung.
    </footer>

    <!-- HelloWorld component, if still desired -->
    <!-- <HelloWorld /> -->
  </div>
</template>

<script>
export default {
  setup() {
    const categories = ref([
      { id: 1, name: '文青特色店', icon: '☕', description: '探索巷弄間的獨立咖啡與選物店' },
      { id: 2, name: '自然景觀', icon: '🌳', description: '享受都市綠地與高美濕地的夕陽' },
      { id: 3, name: '道地美食', icon: '🍜', description: '從夜市小吃到精緻的創意料理' }
    ]);

    const selectCategory = (name) => {
      alert(`準備出發前往：${name}！`);
    };

    return {
      categories,
      selectCategory
    };
  }
};
</script>

<style>
/* Global styles for the app, moved from original <style> block */
body {
  font-family: 'Noto Serif TC', serif;
  background-color: #FDFBF7; /* 米白色背景 */
  color: #004D4D; /* 松綠色字體 */
}
.text-dark-teal {
  color: #004D4D;
}
.bg-dark-teal {
  background-color: #004D4D;
}
.hover-scale {
  transition: transform 0.3s ease;
}
.hover-scale:hover {
  transform: translateY(-5px);
}

@keyframes fadeIn {
  from { opacity: 0; transform: translateY(20px); }
  to { opacity: 1; transform: translateY(0); }
}
.animate-fade-in {
  animation: fadeIn 1s ease-out;
}
</style>
