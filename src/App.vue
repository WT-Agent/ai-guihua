<template>
  <div class="app-container">
    <!-- 成功提示 -->
    <div v-if="copied" class="top-success-toast">
      复制成功
    </div>
    <!-- 常驻悬浮分享按钮 (H5 / 移动端与桌面端通用) -->
    <button class="floating-share-btn" @click="showShareGuide = true">
      <svg class="share-icon" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
        <circle cx="18" cy="5" r="3"></circle>
        <circle cx="6" cy="12" r="3"></circle>
        <circle cx="18" cy="19" r="3"></circle>
        <line x1="8.59" y1="13.51" x2="15.42" y2="17.49"></line>
        <line x1="15.41" y1="6.51" x2="8.59" y2="10.49"></line>
      </svg>
      <span>分享规划神器</span>
    </button>

    <header>
      <h1>{{ appTitle }}</h1>
      <p>SWOT优势诊断 · 职业路径设计 · 技能提升清单 · 落地行动里程碑</p>
    </header>

    <!-- 动态广播轮播 -->
    <UserTicker />

    <!-- 核心操作区卡片 -->
    <main ref="inputCardRef" class="glass-card input-group">
      <!-- 4 种预设类型选择 -->
      <div class="selector-group">
        <label class="selector-label">选择生涯规划类型</label>
        <div class="style-selector">
          <button 
            v-for="ptype in planningTypeOptions" 
            :key="ptype"
            class="style-option"
            :class="{ active: activePlanningType === ptype }"
            @click="activePlanningType = ptype"
          >
            {{ ptype }}
          </button>
        </div>
      </div>

      <!-- 2 组属性: 目标阶段 与 行业方向 -->
      <div class="options-row" style="display: grid; grid-template-columns: repeat(auto-fit, minmax(200px, 1fr)); gap: 1rem;">
        <div class="selector-group">
          <label class="selector-label">目标阶段</label>
          <div class="style-selector">
            <button 
              v-for="stage in stageGoalOptions" 
              :key="stage"
              class="style-option"
              :class="{ active: selectedStageGoal === stage }"
              @click="selectedStageGoal = stage"
            >
              {{ stage }}
            </button>
          </div>
        </div>

        <div class="selector-group">
          <label class="selector-label">行业方向</label>
          <div class="style-selector">
            <button 
              v-for="industry in industryFieldOptions" 
              :key="industry"
              class="style-option"
              :class="{ active: selectedIndustryField === industry }"
              @click="selectedIndustryField = industry"
            >
              {{ industry }}
            </button>
          </div>
        </div>
      </div>

      <!-- 输入框 -->
      <div class="selector-group">
        <div style="display: flex; justify-content: space-between; align-items: center;">
          <label class="selector-label">输入个人背景、优势劣势或发展困惑</label>
          <div style="display: flex; gap: 0.5rem;">
            <button v-if="userInput" class="text-link-btn" @click="userInput = ''">清空输入</button>
            <button class="text-link-btn" @click="showPlanningGuideModal = true">生涯规划模型与性格匹配指南</button>
          </div>
        </div>
        <textarea 
          v-model="userInput" 
          placeholder="请简要描述您的个人现状背景、当前优势短板及生涯发展困惑...（例如：计算机专业应届硕士，面临大厂算法岗Offer与研究所编制的选择，顾虑加班强度与薪资天花板，希望进行SWOT分析与5年路径规划。）"
          style="min-height: 120px;"
        ></textarea>
        <div style="display: flex; justify-content: space-between; font-size: 0.75rem; color: var(--text-secondary);">
          <span>字符数: {{ userInput.length }} 字</span>
          <span>建议包含教育背景、工作年限、核心诉求与顾虑维度</span>
        </div>
      </div>

      <!-- 操作按钮区 -->
      <div style="display: flex; gap: 0.75rem;">
        <button 
          class="action-btn" 
          :disabled="loading || !userInput.trim()"
          @click="handleGenerate"
        >
          {{ loading ? '正在深度诊断SWOT与设计生涯发展路径...' : '开始生成生涯规划方案' }}
        </button>
        <button class="icon-btn" style="padding: 0 1rem; border-radius: 10px;" @click="toggleHistoryDrawer">
          历史方案 ({{ historyList.length }})
        </button>
      </div>

      <!-- 异常提示 -->
      <div v-if="errorMsg" style="color: var(--accent-color); font-size: 0.85rem; text-align: center; margin-top: 0.5rem;">
        {{ errorMsg }}
      </div>
    </main>

    <!-- 生成结果卡片 -->
    <section v-if="result || loading" class="glass-card">
      <div class="result-header">
        <span class="result-title">职业与学业生涯规划报告</span>
        <div class="button-actions">
          <button v-if="result" class="icon-btn" @click="copyText">
            {{ copied ? '已复制报告' : '复制规划方案' }}
          </button>
          <button v-if="result" class="icon-btn" @click="resetResult">
            重置
          </button>
        </div>
      </div>

      <!-- 加载中骨架屏 -->
      <div v-if="loading" class="skeleton">
        <div class="skeleton-line" style="width: 85%"></div>
        <div class="skeleton-line" style="width: 95%"></div>
        <div class="skeleton-line" style="width: 70%"></div>
        <div class="skeleton-line" style="width: 90%"></div>
        <div class="skeleton-line" style="width: 60%"></div>
      </div>

      <!-- 渲染结果 -->
      <div v-else-if="result">
        <!-- AI 评估看板 -->
        <div v-if="aiScores" class="scores-container" style="margin-bottom: 1.5rem; padding: 1.25rem; background: rgba(0,0,0,0.25); border-radius: 12px; border: 1px solid rgba(255,255,255,0.06);">
          <div style="font-weight: 700; font-size: 0.95rem; margin-bottom: 1rem; color: #a5b4fc; display: flex; justify-content: space-between; align-items: center;">
            <span>AI 生涯规划可行性与竞争力评估看板</span>
            <span style="font-size: 0.8rem; font-weight: normal; color: var(--text-secondary);">综合质量分: {{ getAverageScoreFromMap(aiScores) }} / 100</span>
          </div>
          <div class="metrics-grid" style="display: grid; grid-template-columns: repeat(auto-fit, minmax(140px, 1fr)); gap: 1rem;">
            <div v-for="metric in metricsList" :key="metric.key" class="metric-item">
              <div style="display: flex; justify-content: space-between; font-size: 0.8rem; margin-bottom: 0.3rem;">
                <span style="color: var(--text-secondary);">{{ metric.label }}</span>
                <span style="font-weight: bold; color: var(--accent-color);">{{ aiScores[metric.key] || 90 }} 分</span>
              </div>
              <div class="bar-bg" style="height: 6px; background: rgba(255,255,255,0.08); border-radius: 3px; overflow: hidden;">
                <div class="bar-fill" :style="{ width: (aiScores[metric.key] || 90) + '%', background: 'var(--primary-gradient)', height: '100%', borderRadius: '3px', transition: 'width 0.5s ease' }"></div>
              </div>
            </div>
          </div>
        </div>

        <div class="output-content">{{ displayResultText }}</div>
      </div>
    </section>

    <!-- 本地历史记录面板 -->
    <section v-if="showHistory" class="glass-card" style="margin-top: 1rem;">
      <div class="result-header">
        <span class="result-title">本地生涯规划历史记录</span>
        <button class="icon-btn" @click="showHistory = false">关闭记录</button>
      </div>

      <div v-if="historyList.length === 0" style="text-align: center; color: var(--text-secondary); padding: 1.5rem; font-size: 0.85rem;">
        暂无历史生涯规划记录，立即开始生成规划方案吧！
      </div>

      <div v-else class="history-grid" style="display: flex; flex-direction: column; gap: 0.75rem; max-height: 320px; overflow-y: auto;">
        <div v-for="item in historyList" :key="item.id" class="history-item" style="padding: 1rem; background: rgba(0,0,0,0.2); border-radius: 10px; border: 1px solid var(--card-border);">
          <div style="display: flex; justify-content: space-between; font-size: 0.8rem; color: var(--text-secondary); margin-bottom: 0.4rem;">
            <span>{{ item.timestamp }} · [{{ item.planningType }} / {{ item.stageGoal }} / {{ item.industryField }}]</span>
            <span style="color: var(--primary-color);">评分: {{ getAverageScore(item) }}</span>
          </div>
          <div style="font-size: 0.85rem; font-weight: bold; margin-bottom: 0.4rem; white-space: nowrap; overflow: hidden; text-overflow: ellipsis; color: var(--text-primary);">
            背景/诉求: {{ item.input }}
          </div>
          <div style="display: flex; gap: 0.5rem;">
            <button class="icon-btn" style="font-size: 0.75rem;" @click="applyHistory(item)">套用场景</button>
            <button class="icon-btn" style="font-size: 0.75rem;" @click="viewHistoryOutput(item)">查看方案全文</button>
          </div>
        </div>
      </div>
    </section>

    <!-- 实战案例与模版 Showcase -->
    <NomadsShowcase
      @apply-template="handleApplyTemplate"
    />

    <!-- 生涯规划模型与职业性格匹配指南 Modal -->
    <div v-if="showPlanningGuideModal" class="modal-overlay" @click.self="showPlanningGuideModal = false">
      <div class="modal-content" style="max-width: 480px;">
        <h3>生涯规划模型与职业性格匹配指南</h3>
        <p style="text-align: left; font-size: 0.825rem; margin-bottom: 1rem; color: var(--text-secondary);">
          科学定位个人优势、破解转型困局的核心指导原则：
        </p>
        <div class="modal-scroll-area" style="text-align: left; font-size: 0.825rem;">
          <div v-for="(rule, idx) in planningGuideRules" :key="idx" style="margin-bottom: 0.75rem; padding: 0.5rem 0.75rem; background: rgba(255,255,255,0.03); border-radius: 8px; border: 1px solid rgba(255,255,255,0.05);">
            <div style="color: var(--accent-color); font-weight: bold; margin-bottom: 0.2rem;">{{ rule.title }}</div>
            <div style="color: var(--text-primary); margin-bottom: 0.2rem;">建议做法: {{ rule.advice }}</div>
            <div style="color: var(--text-secondary); font-size: 0.775rem;">避坑要点: {{ rule.avoid }}</div>
          </div>
        </div>
        <button class="modal-btn" style="margin-top: 1rem;" @click="showPlanningGuideModal = false">关闭</button>
      </div>
    </div>

    <!-- 微信 H5 悬浮分享引导 Modal -->
    <div v-if="showShareGuide" class="modal-overlay" @click.self="showShareGuide = false">
      <div class="modal-content">
        <h3>分享职业与学业生涯规划专家</h3>
        <p>扫码关注或将链接分享给求职者、在校学生及转型职场人，助力明晰人生方向。</p>
        
        <div class="qr-code-placeholder">
          <svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 100 100" width="100%" height="100%">
            <rect width="100" height="100" fill="white"/>
            <rect x="5" y="5" width="25" height="25" fill="#110e24"/>
            <rect x="9" y="9" width="17" height="17" fill="white"/>
            <rect x="13" y="13" width="9" height="9" fill="#110e24"/>
            <rect x="70" y="5" width="25" height="25" fill="#110e24"/>
            <rect x="74" y="9" width="17" height="17" fill="white"/>
            <rect x="78" y="13" width="9" height="9" fill="#110e24"/>
            <rect x="5" y="70" width="25" height="25" fill="#110e24"/>
            <rect x="9" y="74" width="17" height="17" fill="white"/>
            <rect x="13" y="78" width="9" height="9" fill="#110e24"/>
            <rect x="35" y="10" width="8" height="8" fill="#110e24"/>
            <rect x="48" y="5" width="6" height="12" fill="#110e24"/>
            <rect x="60" y="15" width="5" height="5" fill="#110e24"/>
            <rect x="35" y="35" width="10" height="10" fill="#110e24"/>
            <rect x="50" y="45" width="15" height="8" fill="#110e24"/>
            <rect x="40" y="70" width="8" height="16" fill="#110e24"/>
            <rect x="55" y="65" width="10" height="10" fill="#110e24"/>
            <rect x="75" y="40" width="12" height="12" fill="#110e24"/>
            <rect x="75" y="75" width="15" height="15" fill="#110e24"/>
            <rect x="45" y="80" width="8" height="8" fill="#110e24"/>
          </svg>
        </div>

        <div style="font-size: 0.8rem; color: var(--text-secondary); margin-bottom: 1.5rem;">
          微信号: <span style="color: var(--primary-color); font-weight: bold;">{{ wechatId }}</span>
        </div>

        <button class="modal-btn" @click="showShareGuide = false">关闭</button>
      </div>
    </div>

    <!-- 底部隐私与服务条款链接 -->
    <footer class="footer-links">
      <button class="footer-link-btn" @click="showPrivacy = true">Privacy Policy</button>
      <button class="footer-link-btn" @click="showTerms = true">Terms of Service</button>
      <button class="footer-link-btn" @click="showContact = true">Contact Us</button>
      <a href="https://api.wuxian.xyz/sign-up?aff=OyRY" target="_blank" rel="noopener noreferrer" class="footer-link-btn">API 平台</a>
      <a href="https://www.kutuyun.com/aff/IPJKCKWF" target="_blank" rel="noopener noreferrer" class="footer-link-btn">酷兔云</a>
      <a href="https://bandwagonhost.com/aff.php?aff=48115" target="_blank" rel="noopener noreferrer" class="footer-link-btn">搬瓦工</a>
    </footer>

    <!-- 隐私政策弹窗 -->
    <div v-if="showPrivacy" class="modal-overlay" @click.self="showPrivacy = false">
      <div class="modal-content">
        <h3>Privacy Policy</h3>
        <div class="modal-text-content modal-scroll-area">
          <p>我们非常重视您的个人隐私与职业发展数据保密。您在本应用中输入的履历信息、优势劣势及规划诉求仅用于实时大模型生成，系统不会在云端永久存储或泄露您的个人档案。</p>
          <p>为了记录您的免费生成额度，本应用会在您的浏览器本地（localStorage）记录试用次数与解锁状态。</p>
        </div>
        <button class="modal-btn" @click="showPrivacy = false">关闭</button>
      </div>
    </div>

    <!-- 服务条款弹窗 -->
    <div v-if="showTerms" class="modal-overlay" @click.self="showTerms = false">
      <div class="modal-content">
        <h3>Terms of Service</h3>
        <div class="modal-text-content modal-scroll-area">
          <p>欢迎使用网腾无限 AI 职业与学业生涯规划专家。本工具生成的决策建议、路径拆解与SWOT分析仅供学习参考指导。</p>
          <p>高考志愿填报、考研择校及重大职业转型抉择前，请结合个人实际情况与官方最新考试招生/招聘政策进行综合研判。</p>
        </div>
        <button class="modal-btn" @click="showTerms = false">关闭</button>
      </div>
    </div>

    <!-- 联系我们弹窗 -->
    <div v-if="showContact" class="modal-overlay" @click.self="showContact = false">
      <div class="modal-content contact-modal-content">
        <h3>Contact Us</h3>
        <div class="modal-text-content contact-card-body">
          <p>如果您在使用过程中遇到任何问题，或有合作意向，可以通过以下方式联系我们：</p>
          <div class="contact-qr-container">
            <div class="contact-qr-card">
              <img :src="weixinImg" alt="微信交流" class="contact-qr-img" />
              <span class="contact-qr-label">微信交流</span>
            </div>
            <div class="contact-qr-card">
              <img :src="dingtalkImg" alt="钉钉联系" class="contact-qr-img" />
              <span class="contact-qr-label">钉钉联系</span>
            </div>
          </div>
          <p class="contact-email">反馈邮箱: <span style="color: var(--primary-color);">us@wuxian.xyz</span></p>
        </div>
        <button class="modal-btn" @click="showContact = false">关闭</button>
      </div>
    </div>

    <!-- 裂变拦截弹窗 -->
    <FissionModal 
      :visible="showFission" 
      :wechat-id="wechatId"
      @unlocked="handleUnlocked"
    />
  </div>
