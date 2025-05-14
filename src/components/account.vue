<script setup>
import { reactive, ref, computed } from 'vue'

const props = defineProps({
   data: Object,
   hasRezume: Boolean,
   reviews: Array,
})
const emit = defineEmits(['actionTwo', 'actionThree', 'actionFive', 'updateRezume', 'openMessages', 'updateCard', 'updateBalance'])
const error1 = ref("")
const error2 = ref("")
const error3 = ref("")
const error4 = ref("")
const error11 = ref("")
const error22 = ref("")
const balance = ref(0)
const order = reactive({
   text: "",
   date: "",
   count: "",
   pay: ""
})
const rezume = reactive({
   skills: "",
   pay: "",
   portfolio: "",
})
const cardData = reactive({
   cardNumber: "",
   cardHolder: "",
   expiryDate: "",
   cvv: ""
})

const cardErrors = reactive({
   cardNumber: "",
   cardHolder: "",
   expiryDate: "",
   cvv: ""
})
// Заполняем форму данными существующего резюме при монтировании
const existingRezume = computed(() => {
   return props.data.new_rezumes.find(r => r.author === props.data.name)
})
function loadExistingRezume() {
   if (existingRezume.value) {
      rezume.skills = existingRezume.value.skills;
      rezume.pay = existingRezume.value.pay;
      rezume.portfolio = existingRezume.value.portfolio || "";
   }
}

// Вызываем при монтировании
loadExistingRezume()
function exit() {
   emit('actionTwo')
}
function newOrder() {
   let point = false
   if (order.text === "") {
      error1.value = "В чем заключается суть задания?"
   } else {
      point = true
   }
   if (order.date === "") {
      error2.value = "Вы не обозначили дедлайн"
   } else {
      point = true
   }
   if (order.count === "") {
      error3.value = "Укажите примерный объем для задания"
   } else {
      point = true
   }
   if (order.pay === "") {
      error4.value = "Вы не указали цену для задания"
   } else {
      point = true
   }
   if (point) {
      emit('actionThree', {
         order_text: order.text,
         deadline: order.date,
         author: props.data.name,
         count: order.count,
         pay: order.pay
      })
      order.text = ""
      order.date = ""
      order.count = ""
      order.pay = ""
   }
}
function newWork() {
   let pointt = false
   if (rezume.skills === "") {
      error11.value = "Вы не написали о своих способностях"
   } else {
      pointt = true
   }
   if (rezume.pay === "") {
      error22.value = "Вы не обозначили свои расценки"
   } else {
      pointt = true
   }
   if (pointt) {
      if (props.hasRezume) {
         emit('updateRezume', {
            skills: rezume.skills,
            pay: rezume.pay,
            portfolio: rezume.portfolio,
         })
      } else {
         emit('actionFive', {
            skills: rezume.skills,
            pay: rezume.pay,
            portfolio: rezume.portfolio,
            author: props.data.name,
         })

      }
      error11.value = ""
      error22.value = ""
   }
}

const filteredMessages = computed(() => {
   const result = {};
   const currentUser = props.data.name;

   // Перебираем все диалоги
   for (const [interlocutor, messages] of Object.entries(props.data.messages || {})) {
      // Пропускаем диалог с самим собой
      if (interlocutor === currentUser) continue;

      // Фильтруем только входящие сообщения
      const incoming = messages.filter(msg => msg.sender !== currentUser);

      if (incoming.length > 0) {
         result[interlocutor] = incoming;
      }
   }

   return result;
});

const dialogs = computed(() => {
   const result = {};
   const currentUser = props.data.name;

   // Собираем всех собеседников
   for (const interlocutor of Object.keys(props.data.messages || {})) {
      if (interlocutor !== currentUser) {
         result[interlocutor] = props.data.messages[interlocutor];
      }
   }

   return result;
});

function getLastMessage(messages) {
   return messages.length > 0 ? messages[messages.length - 1] : { text: '', timestamp: '' };
}

