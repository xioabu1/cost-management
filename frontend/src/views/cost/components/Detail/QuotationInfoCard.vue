<template>
  <div class="cost-section">
    <div class="cost-section-header">
      <h3 class="text-sm font-semibold text-gray-900 mb-0 flex items-center gap-2">
        <div class="w-1 h-4 bg-green-500 rounded-full"></div>
        基础信息
      </h3>
    </div>
    <div class="review-info-grid info-grid-override">
      <div class="review-info-card">
        <div class="review-info-label">客户名称</div>
        <div class="review-info-value">{{ quotation.customer_name }}</div>
      </div>
      <div class="review-info-card">
        <div class="review-info-label">客户地区</div>
        <div class="review-info-value">{{ quotation.customer_region || '-' }}</div>
      </div>
      <div class="review-info-card">
        <div class="review-info-label">产品型号</div>
        <div class="review-info-value">{{ quotation.model_name }}</div>
      </div>
      <div class="review-info-card">
        <div class="review-info-label">订单数量</div>
        <div class="review-info-value">{{ formatQuantity(quotation.quantity) }} {{ quotation.quantity === 1 ? 'PC' : 'PCS' }}</div>
      </div>
      <div class="review-info-card lg:col-span-2">
        <div class="review-info-label">包装方式</div>
        <div class="review-info-value packaging-value">
          <span>{{ quotation.packaging_config_name || '-' }}</span>
          <span v-if="quotation.packaging_type" class="packaging-spec">{{ formatPackagingSpec(quotation) }}</span>
        </div>
      </div>
      <div class="review-info-card">
        <div class="review-info-label">创建人</div>
        <div class="review-info-value">{{ quotation.creator_name || '-' }}</div>
      </div>
      <div class="review-info-card">
        <div class="review-info-label">最终价格</div>
        <div class="review-info-value text-blue-600 font-bold">
          {{ formatNumber(quotation.final_price) }} {{ quotation.currency }}
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { formatNumber } from '@/utils/format'

defineProps({
  quotation: Object
})

const formatQuantity = (val) => {
  if (!val) return '0'
  return val.toString().replace(/\B(?=(\d{3})+(?!\d))/g, ",")
}

// 根据包装类型格式化层级规格显示
const formatPackagingSpec = (row) => {
  if (!row.packaging_type) return ''
  // 二层包装类型：no_box, blister_direct
  if (row.packaging_type === 'no_box') {
    return `(${row.layer1_qty}pc/袋, ${row.layer2_qty}袋/箱)`
  } else if (row.packaging_type === 'blister_direct') {
    return `(${row.layer1_qty}pc/泡壳, ${row.layer2_qty}泡壳/箱)`
  } else if (row.packaging_type === 'blister_bag') {
    return `(${row.layer1_qty}pc/袋, ${row.layer2_qty}袋/泡壳, ${row.layer3_qty}泡壳/箱)`
  }
  // 默认三层：standard_box
  return `(${row.layer1_qty}片/袋, ${row.layer2_qty}袋/盒, ${row.layer3_qty}盒/箱)`
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

/* 内容网格样式 */
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

.review-info-sub {
  font-size: 11px;
  color: #475569;
  margin-top: 2px;
}

.packaging-value {
  display: flex;
  align-items: center;
  gap: 8px;
  flex-wrap: nowrap;
}

.packaging-spec {
  font-size: 12px;
  color: #64748b;
  white-space: nowrap;
}

.info-grid-override {
  grid-template-columns: repeat(4, 1fr);
}

@media (max-width: 1024px) {
  .info-grid-override {
    grid-template-columns: repeat(2, 1fr);
  }
}
</style>
