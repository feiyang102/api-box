<script setup lang="ts">
import { computed, h, ref } from 'vue'
import {
  Activity, Bot, Box, Braces, Check, ChevronDown, ChevronRight, CircleHelp,
  Clock3, CloudCog, Code2, Copy, Database, Download, FileJson, FileText,
  Folder, Gauge, Globe2, LayoutDashboard, MessageSquareText, MoreHorizontal,
  Play, Plus, Search, Send, Settings, ShieldCheck, Sparkles, TerminalSquare,
  TestTube2, WandSparkles, X, Zap
} from 'lucide-vue-next'
import {
  NButton, NConfigProvider, NDropdown, NInput, NModal, NProgress, NSelect,
  NSwitch, NTag, createDiscreteApi, darkTheme
} from 'naive-ui'

type Tab = 'params' | 'headers' | 'body' | 'tests'
type MainSection = 'workspace' | 'specs' | 'automation' | 'mock' | 'reports'

const activeSection = ref<MainSection>('workspace')
const activeTab = ref<Tab>('body')
const sending = ref(false)
const sent = ref(true)
const mockRunning = ref(true)
const aiOpen = ref(true)
const showNewSpec = ref(false)
const showRunner = ref(false)
const url = ref('{{baseUrl}}/v1/orders')
const bodyText = ref(`{
  "customer_id": "cus_8f2a1d",
  "items": [
    { "sku": "PRO-001", "quantity": 1 }
  ],
  "shipping_address_id": "addr_203"
}`)
const search = ref('')
const method = ref('POST')
const copied = ref(false)
const { message } = createDiscreteApi(['message'], {
  configProviderProps: { theme: darkTheme }
})

const responseText = computed(() => `{
  "code": 0,
  "message": "success",
  "data": {
    "id": "ord_01JBT7Y8X2M9QK4P",
    "status": "pending_payment",
    "amount": 29900,
    "currency": "CNY",
    "created_at": "2026-06-20T10:24:31Z"
  }
}`)

const methodOptions = ['GET', 'POST', 'PUT', 'PATCH', 'DELETE'].map(v => ({ label: v, value: v }))
const environmentOptions = [
  { label: 'Development', value: 'dev' },
  { label: 'Staging', value: 'staging' },
  { label: 'Production', value: 'prod' }
]
const environment = ref('dev')

const collections = [
  { name: '用户服务', count: 8, open: false },
  { name: '商品中心', count: 12, open: false },
  { name: '订单服务', count: 9, open: true },
  { name: '支付服务', count: 6, open: false }
]

function runRequest() {
  sending.value = true
  sent.value = false
  window.setTimeout(() => {
    sending.value = false
    sent.value = true
    message.success('请求成功 · 201 Created · 186 ms')
  }, 800)
}

async function copyRequest() {
  const text = `${method.value} ${url.value}\nContent-Type: application/json\n\n${bodyText.value}`
  await navigator.clipboard?.writeText(text)
  copied.value = true
  message.success('接口内容已复制')
  window.setTimeout(() => copied.value = false, 1500)
}

function exportMarkdown() {
  const md = `# 创建订单\n\n**${method.value}** \`${url.value}\`\n\n## 请求体\n\n\`\`\`json\n${bodyText.value}\n\`\`\`\n\n## 成功响应\n\n\`\`\`json\n${responseText.value}\n\`\`\`\n`
  const blob = new Blob([md], { type: 'text/markdown' })
  const a = document.createElement('a')
  a.href = URL.createObjectURL(blob)
  a.download = 'create-order.md'
  a.click()
  URL.revokeObjectURL(a.href)
  message.success('Markdown 文档已导出')
}

const dropdownOptions = [
  { label: '导出当前接口', key: 'current', icon: () => h(FileText, { size: 15 }) },
  { label: '导出订单服务', key: 'service', icon: () => h(Folder, { size: 15 }) },
  { label: '导出全部文档', key: 'all', icon: () => h(Download, { size: 15 }) }
]
</script>