function openMessages(author) {
   // Убедимся, что открываем диалог с другим пользователем
   if (author !== props.data.name) {
      emit('openMessages', author);
   }
}
const copywriterRating = computed(() => {
   if (props.data.role !== 'копирайтер') return null;

   const myReviews = props.data.reviews?.filter(r => r.copywriter === props.data.name) || [];
   if (myReviews.length === 0) return 'Нет оценок';

   const sum = myReviews.reduce((acc, review) => acc + review.rating, 0);
   return (sum / myReviews.length).toFixed(1);
});
function validateCard() {
   let isValid = true

   // Валидация номера карты (16 цифр)
   if (!/^\d{16}$/.test(cardData.cardNumber.replace(/\s/g, ''))) {
      cardErrors.cardNumber = "Номер карты должен содержать 16 цифр"
      isValid = false
   } else {
      cardErrors.cardNumber = ""
   }

   // Валидация имени владельца
   if (!cardData.cardHolder.trim()) {
      cardErrors.cardHolder = "Введите имя владельца карты"
      isValid = false
   } else {
      cardErrors.cardHolder = ""
   }

   // Валидация срока действия (MM/YY)
   if (!/^(0[1-9]|1[0-2])\/?([0-9]{2})$/.test(cardData.expiryDate)) {
      cardErrors.expiryDate = "Формат: MM/YY"
      isValid = false
   } else {
      cardErrors.expiryDate = ""
   }

   // Валидация CVV (3 цифры)
   if (!/^\d{3}$/.test(cardData.cvv)) {
      cardErrors.cvv = "CVV должен содержать 3 цифры"
      isValid = false
   } else {
      cardErrors.cvv = ""
   }

   return isValid
}
function saveCard() {
   if (validateCard()) {
      emit('updateCard', {
         cardNumber: cardData.cardNumber,
         cardHolder: cardData.cardHolder,
         expiryDate: cardData.expiryDate,
         cvv: cardData.cvv
      })
      // Очистка формы после успешного сохранения
      cardData.cardNumber = ""
      cardData.cardHolder = ""
      cardData.expiryDate = ""
      cardData.cvv = ""
   }
}
// function addFunds(amount) {
//    balance.value += amount
//    emit('updateBalance', balance.value) // Отправляем обновление на сервер
// }

const addAmount = ref(100) // Сумма для пополнения по умолчанию
const showCardModal = ref(false) // Показывать ли модальное окно для карты