</template>

<script setup lang="ts">
import { ref, computed, onMounted } from 'vue';
import UserTicker from './components/UserTicker.vue';
import FissionModal from './components/FissionModal.vue';
import NomadsShowcase from './components/NomadsShowcase.vue';
import appConfig from './config.json';
import weixinImg from '../asset/weixin.png';
import dingtalkImg from '../asset/dingtalk.png';

// 配置参数
const appTitle = ref(appConfig.title || '网腾无限AI - 职业与学业生涯规划专家');
const wechatId = ref(appConfig.wechatId || 'ai_wuxian_xyz');
const promptTopic = ref(appConfig.promptTopic || '');

const inputCardRef = ref<HTMLElement | null>(null);
const userInput = ref('');
const loading = ref(false);
const errorMsg = ref('');
const result = ref('');
const copied = ref(false);

const showFission = ref(false);
const showPrivacy = ref(false);
const showTerms = ref(false);
const showContact = ref(false);
const showShareGuide = ref(false);
const showPlanningGuideModal = ref(false);

// 解析 Cookie
const getCookie = (name: string): string | null => {
  const nameEQ = name + "=";
  const ca = document.cookie.split(';');
  for (let i = 0; i < ca.length; i++) {
    let c = ca[i];
    while (c.charAt(0) === ' ') c = c.substring(1, c.length);
    if (c.indexOf(nameEQ) === 0) return c.substring(nameEQ.length, c.length);
  }
  return null;
};