<template>
  <n-config-provider :theme="darkTheme">
      <div class="app-shell">
        <header class="topbar">
          <div class="brand"><div class="brand-mark"><Box :size="18" /></div><span>API BOX</span></div>
          <div class="project-switch"><span class="project-dot"></span><strong>商城 API</strong><ChevronDown :size="14" /></div>
          <div class="topbar-spacer" />
          <div class="env-pill"><span></span><n-select v-model:value="environment" :options="environmentOptions" size="small" /></div>
          <n-button quaternary circle title="运行记录"><Clock3 :size="17" /></n-button>
          <n-button quaternary circle title="设置"><Settings :size="17" /></n-button>
          <div class="avatar">N</div>
        </header>

        <aside class="rail">
          <div class="rail-group">
            <button :class="{ active: activeSection === 'workspace' }" @click="activeSection = 'workspace'"><LayoutDashboard /><span>工作台</span></button>
            <button :class="{ active: activeSection === 'specs' }" @click="activeSection = 'specs'"><FileJson /><span>规范</span></button>
            <button :class="{ active: activeSection === 'automation' }" @click="showRunner = true"><TestTube2 /><span>自动化</span></button>
            <button :class="{ active: activeSection === 'mock' }" @click="activeSection = 'mock'"><CloudCog /><span>Mock</span><i class="status-dot"></i></button>
            <button :class="{ active: activeSection === 'reports' }" @click="activeSection = 'reports'"><Gauge /><span>评估</span></button>
          </div>
          <div class="rail-bottom"><button><CircleHelp /><span>帮助</span></button></div>
        </aside>

        <aside class="collections">
          <div class="collection-head"><span>接口集合</span><button @click="showNewSpec = true"><Plus :size="16" /></button></div>
          <div class="search-box"><Search :size="14" /><input v-model="search" placeholder="搜索接口  ⌘ K" /></div>
          <div class="collection-list">
            <div v-for="item in collections" :key="item.name" class="collection-item" :class="{ expanded: item.open }">
              <div class="collection-row"><ChevronRight :size="14" /><Folder :size="15" /><span>{{ item.name }}</span><em>{{ item.count }}</em></div>
              <div v-if="item.open" class="endpoint-list">
                <button><b class="get">GET</b><span>订单列表</span></button>
                <button class="selected"><b class="post">POST</b><span>创建订单</span><MoreHorizontal :size="14" /></button>
                <button><b class="get">GET</b><span>订单详情</span></button>
                <button><b class="patch">PATCH</b><span>更新订单</span></button>
                <button><b class="delete">DEL</b><span>取消订单</span></button>
              </div>
            </div>
          </div>
          <div class="mock-card">
            <div><span class="live-dot"></span><strong>Mock 服务</strong><n-switch v-model:value="mockRunning" size="small" /></div>
            <code>localhost:3100</code><button @click="message.info('Mock 地址已复制')"><Copy :size="13" /></button>
          </div>
        </aside>

        <main class="workspace" :class="{ 'ai-closed': !aiOpen }">
          <section class="request-panel">
            <div class="crumbs"><span>订单服务</span><ChevronRight :size="13"/><strong>创建订单</strong><div class="unsaved"></div></div>
            <div class="request-title-row">
              <div><h1>创建订单</h1><span class="spec-badge"><ShieldCheck :size="13" /> 已关联规范 v1.4</span></div>
              <div class="action-row">
                <n-button size="small" quaternary @click="copyRequest"><Check v-if="copied" :size="15"/><Copy v-else :size="15"/> 复制</n-button>
                <n-dropdown :options="dropdownOptions" @select="exportMarkdown"><n-button size="small" quaternary><Download :size="15"/> 导出 <ChevronDown :size="13"/></n-button></n-dropdown>
                <n-button size="small" type="primary" class="save-btn"><Check :size="15"/> 保存</n-button>
              </div>
            </div>
            <div class="url-bar">
              <n-select v-model:value="method" :options="methodOptions" class="method-select" />
              <n-input v-model:value="url" class="url-input" />
              <n-button type="primary" class="send-btn" :loading="sending" @click="runRequest"><Send v-if="!sending" :size="16"/> 发送</n-button>
            </div>
            <div class="request-tabs">
              <button v-for="tab in ([['params','Params','2'],['headers','Headers','8'],['body','Body',''],['tests','Tests','3']] as const)" :key="tab[0]" :class="{active: activeTab === tab[0]}" @click="activeTab = tab[0]">{{ tab[1] }} <em v-if="tab[2]">{{ tab[2] }}</em></button>
            </div>
            <div class="editor-wrap">
              <div class="editor-toolbar"><span>JSON</span><ChevronDown :size="13"/><div/><button><WandSparkles :size="14"/> 格式化</button><button><Braces :size="14"/> Schema</button></div>
              <div v-if="activeTab === 'body'" class="code-editor"><div class="line-numbers">1<br>2<br>3<br>4<br>5<br>6<br>7</div><textarea v-model="bodyText" spellcheck="false" /></div>
              <div v-else-if="activeTab === 'params'" class="kv-table"><div><span>KEY</span><span>VALUE</span><span>DESCRIPTION</span></div><div><code>expand</code><code>items</code><span>展开订单项</span></div><div><code>locale</code><code>zh-CN</code><span>响应语言</span></div></div>
              <div v-else-if="activeTab === 'headers'" class="kv-table"><div><span>KEY</span><span>VALUE</span><span>DESCRIPTION</span></div><div><code>Content-Type</code><code>application/json</code><span>内容类型</span></div><div><code>Authorization</code><code>Bearer ••••••••</code><span>访问令牌</span></div></div>
              <div v-else class="test-list"><div><Check/> 状态码应为 201 <n-tag type="success" size="small">通过</n-tag></div><div><Check/> 响应时间小于 500ms <n-tag type="success" size="small">通过</n-tag></div><div><Check/> 响应体符合 Schema <n-tag type="success" size="small">通过</n-tag></div></div>
            </div>

            <div class="response-section">
              <div class="response-head"><strong>响应</strong><span v-if="sent" class="status-code"><i></i>201 Created</span><span>186 ms</span><span>1.12 KB</span><div/><button>Pretty</button><button>Raw</button><button><Copy :size="14"/></button></div>
              <div class="response-tabs"><button class="active">Body</button><button>Headers <em>9</em></button><button>Test Results <em class="green">3/3</em></button></div>
              <div class="response-body"><div class="line-numbers">1<br>2<br>3<br>4<br>5<br>6<br>7<br>8<br>9<br>10<br>11</div><pre>{{ responseText }}</pre></div>
            </div>
          </section>

          <aside v-if="aiOpen" class="ai-panel">
            <div class="ai-head"><div class="ai-icon"><Sparkles :size="16"/></div><div><strong>API 助手</strong><span>基于当前接口上下文</span></div><button @click="aiOpen = false"><X :size="16"/></button></div>
            <div class="ai-score-card"><div class="score-ring"><span>92</span><small>/100</small></div><div><strong>交付质量优秀</strong><span>规范完整，建议修复 2 个问题</span></div><ChevronRight :size="16"/></div>
            <div class="ai-content">
              <div class="ai-label">AI 评审发现</div>
              <div class="insight warn"><div><Zap :size="15"/></div><p><strong>缺少幂等性保障</strong><span>创建订单建议增加 <code>Idempotency-Key</code> 请求头，避免重复提交。</span><button @click="message.success('已添加 Idempotency-Key 请求头')">一键修复</button></p></div>
              <div class="insight info"><div><ShieldCheck :size="15"/></div><p><strong>金额字段可更明确</strong><span><code>amount</code> 建议在描述中注明单位为“分”，避免歧义。</span><button @click="message.success('字段描述已补充')">补充描述</button></p></div>
              <div class="passed-title"><Check :size="14"/> 已通过 8 项检查</div>
              <div class="passed-grid"><span>字段命名规范</span><span>状态码正确</span><span>Schema 完整</span><span>示例数据有效</span></div>
            </div>
            <div class="suggestions"><span>你可以问</span><button>帮我生成异常测试用例</button><button>检查这个接口的安全性</button></div>
            <div class="ai-input"><textarea placeholder="询问关于此接口的问题..."/><button><Send :size="15"/></button><span><kbd>⌘</kbd><kbd>↵</kbd> 发送</span></div>
          </aside>
          <button v-else class="open-ai" @click="aiOpen = true"><Sparkles :size="17"/></button>
        </main>

        <n-modal v-model:show="showNewSpec" preset="card" title="创建接口规范" class="app-modal" style="width: 520px">
          <div class="modal-form"><label>接口名称<n-input placeholder="例如：创建退款单" /></label><label>所属服务<n-select :options="[{label:'订单服务',value:'order'},{label:'支付服务',value:'pay'}]" placeholder="选择服务" /></label><label>需求描述<n-input type="textarea" placeholder="描述业务目标、输入与预期输出，AI 将辅助生成规范…" :rows="4" /></label><div class="ai-generate"><Sparkles :size="16"/><span>AI 将生成路径、字段 Schema、示例和验收用例</span></div><n-button type="primary" block @click="showNewSpec = false; message.success('接口规范草稿已生成')"><WandSparkles :size="16"/> 生成规范草稿</n-button></div>
        </n-modal>

        <n-modal v-model:show="showRunner" preset="card" title="自动化测试运行器" class="app-modal" style="width: 640px">
          <div class="runner-summary"><div><Activity/><span><strong>24</strong>测试用例</span></div><div><Check/><span><strong>22</strong>预计通过</span></div><div><Clock3/><span><strong>~18s</strong>预计耗时</span></div></div>
          <div class="suite-row"><span><Play :size="14"/> 订单服务 · 回归测试</span><n-progress type="line" :percentage="92" :show-indicator="false" /></div>
          <n-button type="primary" block @click="showRunner = false; message.success('测试任务已开始运行')"><Play :size="16"/> 开始运行全部测试</n-button>
        </n-modal>
      </div>
  </n-config-provider>
</template>
