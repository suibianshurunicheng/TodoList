<template>
  <div class="edit-container">
    <input v-model="task.inputTitle" placeholder="任务名称" />
    <input v-model="task.content" placeholder="内容" />
    <button @click="saveTask">保存</button>
    <button @click="close">关闭</button>
  </div>
</template>

<script setup>
import { defineProps, defineEmits } from 'vue';
// 接收父组件传来的属性
const props = defineProps({
  task: {
    type: Object,
    required: true
  }
});
// 定义子组件触发的事件
const emit = defineEmits(['save', 'close']);
// 将更新后的数据传给父组件
const saveTask = () => {
  emit('save', props.task, props.task.$index);
};
// 通知父组件关闭子组件
const close = () => {
  emit('close');
};
</script>

<style scoped>
.edit-container {
  border: 1px solid #ccc;
  padding: 10px;
  background-color: #f9f9f9;
}
</style>