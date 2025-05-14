<script setup>
import { computed } from 'vue'
const props = defineProps({
   order: Array,
   userName: String ,
   deleteOrder: Function
})
const emit = defineEmits(['actionFour'])

const filteredOrders = computed(() => {
  if (!props.order) {
    return [];
  }
  return props.order.filter(item => item.author === props.userName);
});

function del(keyToDelete) {
  emit('actionFour', keyToDelete);
}
</script>
<template>
<div class="gridd">
<div v-for="(item, index) in filteredOrders" :key="index" class="square"> 
    <p>Задание: {{ item.order_text }}</p> 
    <p>Дедлайн: {{ item.deadline }}</p> 
    <p>Автор: {{ item.author }}</p> 
    <p>Требуемое количество слов: {{ item.count }}</p> 
    <p>Цена за задание: {{ item.pay }} р.</p> 
    <button @click="props.deleteOrder(item.key)" class="btn">Удалить</button>
</div> 
</div>
</template>
<style scoped>
.gridd{
    display: grid;
    grid-template-columns: repeat(4, 1fr);
    gap: 10px; 
}
 .square{
    width: 270px;
    height: auto;
    color: rgb(0, 0, 0);
    background: linear-gradient(to bottom left, rgba(183, 45, 45, 0.5), rgb(250, 167, 181, 0.5));
    border-radius: 25%;
    padding: 20px;
    margin-top: 50px;
    word-wrap: break-word; 
    white-space: normal;
 }
 .btn{
    width: 150px;
    height: 60px;
    background: linear-gradient(to bottom right, rgb(255, 132, 0), rgb(250, 167, 181));
    border-color: rgb(202, 43, 11);
}
.btn:hover{
   background: linear-gradient(to bottom right, rgb(196, 88, 0), rgb(183, 108, 108));
}
</style>