// 用户登录状态
const userToken = ref(getCookie('wuxian_session'));
const isLoggedIn = computed(() => !!userToken.value);
const authUsesCount = ref(parseInt(localStorage.getItem('auth_uses') || '0', 10));

// 4 种预设类型
const planningTypeOptions = [
  '毕业求职与首份工作选择',
  '职场3-5年瓶颈与转型跳槽',
  '高考志愿与大学专业选择',
  '考研与出国留学路径规划'
];
const activePlanningType = ref(planningTypeOptions[0]);

// 2 组属性：目标阶段 & 行业方向
const stageGoalOptions = ['大学应届生', '初入职场1-3年', '资深骨干转型', '高考选报', '考研冲刺'];
const selectedStageGoal = ref('大学应届生');

const industryFieldOptions = ['互联网科技', '金融财经', '体制内公职', '新兴文化传媒', '传统制造'];
const selectedIndustryField = ref('互联网科技');

// 评估指标列表
const metricsList = [
  { key: 'careerPathClarity', label: '职业路径清晰度' },
  { key: 'skillGapInsight', label: '能力短板剖析度' },
  { key: 'marketDemandFit', label: '市场需求契合度' },
  { key: 'growthActionability', label: '提升行动落地性' },
  { key: 'longTermSustainability', label: '长期可持续发展值' }
];

