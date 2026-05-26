<template>
  <div class="restocking">
    <div class="page-header">
      <h2>Restocking</h2>
      <p>Budget-based restocking recommendations from demand forecasts</p>
    </div>

    <!-- Budget Slider -->
    <div class="card">
      <div class="card-header">
        <h3 class="card-title">Budget</h3>
      </div>
      <div class="budget-control">
        <input
          type="range"
          min="0"
          max="500000"
          step="5000"
          v-model.number="budget"
          class="budget-slider"
        />
        <span class="budget-display">{{ formatCurrency(budget) }}</span>
      </div>
    </div>

    <!-- Loading -->
    <div v-if="loading" class="loading">Loading recommendations...</div>

    <!-- Error -->
    <div v-else-if="error" class="error">{{ error }}</div>

    <!-- Zero budget state -->
    <div v-else-if="budget === 0" class="empty-state">
      <p>Set a budget to see recommendations</p>
    </div>

    <!-- Success banner -->
    <div v-else-if="successOrder" class="success-banner">
      <div class="success-content">
        <div class="success-title">Order placed successfully</div>
        <div class="success-details">
          <span class="success-order-number">{{ successOrder.order_number }}</span>
          <span class="success-delivery">Expected delivery: {{ formatDate(successOrder.expected_delivery) }}</span>
        </div>
      </div>
      <button class="btn btn-secondary" @click="resetView">Place another order</button>
    </div>

    <!-- Recommendations -->
    <template v-else-if="recommendations.length > 0">
      <div class="card">
        <div class="card-header">
          <h3 class="card-title">Recommendations</h3>
        </div>
        <div class="table-container">
          <table>
            <thead>
              <tr>
                <th>SKU</th>
                <th>Item Name</th>
                <th>Demand Gap</th>
                <th>Qty to Order</th>
                <th>Unit Cost</th>
                <th>Line Total</th>
              </tr>
            </thead>
            <tbody>
              <tr v-for="item in recommendations" :key="item.sku">
                <td><strong>{{ item.sku }}</strong></td>
                <td>{{ item.name }}</td>
                <td><span class="badge info">+{{ item.demand_gap }} units</span></td>
                <td>{{ item.quantity }}</td>
                <td>{{ formatCurrency(item.unit_cost) }}</td>
                <td><strong>{{ formatCurrency(item.line_total) }}</strong></td>
              </tr>
            </tbody>
          </table>
        </div>

        <!-- Summary bar -->
        <div class="summary-bar">
          <span class="summary-text">
            {{ recommendations.length }} items
            &middot;
            Total cost: {{ formatCurrency(totalCost) }}
            &middot;
            Budget remaining: {{ formatCurrency(budgetRemaining) }}
          </span>
          <span v-if="totalCost > budget" class="budget-warning">
            Order exceeds budget by {{ formatCurrency(totalCost - budget) }}
          </span>
        </div>
      </div>

      <!-- Place Order -->
      <div class="actions">
        <button
          class="btn btn-primary"
          :disabled="placingOrder || recommendations.length === 0"
          @click="placeOrder"
        >
          {{ placingOrder ? 'Placing...' : 'Place Order' }}
        </button>
      </div>
    </template>

    <!-- No recommendations for given budget -->
    <div v-else-if="budget > 0" class="empty-state">
      <p>No recommendations available for this budget</p>
    </div>
  </div>
</template>

<script>
import { ref, computed, watch } from 'vue'
import { api } from '../api'

