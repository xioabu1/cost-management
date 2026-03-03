<template>
  <el-dialog
    v-model="visible"
    :title="`📋 成本分析详情   ${quotationDetail?.quotation_no || ''}`"
    width="850px"
    :close-on-click-modal="false"
    destroy-on-close
    @close="handleClose"
  >
    <template #header="{ titleId, titleClass }">
      <div class="dialog-header">
        <span :id="titleId" :class="titleClass">📋 成本分析详情   {{ quotationDetail?.quotation_no || '' }}</span>
        <div class="header-actions">
          <el-button type="primary" size="small" icon="Download" @click="handleExport">导出</el-button>
        </div>
      </div>
    </template>

    <div v-loading="loading" class="salesperson-approved-content">
      <template v-if="quotationDetail">
        <!-- 报价单摘要 -->
        <div class="section">
          <div class="section-title">报价单摘要</div>
          <div class="summary-grid">
            <div class="summary-card">
              <div class="card-title">📋 基本信息</div>
              <div class="card-content">
                <div class="info-line"><span class="label">客户名称:</span><span class="value">{{ quotationDetail.customer_name }}</span></div>
                <div class="info-line"><span class="label">客户地区:</span><span class="value">{{ quotationDetail.customer_region || '-' }}</span></div>
                <div class="info-line"><span class="label">销售类型:</span><span class="value">{{ getSalesTypeName(quotationDetail.sales_type) }}</span></div>
                <div class="info-line"><span class="label">法规类别:</span><span class="value">{{ quotationDetail.regulation_name || '-' }}</span></div>
                <div class="info-line"><span class="label">产品型号:</span><span class="value">{{ quotationDetail.model_name }}</span></div>
                <div class="info-line"><span class="label">订单数量:</span><span class="value">{{ formatQuantity(quotationDetail.quantity) }}</span></div>
                <div class="info-line"><span class="label">包装配置:</span><span class="value">{{ formatPackaging(quotationDetail.packaging_config) }}</span></div>
              </div>
            </div>
            <div class="summary-card">
              <div class="card-title">💰 成本分析信息</div>
              <div class="card-content">
                <div class="info-line final-price">
                  <span class="label">最终价格:</span>
                  <span class="value highlight">{{ formatAmount(quotationDetail.final_price, quotationDetail.currency) }}</span>
                </div>
                <div class="profit-table">
                  <div class="profit-header">
                    <span>利润档位</span>
                    <span>单价({{ quotationDetail.currency }})</span>
                  </div>
                  <div v-for="item in profitPricing" :key="item.rate" class="profit-row">
                    <span>{{ item.rate }}%</span>
                    <span>{{ item.price.toFixed(4) }}</span>
                  </div>
                </div>
              </div>
            </div>
          </div>
        </div>

        <!-- 审核结果 -->
        <div class="section">
          <div class="section-title status-approved">✅ 审核结果</div>
          <div class="result-box result-approved">
            <div class="result-icon">✅</div>
            <div class="result-text">恭喜！该成本分析已审核通过</div>
          </div>
          <div v-if="reviewComment" class="comment-section">
            <div class="comment-label">审核批注：</div>
            <div class="comment-content">{{ reviewComment }}</div>
          </div>
        </div>

        <!-- 审核流程 -->
        <div class="section">
          <div class="section-title">审核流程</div>
          <div class="timeline">
            <div v-for="(history, index) in reviewHistory" :key="history.id" class="timeline-item">
              <div class="timeline-dot" :class="{ 'active': index === reviewHistory.length - 1 }"></div>
              <div class="timeline-content">
                <div class="timeline-action">{{ getReviewActionName(history.action) }}</div>
                <div class="timeline-operator">{{ history.operator_name }}</div>
                <div class="timeline-time">{{ formatDateTime(history.created_at) }}</div>
              </div>
              <div v-if="index < reviewHistory.length - 1" class="timeline-line"></div>
            </div>
          </div>
        </div>

        <!-- 注意提示 -->
        <div class="notice-text">
          ⚠️ 注意：已审核通过的成本分析不可修改，如需调整请联系管理员
        </div>
      </template>
    </div>
  </el-dialog>
</template>

<script setup>
import { ref, computed, watch } from 'vue'
import { ElMessage } from 'element-plus'
import { useReviewStore } from '@/store/review'
import {
  getSalesTypeName,
  formatDateTime,
  formatAmount,
  formatQuantity,
  calculateProfitPricing,
  getReviewActionName
} from '@/utils/review'

const props = defineProps({
  modelValue: {
    type: Boolean,
    default: false
  },
  quotationId: {
    type: Number,
    default: null
  }
})

const emit = defineEmits(['update:modelValue'])

const reviewStore = useReviewStore()

const visible = ref(false)
const loading = ref(false)

// 数据
const quotationDetail = ref(null)
const reviewHistory = ref([])
const reviewComment = ref('')

// 监听 modelValue
watch(() => props.modelValue, (val) => {
  visible.value = val
  if (val && props.quotationId) {
    loadDetail()
  }
})

watch(visible, (val) => {
  emit('update:modelValue', val)
})

// 利润分析
const profitPricing = computed(() => {
  if (!quotationDetail.value) return []
  return calculateProfitPricing(
    quotationDetail.value.final_price,
    quotationDetail.value.sales_type
  )
})

