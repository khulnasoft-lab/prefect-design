<template>
  <div class="p-table">
    <table class="p-table__table">
      <slot>
        <p-table-head>
          <slot name="header">
            <p-table-row>
              <template v-for="column in columns" :key="column">
                <p-table-header :style="getColumnStyle(column)">
                  <slot :name="`${column}-heading`" v-bind="{ column }">
                    {{ column.label }}
                  </slot>
                </p-table-header>
              </template>
            </p-table-row>
          </slot>
        </p-table-head>
        <p-table-body>
          <template v-for="(row, index) in data" :key="index">
            <p-table-row>
              <template v-for="column in columns" :key="column">
                <p-table-data>
                  <slot :name="kebabCase(column.property)" :value="row[column.property]" v-bind="{ column, row }">
                    {{ row[column.property] }}
                  </slot>
                </p-table-data>
              </template>
            </p-table-row>
          </template>
        </p-table-body>
        <template v-if="slots.footer">
          <p-table-foot>
            <slot name="footer" />
          </p-table-foot>
        </template>
      </slot>
    </table>
  </div>
</template>

<script lang="ts" setup>
  import { computed, StyleValue, useSlots } from 'vue'
  import PTableBody from './PTableBody.vue'
  import PTableData from './PTableData.vue'
  import PTableFoot from './PTableFoot.vue'
  import PTableHead from './PTableHead.vue'
  import PTableHeader from './PTableHeader.vue'
  import PTableRow from './PTableRow.vue'
  import { TableColumn } from '@/types/tables'
  import { isStrings } from '@/utilities/arrays'
  import { kebabCase } from '@/utilities/strings'

  const props = defineProps<{
    data: Record<string, unknown>[],
    columns?: TableColumn[],
  }>()

  const slots = useSlots()

  const columns = computed<TableColumn[]>(() => {
    if (props.columns) {
      if (isStrings(props.columns)) {
        return convertStringsToTableColumns(props.columns)
      }

      return props.columns
    }

    if (props.data.length) {
      const properties = Object.keys(props.data[0])

      return convertStringsToTableColumns(properties)
    }

    return []
  })

  function getColumnStyle(column: TableColumn): StyleValue {
    if (column.width === undefined) {
      return ''
    }

    return {
      width: column.width,
    }
  }

  function convertStringsToTableColumns(columns: string[]): TableColumn[] {
    return columns.map(property => ({ label: property, property }))
  }
</script>

<style scoped>
.p-table { @apply
  overflow-hidden
  overflow-x-auto
  shadow
  ring-1
  ring-black
  ring-opacity-5
  md:rounded-lg
}
.p-table__table { @apply
  min-w-full
  divide-y
  divide-gray-300
}
</style>