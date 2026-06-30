<template>
  <div>
    <div class="page-header">
      <h1 class="page-title">직원 관리</h1>
      <button class="btn btn-primary" @click="router.push('/admin/employees/new')">
        + 직원 추가
      </button>
    </div>

    <!-- 검색 & 필터 -->
    <div class="toolbar">
      <SearchBox
        placeholder="직원 ID 또는 이름으로 검색"
        @search="onSearch"
      />
      <div class="status-tabs">
        <button
          v-for="tab in STATUS_TABS"
          :key="tab.value"
          class="tab-btn"
          :class="{ active: statusFilter === tab.value }"
          @click="onStatusFilter(tab.value)"
        >
          {{ tab.label }}
          <span class="tab-count">{{ tabCount(tab.value) }}</span>
        </button>
      </div>
    </div>

    <!-- 로딩 -->
    <div v-if="isLoading" class="loading-spinner">
      <div class="spinner"></div>
      <span>불러오는 중...</span>
    </div>

    <!-- 에러 -->
    <div v-else-if="errorMessage" class="alert alert-error">{{ errorMessage }}</div>

    <!-- 테이블 -->
    <div v-else class="card" style="padding:0;overflow:hidden;">
      <div class="table-meta">
        <span class="result-count">
          {{ filteredEmployees.length }}명
          <span v-if="searchQuery" style="color:#9CA3AF;font-weight:400;">
            (전체 {{ employees.length }}명 중)
          </span>
        </span>
      </div>

      <table>
        <thead>
          <tr>
            <th>직원 ID</th>
            <th>이름</th>
            <th>이메일</th>
            <th>부서</th>
            <th>직급</th>
            <th>상태</th>
            <th>입사일</th>
            <th>액션</th>
          </tr>
        </thead>
        <tbody>
          <tr
            v-for="emp in pagedEmployees"
            :key="emp.employeeId"
            :style="emp.status === 'TERMINATED' ? 'opacity:0.55' : ''"
          >
            <td><code style="font-size:12px;">{{ emp.employeeId }}</code></td>
            <td style="font-weight:500;">{{ emp.lastName }} {{ emp.firstName }}</td>
            <td style="color:#6B7280;">{{ emp.email }}</td>
            <td>{{ emp.department || '-' }}</td>
            <td>{{ emp.position || '-' }}</td>
            <td>
              <span :class="['badge', emp.status === 'ACTIVE' ? 'badge-success' : 'badge-danger']">
                {{ emp.status === 'ACTIVE' ? '재직 중' : '퇴사' }}
              </span>
            </td>
            <td style="color:#6B7280;font-size:13px;">{{ emp.hireDate }}</td>
            <td>
              <button
                class="btn btn-outline"
                style="padding:4px 12px;font-size:13px;"
                @click="router.push('/admin/employees/' + emp.employeeId)"
              >
                상세보기
              </button>
            </td>
          </tr>

          <!-- 결과 없음 -->
          <tr v-if="filteredEmployees.length === 0">
            <td colspan="8" class="empty-state">
              <svg width="36" height="36" viewBox="0 0 24 24" fill="none"
                stroke="#D1D5DB" stroke-width="1.5" style="margin-bottom:8px;">
                <circle cx="11" cy="11" r="8"/>
                <line x1="21" y1="21" x2="16.65" y2="16.65"/>
              </svg>
              <div v-if="searchQuery">
                '<strong>{{ searchQuery }}</strong>' 에 해당하는 직원이 없습니다.
              </div>
              <div v-else>등록된 직원이 없습니다.</div>
            </td>
          </tr>
        </tbody>
      </table>

      <!-- 페이지네이션 -->
      <div v-if="totalPages > 1" style="padding:16px 20px;">
        <Pagination
          :current="currentPage"
          :total-pages="totalPages"
          @change="onPageChange"
        />
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, computed, watch } from 'vue'
import { useRouter } from 'vue-router'
import { getEmployees } from '../../api/admin.js'
import SearchBox  from '../../components/SearchBox.vue'
import Pagination from '../../components/Pagination.vue'

