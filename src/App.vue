<script setup>
import { ref, reactive, computed } from 'vue'
import login from './components/login.vue'
import orders from './components/orders.vue'
import works from './components/works.vue'
import account from './components/account.vue'
import modal from './components/modal.vue'
import my_orders from './components/my_orders.vue'
import message from './components/message.vue'
import authorization from './components/authorization.vue'

const activee = ref(1)
const modal_open = ref(false)
const message_open = ref(false)
const text = ref("")
const show1 = ref(false)
const show2 = ref(false)
const isAuthenticated = ref(false)

const data = reactive({
  obj: {
    name: "",
    gender: "",
    age: "",
    role: "",
    email: "",
    new_orders: [],
    new_rezumes: [],
    interlocutor: "",
    messages: {},
    reviews: [],
    balance: 0,
    card: null
  }
})

// для открытия страниц
function active2() {
  if (data.obj.name === "") {
    text.value = ""
    modal_open.value = true
    text.value = "Чтобы перейти на эту страницу, нужно в начале зарегестрироваться"
  } else if (data.obj.role === "копирайтер") {
    activee.value = 2;
  }
}

function active1() {
  activee.value = 1;
}

function active6() {
  activee.value = 6;
}

function active3() {
  if (data.obj.name === "") {
    text.value = ""
    modal_open.value = true
    text.value = "Чтобы перейти на эту страницу, нужно в начале зарегистрироваться"
  } else {
    activee.value = 3;
  }
}

function active4() {
  if (data.obj.name === "") {
    text.value = ""
    modal_open.value = true
    text.value = "Чтобы перейти на эту страницу, нужно в начале зарегестрироваться"
  } else if (data.obj.role === "заказчик") {
    activee.value = 4;
  }
}

function active5() {
  if (data.obj.name === "") {
    text.value = ""
    modal_open.value = true
    text.value = "Чтобы перейти на эту страницу, нужно в начале зарегестрироваться"
  } else if (data.obj.role === "заказчик") {
    activee.value = 5;
  }
}

function close() {
  modal_open.value = false
}

function close_message() {
  message_open.value = false
}

function handleLoginSuccess(userData) {
  data.obj.name = userData.name
  data.obj.gender = userData.gender
  data.obj.age = userData.age
  data.obj.role = userData.role

  isAuthenticated.value = true

  show1.value = (userData.role === "копирайтер")
  show2.value = (userData.role === "заказчик")

  activee.value = 3
}

function appData(inf) {
  data.obj.name = inf.name
  data.obj.age = inf.age
  data.obj.gender = inf.gender
  data.obj.role = inf.role
  isAuthenticated.value = true
  show1.value = (inf.role === "копирайтер");
  show2.value = (inf.role === "заказчик");
  activee.value = 3;
}

function checkAuth(credentials) {
  // Здесь должна быть логика проверки с вашим хранилищем пользователей
  // Это пример - вам нужно адаптировать под вашу структуру данных

  // Проверяем, есть ли пользователь с таким именем и паролем
  const userExists = /* ваша логика проверки */ false;

  if (userExists) {
    isAuthenticated.value = true
    // Загружаем данные пользователя
    data.obj.name = credentials.name
    // ... другие данные пользователя
    activee.value = 3 // Переходим в личный кабинет
  } else {
    text.value = "Неверное имя пользователя или пароль"
    modal_open.value = true
  }
}

function exit() {
  // Очищаем данные
  data.obj.name = ""
  data.obj.age = ""
  data.obj.gender = ""
  data.obj.role = ""

  // Сбрасываем флаги
  isAuthenticated.value = false
  show1.value = false
  show2.value = false

  // Переходим на страницу авторизации
  activee.value = 6
}

function newOrder(order) {
  data.obj.new_orders.push({
    order_text: order.order_text,
    deadline: order.deadline,
    pay: order.pay,
    count: order.count,
    author: order.author,
    key: Date.now()
  })
}

function newRezume(rezume) {
  data.obj.new_rezumes.push({
    skills: rezume.skills,
    pay: rezume.pay,
    portfolio: rezume.portfolio,
    author: rezume.author,
    key: Date.now()
  })
}

function updateRezume(updatedRezume) {
  const index = data.obj.new_rezumes.findIndex(r => r.author === data.obj.name);
  if (index !== -1) {
    data.obj.new_rezumes[index] = {
      ...updatedRezume,
      key: data.obj.new_rezumes[index].key, // сохраняем тот же ключ
      author: data.obj.name
    };
  }
}

function del(keyToDelete) {
  deleteModal.show = true;
  deleteModal.orderKey = keyToDelete;
  deleteModal.reason = null;
  deleteModal.rating = null;
}

// // Добавьте функцию подтверждения удаления
// function confirmDelete() {
//   if (!deleteModal.reason) return;

