<script setup>
import { computed, onMounted, ref, watch } from 'vue'

const initialProducts = [
  { id: 1001, name: 'Studio Headphones', category: 'Electronics', price: 129, stock: 48, status: 'Active', updated: '2026-11-10' },
  { id: 1002, name: 'Everyday Backpack', category: 'Accessories', price: 79, stock: 12, status: 'Active', updated: '2026-11-09' },
  { id: 1003, name: 'Ceramic Coffee Set', category: 'Home & Living', price: 54, stock: 0, status: 'Out of stock', updated: '2026-11-08' },
  { id: 1004, name: 'Desk Lamp Mini', category: 'Home & Living', price: 42, stock: 7, status: 'Low stock', updated: '2026-11-07' },
  { id: 1005, name: 'Canvas Tote', category: 'Accessories', price: 24, stock: 86, status: 'Active', updated: '2026-11-06' },
  { id: 1006, name: 'Wireless Charger', category: 'Electronics', price: 39, stock: 31, status: 'Active', updated: '2026-11-05' },
]

const saved = localStorage.getItem('orbit-products')
const products = ref(saved ? JSON.parse(saved) : initialProducts)
watch(products, value => localStorage.setItem('orbit-products', JSON.stringify(value)), { deep: true })

const activeNav = ref('Products')
const search = ref('')
const statusFilter = ref('All')
const showModal = ref(false)
const editingId = ref(null)
const toast = ref('')
const mobileMenuOpen = ref(false)

const blankForm = () => ({ name: '', category: 'Electronics', price: 0, stock: 0 })
const form = ref(blankForm())

const filteredProducts = computed(() => products.value.filter(product => {
  const query = search.value.trim().toLowerCase()
  const matchesSearch = !query || product.name.toLowerCase().includes(query) || product.category.toLowerCase().includes(query) || String(product.id).includes(query)
  const matchesStatus = statusFilter.value === 'All' || product.status === statusFilter.value
  return matchesSearch && matchesStatus
}))

const totalValue = computed(() => products.value.reduce((sum, item) => sum + item.price * item.stock, 0))
const lowStockCount = computed(() => products.value.filter(item => item.stock > 0 && item.stock < 10).length)
const activeCount = computed(() => products.value.filter(item => item.status === 'Active').length)

function notify(message) {
  toast.value = message
  window.setTimeout(() => { toast.value = '' }, 2400)
}

function statusFor(stock) {
  if (stock === 0) return 'Out of stock'
  if (stock < 10) return 'Low stock'
  return 'Active'
}

function openCreate() {
  editingId.value = null
  form.value = blankForm()
  showModal.value = true
}

function openEdit(product) {
  editingId.value = product.id
  form.value = { name: product.name, category: product.category, price: product.price, stock: product.stock }
  showModal.value = true
}

function saveProduct() {
  if (!form.value.name.trim()) {
    notify('Please enter a product name.')
    return
  }
  const clean = {
    name: form.value.name.trim(),
    category: form.value.category,
    price: Math.max(0, Number(form.value.price) || 0),
    stock: Math.max(0, Number(form.value.stock) || 0),
  }
  if (editingId.value) {
    const index = products.value.findIndex(item => item.id === editingId.value)
    if (index !== -1) products.value[index] = { ...products.value[index], ...clean, status: statusFor(clean.stock), updated: new Date().toISOString().slice(0, 10) }
    notify('Product updated.')
  } else {
    products.value.unshift({ id: Date.now(), ...clean, status: statusFor(clean.stock), updated: new Date().toISOString().slice(0, 10) })
    notify('Product added.')
  }
  showModal.value = false
}

function deleteProduct(product) {
  if (window.confirm(`Delete "${product.name}"?`)) {
    products.value = products.value.filter(item => item.id !== product.id)
    notify('Product deleted.')
  }
}

function selectNav(item) {
  activeNav.value = item
  mobileMenuOpen.value = false
  if (item !== 'Products') notify(`${item} is a demo navigation item.`)
}

function resetDemo() {
  if (window.confirm('Restore the original demo products?')) {
    products.value = [...initialProducts]
    notify('Demo data restored.')
  }
}

onMounted(() => {
  // The dashboard is intentionally self-contained; no API connection is required.
})
</script>