// 加载详情
const loadDetail = async () => {
  loading.value = true
  try {
    const response = await reviewStore.fetchReviewDetail(props.quotationId)
    if (response.success) {
      quotationDetail.value = response.data.quotation
      reviewHistory.value = response.data.history || []
      // 获取审核批注
      const comments = response.data.comments || []
      reviewComment.value = comments.length > 0 ? comments[comments.length - 1].content : ''
    }
  } catch (error) {
    ElMessage.error('加载详情失败')
  } finally {
    loading.value = false
  }
}

// 格式化包装配置
const formatPackaging = (config) => {
  if (!config) return '-'
  if (typeof config === 'string') {
    try {
      config = JSON.parse(config)
    } catch {
      return config
    }
  }
  const parts = []
  if (config.pieces_per_bag) parts.push(`${config.pieces_per_bag}片/袋`)
  if (config.bags_per_box) parts.push(`${config.bags_per_box}袋/盒`)
  if (config.boxes_per_carton) parts.push(`${config.boxes_per_carton}盒/箱`)
  return parts.join(', ') || '-'
}

// 导出
const handleExport = () => {
  ElMessage.info('导出功能开发中')
}

// 关闭
const handleClose = () => {
  visible.value = false
  quotationDetail.value = null
  reviewHistory.value = []
  reviewComment.value = ''
}
</script>

<style scoped>
.dialog-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  width: 100%;
}

.header-actions {
  margin-right: 30px;
}

.salesperson-approved-content {
  max-height: 65vh;
  overflow-y: auto;
}

.section {
  margin-bottom: 20px;
  padding: 16px;
  background: #fafafa;
  border-radius: 8px;
}

.section-title {
  font-weight: 600;
  font-size: 14px;
  color: #303133;
  margin-bottom: 12px;
  padding-bottom: 8px;
  border-bottom: 1px solid #e4e7ed;
}

.status-approved {
  color: #67c23a;
}

.summary-grid {
  display: flex;
  gap: 20px;
}

.summary-card {
  flex: 1;
  background: #fff;
  border: 1px solid #e4e7ed;
  border-radius: 8px;
  padding: 16px;
}

.card-title {
  font-weight: 600;
  font-size: 14px;
  color: #303133;
  margin-bottom: 12px;
}

.card-content {
  display: flex;
  flex-direction: column;
  gap: 8px;
}

.info-line {
  display: flex;
  font-size: 13px;
}

.info-line .label {
  width: 70px;
  color: #909399;
}

.info-line .value {
  color: #303133;
}

.info-line .value.highlight {
  color: #409eff;
  font-weight: 600;
  font-size: 16px;
}

.final-price {
  margin-bottom: 12px;
}

.profit-table {
  background: #f5f7fa;
  border-radius: 4px;
  overflow: hidden;
}

.profit-header {
  display: flex;
  justify-content: space-between;
  padding: 8px 12px;
  background: #e4e7ed;
  font-size: 12px;
  color: #606266;
  font-weight: 600;
}

.profit-row {
  display: flex;
  justify-content: space-between;
  padding: 8px 12px;
  font-size: 13px;
  color: #303133;
  border-bottom: 1px solid #e4e7ed;
}

.profit-row:last-child {
  border-bottom: none;
}

.result-box {
  display: flex;
  flex-direction: column;
  align-items: center;
  padding: 24px;
  border-radius: 8px;
}

.result-approved {
  background: #f0f9eb;
  border: 1px solid #e1f3d8;
}

.result-icon {
  font-size: 36px;
  margin-bottom: 12px;
}

.result-text {
  font-size: 18px;
  font-weight: 600;
  color: #67c23a;
}

.comment-section {
  margin-top: 16px;
}

.comment-label {
  font-size: 13px;
  color: #909399;
  margin-bottom: 8px;
}

.comment-content {
  padding: 12px;
  background: #fff;
  border: 1px solid #e4e7ed;
  border-radius: 4px;
  font-size: 13px;
  color: #303133;
}

.timeline {
  display: flex;
  align-items: flex-start;
  padding: 16px 0;
}

.timeline-item {
  display: flex;
  flex-direction: column;
  align-items: center;
  position: relative;
  flex: 1;
}

.timeline-dot {
  width: 12px;
  height: 12px;
  border-radius: 50%;
  background: #dcdfe6;
  margin-bottom: 8px;
}

.timeline-dot.active {
  background: #67c23a;
}

.timeline-content {
  text-align: center;
}

.timeline-action {
  font-size: 13px;
  font-weight: 600;
  color: #303133;
}

.timeline-operator {
  font-size: 12px;
  color: #606266;
  margin-top: 4px;
}

.timeline-time {
  font-size: 12px;
  color: #909399;
  margin-top: 2px;
}

.timeline-line {
  position: absolute;
  top: 6px;
  left: 50%;
  width: 100%;
  height: 2px;
  background: #dcdfe6;
  z-index: -1;
}

.notice-text {
  padding: 12px 16px;
  background: #fdf6ec;
  border: 1px solid #faecd8;
  border-radius: 4px;
  font-size: 13px;
  color: #e6a23c;
}
</style>
