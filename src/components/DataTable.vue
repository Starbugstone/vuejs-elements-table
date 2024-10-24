<script setup lang="ts">
import { ref, computed, onMounted } from 'vue'
import { ElMessage } from 'element-plus'
import axios from 'axios'

interface DataItem {
  [key: string]: any
}

const tableData = ref<DataItem[]>([])
const loading = ref(true)
const searchQuery = ref('')
const currentPage = ref(1)
const pageSize = ref(10)
const sortBy = ref<string | null>(null)
const sortOrder = ref<'ascending' | 'descending' | null>(null)
const filters = ref<Record<string, string>>({})

const API_URL = 'https://jsonplaceholder.typicode.com/posts'

const fetchData = async () => {
  try {
    loading.value = true
    const response = await axios.get(API_URL)
    tableData.value = response.data
  } catch (error) {
    ElMessage.error('Failed to fetch data')
    console.error(error)
  } finally {
    loading.value = false
  }
}

const filteredData = computed(() => {
  let result = [...tableData.value]

  if (searchQuery.value) {
    const query = searchQuery.value.toLowerCase()
    result = result.filter(item => 
      Object.values(item).slice(0, 3).some(value => 
        String(value).toLowerCase().includes(query)
      )
    )
  }

  Object.entries(filters.value).forEach(([key, value]) => {
    if (value) {
      result = result.filter(item => String(item[key]).toLowerCase().includes(String(value).toLowerCase()))
    }
  })

  if (sortBy.value && sortOrder.value) {
    result.sort((a, b) => {
      const valA = a[sortBy.value!]
      const valB = b[sortBy.value!]
      if (valA < valB) return sortOrder.value === 'ascending' ? -1 : 1
      if (valA > valB) return sortOrder.value === 'ascending' ? 1 : -1
      return 0
    })
  }

  return result
})

const paginatedData = computed(() => {
  const start = (currentPage.value - 1) * pageSize.value
  const end = start + pageSize.value
  return filteredData.value.slice(start, end)
})

const totalItems = computed(() => filteredData.value.length)

const handleSort = ({ prop, order }: { prop: string, order: string }) => {
  sortBy.value = prop
  sortOrder.value = order
}

const handleFilter = (value: string, column: string) => {
  filters.value = { ...filters.value, [column]: value }
}

onMounted(() => {
  fetchData()
})
</script>

<template>
  <div class="data-table">
    <div class="table-toolbar">
      <el-input
        v-model="searchQuery"
        placeholder="Search in first 3 columns..."
        class="search-input"
        clearable
      >
        <template #prefix>
          <el-icon><search /></el-icon>
        </template>
      </el-input>
    </div>

    <el-table
      v-loading="loading"
      :data="paginatedData"
      style="width: 100%"
      @sort-change="handleSort"
      border
    >
      <el-table-column
        prop="id"
        label="ID"
        sortable
        width="80"
        fixed
      />
      <el-table-column
        prop="title"
        label="Title"
        sortable="custom"
        min-width="300"
      >
        <template #header>
          <div class="column-header">
            <div class="header-content">
              <span>Title</span>
              <el-input
                v-model="filters.title"
                placeholder="Filter title"
                size="small"
                clearable
                @input="handleFilter($event, 'title')"
              />
            </div>
          </div>
        </template>
      </el-table-column>
      <el-table-column
        prop="body"
        label="Body"
        sortable="custom"
        min-width="400"
      >
        <template #header>
          <div class="column-header">
            <div class="header-content">
              <span>Body</span>
              <el-input
                v-model="filters.body"
                placeholder="Filter body"
                size="small"
                clearable
                @input="handleFilter($event, 'body')"
              />
            </div>
          </div>
        </template>
      </el-table-column>
    </el-table>

    <div class="pagination">
      <el-pagination
        v-model:current-page="currentPage"
        v-model:page-size="pageSize"
        :total="totalItems"
        :page-sizes="[10, 20, 50, 100]"
        layout="total, sizes, prev, pager, next, , jumper"
        background
      />
    </div>
  </div>
</template>

<style scoped>
.data-table {
  padding: 20px;
  display: flex;
  flex-direction: column;
  gap: 20px;
}

.table-toolbar {
  display: flex;
  gap: 16px;
  align-items: center;
}

.search-input {
  max-width: 300px;
}

.column-header {
  width: 100%;
}

.header-content {
  display: flex;
  flex-direction: column;
  gap: 8px;
  position: relative;
  padding-right: 24px; /* Make room for the sort icon */
}

.header-content span {
  font-weight: bold;
}

/* Custom styling for the sort icon positioning */
:deep(.el-table__column-filter-trigger) {
  position: absolute;
  right: 0;
  top: 50%;
  transform: translateY(-50%);
}

:deep(.caret-wrapper) {
  position: absolute !important;
  right: 0;
  top: 50%;
}

.pagination {
  display: flex;
  justify-content: flex-end;
  margin-top: 16px;
}

.el-table__header-wrapper {
  background-color: #f5f7fa;
}

.el-table th {
  background-color: #f5f7fa;
}

/* Ensure sort icons stay on the right */
.el-table .sort-caret {
  position: absolute;
  right: 4px;
}

</style>