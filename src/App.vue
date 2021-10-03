<template>
  <!-- <div class="app-form"> 
    <el-card class="index-filter">
      <form-filter />
    </el-card> -->
  <el-card class="box-card">
    <template #header>
      <div>
        <h1>TODO-List</h1>
      </div>
      <div class="card-header">
        <el-tag>Total: {{ total }}</el-tag>
        <el-tag type="success">Success: {{ success }}</el-tag>
        <el-tag type="danger">Pending: {{ pending }}</el-tag>
      </div>
      <div>
        <el-input
          v-model="input1"
          clearable
          placeholder="请输入新任务"
          @keyup.enter="addRow(tableData)"
        >
          <template #append>
            <el-button
              icon="el-icon-right"
              @click="addRow(tableData)"
            ></el-button>
          </template>
        </el-input>
      </div>
      <div>
        <el-select
          v-model="selectValue"
          placeholder="Priority Selector"
          @change="searchSelect()"
        >
          <el-option
            v-for="item in options"
            :key="item.value"
            :label="item.label"
            :value="item.value"
          >
          </el-option>
        </el-select>
      </div>
    </template>
    <el-table :data="tableData" style="width: 100%">
      <el-table-column
        prop="mission"
        :index="indexMethod"
        label="任务"
        width="400"
        :cell-class-name="finished"
      ></el-table-column>
      <el-table-column prop="priority" label="目的" width="400">
        <template #default="scope">
          <el-select
            v-model="scope.row.priority"
            placeholder="Priority"
            @change="priorityChange(scope.row.priority, scope.$index, tableData)"
          >
            <el-option
              v-for="item in options"
              :key="item.value"
              :label="item.label"
              :value="item.value"
            >
            </el-option>
          </el-select>
        </template>
      </el-table-column>
      <el-table-column prop="dueDate" label="截至时间" width="400">
        <template #default="scope">
            <div class="block">
              <el-date-picker v-model="scope.row.dueDay" type="date" placeholder="选择日期">
              </el-date-picker>
            </div>
        </template>
      </el-table-column>
      <!-- <el-table-column prop="date" label="完成时间" width="300">
        <el-date-picker v-model="dateValue" type="date" placeholder="选择日期">
        </el-date-picker>
      </el-table-column> -->
      <el-table-column prop="completed" label="完成">
        <template #default="scope">
          <el-button
            type="primary"
            icon="el-icon-check"
            @click="completeRow(scope.$index, tableData)"
          >
            完成
          </el-button>
        </template>
      </el-table-column>
      <el-table-column prop="deleted" label="删除">
        <template #default="scope">
          <el-button
            type="primary"
            icon="el-icon-delete"
            @click.prevent="deleteRow(scope.$index, tableData)"
          >
            移除
          </el-button>
        </template>
      </el-table-column>
    </el-table>

    <el-table :data="tableData1" style="width: 100%">
      <el-table-column
        prop="mission1"
        label="任务"
        width="600"
      ></el-table-column>
      <el-table-column prop="finishTime" label="完成时间"> </el-table-column>
    </el-table>
  </el-card>
</template>

<script>
import { defineComponent, ref } from 'vue'
import { ElMessage } from 'element-plus'
export default defineComponent({
  setup() {
    let missionList = new Array()
    var priorityList = new Array()
    var today = new Date()
    let missionlocal = 'missionlocal'
    let prioritylocal = 'prioritylocal'
    let mission2 = new Array()
    var time =
      today.getFullYear() + '-' + (today.getMonth() + 1) + '-' + today.getDate()
    let success = ref(0)
    let pending = ref(1)
    let total = ref(1)
    let input1 = ref('')
    let selectValue = ref('')
    //let dateValue = ref('')
    //初始化本地储存的数组
    let storageMission = JSON.parse(localStorage.getItem(missionlocal))
    let storagePriority = JSON.parse(localStorage.getItem(prioritylocal))
    if (storageMission != null) {
      mission2 = storageMission
      priorityList = storagePriority
      var tableData = ref([{ mission: storageMission[0], priority: storagePriority[0]}])
      for (let i = 1; storageMission[i] != null; i++) {
        total.value += 1
        pending.value += 1
        tableData.value.push({
          mission: storageMission[i],
          priority: storagePriority[i]
        })
      }
    } else {
      tableData = ref([{ mission: '创建第一个任务吧', priority: '' }])
    }
    var tableData1 = ref([{}])
    const addRow = () => {
      let inputValue = input1.value
      if (inputValue != '') {
        tableData.value.splice(0, 0, {
          mission: inputValue,
          priority: ''
        })
        total.value += 1
        pending.value += 1
        mission2.unshift(inputValue)
        localStorage.setItem(missionlocal, JSON.stringify(mission2))
      } else {
        ElMessage('任务不能为空')
      }
      console.log(tableData.value)
    }
    const searchSelect = () => {
      for (let i = 0; i < missionList.length; i++) {
        tableData.value.push({
          mission: missionList[i],
          priority: ''
        })
      }
      missionList.splice(0, missionList.length)
      for (let i = 0; i < tableData.value.length; i++) {
        if (selectValue.value != tableData.value[i].priority) {
          missionList.push(tableData.value[i].mission)
          tableData.value.splice(i, 1)
          i -= 1
        }
      }
    }
    const deleteRow = (index, rows) => {
      rows.splice(index, 1)
      total.value -= 1
      pending.value -= 1
      ElMessage('任务已删除')
      mission2.splice(index, 1)
      priorityList.splice(index, 1)
      localStorage.setItem(prioritylocal, JSON.stringify(priorityList))
      localStorage.setItem(missionlocal, JSON.stringify(mission2))
    }
    const completeRow = (index, rows) => {
      rows[index].mission += '(已完成)'
      tableData1.value.unshift({
        mission1: rows[index].mission,
        finishTime: time
      })
      rows.splice(index, 1)
      pending.value -= 1
      success.value += 1
      total.value -= 1
      mission2.splice(index, 1)
      priorityList.splice(index, 1)
      localStorage.setItem(prioritylocal, JSON.stringify(priorityList))
      localStorage.setItem(missionlocal, JSON.stringify(mission2))
      ElMessage.success({
        message: '任务成功完成',
        type: 'success'
      })
    }
    const priorityChange = (priority, index, rows) => {
      rows[index].priority = priority
      console.log(tableData)
      priorityList[index] = priority
      localStorage.setItem(prioritylocal, JSON.stringify(priorityList))
      console.log(tableData.value)
    }
    return {
      input1,
      selectValue,
      tableData,
      tableData1,
      addRow,
      searchSelect,
      deleteRow,
      completeRow,
      priorityChange,
      success,
      total,
      pending,
      currentPage: 1,
      intPageSize: 10
    }
  },
  data() {
    return {
      dueDay: '',
      options: [
        {
          value: '练习',
          label: '练习'
        },
        {
          value: '算分',
          label: '算分'
        },
        {
          value: '考试',
          label: '考试'
        }
      ]
    }
  },
  method: {
    indexMethod(index) {
      return (this.currentPage - 1) * this.intPageSize + index + 1
    }
  }
})
</script>

<style>
.card-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
}
.finished {
  text-decoration: line-through;
}
</style>
