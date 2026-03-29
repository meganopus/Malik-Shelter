<script setup lang="ts">
import { ref, onMounted } from 'vue'
import FilterSidebar from '@/components/FilterSidebar.vue'
import PetCard from '@/components/PetCard.vue'
import LoadingSpinner from '@/components/LoadingSpinner.vue'

interface Animal {
  id: string
  name: string
  species: string
  breed: string
  gender: string
  age_months: number
  status: string
  size?: string
  tags?: string[]
  photos: { photo_url: string; is_main: boolean }[]
}

interface SearchResult {
  total: number
  page: number
  limit: number
  total_pages: number
  items: Animal[]
}

const API_BASE = 'http://localhost:3000'

const animals = ref<Animal[]>([])
const loading = ref(false)
const loadingMore = ref(false)
const error = ref<string | null>(null)
const pagination = ref({ page: 1, total_pages: 1, total: 0 })
const viewMode = ref<'grid' | 'list'>('grid')

const currentFilters = ref({
  species: '',
  gender: '',
  status: ''
})

async function fetchAnimals(append = false) {
  if (append) loadingMore.value = true
  else loading.value = true
  error.value = null

  try {
    const params = new URLSearchParams()
    params.set('page', String(pagination.value.page))
    params.set('limit', '6')

    if (currentFilters.value.species) params.set('species', currentFilters.value.species)
    if (currentFilters.value.gender) params.set('gender', currentFilters.value.gender)
    if (currentFilters.value.status) params.set('status', currentFilters.value.status)

    const res = await fetch(`${API_BASE}/animals?${params.toString()}`)
    if (!res.ok) throw new Error('Failed to fetch animals')

    const data: SearchResult = await res.json()

    if (append) {
      animals.value = [...animals.value, ...data.items]
    } else {
      animals.value = data.items
    }

    pagination.value = {
      page: data.page,
      total_pages: data.total_pages,
      total: data.total
    }
  } catch (e: any) {
    error.value = e.message
  } finally {
    loading.value = false
    loadingMore.value = false
  }
}

async function loadMore() {
  if (pagination.value.page >= pagination.value.total_pages) return
  pagination.value.page++
  await fetchAnimals(true)
}

function onFilterChange(filters: typeof currentFilters.value) {
  currentFilters.value = filters
  pagination.value.page = 1
  fetchAnimals()
}

onMounted(() => {
  fetchAnimals()
})
</script>