//   if (deleteModal.reason === 'completed' && !deleteModal.rating) {
//     return; // Ждем пока пользователь поставит оценку
//   }

//   // Логируем данные перед удалением
//   console.log('Удаление заказа:', {
//     key: deleteModal.orderKey,
//     reason: deleteModal.reason,
//     rating: deleteModal.rating
//   });

//     if (deleteModal.reason === 'completed' && deleteModal.rating) {
//     // Добавляем поиск копирайтера
//     const selectedCopywriter = prompt("Введите имя копирайтера, который выполнил заказ:");
//     if (selectedCopywriter) {
//       data.obj.reviews.push({
//         copywriter: selectedCopywriter,
//         rating: deleteModal.rating,
//         date: new Date().toISOString(),
//         orderKey: deleteModal.orderKey
//       });
//     }
//   }

//   // Удаляем заказ
//   data.obj.new_orders = data.obj.new_orders.filter(
//     order => order.key !== deleteModal.orderKey
//   );

//   // Автоматически закрываем модальное окно
//   deleteModal.show = false;
// }


function confirmDelete() {
  if (deleteModal.reason === 'completed') {
    if (!copywriterName.value || !deleteModal.rating) return;

    data.obj.reviews.push({
      copywriter: copywriterName.value,
      rating: deleteModal.rating,
      date: new Date().toISOString(),
      orderKey: deleteModal.orderKey
    });

    copywriterName.value = '';
  }

  data.obj.new_orders = data.obj.new_orders.filter(
    order => order.key !== deleteModal.orderKey
  );

  deleteModal.show = false;
}

const copywriterName = ref('');

// Функция для поиска копирайтеров
const availableCopywriters = computed(() => {
  return [...new Set(data.obj.reviews.map(r => r.copywriter))];
});

// Добавляем функцию для обработки выбора причины
function handleReasonSelect(reason) {
  deleteModal.reason = reason;
  if (reason !== 'completed') {
    confirmDelete(); // Автоматически подтверждаем если не нужна оценка
  }
}

function send_message(author) {
  if (author && author !== data.obj.name) {
    message_open.value = true;
    data.obj.interlocutor = author;

    // Инициализация диалога, если его нет
    if (!data.obj.messages[author]) {
      data.obj.messages[author] = [];
    }
  }
}

function receiveMessage({ author, message, file }) {
  console.log('Получено сообщение:', { author, message, file: file?.name });
  if (!author || !message) {
    console.error('Не указан получатель или текст сообщения');
    return;
  }

  // Создаем сообщение для отправителя
  const newMessage = {
    text: message,
    sender: data.obj.name,
    timestamp: new Date().toLocaleTimeString(),
    file: file ? { ...file } : null
  };

  // Сохраняем у отправителя
  if (!data.obj.messages[author]) {
    data.obj.messages[author] = [];
  }
  data.obj.messages[author].push(newMessage);

  // Сохраняем у получателя (если это не сообщение самому себе)
  if (author !== data.obj.name) {
    if (!data.obj.messages[data.obj.name]) {
      data.obj.messages[data.obj.name] = [];
    }

    const recipientMessage = {
      ...newMessage,
      sender: data.obj.name // Для получателя sender - это отправитель
    };

    data.obj.messages[data.obj.name].push(recipientMessage);
  }

  // Форсируем обновление
  data.obj.messages = { ...data.obj.messages };
}

function saveMessage(recipient, message, file) {
  if (!data.obj.messages[recipient]) {
    data.obj.messages[recipient] = [];
  }

  data.obj.messages[recipient].push({
    text: message,
    sender: data.obj.name,
    timestamp: new Date().toLocaleTimeString(),
    file: file ? { ...file } : null  // Полная копия файла
  });

  // Форсируем обновление
  data.obj.messages = { ...data.obj.messages };
}

const deleteModal = reactive({
  show: false,
  orderKey: null,
  reason: null,
  rating: null
});

function updateBalance(newBalance) {
  data.obj.balance = newBalance;
}

function updateCard(cardDetails) {
  data.obj.card = cardDetails;
}
</script>