const aiScores = ref<Record<string, number> | null>(null);

// 历史记录定义
interface HistoryItem {
  id: string;
  timestamp: string;
  planningType: string;
  stageGoal: string;
  industryField: string;
  input: string;
  aiScores: Record<string, number> | null;
  output: string;
}

const historyList = ref<HistoryItem[]>([]);
const showHistory = ref(false);

// 生涯规划模型与指南
const planningGuideRules = [
  { 
    title: 'SWOT 诊断四象限原则', 
    advice: '客观分析自身优势(S)与劣势(W)，善用外部行业机遇(O)抵御竞争风险(T)。', 
    avoid: '切忌盲目跟风热点行业，忽视个人核心兴趣与专业基础。' 
  },
  { 
    title: '能力与能力圈交叉匹配', 
    advice: '结合已有硬核技能与软实力，寻找高可迁移度的交叉岗位形态。', 
    avoid: '切忌无准备地进行跨度过大的断崖式转行。' 
  },
  { 
    title: '阶段性里程碑拆解', 
    advice: '将5年中长期目标拆解为季度技能学习与月度实践项目。', 
    avoid: '切忌设定空洞抽象的宏大愿景而缺乏本周可落地的行动项。' 
  }
];

// 计算纯结果文本 (剔除打分标签 [GUIHUA_SCORES])
const displayResultText = computed(() => {
  if (!result.value) return '';
  return result.value.replace(/\[GUIHUA_SCORES\][\s\S]*?\[\/GUIHUA_SCORES\]/g, '').trim();
});

