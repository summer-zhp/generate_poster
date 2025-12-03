<script setup>
import { ref } from 'vue';
import { snapdom } from '@zumer/snapdom';

const formData = ref({
  name: '张三',
  title: '我的精美海报',
  content: '这是一段示例海报文案。你可以编辑这里的内容，实时预览海报生成的精美效果。Snapdom 让前端生成图片变得如此简单且高质量。',
  theme: 'modern'
});

const generatedImage = ref(null);
const posterRef = ref(null);

const generatePoster = async () => {
  if (!posterRef.value) return;
  
  try {
    // 使用 snapdom 生成图片
    const img = await snapdom.toPng(posterRef.value);
    generatedImage.value = img.src;
    
  } catch (error) {
    console.error('生成失败:', error);
    alert('海报生成失败');
  }
};

const themes = {
  modern: 'background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);',
  nature: 'background: linear-gradient(135deg, #11998e 0%, #38ef7d 100%);',
  sunset: 'background: linear-gradient(135deg, #ff9966 0%, #ff5e62 100%);',
  dark: 'background: linear-gradient(135deg, #232526 0%, #414345 100%);',
};
</script>

<template>
  <div class="demo-container">
    <div class="controls">
      <h2>Snapdom 生成器</h2>
      <div class="form-group">
        <label>作者姓名</label>
        <input v-model="formData.name" type="text" placeholder="输入姓名" />
      </div>
      <div class="form-group">
        <label>海报标题</label>
        <input v-model="formData.title" type="text" placeholder="输入标题" />
      </div>
      <div class="form-group">
        <label>主要内容</label>
        <textarea v-model="formData.content" rows="4" placeholder="输入内容"></textarea>
      </div>
      <div class="form-group">
        <label>选择主题</label>
        <select v-model="formData.theme">
          <option value="modern">现代紫韵</option>
          <option value="nature">清新自然</option>
          <option value="sunset">落日余晖</option>
          <option value="dark">深邃暗夜</option>
        </select>
      </div>
      <button @click="generatePoster" class="generate-btn">
        生成海报
      </button>
    </div>

    <div class="preview-area">
      <h3>实时预览</h3>
      <div 
        ref="posterRef" 
        class="poster-card"
        :style="themes[formData.theme]"
      >
        <!-- 装饰性背景圆 -->
        <div class="decorative-circle circle-1"></div>
        <div class="decorative-circle circle-2"></div>

        <div class="poster-glass-container">
          <div class="poster-header">
            <span class="tag">每日精选</span>
            <span class="date">{{ new Date().toLocaleDateString('zh-CN') }}</span>
          </div>
          
          <h1 class="poster-title">{{ formData.title }}</h1>
          
          <div class="poster-divider"></div>
          
          <p class="poster-text">{{ formData.content }}</p>
          
          <div class="poster-footer">
            <div class="author-info">
              <div class="avatar">{{ formData.name[0] }}</div>
              <div class="author-text">
                <span class="created-by">Created by</span>
                <span class="author-name">{{ formData.name }}</span>
              </div>
            </div>
            <div class="qr-placeholder">
              <span>扫码查看</span>
            </div>
          </div>
        </div>
      </div>
    </div>

    <div class="result-area" v-if="generatedImage">
      <h3>生成结果</h3>
      <div class="result-image-wrapper">
        <img :src="generatedImage" alt="Generated Poster" />
        <a :href="generatedImage" download="snapdom-poster.png" class="download-link">下载海报</a>
      </div>
    </div>
  </div>
</template>

<style scoped>
.demo-container {
  display: grid;
  grid-template-columns: 320px 1fr 1fr;
  gap: 2rem;
  padding: 2rem;
  max-width: 1400px;
  margin: 0 auto;
  font-family: 'PingFang SC', 'Microsoft YaHei', sans-serif;
}

.controls {
  background: #fff;
  padding: 1.5rem;
  border-radius: 16px;
  box-shadow: 0 4px 20px rgba(0, 0, 0, 0.05);
  height: fit-content;
}

.form-group {
  margin-bottom: 1.2rem;
}

.form-group label {
  display: block;
  margin-bottom: 0.5rem;
  font-weight: 600;
  color: #374151;
  font-size: 0.9rem;
}

input, textarea, select {
  width: 100%;
  padding: 0.75rem;
  border: 1px solid #e5e7eb;
  border-radius: 8px;
  font-size: 0.95rem;
  transition: all 0.2s;
  background: #f9fafb;
}

input:focus, textarea:focus, select:focus {
  outline: none;
  border-color: #6366f1;
  background: #fff;
  box-shadow: 0 0 0 3px rgba(99, 102, 241, 0.1);
}

