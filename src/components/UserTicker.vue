<template>
  <div class="glass-card ticker-card">
    <div class="ticker-wrapper">
      <div v-for="(item, index) in tickerItems" :key="index" class="ticker-item">
        <span>{{ item.text }}</span>
        <span style="color: var(--success-color); font-weight: bold;">[规划成功]</span>
      </div>
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref } from 'vue';

const tickerItems = ref([
  { text: '用户 138****8819 成功生成了《互联网大厂3年P6升级P7晋升与技能提升方案》' },
  { text: '用户 159****3201 成功生成了《2026年高考志愿理科智能制造专业选报规划》' },
  { text: '用户 177****5022 成功生成了《金融学本科考研与券商实习双轨规划图谱》' },
  { text: '用户 131****9182 成功生成了《传统制造业项目经理转型产品经理路线图》' },
  { text: '用户 186****6640 成功生成了《双非应届生校招冲刺与SWOT优势分析报告》' }
]);
</script>