// 解析打分标签
const parseAiScores = (rawText: string) => {
  const match = rawText.match(/\[GUIHUA_SCORES\](.*?)\[\/GUIHUA_SCORES\]/);
  if (!match) return null;
  const content = match[1];
  const scoresObj: Record<string, number> = {};
  content.split(',').forEach(item => {
    const [key, val] = item.split(':');
    if (key && val) {
      scoresObj[key.trim()] = parseInt(val.trim(), 10) || 90;
    }
  });
  return Object.keys(scoresObj).length > 0 ? scoresObj : null;
};

// 计算平均分
const getAverageScoreFromMap = (scores: Record<string, number>) => {
  const keys = Object.keys(scores);
  if (keys.length === 0) return '92.5';
  const sum = keys.reduce((acc, k) => acc + (scores[k] || 90), 0);
  return (sum / keys.length).toFixed(1);
};

const getAverageScore = (item: HistoryItem) => {
  if (!item.aiScores) return '92.5';
  return getAverageScoreFromMap(item.aiScores);
};

// 本地历史记录读取与保存
const HISTORY_KEY = 'guihua_history_records';

const loadHistory = () => {
  try {
    const dataStr = localStorage.getItem(HISTORY_KEY);
    if (dataStr) {
      historyList.value = JSON.parse(dataStr);
    }
  } catch (e) {
    console.error('读取历史记录失败:', e);
  }
};

const saveHistory = (newItem: HistoryItem) => {
  try {
    historyList.value.unshift(newItem);
    if (historyList.value.length > 20) {
      historyList.value = historyList.value.slice(0, 20);
    }
    localStorage.setItem(HISTORY_KEY, JSON.stringify(historyList.value));
  } catch (e) {
    console.error('保存历史记录失败:', e);
  }
};

