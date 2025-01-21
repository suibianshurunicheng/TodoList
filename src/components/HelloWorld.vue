<template>
    <div class="task-manager-container">
        <!--添加任务-->
        <div class="input-button-section">
            <input v-model="inputTitle" placeholder="请输入任务内容" @keyup.enter="addTableRow" />
            <div class="button-group">
                <button @click="addTableRow">添加任务</button>
                <!--排序-->
                <button @click="sortByTime">按时间排序</button>
                <button @click="restoreDefaultSort">恢复默认排序</button>
                <!--清除已完成任务-->
                <button @click="clearCompletedRows">清除已完成的任务</button>
            </div>
        </div>
        <!--任务列表-->
        <table>
            <thead>
                <tr>
                    <th>完成情况</th>
                    <th>添加时间</th>
                    <th>任务名称</th>
                    <th>设置优先级</th>
                    <th>操作</th>
                </tr>
            </thead>
            <tbody>
                <tr v-for="(row, index) in filteredTableData" :key="index" :class="{ 'check-row': row.isChecked }">
                    <td>
                        <input type="checkbox" v-model="row.isChecked" />
                        <span>{{ row.isChecked ? '已完成' : '未完成' }}</span>
                    </td>
                    <td>{{ row.time }}</td>
                    <td>
                        <p @click="handler(index)">{{ row.inputTitle }}</p>
                        <Edit v-if="editIndex === index" :task="row" @save="saveEdit" @close="closeEdit"></Edit>
                    </td>
                    <td>
                        <input type="radio" v-model="row.priority" value="tall" :name="`row${index}`"/>高
                        <input type="radio" v-model="row.priority" value="well" :name="`row${index}`"/>中
                        <input type="radio" v-model="row.priority" value="low" :name="`row${index}`"/>低
                    </td>
                    <td><button @click="adddelete(index)">删除任务</button></td>
                </tr>
            </tbody>
        </table>
        <div class="filter-option-group">
            <input type="radio" v-model="filterOption" value="all" />显示全部任务
            <input type="radio" v-model="filterOption" value="uncompleted" />仅显示未完成的任务
            <input type="radio" v-model="filterOption" value="completed" />仅显示已完成的任务
        </div>
    </div>
</template>
<script setup>
import Edit from './Edit/index.vue'
import { ref, onMounted, watch, computed, nextTick } from 'vue';
// 定义表格数据
const tableData = ref([]);
// 定义输入框绑定的变量
const inputTitle = ref('');
// 过滤选项
const filterOption = ref('all');
// 按时间排序
const defaultTableData = ref([]);
// 定义子组件的显示与隐藏
const editIndex = ref(null)
// 定义添加表格行的方法
const addTableRow = () => {
    if (inputTitle.value.trim() !== '') {
        tableData.value.push({
            inputTitle: inputTitle.value,
            isChecked: false,
            priority: 'low',
            time: new Date().toISOString()
        });
        inputTitle.value = '';
    } else {
        alert('内容不能为空')
    }
};
// 删除操作
const adddelete = (index) => {
    tableData.value.splice(index, 1)
}
// 修改页面
const handler = (index) => {
  editIndex.value = index;
};
// 接收更新的数据
const saveEdit = (updatedTask, index) => {
  tableData.value[index] = updatedTask;
  editIndex.value = null;
};
// 关闭编辑状态
const closeEdit = () => {
  editIndex.value = null;
};
// 从localStorage中读取数据并初始化tableData
onMounted(() => {
    const storedData = localStorage.getItem('tableData');
    if (storedData) {
        tableData.value = JSON.parse(storedData);
    }
});
// 监听tableData变化，保存到localStorage
watch(tableData, (newData) => {
    localStorage.setItem('tableData', JSON.stringify(newData));
}, { deep: true });
// 过滤后的数据
const filteredTableData = computed(() => {
    if (filterOption.value === 'all') {
        return tableData.value;
    } else if (filterOption.value === 'uncompleted') {
        return tableData.value.filter(row => !row.isChecked);
    } else {
        return tableData.value.filter(row => row.isChecked);
    }
});
// 按时间排序
const sortByTime = () => {
    if (defaultTableData.value.length === 0) {
        defaultTableData.value = [...tableData.value];
    }
    tableData.value.sort((a, b) => {
        return new Date(a.time) - new Date(b.time);
    });
};
// 默认排序
const restoreDefaultSort = () => {
    tableData.value = [...defaultTableData.value];
};
// 清除已完成的任务
const clearCompletedRows = () => {
    tableData.value = tableData.value.filter(row => !row.isChecked);
    // 同步更新defaultTableData
    defaultTableData.value = tableData.value.filter(row => !row.isChecked);
};
// 优先级排序函数
const sortByPriority = () => {
    const priorityOrder = { tall: 3, well: 2, low: 1 };
    tableData.value.sort((a, b) => priorityOrder[b.priority] - priorityOrder[a.priority]);
};
watch(() => tableData.value.map(row => row.priority), () => {
    sortByPriority();
});
</script>

<style scoped>
input {
    padding: 5px;
    margin: 5px;
}

table {
    border-collapse: collapse;
}

th,
td {
    border: 1px solid black;
    padding: 5px;
    text-align: center;
    vertical-align: middle;
}

.check-row {
    background-color: chartreuse;
}
/*手机端能用*/
.task-manager-container {
  display: flex;
  flex-direction: column;
  min-height: 100vh;
  padding: 10px;
}

.input-button-section {
  margin-bottom: 10px;
}

.button-group {
  display: flex;
  flex-wrap: wrap;
  gap: 10px;
}

.button-group button {
  flex: 1 1 auto;
  padding: 10px;
  background-color: #007bff;
  color: white;
  border: none;
  cursor: pointer;
}

.filter-option-group {
  display: flex;
  flex-wrap: wrap;
  gap: 10px;
  margin-top: 10px;
}

.filter-option-group input {
  margin-right: 5px;
}
</style>