const PAGE_SIZE = 10

const STATUS_TABS = [
  { label: '전체',    value: 'ALL' },
  { label: '재직 중', value: 'ACTIVE' },
  { label: '퇴사',   value: 'TERMINATED' },
]

const router       = useRouter()
const employees    = ref([])
const isLoading    = ref(false)
const errorMessage = ref('')
const searchQuery  = ref('')
const statusFilter = ref('ALL')
const currentPage  = ref(1)

// 검색·필터 변경 시 첫 페이지로 리셋
watch([searchQuery, statusFilter], () => { currentPage.value = 1 })

// 필터링 (ID + 이름 검색, 상태 필터)
const filteredEmployees = computed(() => {
  const q = searchQuery.value.toLowerCase()
  return employees.value.filter(emp => {
    const matchStatus =
      statusFilter.value === 'ALL' || emp.status === statusFilter.value
    const fullName = `${emp.lastName}${emp.firstName}`.toLowerCase()
    const fullNameSpace = `${emp.lastName} ${emp.firstName}`.toLowerCase()
    const matchQuery =
      !q ||
      emp.employeeId.toLowerCase().includes(q) ||
      fullName.includes(q) ||
      fullNameSpace.includes(q)
    return matchStatus && matchQuery
  })
})

// 페이지네이션
const totalPages = computed(() =>
  Math.ceil(filteredEmployees.value.length / PAGE_SIZE)
)
const pagedEmployees = computed(() => {
  const start = (currentPage.value - 1) * PAGE_SIZE
  return filteredEmployees.value.slice(start, start + PAGE_SIZE)
})

// 탭별 카운트
function tabCount(status) {
  if (status === 'ALL') return employees.value.length
  return employees.value.filter(e => e.status === status).length
}

function onSearch(q)        { searchQuery.value  = q }
function onStatusFilter(s)  { statusFilter.value = s }
function onPageChange(page) { currentPage.value  = page }

async function loadEmployees() {
  isLoading.value    = true
  errorMessage.value = ''
  try {
    const res = await getEmployees()
    employees.value = res.data.data
  } catch (err) {
    // Axios interceptor가 error.response.data.error를 reject하므로
    // err.message = 서버 ErrorCode 메시지
    errorMessage.value = err?.message || '직원 목록을 불러오는 데 실패했습니다.'
  } finally {
    isLoading.value = false
  }
}

loadEmployees()
</script>

<style scoped>
.toolbar {
  display: flex;
  align-items: center;
  gap: 16px;
  margin-bottom: 16px;
  flex-wrap: wrap;
}

.status-tabs {
  display: flex;
  gap: 4px;
  background: #F3F4F6;
  padding: 4px;
  border-radius: 8px;
}

.tab-btn {
  display: flex;
  align-items: center;
  gap: 6px;
  padding: 6px 14px;
  border: none;
  background: transparent;
  border-radius: 6px;
  font-size: 13px;
  font-weight: 500;
  color: #6B7280;
  cursor: pointer;
  transition: all 0.15s;
}

.tab-btn:hover { color: #374151; }

.tab-btn.active {
  background: #fff;
  color: #111827;
  box-shadow: 0 1px 3px rgba(0,0,0,0.08);
}

.tab-count {
  background: #E5E7EB;
  color: #6B7280;
  font-size: 11px;
  font-weight: 600;
  padding: 1px 6px;
  border-radius: 10px;
  min-width: 20px;
  text-align: center;
}

.tab-btn.active .tab-count {
  background: #EEF2FF;
  color: #6366F1;
}

.table-meta {
  padding: 12px 20px 8px;
  border-bottom: 1px solid #F3F4F6;
}

.result-count {
  font-size: 13px;
  font-weight: 600;
  color: #374151;
}

.empty-state {
  text-align: center;
  color: #9CA3AF;
  padding: 48px 20px;
  font-size: 14px;
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 4px;
}
</style>
