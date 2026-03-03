<template>
  <el-dialog
    :model-value="modelValue"
    :title="`📤 重新提交成本分析  ${quotation?.quotation_no || ''}`"
    width="500px"
    :close-on-click-modal="false"
    :before-close="handleBeforeClose"
    append-to-body
  >
    <div class="resubmit-content">
      <el-divider />
      
      <p class="confirm-text">确认将此成本分析重新提交审核？</p>

      <!-- 成本分析摘要 -->
      <div class="summary-box">
        <div class="info-line"><span class="label">客户名称:</span><span class="value">{{ quotation?.customer_name }}</span></div>
        <div class="info-line"><span class="label">产品型号:</span><span class="value">{{ quotation?.model_name }}</span></div>
        <div class="info-line"><span class="label">订单数量:</span><span class="value">{{ formatQuantity(quotation?.quantity) }}</span></div>
        <div class="info-line"><span class="label">最终价格:</span><span class="value highlight">{{ formatAmount(quotation?.final_price, quotation?.currency) }}</span></div>
        
        <!-- 利润区间 -->
        <div class="profit-section">
          <div class="profit-label">利润区间:</div>
          <div class="profit-list">
            <div v-for="item in profitPricing" :key="item.rate" class="profit-item">
              {{ item.rate }}%: {{ item.price.toFixed(4) }} {{ item.currency }}
            </div>
          </div>
        </div>
      </div>

      <!-- 提示 -->
      <div class="hint-text">
        💡 提示：提交后成本分析将进入待审核状态，等待审核人审核
      </div>
    </div>

    <template #footer>
      <div class="dialog-footer">
        <el-button @click="handleClose">取消</el-button>
        <el-button type="primary" @click="handleConfirm" :loading="loading">确认提交</el-button>
      </div>
    </template>
  </el-dialog>
</template>

<script setup>
import { ref, watch } from 'vue'
import { formatAmount, formatQuantity } from '@/utils/review'

const props = defineProps({
  modelValue: {
    type: Boolean,
    default: false
  },
  quotation: {
    type: Object,
    default: null
  },
  profitPricing: {
    type: Array,
    default: () => []
  }
})

const emit = defineEmits(['update:modelValue', 'confirm'])

const loading = ref(false)

const handleBeforeClose = (done) => {
  emit('update:modelValue', false)
  done()
}

const handleClose = () => {
  emit('update:modelValue', false)
}

const handleConfirm = () => {
  loading.value = true
  emit('confirm')
  setTimeout(() => {
    loading.value = false
  }, 500)
}
</script>

<style scoped>
.resubmit-content {
  padding: 0 10px;
}

.confirm-text {
  font-size: 15px;
  color: #303133;
  margin-bottom: 16px;
}

.summary-box {
  background: #f5f7fa;
  border: 1px solid #e4e7ed;
  border-radius: 8px;
  padding: 16px;
  display: flex;
  flex-direction: column;
  gap: 10px;
}

.info-line {
  display: flex;
  font-size: 14px;
}

.info-line .label {
  width: 80px;
  color: #909399;
}

.info-line .value {
  color: #303133;
}

.info-line .value.highlight {
  color: #409eff;
  font-weight: 600;
}

.profit-section {
  margin-top: 8px;
  padding-top: 12px;
  border-top: 1px dashed #e4e7ed;
}

.profit-label {
  font-size: 14px;
  color: #909399;
  margin-bottom: 8px;
}

.profit-list {
  display: flex;
  flex-direction: column;
  gap: 4px;
  padding-left: 80px;
}

.profit-item {
  font-size: 14px;
  color: #303133;
}

.hint-text {
  margin-top: 16px;
  padding: 10px 12px;
  background: #ecf5ff;
  border-radius: 4px;
  font-size: 13px;
  color: #409eff;
}

.dialog-footer {
  display: flex;
  justify-content: center;
  gap: 20px;
}
</style>