function addFunds() {
   if (addAmount.value < 100) {
      alert('Минимальная сумма пополнения - 100 ₽')
      return
   }

   emit('updateBalance', props.data.balance + addAmount.value)
   addAmount.value = 100 // Сбрасываем сумму
   showCardModal.value = false // Закрываем модальное окно
}
</script>
<template>
   <div class="gridd">
      <div class="square">
         <p>Имя: {{ data.name }}</p>
         <p>Пол: {{ data.gender }}</p>
         <p>Возраст: {{ data.age }}</p>
         <p>Роль: {{ data.role }}</p>
         <p v-if="data.role === 'копирайтер'">Рейтинг: {{ copywriterRating }}</p>
         <button @click="exit" class="btn">Выйти</button>
      </div>

      <div class="create">
         <span v-if="data.role === 'заказчик'">
            <button @click="newOrder" class="btn">Создать новый заказ</button>
            <div class="form-group">
               <div class="label">Опишите суть своего заказа</div>
               <textarea v-model="order.text" rows="5"></textarea>
               <div class="error">{{ error1 }}</div>
            </div>

            <div class="form-group">
               <div class="label">Обозначьте дедлайн своего заказа</div>
               <input type="date" v-model="order.date" class="width" min="2025-05-10" max="2030-12-31" />
               <div class="error">{{ error2 }}</div>
            </div>

            <div class="form-group">
               <div class="label">Обозначьте необходимое количество слов</div>
               <input type="number" v-model="order.count" class="width" />
               <div class="error">{{ error3 }}</div>
            </div>

            <div class="form-group">
               <div class="label">Сколько стоит выполнить ваш заказ</div>
               <input type="number" v-model="order.pay" class="width" /> р.
               <div class="error">{{ error4 }}</div>
            </div>

         </span>
         <span v-else>
            <button @click="newWork" class="btn">
               {{ hasRezume ? 'Обновить резюме' : 'Создать резюме' }}
            </button>

            <div class="form-group">
               <div class="label">Расскажите о своих навыках и умениях</div>
               <textarea v-model="rezume.skills" rows="5"></textarea>
               <div class="error">{{ error11 }}</div>
            </div>

            <div class="form-group">
               <div class="label">Какие у вас расценки</div>
               <textarea v-model="rezume.pay" rows="5"></textarea>
               <div class="error">{{ error22 }}</div>
            </div>

            <div class="form-group">
               <div class="label">Сюда можно прикрепить ссылку на ваше портфолио</div>
               <input type="text" v-model="rezume.portfolio" class="width" />
            </div>
         </span>
      </div>

      <div class="square">

         <h3>Диалоги</h3>
         <div v-if="Object.keys(dialogs).length > 0">
            <div v-for="(messages, interlocutor) in dialogs" :key="interlocutor" @click="openMessages(interlocutor)"
               class="message-preview">
               <strong>Диалог с: {{ interlocutor }}</strong>
               <div>Последнее сообщение: {{ getLastMessage(messages).text }}</div>
               <small>{{ getLastMessage(messages).timestamp }}</small>
            </div>
         </div>
         <div v-else>
            Нет активных диалогов
         </div>
      </div>



      <div class="square">
         <h3>Мой баланс</h3>
         <div class="balance-display">
            {{ data.balance }} ₽
         </div>

         <button @click="showCardModal = true" class="btn">Пополнить баланс</button>

         <!-- <div class="add-funds">
            <h4>Пополнить баланс</h4>
            <input v-model.number="addAmount" type="number" min="100" placeholder="Сумма" class="funds-input">
            <button @click="addFunds(addAmount)" class="btn">Пополнить</button>
         </div> -->
      </div>

      <div v-if="showCardModal" class="modal-overlay">
         <div class="modal-content">
            <h3>Пополнение баланса</h3>

            <div class="form-group">
               <label>Сумма пополнения</label>
               <input v-model.number="addAmount" type="number" min="100" class="width">
            </div>

            <h4>Данные карты</h4>

            <div class="form-group">
               <label>Номер карты  </label>
               <input v-model="cardData.cardNumber" type="text" placeholder="1234 5678 9012 3456" maxlength="19"
                  @input="cardData.cardNumber = cardData.cardNumber.replace(/\D/g, '').replace(/(\d{4})/g, '$1 ').trim()">
            </div>

            <div class="form-group">
               <label>Имя владельца  </label>
               <input v-model="cardData.cardHolder" type="text" placeholder="IVAN IVANOV">
            </div>

            <div class="form-group">
               <label>Срок действия (MM/YY)  </label>
               <input v-model="cardData.expiryDate" type="text" placeholder="MM/YY" maxlength="5"
                  @input="cardData.expiryDate = cardData.expiryDate.replace(/\D/g, '').replace(/(\d{2})(\d{0,2})/, '$1/$2')">
            </div>

            <div class="form-group">
               <label>CVV  </label>
               <input v-model="cardData.cvv" type="password" placeholder="123" maxlength="3"
                  @input="cardData.cvv = cardData.cvv.replace(/\D/g, '')">
            </div>

            <div class="modal-actions">
               <button @click="showCardModal = false" class="btn cancel">Отмена</button>
               <button @click="addFunds" class="btn">Пополнить</button>
            </div>
         </div>
      </div>

   </div>
</template>

<style scoped>
.gridd {
   display: grid;
   grid-template-columns: repeat(4, 1fr);
   gap: 50px;
}