export default {
  name: 'Restocking',
  setup() {
    const budget = ref(0)
    const recommendations = ref([])
    const loading = ref(false)
    const error = ref(null)
    const placingOrder = ref(false)
    const successOrder = ref(null)

    let debounceTimer = null

    const totalCost = computed(() =>
      recommendations.value.reduce((sum, item) => sum + item.line_total, 0)
    )

    const budgetRemaining = computed(() => budget.value - totalCost.value)

    const formatCurrency = (value) =>
      value.toLocaleString('en-US', { style: 'currency', currency: 'USD' })

    const formatDate = (dateString) => {
      const date = new Date(dateString)
      if (isNaN(date.getTime())) return dateString
      return date.toLocaleDateString('en-US', {
        year: 'numeric',
        month: 'short',
        day: 'numeric'
      })
    }

    const loadRecommendations = async () => {
      if (budget.value === 0) {
        recommendations.value = []
        return
      }
      loading.value = true
      error.value = null
      try {
        recommendations.value = await api.getRestockingRecommendations(budget.value)
      } catch (err) {
        error.value = 'Failed to load recommendations'
        console.error(err)
      } finally {
        loading.value = false
      }
    }

    watch(budget, () => {
      successOrder.value = null
      clearTimeout(debounceTimer)
      debounceTimer = setTimeout(loadRecommendations, 400)
    })

    const placeOrder = async () => {
      placingOrder.value = true
      error.value = null
      try {
        const result = await api.createRestockingOrder({
          items: recommendations.value,
          total_budget: budget.value,
          total_cost: totalCost.value
        })
        successOrder.value = result
        recommendations.value = []
      } catch (err) {
        error.value = 'Failed to place order'
        console.error(err)
      } finally {
        placingOrder.value = false
      }
    }

    const resetView = () => {
      successOrder.value = null
      budget.value = 0
      recommendations.value = []
      error.value = null
    }

    return {
      budget,
      recommendations,
      loading,
      error,
      placingOrder,
      successOrder,
      totalCost,
      budgetRemaining,
      formatCurrency,
      formatDate,
      placeOrder,
      resetView
    }
  }
}
</script>

<style scoped>
.restocking {
  padding: 0;
}

.budget-control {
  display: flex;
  align-items: center;
  gap: 1.5rem;
  padding: 0.5rem 0;
}

.budget-slider {
  flex: 1;
  height: 6px;
  accent-color: #2563eb;
  cursor: pointer;
}

.budget-display {
  font-size: 1.25rem;
  font-weight: 700;
  color: #0f172a;
  min-width: 120px;
  text-align: right;
}

.summary-bar {
  display: flex;
  align-items: center;
  gap: 1.5rem;
  padding: 0.875rem 0.75rem;
  background: #f8fafc;
  border-top: 1px solid #e2e8f0;
  margin-top: 0.5rem;
  border-radius: 0 0 6px 6px;
}

.summary-text {
  font-size: 0.875rem;
  color: #334155;
  font-weight: 500;
}

.budget-warning {
  font-size: 0.875rem;
  font-weight: 600;
  color: #dc2626;
}

.actions {
  display: flex;
  justify-content: flex-end;
  margin-bottom: 1.25rem;
}

.btn {
  padding: 0.625rem 1.5rem;
  border-radius: 8px;
  font-size: 0.938rem;
  font-weight: 600;
  cursor: pointer;
  border: none;
  transition: all 0.2s ease;
}

.btn:disabled {
  opacity: 0.5;
  cursor: not-allowed;
}

.btn-primary {
  background: #2563eb;
  color: white;
}

.btn-primary:hover:not(:disabled) {
  background: #1d4ed8;
}

.btn-secondary {
  background: white;
  color: #334155;
  border: 1px solid #e2e8f0;
}

.btn-secondary:hover:not(:disabled) {
  background: #f8fafc;
  border-color: #cbd5e1;
}

.success-banner {
  display: flex;
  align-items: center;
  justify-content: space-between;
  gap: 1.5rem;
  background: #d1fae5;
  border: 1px solid #6ee7b7;
  border-radius: 10px;
  padding: 1.25rem 1.5rem;
  margin-bottom: 1.25rem;
}

.success-title {
  font-size: 1rem;
  font-weight: 700;
  color: #065f46;
  margin-bottom: 0.375rem;
}

.success-details {
  display: flex;
  gap: 1rem;
  font-size: 0.875rem;
  color: #065f46;
}

.success-order-number {
  font-weight: 600;
}

.empty-state {
  background: white;
  border: 1px solid #e2e8f0;
  border-radius: 10px;
  padding: 3rem;
  text-align: center;
  color: #64748b;
  font-size: 0.938rem;
}
</style>
