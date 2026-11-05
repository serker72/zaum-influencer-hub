<script setup>
import { computed, ref } from 'vue'
import influencersData from './data.json'

// ── State ──────────────────────────────────────────────────────────
const searchQuery = ref('')
const statusFilter = ref('Все')
const platformFilter = ref('Все')

const statuses = ['Все', 'Согласование', 'Запланировано', 'Опубликовано', 'Результаты получены']
const platforms = ['Все', 'Telegram', 'VK', 'YouTube']

// ── Helpers ────────────────────────────────────────────────────────
function formatCurrency(value) {
  return new Intl.NumberFormat('ru-RU', {
    style: 'currency',
    currency: 'RUB',
    minimumFractionDigits: 0,
    maximumFractionDigits: 0,
  }).format(value)
}

function formatNumber(value) {
  return new Intl.NumberFormat('ru-RU').format(value)
}

const normalizedQuery = computed(() => searchQuery.value.toLowerCase().trim())

function matchesSearch(item, query) {
  return (
    !query ||
    item.bloggerName.toLowerCase().includes(query) ||
    item.article.toLowerCase().includes(query)
  )
}

function matchesStatus(item, status) {
  return status === 'Все' || item.status === status
}

function matchesPlatform(item, platform) {
  return platform === 'Все' || item.platform === platform
}

// ── KPI (computed) ─────────────────────────────────────────────────
const kpi = computed(() => {
  const totalIntegrations = influencersData.length
  const totalBudget = influencersData.reduce((sum, i) => sum + i.placementCost, 0)
  const totalStoriesReach = influencersData.reduce((sum, i) => sum + i.storiesReach, 0)
  const avgCPM =
    totalIntegrations > 0
      ? influencersData.reduce((sum, i) => sum + i.cpm, 0) / totalIntegrations
      : 0

  return {
    totalIntegrations,
    totalBudget,
    totalStoriesReach,
    avgCPM,
  }
})

// ── Filtered list (computed) ───────────────────────────────────────
const filteredData = computed(() => {
  const query = normalizedQuery.value
  return influencersData.filter(
    (item) => matchesStatus(item, statusFilter.value) && matchesPlatform(item, platformFilter.value) && matchesSearch(item, query)
  )
})

// ── Facet counters (computed) ──────────────────────────────────────
// Счётчики считаются по данным, отфильтрованным по ОСТАЛЬНЫМ условиям,
// поэтому цифры отражают реальный результат клика по кнопке.
const baseForStatusCounts = computed(() => {
  const query = normalizedQuery.value
  return influencersData.filter(
    (item) => matchesPlatform(item, platformFilter.value) && matchesSearch(item, query)
  )
})

const baseForPlatformCounts = computed(() => {
  const query = normalizedQuery.value
  return influencersData.filter(
    (item) => matchesStatus(item, statusFilter.value) && matchesSearch(item, query)
  )
})

function countBy(list, key) {
  const counts = {}
  list.forEach((item) => {
    counts[item[key]] = (counts[item[key]] || 0) + 1
  })
  return counts
}

const statusCounts = computed(() => {
  const list = baseForStatusCounts.value
  const byStatus = countBy(list, 'status')
  const counts = { 'Все': list.length }
  statuses.forEach((s) => {
    if (s !== 'Все') counts[s] = byStatus[s] || 0
  })
  return counts
})

const platformCounts = computed(() => {
  const list = baseForPlatformCounts.value
  const byPlatform = countBy(list, 'platform')
  const counts = { 'Все': list.length }
  platforms.forEach((p) => {
    if (p !== 'Все') counts[p] = byPlatform[p] || 0
  })
  return counts
})

// ── Status helpers ──────────────────────────────────────────────────
function statusBadgeClass(status) {
  const map = {
    'Согласование': 'bg-amber-500/20 text-amber-400 ring-amber-500/30',
    'Запланировано': 'bg-sky-500/20 text-sky-400 ring-sky-500/30',
    'Опубликовано': 'bg-emerald-500/20 text-emerald-400 ring-emerald-500/30',
    'Результаты получены': 'bg-emerald-500/20 text-emerald-400 ring-emerald-500/30',
  }
  return map[status] || 'bg-neutral-700 text-neutral-300 ring-neutral-600'
}