<template>
  <div class="app-shell">
    <div v-if="mobileMenuOpen" class="mobile-backdrop" @click="mobileMenuOpen = false"></div>
    <aside class="sidebar" :class="{ 'sidebar-open': mobileMenuOpen }">
      <a class="brand" href="#" @click.prevent="selectNav('Overview')">
        <span class="brand-mark">o</span>
        <span>orbit<span class="brand-dot">.</span></span>
      </a>

      <div class="workspace-switcher">
        <div class="workspace-avatar">S</div>
        <div class="workspace-copy"><strong>Studio Workspace</strong><small>Free plan</small></div>
        <span class="chevron">⌄</span>
      </div>

      <p class="nav-label">WORKSPACE</p>
      <nav class="nav-list" aria-label="Main navigation">
        <button class="nav-item" :class="{ selected: activeNav === 'Overview' }" @click="selectNav('Overview')"><span class="nav-icon">▦</span> Overview</button>
        <button class="nav-item" :class="{ selected: activeNav === 'Products' }" @click="selectNav('Products')"><span class="nav-icon">◇</span> Products <span class="nav-count">{{ products.length }}</span></button>
        <button class="nav-item" :class="{ selected: activeNav === 'Orders' }" @click="selectNav('Orders')"><span class="nav-icon">▤</span> Orders</button>
        <button class="nav-item" :class="{ selected: activeNav === 'Customers' }" @click="selectNav('Customers')"><span class="nav-icon">♧</span> Customers</button>
        <button class="nav-item" :class="{ selected: activeNav === 'Analytics' }" @click="selectNav('Analytics')"><span class="nav-icon">⌁</span> Analytics</button>
      </nav>

      <p class="nav-label second-label">PREFERENCES</p>
      <nav class="nav-list">
        <button class="nav-item" :class="{ selected: activeNav === 'Settings' }" @click="selectNav('Settings')"><span class="nav-icon">⚙</span> Settings</button>
        <button class="nav-item" :class="{ selected: activeNav === 'Help center' }" @click="selectNav('Help center')"><span class="nav-icon">?</span> Help center</button>
      </nav>

      <div class="sidebar-bottom">
        <div class="help-card">
          <div class="help-icon">✦</div>
          <strong>Need a hand?</strong>
          <p>Visit our help center to learn more about Orbit.</p>
          <button @click="selectNav('Help center')">Get support <span>↗</span></button>
        </div>
        <div class="profile">
          <div class="profile-avatar">AL</div>
          <div class="profile-copy"><strong>Alex Lee</strong><small>alex@studio.co</small></div>
          <span class="profile-more">···</span>
        </div>
      </div>
    </aside>

    <main class="main-content">
      <header class="topbar">
        <button class="mobile-menu" aria-label="Open navigation" @click="mobileMenuOpen = !mobileMenuOpen">☰</button>
        <div class="breadcrumb">Workspace <span>/</span> <strong>{{ activeNav }}</strong></div>
        <div class="topbar-actions">
          <button class="icon-button" aria-label="Notifications" @click="notify('You are all caught up.')">♧<i></i></button>
          <div class="top-avatar">AL</div>
        </div>
      </header>

      <div class="page-wrap">
        <section class="page-heading">
          <div>
            <p class="eyebrow">MONDAY, NOVEMBER 11, 2026</p>
            <h1>Good morning, Alex <span>✦</span></h1>
            <p class="subheading">Here’s what’s happening with your store today.</p>
          </div>
          <button class="primary-button" @click="openCreate"><span>＋</span> Add product</button>
        </section>

        <section class="stats-grid" aria-label="Store summary">
          <article class="stat-card">
            <div class="stat-top"><span>Total products</span><span class="stat-icon lavender">◇</span></div>
            <div class="stat-value">{{ products.length.toLocaleString() }}</div>
            <div class="stat-foot"><span class="trend">↗ 12.8%</span> <span>vs. last month</span></div>
            <div class="sparkline spark-one"><i></i><i></i><i></i><i></i><i></i><i></i><i></i><i></i><i></i></div>
          </article>
          <article class="stat-card">
            <div class="stat-top"><span>Inventory value</span><span class="stat-icon peach">$</span></div>
            <div class="stat-value">${{ totalValue.toLocaleString() }}</div>
            <div class="stat-foot"><span class="trend">↗ 8.2%</span> <span>vs. last month</span></div>
            <div class="sparkline spark-two"><i></i><i></i><i></i><i></i><i></i><i></i><i></i><i></i><i></i></div>
          </article>
          <article class="stat-card">
            <div class="stat-top"><span>Active products</span><span class="stat-icon mint">✓</span></div>
            <div class="stat-value">{{ activeCount }}</div>
            <div class="stat-foot"><span class="trend">↗ 4.6%</span> <span>vs. last month</span></div>
            <div class="sparkline spark-three"><i></i><i></i><i></i><i></i><i></i><i></i><i></i><i></i><i></i></div>
          </article>
          <article class="stat-card">
            <div class="stat-top"><span>Low stock alerts</span><span class="stat-icon yellow">!</span></div>
            <div class="stat-value">{{ lowStockCount }}</div>
            <div class="stat-foot"><span class="warning-text">Needs attention</span> <span>in your inventory</span></div>
            <div class="sparkline spark-four"><i></i><i></i><i></i><i></i><i></i><i></i><i></i><i></i><i></i></div>
          </article>
        </section>

        <section class="content-card">
          <div class="table-heading">
            <div>
              <div class="title-line"><h2>Product inventory</h2><span class="soft-count">{{ products.length }} products</span></div>
              <p>Manage and keep track of your products.</p>
            </div>
            <button class="secondary-button export-button" @click="notify('Export is a demo action. You can add CSV export next.')">↓ <span>Export</span></button>
          </div>

          <div class="table-toolbar">
            <label class="search-box">
              <span>⌕</span>
              <input v-model="search" type="search" placeholder="Search products..." aria-label="Search products" />
              <kbd>⌘ K</kbd>
            </label>
            <div class="toolbar-right">
              <label class="filter-select"><span>☷</span><select v-model="statusFilter" aria-label="Filter by status"><option>All</option><option>Active</option><option>Low stock</option><option>Out of stock</option></select></label>
              <button class="secondary-button filter-button" @click="resetDemo">↺ <span>Reset</span></button>
            </div>
          </div>

          <div class="table-scroll">
            <table>
              <thead><tr><th class="check-col"><input type="checkbox" aria-label="Select all products" @change="notify('Bulk selection is a demo interaction.')" /></th><th>PRODUCT</th><th>PRODUCT ID</th><th>CATEGORY</th><th>PRICE</th><th>STOCK</th><th>STATUS</th><th> </th></tr></thead>
              <tbody>
                <tr v-for="product in filteredProducts" :key="product.id">
                  <td><input type="checkbox" :aria-label="`Select ${product.name}`" /></td>
                  <td><div class="product-cell"><div class="product-thumb" :class="`thumb-${product.id % 5}`">{{ product.name.slice(0, 1) }}</div><strong>{{ product.name }}</strong></div></td>
                  <td class="muted-cell">#{{ product.id }}</td>
                  <td><span class="category-pill">{{ product.category }}</span></td>
                  <td class="price-cell">${{ Number(product.price).toFixed(2) }}</td>
                  <td>{{ product.stock }} units</td>
                  <td><span class="status-pill" :class="product.status.toLowerCase().replaceAll(' ', '-')"><i></i>{{ product.status }}</span></td>
                  <td><div class="row-actions"><button aria-label="Edit product" @click="openEdit(product)">✎</button><button aria-label="Delete product" @click="deleteProduct(product)">···</button></div></td>
                </tr>
                <tr v-if="filteredProducts.length === 0"><td colspan="8" class="empty-state">No products found. Try another search or add a product.</td></tr>
              </tbody>
            </table>
          </div>
          <div class="table-footer"><span>Showing <strong>{{ filteredProducts.length }}</strong> of <strong>{{ products.length }}</strong> products</span><div class="pagination"><button disabled>← Previous</button><button class="current-page">1</button><button disabled>Next →</button></div></div>
        </section>
        <footer class="page-footer"><span>© 2026 Orbit Workspace</span><span>Made for better work <b>✦</b></span></footer>
      </div>
    </main>

    <div v-if="showModal" class="modal-backdrop" @click.self="showModal = false">
      <section class="modal" role="dialog" aria-modal="true" aria-labelledby="modal-title">
        <div class="modal-heading"><div><p class="eyebrow">{{ editingId ? 'UPDATE INVENTORY' : 'NEW INVENTORY ITEM' }}</p><h2 id="modal-title">{{ editingId ? 'Edit product' : 'Add a product' }}</h2></div><button class="close-button" aria-label="Close" @click="showModal = false">×</button></div>
        <form @submit.prevent="saveProduct">
          <label class="form-field">Product name<input v-model="form.name" required maxlength="80" placeholder="e.g. Wireless keyboard" /></label>
          <label class="form-field">Category<select v-model="form.category"><option>Electronics</option><option>Accessories</option><option>Home & Living</option><option>Apparel</option><option>Other</option></select></label>
          <div class="form-row"><label class="form-field">Price (USD)<input v-model.number="form.price" type="number" min="0" step="0.01" required /></label><label class="form-field">Stock quantity<input v-model.number="form.stock" type="number" min="0" step="1" required /></label></div>
          <div class="modal-actions"><button type="button" class="secondary-button" @click="showModal = false">Cancel</button><button type="submit" class="primary-button">{{ editingId ? 'Save changes' : 'Add product' }}</button></div>
        </form>
      </section>
    </div>

    <Transition name="toast"><div v-if="toast" class="toast-message">✓ {{ toast }}</div></Transition>
  </div>
</template>