<template>
  <div class="block">
    <button v-if="show1" @click="active2" :class="['btn', { active: activee === 2 }]">Заказы</button>
    <button v-if="show2" @click="active4" :class="['btn', { active: activee === 4 }]">Копирайтеры</button>
    <button v-if="show2" @click="active5" :class="['btn', { active: activee === 5 }]">Мои заказы</button>
    <button @click="active3" v-if="isAuthenticated" :class="['btn', { active: activee === 3 }]">Личный кабинет</button>
    <button @click="active1" :class="['btn', { active: activee === 1 }]">Регистрация</button>
    <button @click="active6" :class="['btn', { active: activee === 6 }]">Авторизация</button>
  </div>

  <modal v-if="deleteModal.show" @modulClose="() => deleteModal.show = false">
    <template #default>
      <h3>По какой причине удаляете свой заказ?</h3>
      <div class="delete-reasons">
        <button @click="handleReasonSelect('irrelevant')" class="reason-btn">
          Уже не актуален
        </button>
        <button @click="handleReasonSelect('completed')" class="reason-btn">
          Его сделал копирайтер
        </button>
      </div>

      <div v-if="deleteModal.reason === 'completed'" class="rating-section">
        <h4>Оцените работу копирайтера:</h4>
        <input v-model="copywriterName" placeholder="Введите имя копирайтера" class="copywriter-input">
        <div class="rating-stars">
          <span v-for="n in 5" @click="deleteModal.rating = n" :class="{ active: deleteModal.rating >= n }">
            ★
          </span>
        </div>
        <button @click="confirmDelete" :disabled="!copywriterName || !deleteModal.rating" class="confirm-rating-btn">
          Подтвердить оценку
        </button>
      </div>
    </template>
  </modal>

  <login v-if="activee === 1" 
  @actionOne="appData" />
  <authorization v-if="activee === 6" 
  @loginSuccess="handleLoginSuccess" />
  <my_orders v-if="activee === 5" 
  :order="data.obj.new_orders" 
  :userName="data.obj.name" 
  :deleteOrder="del"
    @actionFour="del" />
  <orders v-if="activee === 2" 
  :order="data.obj.new_orders" 
  @actionSeven="send_message" />
  <works v-if="activee === 4" 
  :rezumes="data.obj.new_rezumes" 
  @actionSix="send_message" />
  <account v-if="activee === 3" 
  :data="data.obj" 
  :hasRezume="data.obj.new_rezumes.some(r => r.author === data.obj.name)"
    :reviews="data.obj.reviews" 
    @actionTwo="exit" 
    @actionThree="newOrder" 
    @actionFive="newRezume"
    @updateRezume="updateRezume" 
    @openMessages="send_message" 
    @updateBalance="updateBalance"
    @updateCard="updateCard"/>
  <message v-if="message_open" :interlocutor="data.obj.interlocutor"
    :messages="data.obj.messages[data.obj.interlocutor] || []" :currentUser="data.obj.name" @modulClose="close_message"
    @sendMessage="receiveMessage" />


</template>

<style>
body,
html {
  background-image: url("./assets/sky.jpg");
  background-size: cover;
}
</style>

<style scoped>
* {
  font-family: Comic Sans MS, Comic Sans, cursive;
  font-size: 1.2rem;
}

.block {
  display: flex;
  position: fixed;
  top: 0;
  left: 50%;
  transform: translate(-50%, 0%);
  justify-content: center;
  width: 100%;
  height: 20%;
  background: linear-gradient(rgb(169, 28, 47), rgba(255, 192, 203, 0));
}

.btn {
  background-color: #e3bebe83;
  color: #601818;
  margin: 10px;
  height: 35%;

}

.btn.active {
  background-color: #771b1be7;
  color: white;
  margin: 10px;
}

.btn:hover {
  background-color: #f29595;
}

.hidden {
  display: none;
}

.delete-reasons {
  display: flex;
  flex-direction: column;
  gap: 10px;
  margin: 20px 0;
}

.delete-reasons label {
  display: flex;
  align-items: center;
  gap: 8px;
  cursor: pointer;
}

.rating-section {
  margin-top: 20px;
}

.rating-select {
  padding: 8px;
  border-radius: 4px;
  border: 1px solid #ccc;
  font-size: 16px;
}

.confirm-btn {
  background-color: #4CAF50;
  color: white;
  margin-right: 10px;
}

.cancel-btn {
  background-color: #f44336;
  color: white;
}

.delete-reasons {
  display: flex;
  gap: 15px;
  margin: 20px 0;
  justify-content: center;
}

.reason-btn {
  padding: 10px 20px;
  background: #4CAF50;
  color: white;
  border: none;
  border-radius: 5px;
  cursor: pointer;
  font-size: 16px;
}

.reason-btn:hover {
  background: #45a049;
}

.rating-stars {
  font-size: 24px;
  color: #ccc;
  cursor: pointer;
  margin-top: 10px;
}

.rating-stars span {
  transition: color 0.2s;
}

.rating-stars span.active {
  color: #ffc107;
}

.rating-stars span:hover {
  color: #ffc107;
}

.copywriter-input {
  padding: 8px;
  margin: 10px 0;
  width: 100%;
  border-radius: 4px;
  border: 1px solid #ccc;
}

.confirm-rating-btn {
  padding: 10px;
  background: #4CAF50;
  color: white;
  border: none;
  border-radius: 4px;
  cursor: pointer;
  margin-top: 10px;
}

.confirm-rating-btn:disabled {
  background: #cccccc;
  cursor: not-allowed;
}
</style>