function platformBadgeClass(platform) {
  const map = {
    Telegram: 'bg-violet-500/20 text-violet-400',
    VK: 'bg-blue-500/20 text-blue-400',
    YouTube: 'bg-red-500/20 text-red-400',
  }
  return map[platform] || 'bg-neutral-700 text-neutral-300'
}

const platformIcons = {
  Telegram: '✈',
  VK: 'V',
  YouTube: '▶',
}
</script>

<template>
  <div class="min-h-screen bg-neutral-950 text-neutral-200">
    <!-- ════════════ HEADER ════════════ -->
    <header class="border-b border-neutral-800 bg-neutral-900/80 backdrop-blur">
      <div class="mx-auto flex max-w-7xl items-center justify-between px-4 py-5 sm:px-6 lg:px-8">
        <div class="flex items-center gap-3">
          <div
            class="flex size-10 items-center justify-center rounded-xl bg-gradient-to-br from-emerald-400 to-teal-500 text-sm font-bold text-neutral-950"
          >
            Z
          </div>
          <div>
            <h1 class="text-lg font-semibold tracking-tight text-white sm:text-xl">
              ZAUM Influencer Hub
            </h1>
            <p class="text-xs text-neutral-500">Управление блогерами</p>
          </div>
        </div>
        <div class="text-right">
          <p class="text-sm text-neutral-400">Добро пожаловать 👋</p>
          <p class="text-xs text-neutral-600">{{ new Date().toLocaleDateString('ru-RU', { day: 'numeric', month: 'long', year: 'numeric' }) }}</p>
        </div>
      </div>
    </header>

    <main class="mx-auto max-w-7xl px-4 py-6 sm:px-6 lg:px-8">
      <!-- ════════════ KPI BLOCK ════════════ -->
      <section class="mb-6 grid grid-cols-2 gap-3 sm:grid-cols-4 sm:gap-4">
        <!-- Всего интеграций -->
        <div
          class="rounded-2xl border border-neutral-800 bg-neutral-900 p-4 transition hover:border-neutral-700 sm:p-5"
        >
          <p class="text-xs font-medium uppercase tracking-wider text-neutral-500">
            Всего интеграций
          </p>
          <p class="mt-2 text-2xl font-bold text-white sm:text-3xl">{{ kpi.totalIntegrations }}</p>
        </div>

        <!-- Общий бюджет -->
        <div
          class="rounded-2xl border border-neutral-800 bg-neutral-900 p-4 transition hover:border-neutral-700 sm:p-5"
        >
          <p class="text-xs font-medium uppercase tracking-wider text-neutral-500">
            Общий бюджет
          </p>
          <p class="mt-2 text-2xl font-bold text-emerald-400 sm:text-3xl">
            {{ formatCurrency(kpi.totalBudget) }}
          </p>
        </div>

        <!-- Суммарный охват сторис -->
        <div
          class="rounded-2xl border border-neutral-800 bg-neutral-900 p-4 transition hover:border-neutral-700 sm:p-5"
        >
          <p class="text-xs font-medium uppercase tracking-wider text-neutral-500">
            Охват сторис
          </p>
          <p class="mt-2 text-2xl font-bold text-sky-400 sm:text-3xl">
            {{ formatNumber(kpi.totalStoriesReach) }}
          </p>
        </div>

        <!-- Средний CPM -->
        <div
          class="rounded-2xl border border-neutral-800 bg-neutral-900 p-4 transition hover:border-neutral-700 sm:p-5"
        >
          <p class="text-xs font-medium uppercase tracking-wider text-neutral-500">
            Средний CPM
          </p>
          <p class="mt-2 text-2xl font-bold text-amber-400 sm:text-3xl">
            {{ kpi.avgCPM.toFixed(1) }}
          </p>
        </div>
      </section>

      <!-- ════════════ CONTROL PANEL ════════════ -->
      <section class="mb-5 space-y-3">
        <!-- Row 1: Status filters + counters -->
        <div class="flex flex-col gap-3 sm:flex-row sm:items-center sm:gap-4">
          <div class="flex flex-wrap gap-2">
            <button
              v-for="s in statuses"
              :key="s"
              @click="statusFilter = s"
              class="inline-flex items-center gap-1.5 rounded-lg px-3 py-1.5 text-xs font-medium transition"
              :class="
                statusFilter === s
                  ? 'bg-neutral-200 text-neutral-900'
                  : statusCounts[s] === 0
                    ? 'bg-neutral-800/60 text-neutral-600 hover:bg-neutral-800 hover:text-neutral-500'
                    : 'bg-neutral-800 text-neutral-400 hover:bg-neutral-700 hover:text-neutral-300'
              "
            >
              {{ s }}
              <span
                class="rounded-full px-1.5 py-0.5 text-[10px] font-semibold leading-none"
                :class="
                  statusFilter === s
                    ? 'bg-neutral-900/15 text-neutral-700'
                    : statusCounts[s] === 0
                      ? 'bg-neutral-800 text-neutral-700'
                      : 'bg-neutral-700 text-neutral-400'
                "
              >
                {{ statusCounts[s] }}
              </span>
            </button>
          </div>

          <!-- Search -->
          <div class="relative flex-1 sm:max-w-xs">
            <svg
              class="pointer-events-none absolute left-3 top-1/2 size-4 -translate-y-1/2 text-neutral-500"
              fill="none"
              stroke="currentColor"
              stroke-width="2"
              viewBox="0 0 24 24"
            >
              <path
                stroke-linecap="round"
                stroke-linejoin="round"
                d="M21 21l-5.197-5.197m0 0A7.5 7.5 0 105.196 5.196a7.5 7.5 0 0010.607 10.607z"
              />
            </svg>
            <input
              v-model="searchQuery"
              type="text"
              placeholder="Поиск по имени или артикулу…"
              class="w-full rounded-lg border border-neutral-800 bg-neutral-900 py-2 pl-9 pr-4 text-sm text-neutral-200 placeholder-neutral-600 outline-none ring-1 ring-transparent transition focus:border-neutral-700 focus:ring-neutral-700"
            />
          </div>
        </div>

        <!-- Row 2: Platform filters + counters -->
        <div class="flex flex-wrap items-center gap-2">
          <span class="text-[10px] font-medium uppercase tracking-wider text-neutral-600">Площадка</span>
          <button
            v-for="p in platforms"
            :key="p"
            @click="platformFilter = p"
            class="inline-flex items-center gap-1.5 rounded-lg px-2.5 py-1 text-xs font-medium transition"
            :class="
              platformFilter === p
                ? 'bg-neutral-200 text-neutral-900'
                : platformCounts[p] === 0
                  ? 'bg-neutral-800/60 text-neutral-600 hover:bg-neutral-800 hover:text-neutral-500'
                  : 'bg-neutral-800 text-neutral-400 hover:bg-neutral-700 hover:text-neutral-300'
            "
          >
            <span v-if="p !== 'Все' && platformIcons[p]" class="text-[10px]">{{ platformIcons[p] }}</span>
            {{ p }}
            <span
              class="rounded-full px-1.5 py-0.5 text-[10px] font-semibold leading-none"
              :class="
                platformFilter === p
                  ? 'bg-neutral-900/15 text-neutral-700'
                  : platformCounts[p] === 0
                    ? 'bg-neutral-800 text-neutral-700'
                    : 'bg-neutral-700 text-neutral-400'
              "
            >
              {{ platformCounts[p] }}
            </span>
          </button>
        </div>
      </section>

      <!-- ════════════ BLOGGERS GRID ════════════ -->
      <section
        v-if="filteredData.length"
        class="grid grid-cols-1 gap-4 sm:grid-cols-2 lg:grid-cols-3"
      >
        <article
          v-for="item in filteredData"
          :key="item.id"
          class="group flex flex-col justify-between rounded-2xl border border-neutral-800 bg-neutral-900 p-5 transition-all hover:border-neutral-700 hover:shadow-lg hover:shadow-neutral-900/60"
        >
          <!-- Top row: name + link -->
          <div>
            <div class="mb-3 flex items-start justify-between gap-2">
              <div class="min-w-0">
                <h3 class="truncate text-base font-semibold text-white sm:text-lg">
                  {{ item.bloggerName }}
                </h3>
                <p class="text-xs text-neutral-500">{{ item.article }}</p>
              </div>
              <a
                :href="item.adLink"
                target="_blank"
                rel="noopener noreferrer"
                class="shrink-0 rounded-lg p-1.5 text-neutral-500 opacity-0 transition-opacity group-hover:opacity-100 hover:bg-neutral-800 hover:text-emerald-400"
                title="Открыть рекламу"
              >
                <svg class="size-5" fill="none" stroke="currentColor" stroke-width="2" viewBox="0 0 24 24">
                  <path stroke-linecap="round" stroke-linejoin="round" d="M13.5 6H5.25A2.25 2.25 0 003 8.25v10.5A2.25 2.25 0 005.25 21h10.5A2.25 2.25 0 0018 18.75V10.5m-10.5 6L21 3m0 0h-5.25M21 3v5.25" />
                </svg>
              </a>
            </div>

            <!-- Badges -->
            <div class="mb-4 flex flex-wrap gap-2">
              <span
                class="inline-flex items-center gap-1.5 rounded-full px-2.5 py-0.5 text-xs font-medium ring-1 ring-inset"
                :class="statusBadgeClass(item.status)"
              >
                <span class="size-1.5 rounded-full bg-current opacity-70"></span>
                {{ item.status }}
              </span>
              <span
                class="inline-flex items-center gap-1 rounded-full px-2.5 py-0.5 text-xs font-medium"
                :class="platformBadgeClass(item.platform)"
              >
                <span class="text-[10px]">{{ platformIcons[item.platform] }}</span>
                {{ item.platform }}
              </span>
            </div>

            <!-- Metrics grid -->
            <div class="grid grid-cols-2 gap-x-4 gap-y-2 rounded-xl bg-neutral-950/50 p-3">
              <div>
                <p class="text-[11px] uppercase tracking-wider text-neutral-500">Бюджет</p>
                <p class="text-sm font-semibold text-white sm:text-base">
                  {{ formatCurrency(item.placementCost) }}
                </p>
              </div>
              <div>
                <p class="text-[11px] uppercase tracking-wider text-neutral-500">CPM</p>
                <p class="text-sm font-semibold text-white sm:text-base">{{ item.cpm.toFixed(1) }}</p>
              </div>
              <div>
                <p class="text-[11px] uppercase tracking-wider text-neutral-500">Охват сторис</p>
                <p class="text-sm font-semibold text-sky-400 sm:text-base">
                  {{ formatNumber(item.storiesReach) }}
                </p>
              </div>
              <div>
                <p class="text-[11px] uppercase tracking-wider text-neutral-500">Reels / просм.</p>
                <p class="text-sm font-semibold text-violet-400 sm:text-base">
                  {{ formatNumber(item.reelsViews) }}
                </p>
              </div>
            </div>
          </div>

          <!-- Footer: ad date + format -->
          <div class="mt-4 flex items-center justify-between border-t border-neutral-800 pt-3 text-xs text-neutral-500">
            <span>
              Дата: {{ new Date(item.adDate).toLocaleDateString('ru-RU', { day: 'numeric', month: 'short' }) }}
            </span>
            <span class="text-neutral-600">{{ item.adFormat }}</span>
          </div>
        </article>
      </section>

      <!-- Empty state -->
      <section v-else class="flex flex-col items-center justify-center py-20 text-center">
        <svg class="mb-4 size-12 text-neutral-700" fill="none" stroke="currentColor" stroke-width="1.5" viewBox="0 0 24 24">
          <path stroke-linecap="round" stroke-linejoin="round" d="M15.182 16.318A4.486 4.486 0 0012.016 15a4.486 4.486 0 00-3.198 1.318M21 12a9 9 0 11-18 0 9 9 0 0118 0zM9.75 9.75c0 .414-.168.75-.375.75S9 10.164 9 9.75 9.168 9 9.375 9s.375.336.375.75zm-.375 0h.008v.015h-.008V9.75zm5.625 0c0 .414-.168.75-.375.75s-.375-.336-.375-.75.168-.75.375-.75.375.336.375.75zm-.375 0h.008v.015h-.008V9.75z" />
        </svg>
        <p class="text-lg font-medium text-neutral-400">Ничего не найдено</p>
        <p class="mt-1 text-sm text-neutral-600">Попробуйте изменить параметры фильтрации</p>
      </section>
    </main>
  </div>
</template>