.generate-btn {
  width: 100%;
  padding: 0.9rem;
  background: linear-gradient(to right, #4f46e5, #6366f1);
  color: white;
  border: none;
  border-radius: 8px;
  font-weight: 600;
  font-size: 1rem;
  cursor: pointer;
  transition: transform 0.1s, box-shadow 0.2s;
  margin-top: 1rem;
}

.generate-btn:hover {
  transform: translateY(-1px);
  box-shadow: 0 4px 12px rgba(79, 70, 229, 0.3);
}

.generate-btn:active {
  transform: translateY(0);
}

.preview-area, .result-area {
  display: flex;
  flex-direction: column;
  gap: 1rem;
}

h3 {
  font-size: 1.1rem;
  color: #4b5563;
  font-weight: 600;
}

/* 海报样式优化 */
.poster-card {
  width: 100%;
  aspect-ratio: 3/4;
  border-radius: 20px;
  box-shadow: 0 20px 40px -10px rgba(0, 0, 0, 0.2);
  position: relative;
  overflow: hidden;
  display: flex;
  align-items: center;
  justify-content: center;
  padding: 2rem;
}

/* 背景装饰 */
.decorative-circle {
  position: absolute;
  border-radius: 50%;
  filter: blur(40px);
  opacity: 0.6;
}

.circle-1 {
  width: 200px;
  height: 200px;
  background: rgba(255, 255, 255, 0.2);
  top: -50px;
  left: -50px;
}

.circle-2 {
  width: 150px;
  height: 150px;
  background: rgba(255, 255, 255, 0.15);
  bottom: -20px;
  right: -20px;
}

/* 毛玻璃容器 */
.poster-glass-container {
  width: 100%;
  height: 100%;
  background: rgba(255, 255, 255, 0.15);
  backdrop-filter: blur(12px);
  -webkit-backdrop-filter: blur(12px);
  border-radius: 16px;
  border: 1px solid rgba(255, 255, 255, 0.3);
  padding: 2rem;
  display: flex;
  flex-direction: column;
  color: white;
  box-shadow: 0 8px 32px rgba(0, 0, 0, 0.1);
}

.poster-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 2rem;
}

.tag {
  background: rgba(255, 255, 255, 0.25);
  padding: 4px 12px;
  border-radius: 20px;
  font-size: 0.8rem;
  font-weight: 600;
  letter-spacing: 1px;
}

.date {
  font-size: 0.9rem;
  opacity: 0.9;
  font-family: monospace;
}

.poster-title {
  font-size: 2.2rem;
  font-weight: 800;
  line-height: 1.2;
  margin-bottom: 1.5rem;
  text-shadow: 0 2px 10px rgba(0,0,0,0.1);
}

.poster-divider {
  width: 40px;
  height: 4px;
  background: rgba(255, 255, 255, 0.6);
  border-radius: 2px;
  margin-bottom: 1.5rem;
}

.poster-text {
  font-size: 1.05rem;
  line-height: 1.7;
  opacity: 0.95;
  flex-grow: 1;
  white-space: pre-wrap;
}

.poster-footer {
  margin-top: auto;
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding-top: 1.5rem;
  border-top: 1px solid rgba(255, 255, 255, 0.2);
}

.author-info {
  display: flex;
  align-items: center;
  gap: 10px;
}

.avatar {
  width: 40px;
  height: 40px;
  background: rgba(255, 255, 255, 0.3);
  border-radius: 50%;
  display: flex;
  align-items: center;
  justify-content: center;
  font-weight: bold;
  font-size: 1.2rem;
}

.author-text {
  display: flex;
  flex-direction: column;
}

.created-by {
  font-size: 0.7rem;
  opacity: 0.7;
  text-transform: uppercase;
}

.author-name {
  font-weight: 600;
  font-size: 0.95rem;
}

.qr-placeholder {
  width: 50px;
  height: 50px;
  background: white;
  border-radius: 6px;
  display: flex;
  align-items: center;
  justify-content: center;
}

.qr-placeholder span {
  font-size: 0.6rem;
  color: #333;
  text-align: center;
  line-height: 1;
}

.result-image-wrapper img {
  width: 100%;
  border-radius: 12px;
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.1);
}

.download-link {
  display: inline-block;
  margin-top: 1rem;
  color: #4f46e5;
  text-decoration: none;
  font-weight: 600;
  font-size: 0.95rem;
}

.download-link:hover {
  text-decoration: underline;
}

@media (max-width: 1024px) {
  .demo-container {
    grid-template-columns: 1fr;
  }
}
</style>
