<template>
  <div v-if="isOpen" class="modal-overlay" @click.self="$emit('close')">
    <div class="modal">
      <div class="modal-header">
        <h2>{{ mode === 'create' ? 'Create Purchase Order' : 'Purchase Order Details' }}</h2>
        <button class="close-btn" @click="$emit('close')">
          <svg width="20" height="20" viewBox="0 0 20 20" fill="none">
            <path d="M15 5L5 15M5 5L15 15" stroke="currentColor" stroke-width="2" stroke-linecap="round"/>
          </svg>
        </button>
      </div>

      <div v-if="backlogItem" class="backlog-summary">
        <div class="summary-row">
          <span class="summary-label">Backlog Item</span>
          <span class="summary-value">{{ backlogItem.item_name }} ({{ backlogItem.item_sku }})</span>
        </div>
        <div class="summary-row">
          <span class="summary-label">Shortage</span>
          <span class="summary-value danger">{{ Math.abs(backlogItem.quantity_needed - backlogItem.quantity_available) }} units short</span>
        </div>
      </div>

      <!-- Create mode: form -->
      <form v-if="mode === 'create'" @submit.prevent="submitForm" class="modal-form">
        <div class="form-group">
          <label>Supplier Name</label>
          <input v-model="form.supplier_name" type="text" required placeholder="Enter supplier name" class="form-input" />
        </div>
        <div class="form-row">
          <div class="form-group">
            <label>Quantity</label>
            <input v-model.number="form.quantity" type="number" min="1" required class="form-input" />
          </div>
          <div class="form-group">
            <label>Unit Cost ($)</label>
            <input v-model.number="form.unit_cost" type="number" min="0" step="0.01" required class="form-input" />
          </div>
        </div>
        <div class="form-group">
          <label>Expected Delivery Date</label>
          <input v-model="form.expected_delivery_date" type="date" required class="form-input" />
        </div>
        <div class="form-group">
          <label>Notes (optional)</label>
          <textarea v-model="form.notes" placeholder="Any additional notes..." class="form-textarea" rows="3"></textarea>
        </div>
        <div v-if="totalCost > 0" class="total-cost">
          Total Cost: <strong>{{ formatCurrency(totalCost) }}</strong>
        </div>
        <div class="modal-actions">
          <button type="button" class="btn-secondary" @click="$emit('close')">Cancel</button>
          <button type="submit" class="btn-primary" :disabled="submitting">
            {{ submitting ? 'Creating...' : 'Create Purchase Order' }}
          </button>
        </div>
      </form>

      <!-- View mode: display PO details -->
      <div v-else class="po-details">
        <div v-if="loadingPO" class="loading">Loading...</div>
        <div v-else-if="purchaseOrder" class="details-grid">
          <div class="detail-row">
            <span class="detail-label">PO Number</span>
            <span class="detail-value po-id">{{ purchaseOrder.id }}</span>
          </div>
          <div class="detail-row">
            <span class="detail-label">Supplier</span>
            <span class="detail-value">{{ purchaseOrder.supplier_name }}</span>
          </div>
          <div class="detail-row">
            <span class="detail-label">Quantity</span>
            <span class="detail-value">{{ purchaseOrder.quantity }} units</span>
          </div>
          <div class="detail-row">
            <span class="detail-label">Unit Cost</span>
            <span class="detail-value">{{ formatCurrency(purchaseOrder.unit_cost) }}</span>
          </div>
          <div class="detail-row">
            <span class="detail-label">Total Cost</span>
            <span class="detail-value strong">{{ formatCurrency(purchaseOrder.quantity * purchaseOrder.unit_cost) }}</span>
          </div>
          <div class="detail-row">
            <span class="detail-label">Expected Delivery</span>
            <span class="detail-value">{{ purchaseOrder.expected_delivery_date }}</span>
          </div>
          <div class="detail-row">
            <span class="detail-label">Status</span>
            <span :class="['badge', purchaseOrder.status]">{{ purchaseOrder.status }}</span>
          </div>
          <div v-if="purchaseOrder.notes" class="detail-row">
            <span class="detail-label">Notes</span>
            <span class="detail-value">{{ purchaseOrder.notes }}</span>
          </div>
        </div>
        <div class="modal-actions">
          <button class="btn-primary" @click="$emit('close')">Close</button>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import { ref, computed, watch } from 'vue'
import { api } from '../api'

export default {
  name: 'PurchaseOrderModal',
  props: {
    isOpen: { type: Boolean, default: false },
    backlogItem: { type: Object, default: null },
    mode: { type: String, default: 'create' }
  },
  emits: ['close', 'po-created'],
  setup(props, { emit }) {
    const submitting = ref(false)
    const loadingPO = ref(false)
    const purchaseOrder = ref(null)

    const form = ref({
      supplier_name: '',
      quantity: props.backlogItem ? Math.abs(props.backlogItem.quantity_needed - props.backlogItem.quantity_available) : 1,
      unit_cost: 0,
      expected_delivery_date: '',
      notes: ''
    })

    const totalCost = computed(() => {
      return (form.value.quantity || 0) * (form.value.unit_cost || 0)
    })

    const formatCurrency = (value) => {
      return new Intl.NumberFormat('en-US', { style: 'currency', currency: 'USD' }).format(value)
    }

    const resetForm = () => {
      form.value = {
        supplier_name: '',
        quantity: props.backlogItem ? Math.abs(props.backlogItem.quantity_needed - props.backlogItem.quantity_available) : 1,
        unit_cost: 0,
        expected_delivery_date: '',
        notes: ''
      }
      purchaseOrder.value = null
    }

    const loadPO = async () => {
      if (!props.backlogItem) return
      loadingPO.value = true
      try {
        purchaseOrder.value = await api.getPurchaseOrderByBacklogItem(props.backlogItem.id)
      } catch {
        purchaseOrder.value = null
      } finally {
        loadingPO.value = false
      }
    }

    watch(() => props.isOpen, (open) => {
      if (open) {
        resetForm()
        if (props.mode === 'view') loadPO()
      }
    })

    const submitForm = async () => {
      if (!props.backlogItem) return
      submitting.value = true
      try {
        const poData = await api.createPurchaseOrder({
          backlog_item_id: props.backlogItem.id,
          supplier_name: form.value.supplier_name,
          quantity: form.value.quantity,
          unit_cost: form.value.unit_cost,
          expected_delivery_date: form.value.expected_delivery_date,
          notes: form.value.notes || null
        })
        emit('po-created', poData)
      } catch (err) {
        console.error('Failed to create PO:', err)
      } finally {
        submitting.value = false
      }
    }

    return { form, submitting, loadingPO, purchaseOrder, totalCost, formatCurrency, submitForm }
  }
}
</script>

