<script setup>
const props = defineProps({
   order: Array,
})
const emit = defineEmits(['actionSeven'])
function connect(author){
   emit('actionSeven', author)
}
</script>
<template>
<div class="gridd">
   <div class="square"> 
    <p>Задание: Написать доклад про северную природу</p> 
    <p>Дедлайн: 12.04.2024</p> 
    <p>Автор: Tom</p> 
    <p>Требуемое количество слов: 300</p> 
    <p>Цена за задание: 200р.</p> 
    <button @click="connect('Tom')" class="btn">Связаться с заказчиком</button>
</div> 

<div v-for="(item, index) in props.order" :key="index" class="square"> 
    <p>Задание: {{ item.order_text }}</p> 
    <p>Дедлайн: {{ item.deadline }}</p> 
    <p>Автор: {{ item.author }}</p> 
    <p>Требуемое количество слов: {{ item.count }}</p> 
    <p>Цена за задание: {{ item.pay }} р.</p> 
    <button @click="connect(item.author)" class="btn">Связаться с заказчиком</button>
</div> 
</div>
</template>
<style scoped>
.gridd{
    display: grid;
    grid-template-columns: repeat(5, 1fr);
    gap: 10px; 
}
 .square{
    width: 270px;
    height: auto;
    color: black;
    background: linear-gradient(to bottom left, rgba(183, 45, 45, 0.5), rgb(250, 167, 181, 0.5));
    border-radius: 25%;
    padding: 20px;
    margin-top: 50px;
    word-wrap: break-word; /* Разрешаем перенос слов на новые строки */
    white-space: normal; /* Отключаем сжатие текста в одну строку */
 }
 .btn {
   width: 200px;
   height: auto;
   background: linear-gradient(to bottom right, rgb(255, 132, 0), rgb(250, 167, 181));
   border-color: rgb(202, 43, 11);
}

.btn:hover{
   background: linear-gradient(to bottom right, rgb(196, 88, 0), rgb(183, 108, 108));
}
</style>
