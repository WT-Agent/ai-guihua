<template>
  <section class="nomads-showcase-section">
    <div class="showcase-header">
      <div class="header-left">
        <h2 class="showcase-title">实战案例与规划模板库</h2>
        <p class="showcase-subtitle">精选高频职业与学业发展场景，点击“一键套用”快速精准测评</p>
      </div>
      <span class="showcase-badge">已收录 {{ showcaseItems.length }} 个实战模板</span>
    </div>

    <div class="showcase-grid">
      <div 
        v-for="item in showcaseItems" 
        :key="item.id" 
        class="glass-card showcase-card"
      >
        <div class="card-header">
          <span class="scenario-tag">{{ item.tag }}</span>
          <span class="usage-count">{{ item.usageCount }} 次应用</span>
        </div>

        <div class="card-content">
          <h3 class="item-title">{{ item.title }}</h3>
          <p class="item-prompt">“{{ item.prompt }}”</p>
        </div>

        <div class="card-action">
          <button class="apply-btn" @click="applyTemplate(item)">
            <span>一键套用</span>
            <svg class="arrow-icon" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
              <line x1="5" y1="12" x2="19" y2="12"></line>
              <polyline points="12 5 19 12 12 19"></polyline>
            </svg>
          </button>
        </div>
      </div>
    </div>
  </section>
</template>

<script setup lang="ts">
import { computed } from 'vue';

const emit = defineEmits<{
  (e: 'apply-template', payload: { prompt: string; planningType?: string; stageGoal?: string; industryField?: string }): void;
}>();

export interface ShowcaseItem {
  id: string;
  tag: string;
  title: string;
  prompt: string;
  planningType: string;
  stageGoal: string;
  industryField: string;
  usageCount: string;
}

const showcaseItems = computed<ShowcaseItem[]>(() => [
  {
    id: 'plan-1',
    tag: '职场跳槽',
    title: '互联网大厂3年P6升P7瓶颈破解',
    prompt: '本人目前在电商大厂任职资深后端开发3年，面临技术业务瓶颈与绩效考核压力，希望评估跳槽中厂架构师或留在原大厂转做AI大模型工程化的路径与SWOT。',
    planningType: '职场3-5年瓶颈与转型跳槽',
    stageGoal: '资深骨干转型',
    industryField: '互联网科技',
    usageCount: '32.8k'
  },
  {
    id: 'plan-2',
    tag: '高考选报',
    title: '高考理科580分专业选报与大学规划',
    prompt: '预估高考580分，理科生，对人工智能与自动化感兴趣，希望评估计算机科学、智能制造与电子信息工程的未来就职前景与院校填报梯队策略。',
    planningType: '高考志愿与大学专业选择',
    stageGoal: '高考选报',
    industryField: '互联网科技',
    usageCount: '28.4k'
  },
  {
    id: 'plan-3',
    tag: '考研备考',
    title: '计算机专业考研与校招两手准备规划',
    prompt: '软件工程大三学生，GPA前30%，犹豫冲刺985名校计算机专硕还是全力备战秋招互联网大厂，希望能有时间节点拆解与风险应对预案。',
    planningType: '考研与出国留学路径规划',
    stageGoal: '考研冲刺',
    industryField: '互联网科技',
    usageCount: '45.1k'
  },
  {
    id: 'plan-4',
    tag: '应届求职',
    title: '文科双非应届生首份工作避坑指南',
    prompt: '新闻传播专业应届毕业，拿到新媒体运营与传统媒体记者两个Offer，希望剖析两个岗位的技能提升空间与3年薪资增长路径。',
    planningType: '毕业求职与首份工作选择',
    stageGoal: '大学应届生',
    industryField: '新兴文化传媒',
    usageCount: '19.7k'
  },
  {
    id: 'plan-5',
    tag: '跨界转型',
    title: '传统制造工程师转行数字营销与AI应用',
    prompt: '机械制造行业从业2年，月薪8k增长乏力，自学了Python与AI文生图工具，希望制定转行数字营销或AI应用方案的技能补强计划。',
    planningType: '职场3-5年瓶颈与转型跳槽',
    stageGoal: '初入职场1-3年',
    industryField: '新兴文化传媒',
    usageCount: '36.2k'
  },
  {
    id: 'plan-6',
    tag: '职业危机',
    title: '35岁职场瓶颈与国企/体制内稳定转型',
    prompt: '外企35岁中层管理，面对裁员风险与家庭压力，考虑备考事业单位或国企管理岗，需要诊断技能可迁移性与中长期稳定性。',
    planningType: '职场3-5年瓶颈与转型跳槽',
    stageGoal: '资深骨干转型',
    industryField: '体制内公职',
    usageCount: '22.0k'
  }
]);