<style scoped>
.modal-overlay {
  position: fixed;
  inset: 0;
  background: rgba(0, 0, 0, 0.5);
  display: flex;
  align-items: center;
  justify-content: center;
  z-index: 1000;
  padding: 1rem;
}

.modal {
  background: white;
  border-radius: 12px;
  width: 100%;
  max-width: 520px;
  box-shadow: 0 20px 60px rgba(0, 0, 0, 0.2);
}

.modal-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 1.25rem 1.5rem;
  border-bottom: 1px solid #e2e8f0;
}

.modal-header h2 {
  font-size: 1.125rem;
  font-weight: 700;
  color: #0f172a;
}

.close-btn {
  background: none;
  border: none;
  cursor: pointer;
  color: #64748b;
  padding: 0.25rem;
  border-radius: 4px;
  display: flex;
}

.close-btn:hover {
  background: #f1f5f9;
  color: #0f172a;
}

.backlog-summary {
  padding: 1rem 1.5rem;
  background: #f8fafc;
  border-bottom: 1px solid #e2e8f0;
  display: flex;
  flex-direction: column;
  gap: 0.5rem;
}

.summary-row {
  display: flex;
  gap: 0.75rem;
  font-size: 0.875rem;
}

.summary-label {
  color: #64748b;
  font-weight: 600;
  min-width: 100px;
}

.summary-value { color: #0f172a; }
.summary-value.danger { color: #ef4444; font-weight: 600; }

.modal-form, .po-details {
  padding: 1.5rem;
  display: flex;
  flex-direction: column;
  gap: 1rem;
}

.form-row {
  display: grid;
  grid-template-columns: 1fr 1fr;
  gap: 1rem;
}

.form-group {
  display: flex;
  flex-direction: column;
  gap: 0.375rem;
}

.form-group label {
  font-size: 0.813rem;
  font-weight: 600;
  color: #374151;
}

.form-input, .form-textarea {
  padding: 0.5rem 0.75rem;
  border: 1px solid #cbd5e1;
  border-radius: 6px;
  font-size: 0.875rem;
  color: #0f172a;
  font-family: inherit;
  transition: border-color 0.2s;
}

.form-input:focus, .form-textarea:focus {
  outline: none;
  border-color: #3b82f6;
  box-shadow: 0 0 0 3px rgba(59, 130, 246, 0.1);
}

.form-textarea { resize: vertical; }

.total-cost {
  font-size: 0.875rem;
  color: #475569;
  padding: 0.75rem;
  background: #eff6ff;
  border-radius: 6px;
}

.total-cost strong { color: #1d4ed8; }

.modal-actions {
  display: flex;
  justify-content: flex-end;
  gap: 0.75rem;
  padding-top: 0.5rem;
}

.btn-primary {
  padding: 0.625rem 1.25rem;
  background: #2563eb;
  color: white;
  border: none;
  border-radius: 6px;
  font-size: 0.875rem;
  font-weight: 600;
  cursor: pointer;
  transition: background 0.2s;
}

.btn-primary:hover:not(:disabled) { background: #1d4ed8; }
.btn-primary:disabled { opacity: 0.6; cursor: not-allowed; }

.btn-secondary {
  padding: 0.625rem 1.25rem;
  background: white;
  color: #374151;
  border: 1px solid #d1d5db;
  border-radius: 6px;
  font-size: 0.875rem;
  font-weight: 600;
  cursor: pointer;
  transition: all 0.2s;
}

.btn-secondary:hover { background: #f9fafb; }

.loading {
  padding: 2rem;
  text-align: center;
  color: #64748b;
}

.details-grid {
  display: flex;
  flex-direction: column;
  gap: 0.875rem;
}

.detail-row {
  display: flex;
  gap: 1rem;
  font-size: 0.875rem;
  align-items: center;
}

.detail-label {
  min-width: 130px;
  color: #64748b;
  font-weight: 600;
  font-size: 0.813rem;
}

.detail-value { color: #0f172a; }
.detail-value.strong { font-weight: 700; color: #0f172a; }
.po-id { font-family: monospace; font-weight: 700; color: #2563eb; }

.badge {
  padding: 0.25rem 0.625rem;
  border-radius: 20px;
  font-size: 0.75rem;
  font-weight: 600;
  text-transform: capitalize;
}

.badge.pending { background: #fef9c3; color: #854d0e; }
.badge.completed { background: #dcfce7; color: #166534; }
.badge.cancelled { background: #fee2e2; color: #991b1b; }
</style>
