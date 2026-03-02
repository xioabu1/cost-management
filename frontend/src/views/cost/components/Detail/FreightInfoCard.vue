<template>
  <div class="cost-section">
    <div class="cost-section-header">
      <h3 class="text-sm font-semibold text-gray-900 mb-0 flex items-center gap-2">
        <div class="w-1 h-4 bg-green-500 rounded-full"></div>
        运费及物流
      </h3>
    </div>
    <div class="review-logistics-panel">
      <div class="review-info-grid">
        <div class="review-info-card">
          <div class="review-info-label">销售类型</div>
          <div class="review-info-value">
            <StatusBadge type="sales_type" :value="quotation.sales_type" />
          </div>
        </div>
        <div class="review-info-card">
          <div class="review-info-label">发货方式</div>
          <div class="review-info-value">{{ formatShippingMethod(quotation.shipping_method) || '-' }}</div>
        </div>
        <div class="review-info-card">
          <div class="review-info-label">发货港口</div>
          <div class="review-info-value">{{ quotation.port_name || quotation.port || '-' }}</div>
        </div>
        <div class="review-info-card">
          <div class="review-info-label">运费计入成本</div>
          <div class="review-info-value" :class="quotation.include_freight_in_base ? 'text-green-600' : 'text-gray-400'">
            {{ quotation.include_freight_in_base ? '已计入' : '未计入' }}
          </div>
        </div>
        <div class="review-info-card">
          <div class="review-info-label">总箱数</div>
          <div class="review-info-value">{{ shippingInfo.cartons || '-' }} 箱</div>
        </div>
        <div class="review-info-card">
          <div class="review-info-label">总体积 (CBM)</div>
          <div class="review-info-value">{{ shippingInfo.cbm || '-' }} m³</div>
        </div>
        <div class="review-info-card">
          <div class="review-info-label">总运费</div>
          <div class="review-info-value">¥{{ formatNumber(quotation.freight_total) }}</div>
        </div>
        <div class="review-info-card">
          <div class="review-info-label">每片分摊</div>
          <div class="review-info-value highlight">¥{{ formatNumber(quotation.freight_per_unit) }}</div>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { formatNumber } from '@/utils/format'
import StatusBadge from '@/components/common/StatusBadge.vue'

defineProps({
  quotation: Object,
  shippingInfo: Object
})

const formatShippingMethod = (method) => {
  const map = { fcl_40: '40GP', fcl_20: '20GP', lcl: 'LCL散货' }
  return map[method] || method
}
</script>

<style scoped>
/* 卡片容器样式 - 与 CostDetailTabs 一致 */
.cost-section {
  background: #fff;
  border: 1px solid #e5e7eb;
  border-radius: 12px;
  overflow: hidden;
}

.cost-section-header {
  padding: 20px 20px 16px 20px;
  border-bottom: 1px solid #e5e7eb;
  background: #fff;
}

/* 内容网格样式 - 与基础信息卡片一致 */
.review-info-grid {
  display: grid;
  grid-template-columns: repeat(4, 1fr);
  gap: 12px;
  padding: 16px 20px;
}

.review-info-card {
  background: #fff;
  border: 1px solid #e5e7eb;
  border-radius: 6px;
  padding: 12px 14px;
}

.review-info-label {
  font-size: 11px;
  font-weight: 500;
  color: #94a3b8;
  text-transform: uppercase;
  letter-spacing: 0.5px;
  margin-bottom: 6px;
}

.review-info-value {
  font-size: 13px;
  font-weight: 500;
  color: #0f172a;
  line-height: 1.4;
}

.highlight {
  color: #3b82f6;
  font-weight: 600;
}
</style>