function applyTemplate(item: ShowcaseItem) {
  emit('apply-template', {
    prompt: item.prompt,
    planningType: item.planningType,
    stageGoal: item.stageGoal,
    industryField: item.industryField
  });
}
</script>

<style scoped>
.nomads-showcase-section {
  margin-top: 2rem;
  width: 100%;
}

.showcase-header {
  display: flex;
  justify-content: space-between;
  align-items: flex-end;
  margin-bottom: 1.25rem;
  padding-bottom: 0.75rem;
  border-bottom: 1px solid var(--card-border);
}

.showcase-title {
  font-size: 1.2rem;
  font-weight: 700;
  color: var(--text-primary);
  background: var(--primary-gradient);
  -webkit-background-clip: text;
  -webkit-text-fill-color: transparent;
}

.showcase-subtitle {
  font-size: 0.825rem;
  color: var(--text-secondary);
  margin-top: 0.25rem;
}

.showcase-badge {
  font-size: 0.75rem;
  color: #a5b4fc;
  background: rgba(99, 102, 241, 0.12);
  border: 1px solid rgba(99, 102, 241, 0.25);
  padding: 4px 10px;
  border-radius: 20px;
}

.showcase-grid {
  display: grid;
  grid-template-columns: repeat(1, 1fr);
  gap: 1.25rem;
}

@media (min-width: 640px) {
  .showcase-grid {
    grid-template-columns: repeat(2, 1fr);
  }
}

@media (min-width: 1024px) {
  .showcase-grid {
    grid-template-columns: repeat(3, 1fr);
  }
}

.showcase-card {
  display: flex;
  flex-direction: column;
  justify-content: space-between;
  height: 100%;
  padding: 1.25rem;
  background: rgba(255, 255, 255, 0.02);
  border: 1px solid var(--card-border);
  border-radius: 14px;
  transition: all 0.25s ease;
}

.showcase-card:hover {
  background: rgba(255, 255, 255, 0.05);
  border-color: rgba(99, 102, 241, 0.4);
  transform: translateY(-3px);
  box-shadow: 0 10px 25px rgba(0, 0, 0, 0.4);
}

.card-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 0.75rem;
}

.scenario-tag {
  font-size: 0.75rem;
  font-weight: 600;
  padding: 3px 8px;
  border-radius: 6px;
  background: rgba(168, 85, 247, 0.15);
  color: #c084fc;
  border: 1px solid rgba(168, 85, 247, 0.3);
}

.usage-count {
  font-size: 0.75rem;
  color: var(--text-secondary);
}

.card-content {
  margin-bottom: 1rem;
  flex: 1;
}

.item-title {
  font-size: 0.95rem;
  font-weight: 600;
  color: var(--text-primary);
  margin-bottom: 0.4rem;
}

.item-prompt {
  font-size: 0.825rem;
  color: var(--text-secondary);
  line-height: 1.45;
  display: -webkit-box;
  -webkit-line-clamp: 3;
  -webkit-box-orient: vertical;
  overflow: hidden;
  font-style: italic;
}

.card-action {
  padding-top: 0.75rem;
  border-top: 1px solid rgba(255, 255, 255, 0.04);
}

.apply-btn {
  width: 100%;
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 6px;
  padding: 0.5rem 1rem;
  background: rgba(99, 102, 241, 0.1);
  border: 1px solid rgba(99, 102, 241, 0.3);
  border-radius: 8px;
  color: #a5b4fc;
  font-size: 0.825rem;
  font-weight: 600;
  cursor: pointer;
  transition: all 0.2s ease;
}

.showcase-card:hover .apply-btn {
  background: var(--primary-gradient);
  border-color: transparent;
  color: white;
}

.arrow-icon {
  width: 14px;
  height: 14px;
  transition: transform 0.2s ease;
}

.apply-btn:hover .arrow-icon {
  transform: translateX(3px);
}
</style>