const toggleHistoryDrawer = () => {
  showHistory.value = !showHistory.value;
};

const applyHistory = (item: HistoryItem) => {
  activePlanningType.value = item.planningType || planningTypeOptions[0];
  selectedStageGoal.value = item.stageGoal || stageGoalOptions[0];
  selectedIndustryField.value = item.industryField || industryFieldOptions[0];
  userInput.value = item.input;
  showHistory.value = false;
  if (inputCardRef.value) {
    inputCardRef.value.scrollIntoView({ behavior: 'smooth', block: 'center' });
  }
};

const viewHistoryOutput = (item: HistoryItem) => {
  result.value = item.output;
  aiScores.value = item.aiScores;
  showHistory.value = false;
};

// 重置结果
const resetResult = () => {
  result.value = '';
  aiScores.value = null;
  userInput.value = '';
};

// 判断是否达到免费次数上限
const isLimitReached = computed(() => {
  if (isLoggedIn.value) {
    return authUsesCount.value >= 15;
  }
  const uses = parseInt(localStorage.getItem('free_uses') || '0', 10);
  const shared = localStorage.getItem('shared_fission') === 'true';
  return uses >= 3 && !shared;
});

// 获取 API 请求端点
const apiEndpoint = import.meta.env.DEV
  ? '/api/local/generate'
  : (import.meta.env.VITE_API_ENDPOINT || 'https://api.wuxian.xyz/api/v1/generate');

const handleGenerate = async () => {
  if (isLimitReached.value) {
    showFission.value = true;
    return;
  }

  loading.value = true;
  errorMsg.value = '';
  result.value = '';
  aiScores.value = null;

  try {
    const fullPrompt = `场景类别：${promptTopic.value}\n规划类型：${activePlanningType.value}\n目标阶段：${selectedStageGoal.value}\n行业方向：${selectedIndustryField.value}\n背景与诉求细节：${userInput.value}`;

    const response = await fetch(apiEndpoint, {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json',
      },
      credentials: 'include',
      body: JSON.stringify({
        taskType: 'text',
        prompt: fullPrompt,
        style: activePlanningType.value
      })
    });

    const data = await response.json();
    if (data.error) {
      errorMsg.value = data.error;
    } else {
      result.value = data.result;
      const parsedScores = parseAiScores(data.result);
      aiScores.value = parsedScores;

      // 保存历史记录
      const historyItem: HistoryItem = {
        id: Date.now().toString(),
        timestamp: new Date().toLocaleString(),
        planningType: activePlanningType.value,
        stageGoal: selectedStageGoal.value,
        industryField: selectedIndustryField.value,
        input: userInput.value,
        aiScores: parsedScores,
        output: data.result
      };
      saveHistory(historyItem);
      
      if (isLoggedIn.value) {
        const nextAuthUses = authUsesCount.value + 1;
        localStorage.setItem('auth_uses', nextAuthUses.toString());
        authUsesCount.value = nextAuthUses;
      } else {
        const currentUses = parseInt(localStorage.getItem('free_uses') || '0', 10);
        localStorage.setItem('free_uses', (currentUses + 1).toString());
      }
    }
  } catch (err: any) {
    errorMsg.value = '请求接口失败，请检查网络或本地代理服务。';
  } finally {
    loading.value = false;
  }
};

const handleApplyTemplate = (payload: { prompt: string; planningType?: string; stageGoal?: string; industryField?: string }) => {
  userInput.value = payload.prompt;
  if (payload.planningType) {
    activePlanningType.value = payload.planningType;
  }
  if (payload.stageGoal) {
    selectedStageGoal.value = payload.stageGoal;
  }
  if (payload.industryField) {
    selectedIndustryField.value = payload.industryField;
  }
  if (inputCardRef.value) {
    inputCardRef.value.scrollIntoView({ behavior: 'smooth', block: 'center' });
  }
};

const handleUnlocked = () => {
  showFission.value = false;
  handleGenerate();
};

const copyText = async () => {
  try {
    await navigator.clipboard.writeText(displayResultText.value);
    copied.value = true;
    setTimeout(() => {
      copied.value = false;
    }, 2000);
  } catch (err) {
    errorMsg.value = '复制失败，请手动选择复制。';
  }
};

onMounted(() => {
  loadHistory();
});
</script>
