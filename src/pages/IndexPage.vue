<template>
  <q-page class="row q-pt-xl">
    <div class="full-width q-px-xl">
      <div class="q-mb-xl">
        <q-input v-model="tempData.name" label="姓名" />
        <q-input v-model="tempData.age" label="年齡" />
        <q-btn color="primary" class="q-mt-md" @click="createTableData()">新增</q-btn>
      </div>

      <q-table
        flat
        bordered
        ref="tableRef"
        :rows="blockData"
        :columns="(tableConfig as QTableProps['columns'])"
        row-key="id"
        hide-pagination
        separator="cell"
        :rows-per-page-options="[0]"
      >
        <template v-slot:header="props">
          <q-tr :props="props">
            <q-th v-for="col in props.cols" :key="col.name" :props="props">
              {{ col.label }}
            </q-th>
            <q-th></q-th>
          </q-tr>
        </template>

        <template v-slot:body="props">
          <q-tr :props="props">
            <q-td
              v-for="col in props.cols"
              :key="col.name"
              :props="props"
              style="min-width: 120px"
            >
              <q-input v-if="currDate.id === props.row.id" v-model="currDate[col.name]" :label="col.label" @change="editData()" />
              <div v-else>{{ col.value }}</div>
            </q-td>
            <q-td class="text-right" auto-width v-if="tableButtons.length > 0">
              <q-btn
                v-for="(btn, index) in tableButtons"
                :key="index"
                size="sm"
                color="grey-6"
                round
                dense
                :icon="btn.icon"
                class="q-ml-md"
                padding="5px 5px"
                @click="handleClickOption(btn, props.row)"
              >
                <q-tooltip
                  transition-show="scale"
                  transition-hide="scale"
                  anchor="top middle"
                  self="bottom middle"
                  :offset="[10, 10]"
                >
                  {{ btn.label }}
                </q-tooltip>
              </q-btn>
            </q-td>
          </q-tr>
        </template>
        <template v-slot:no-data="{ icon }">
          <div
            class="full-width row flex-center items-center text-primary q-gutter-sm"
            style="font-size: 18px"
          >
            <q-icon size="2em" :name="icon" />
            <span> 無相關資料 </span>
          </div>
        </template>
      </q-table>
    </div>
  </q-page>
</template>

<script setup lang="ts">
import axios from 'axios';
import { QTableProps } from 'quasar';
import { ref } from 'vue';
interface btnType {
  label: string;
  icon: string;
  status: string;
}

const blockData = ref([]);

const getTableDataList = () => {
  axios
    .get('https://dahua.metcfire.com.tw/api/CRUDTest/a')
    .then((response) => {
      if (response.status === 200) {
        blockData.value = response.data;
      }
    })
    .catch((error) => {
      blockData.value = [];
    });
}

const tableConfig = ref([
  {
    label: '姓名',
    name: 'name',
    field: 'name',
    align: 'left',
  },
  {
    label: '年齡',
    name: 'age',
    field: 'age',
    align: 'left',
  },
]);
const tableButtons = ref([
  {
    label: '編輯',
    icon: 'edit',
    status: 'edit',
  },
  {
    label: '刪除',
    icon: 'delete',
    status: 'delete',
  },
]);

const tempData = ref({
  name: '',
  age: '',
});

const createTableData = async () => {
  axios
    .post('https://dahua.metcfire.com.tw/api/CRUDTest', tempData.value)
    .then((response) => {
      if (response.status === 200) {
        getTableDataList()
      }
    })
    .catch((error) => {
      console.log(error)
    })
}
const deleteTableData = (id) => {
  axios
    .delete(`https://dahua.metcfire.com.tw/api/CRUDTest/${id}`)
    .then((response) => {
      if (response.status === 200) {
        getTableDataList()
      }
    })
    .catch((error) => {
      console.log(error)
    })
}

const currDate = ref({});

const editData = () => {
  axios
    .patch('https://dahua.metcfire.com.tw/api/CRUDTest', currDate.value)
    .then((respones) => {
      console.log(respones)
      if (respones.status === 200) {
        currDate.value = {};
      }
    })
    .catch((error) => {
      console.log(error)
    })
}
const handleClickOption = (btnType, rowData) => {
  if (btnType.status === 'edit') {
    currDate.value = rowData;
  } else if (btnType.status === 'delete') {
    deleteTableData(rowData.id);
  }
}

// init
getTableDataList()
</script>

<style lang="scss" scoped>
.q-table th {
  font-size: 20px;
  font-weight: bold;
}

.q-table tbody td {
  font-size: 18px;
}
</style>