.square {
   width: 270px;
   height: auto;
   color: black;
   background: linear-gradient(to bottom left, rgba(183, 45, 45, 0.5), rgb(250, 167, 181, 0.5));
   border-radius: 25%;
   padding: 20px;
   margin-top: 50px;
}

.create {
   width: 270px;
   height: auto;
   color: black;
   background: linear-gradient(to bottom left, rgb(183, 45, 45, 0.5), rgb(250, 167, 181, 0.5));
   border-radius: 25%;
   padding: 20px;
   margin-top: 50px;
}

.form-group {
   width: 100%;
   /* Занимаем всю доступную ширину контейнера */
   margin-bottom: 10px;
}

.label {
   text-align: center;
   /* Центрируем label */
   margin-bottom: 5px;
}

.width {
   width: 60%;
   /* Ширина input теперь 100% form-group */
   height: 25px;
   padding: 8px;
   box-sizing: border-box;
   /* Важно, чтобы padding не увеличивал ширину */
}

.error {
   text-align: center;
   /* Центрируем текст ошибки */
   color: red;
   font-size: 1em;
   margin-top: 3px;
}

input[type=text] {
   background-color: #f1bf7e;
   border-radius: 20px 20px 20px 20px;
   border-color: #831010;
   color: #954b1e;
}

input[type=text]:focus {
   background-color: rgb(189, 114, 102);
   color: #832510;
}

input[type=number] {
   background-color: #f1bf7e;
   border-radius: 20px 20px 20px 20px;
   border-color: #831010;
   color: #954b1e;
}

input[type=number]:focus {
   background-color: rgb(189, 114, 102);
   color: #832510;
}

input[type=date] {
   background-color: #f1bf7e;
   border-radius: 20px 20px 20px 20px;
   border-color: #831010;
   color: #954b1e;
}

input[type=date]:focus {
   background-color: rgb(189, 114, 102);
   color: #832510;
}

input[type="password"] {
   background-color: #f1bf7e;
   border-radius: 20px 20px 20px 20px;
   border-color: #831010;
   color: #954b1e;
}

input[type="password"]:focus {
   background-color: rgb(189, 114, 102);
   color: #832510;
}

.card-preview {
   background: linear-gradient(135deg, #3a4a6b, #1e2b4d);
   color: white;
   padding: 15px;
   border-radius: 10px;
   margin-bottom: 15px;
   font-family: 'Courier New', monospace;
}

textarea {
   background-color: #f1bf7e;
   border-radius: 20px 20px 20px 20px;
   border-color: #831010;
   color: #954b1e;
   padding: 5%;
}

textarea:focus {
   background-color: rgb(189, 114, 102);
   color: #832510;
}

.balance-display {
   font-size: 2rem;
   font-weight: bold;
   text-align: center;
   margin: 20px 0;
   color: #2c3e50;
}

.add-funds {
   margin-top: 20px;
   padding: 15px;
   background: rgba(255, 255, 255, 0.2);
   border-radius: 10px;
}

.funds-input {
   width: 100%;
   padding: 8px;
   margin: 10px 0;
   border-radius: 5px;
   border: 1px solid #ccc;
}

.modal-overlay {
  position: fixed;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background-color: rgba(0, 0, 0, 0.5);
  display: flex;
  justify-content: center;
  align-items: center;
  z-index: 1000;
}

.modal-content {
  background-color: white;
  padding: 20px;
  border-radius: 10px;
  width: 400px;
  max-width: 90%;
}

.modal-actions {
  display: flex;
  justify-content: space-between;
  margin-top: 20px;
}

.btn.cancel {
  background-color: #f44336;
}

.balance-display {
  font-size: 2rem;
  font-weight: bold;
  text-align: center;
  margin: 20px 0;
  color: #2c3e50;
}

.btn {
   width: 150px;
   height: auto;
   background-color: #e3bebe83;
   color: #601818;
}

.btn:hover {
   background-color: #771b1be7;
   color: white;
}
</style>
