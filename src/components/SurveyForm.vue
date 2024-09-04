<template>
  <q-page padding>
    <h4 class="text-center">Spiritual Gift Survey</h4>
    <q-form @submit="submitForm" @reset="resetForm" class="q-gutter-md">
      <!-- Navigation Buttons -->
      <div class="row justify-between q-mt-md">
        <q-btn
          @click="previousTab"
          :disable="activeTab === 0"
          label="Previous"
          color="grey-7"
        />

        <div>
          <q-btn
            :disable="activeTab != paginatedItems.length - 1"
            type="submit"
            label="Submit"
            color="secondary"
            class="q-mr-sm"
          />
          <q-btn
            @click="nextTab"
            :disable="activeTab === paginatedItems.length - 1"
            label="Next"
            color="primary"
          />
        </div>
      </div>

      <!-- Tab Panels -->
      <q-tab-panels v-model="activeTab" animated swipeable>
        <q-tab-panel
          v-for="(itemGroup, index) in paginatedItems"
          :key="index"
          :name="index"
        >
          <div v-for="item in itemGroup" :key="item.id" class="q-pa-xs">
            <q-card class="q-mb-md">
              <q-card-section horizontal class="row">
                <q-card-section class="col-11">
                  <div>{{ item.statement }}</div>
                  <q-slider
                    v-model="answers[item.id]"
                    :min="0"
                    :max="5"
                    :step="1"
                    snap
                    dense
                    color="primary"
                  />
                </q-card-section>
                <q-card-section class="col-1">
                  <p class="text-h6 no-margin">{{ answers[item.id] ?? 0 }}</p>
                </q-card-section>
              </q-card-section>
            </q-card>
          </div>
        </q-tab-panel>
      </q-tab-panels>
    </q-form>

    <q-dialog v-model="resultsDialog">
      <q-card style="min-width: 300px; max-width: 40vw">
        <q-card-section>
          <div class="text-h6">Survey Results</div>
        </q-card-section>
        <div>

        <q-list dense bordered>
          <q-item
            class="justify-between text-bold"
            v-for="category in sortedCategories"
            :key="category.id"
          >
            <q-item-section>{{ category.desc }}</q-item-section>

            <q-item-section class="text-right">{{ category.total }}</q-item-section>
          </q-item>
        </q-list>
        </div>

        <q-card-actions align="right">
          <q-btn flat label="Close" color="primary" text-color="black" v-close-popup />
        </q-card-actions>
      </q-card>
    </q-dialog>
  </q-page>
</template>

<script setup>
import { ref, computed } from 'vue'
import { items } from '../data/items' // Import items from the provided file
import { categories } from '../data/category' // Import categories from the provided file

const user = ref({
  name: '',
  email: ''
})
const answers = ref({})
const activeTab = ref(0) // Track the current tab

// Shuffle items and split into groups of 10
const shuffledItems = items.sort(() => Math.random() - 0.5) // Shuffle items
const paginatedItems = computed(() => {
  const chunkSize = 5
  const groups = []
  for (let i = 0; i < shuffledItems.length; i += chunkSize) {
    groups.push(shuffledItems.slice(i, i + chunkSize))
  }
  return groups
})

const resultsDialog = ref(false)

const submitForm = () => {
  const categoryTotals = calculateCategoryTotals()
  sortedCategories.value = sortCategoriesByTotal(categoryTotals)
  resultsDialog.value = true
}

const resetForm = () => {
  user.value = { name: '', email: '' }
  answers.value = {}
  activeTab.value = 0
}

const calculateCategoryTotals = () => {
  const totals = {}

  for (const categoryKey in categories) {
    const category = categories[categoryKey]
    let total = 0
    category.itemIds.forEach((itemId) => {
      if (answers.value[itemId]) {
        total += answers.value[itemId]
      }
    })
    totals[categoryKey] = {
      desc: category.desc,
      total
    }
  }

  return totals
}

const sortedCategories = ref([])
const sortCategoriesByTotal = (categoryTotals) => {
  return Object.values(categoryTotals).sort((a, b) => b.total - a.total)
}

// Navigate between tabs
const nextTab = () => {
  if (activeTab.value < paginatedItems.value.length - 1) {
    activeTab.value++
  }
}

const previousTab = () => {
  if (activeTab.value > 0) {
    activeTab.value--
  }
}
</script>

<style scoped>
.q-card {
  max-width: 500px;
  margin: 0 auto;
}
</style>