<template>
  <div class="min-h-screen bg-[#f5f0ea] flex flex-col">
    <!-- Page Body -->
    <div class="flex gap-6 px-8 py-8 flex-1">
      <!-- Sidebar -->
      <FilterSidebar @filter-change="onFilterChange" />

      <!-- Main Content -->
      <main class="flex-1 min-w-0">
        <!-- Section Header -->
        <div class="flex items-start justify-between mb-6">
          <div>
            <h1 class="text-3xl font-bold text-[#2d2a1e] leading-tight">Available for Adoption</h1>
            <p class="text-sm text-gray-500 mt-1 font-[family-name:var(--font-form)]">
              Meet the residents of Malik Shelter looking for a forever home.
            </p>
          </div>

          <!-- View Toggle -->
          <div class="flex items-center gap-1 bg-white rounded-lg border border-gray-200 p-1">
            <button
              @click="viewMode = 'grid'"
              class="p-1.5 rounded transition-colors"
              :class="viewMode === 'grid' ? 'bg-gray-100 text-gray-700' : 'text-gray-400 hover:text-gray-600'"
            >
              <svg class="w-4 h-4" fill="currentColor" viewBox="0 0 16 16">
                <rect x="1" y="1" width="6" height="6" rx="1"/>
                <rect x="9" y="1" width="6" height="6" rx="1"/>
                <rect x="1" y="9" width="6" height="6" rx="1"/>
                <rect x="9" y="9" width="6" height="6" rx="1"/>
              </svg>
            </button>
            <button
              @click="viewMode = 'list'"
              class="p-1.5 rounded transition-colors"
              :class="viewMode === 'list' ? 'bg-gray-100 text-gray-700' : 'text-gray-400 hover:text-gray-600'"
            >
              <svg class="w-4 h-4" fill="currentColor" viewBox="0 0 16 16">
                <rect x="1" y="2" width="14" height="2" rx="1"/>
                <rect x="1" y="7" width="14" height="2" rx="1"/>
                <rect x="1" y="12" width="14" height="2" rx="1"/>
              </svg>
            </button>
          </div>
        </div>

        <!-- Loading State -->
        <div v-if="loading" class="flex justify-center py-12">
          <LoadingSpinner />
        </div>

        <!-- Error State -->
        <div v-else-if="error" class="text-center py-12">
          <p class="text-red-500 font-[family-name:var(--font-form)]">{{ error }}</p>
          <button @click="fetchAnimals()" class="btn-primary mt-4">Try Again</button>
        </div>

        <!-- Empty State -->
        <div v-else-if="animals.length === 0" class="text-center py-12">
          <p class="text-gray-400 font-[family-name:var(--font-form)]">No pets found matching your filters.</p>
        </div>

        <!-- Grid -->
        <template v-else>
          <div
            :class="viewMode === 'grid'
              ? 'grid grid-cols-3 gap-5'
              : 'flex flex-col gap-4'"
          >
            <PetCard
              v-for="animal in animals"
              :key="animal.id"
              :animal="animal"
            />
          </div>

          <!-- Load More -->
          <div v-if="pagination.page < pagination.total_pages" class="flex justify-center mt-10">
            <button
              @click="loadMore"
              :disabled="loadingMore"
              class="flex items-center gap-2 px-8 py-3 bg-[#ede8e0] hover:bg-[#e0d9ce] text-[#6b5a3e] font-semibold rounded-full text-sm font-[family-name:var(--font-form)] transition-colors disabled:opacity-60"
            >
              <span>{{ loadingMore ? 'Loading...' : 'Load more friends' }}</span>
              <svg v-if="!loadingMore" class="w-4 h-4" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M19 9l-7 7-7-7"/>
              </svg>
            </button>
          </div>
        </template>
      </main>
    </div>

    <!-- Footer -->
    <footer class="bg-[#2d2a1e] text-white mt-8">
      <div class="px-8 py-8 flex items-center justify-between">
        <!-- Brand -->
        <div>
          <div class="text-lg font-bold">Malik Shelter</div>
          <p class="text-xs text-gray-400 mt-1 font-[family-name:var(--font-form)]">
            © 2024 Malik Shelter. Every pet deserves a home.
          </p>
        </div>

        <!-- Links -->
        <div class="flex items-center gap-6 text-xs text-gray-400 font-[family-name:var(--font-form)]">
          <a href="#" class="hover:text-white transition-colors">Privacy Policy</a>
          <a href="#" class="hover:text-white transition-colors">Terms of Service</a>
          <a href="#" class="hover:text-white transition-colors">Contact Us</a>
          <a href="#" class="hover:text-white transition-colors">Volunteer</a>
        </div>

        <!-- Social -->
        <div class="flex items-center gap-3">
          <button class="w-8 h-8 rounded-full border border-gray-600 flex items-center justify-center hover:border-gray-400 transition-colors">
            <svg class="w-3.5 h-3.5 text-gray-400" fill="currentColor" viewBox="0 0 24 24">
              <path d="M18 2h-3a5 5 0 00-5 5v3H7v4h3v8h4v-8h3l1-4h-4V7a1 1 0 011-1h3z"/>
            </svg>
          </button>
          <button class="w-8 h-8 rounded-full border border-gray-600 flex items-center justify-center hover:border-gray-400 transition-colors">
            <svg class="w-3.5 h-3.5 text-gray-400" fill="currentColor" viewBox="0 0 24 24">
              <path d="M4 4h16a2 2 0 012 2v12a2 2 0 01-2 2H4a2 2 0 01-2-2V6a2 2 0 012-2zm0 2v.01L12 13 20 6.01V6H4zm0 12h16V8l-8 5.99L4 8v10z"/>
            </svg>
          </button>
        </div>
      </div>
    </footer>
  </div>
</template>
