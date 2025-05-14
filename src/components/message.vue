<script setup>
import { ref, onMounted, reactive, computed } from 'vue';
import { OverlayScrollbars } from 'overlayscrollbars';
import 'overlayscrollbars/styles/overlayscrollbars.css'; // Импортируем CSS

const props = defineProps({
    interlocutor: String,
    messages: Array,
    currentUser: String // Добавьте этот пропс
});
const emit = defineEmits(['modulClose', 'sendMessage'])
const fileInput = ref(null);
const selectedFile = ref(null);
const messageText = ref("")
const log = ref(false);
const messagesContainer = ref(null);

// Добавьте эти функции в начало компонента
const formatFileSize = (bytes) => {
  if (bytes === 0) return '0 Bytes';
  const k = 1024;
  const sizes = ['Bytes', 'KB', 'MB'];
  const i = Math.floor(Math.log(bytes) / Math.log(k));
  return parseFloat((bytes / Math.pow(k, i)).toFixed(2)) + ' ' + sizes[i];
};

const isTextFile = (type) => {
  return type?.startsWith('text/') || 
         ['application/json'].includes(type);
};

const decodeFileContent = (base64) => {
  try {
    const binaryString = atob(base64);
    return decodeURIComponent(escape(binaryString));
  } catch {
    return '[Содержимое не может быть отображено]';
  }
};

const dialogTitle = computed(() => {
    return `Переписка с ${props.interlocutor}`;
});

const sortedMessages = computed(() => {
    if (!props.messages) return [];
    return [...props.messages].sort((a, b) => {
        return new Date(a.timestamp) - new Date(b.timestamp);
    });
});

onMounted(() => {
    OverlayScrollbars(messagesContainer.value, {
        // Настройки библиотеки (по желанию)
        overflowBehavior: {
            x: "hidden",
            y: "scroll"
        },
        scrollbars: {
            visibility: "hidden",  // Hide the scrollbar
            autoHide: "move",       // Hide after a specific amount of time
            autoHideDelay: 800
        }
    });
});

function send() {
    if (!props.interlocutor || props.interlocutor === props.currentUser) {
        console.error('Нельзя отправить сообщение самому себе');
        return;
    }

    if (messageText.value.trim()) {
        emit('sendMessage', {
            author: props.interlocutor,
            message: messageText.value
        });
        messageText.value = "";
    }
}

function handleFileSelect(event) {
    selectedFile.value = event.target.files[0];
}

async function sendFile() {
    if (!selectedFile.value || !props.interlocutor) return;

    try {
        // Читаем файл как ArrayBuffer
        const arrayBuffer = await selectedFile.value.arrayBuffer();
        // Конвертируем в Base64
        const base64String = btoa(
            new Uint8Array(arrayBuffer).reduce(
                (data, byte) => data + String.fromCharCode(byte), ''
            )
        );

        emit('sendMessage', {
            author: props.interlocutor,
            message: `[Файл: ${selectedFile.value.name}]`,
            file: {
                name: selectedFile.value.name,
                type: selectedFile.value.type || 'text/plain',
                size: selectedFile.value.size,
                data: base64String // Отправляем чистый Base64
            }
        });

        selectedFile.value = null;
        fileInput.value.value = '';
    } catch (error) {
        console.error('Ошибка отправки файла:', error);
        alert('Не удалось отправить файл');
    }
}

function readFileAsDataURL(file) {
    return new Promise((resolve, reject) => {
        const reader = new FileReader();
        reader.onload = e => resolve(e.target.result);
        reader.onerror = reject;
        reader.readAsDataURL(file);
    });
}

function downloadFile(file) {
    try {
        // Декодируем Base64
        const binaryString = atob(file.data);
        const bytes = new Uint8Array(binaryString.length);

        for (let i = 0; i < binaryString.length; i++) {
            bytes[i] = binaryString.charCodeAt(i);
        }

        const blob = new Blob([bytes], { type: file.type });
        const url = URL.createObjectURL(blob);

        const a = document.createElement('a');
        a.href = url;
        a.download = file.name || 'file.txt';
        document.body.appendChild(a);
        a.click();

        setTimeout(() => {
            document.body.removeChild(a);
            URL.revokeObjectURL(url);
        }, 100);

    } catch (error) {
        console.error('Ошибка скачивания:', error);
        alert('Не удалось скачать файл');
    }
}
</script>
<template>
    <div class="container">
        <div class="modal_main">
            <div class="back_main">
                <div class="square">
                    <div class="messenger-container">
                        {{ dialogTitle }}
                        <div class="messages" ref="messagesContainer">
                            <div v-for="(message, index) in sortedMessages" :key="index" :class="['message', {
                                'outgoing': message.sender === currentUser,
                                'incoming': message.sender !== currentUser
                            }]">
                                <div class="message-sender">{{ message.sender }}</div>
                                <div class="message-text">{{ message.text }}</div>

                                <!-- Блок для отображения файла -->
                                <div v-if="message.file" class="message-file">
                                    <strong>Файл:</strong> {{ message.file.name }}
                                    <a href="#" @click.prevent="downloadFile(message.file)" class="file-link">
                                        Скачать ({{ formatFileSize(message.file.size) }})
                                    </a>
                                    <div v-if="isTextFile(message.file.type)" class="file-preview">
                                        <pre>{{ decodeFileContent(message.file.data) }}</pre>
                                    </div>
                                </div>

                                <div class="message-time">{{ message.timestamp }}</div>
                            </div>
                        </div>

                        <div class="input-area">
                            <input type="text" v-model="messageText" @keyup.enter="send">
                            <button @click="send">Отправить</button>
                            <div class="file-upload">
                                <input type="file" ref="fileInput" accept=".txt,text/plain" @change="handleFileSelect"
                                    style="display: none">
                                <button @click="fileInput.click()"> Прикрепить файл </button>
                                <span v-if="selectedFile">{{ selectedFile.name }}</span>
                                <button v-if="selectedFile" @click="sendFile" class="send-file-btn">
                                    Отправить файл
                                </button>
                            </div>
                        </div>
                    </div>
                </div>
                <button @click="$emit('modulClose')" class="btn">Закрыть</button>
            </div>
        </div>
    </div>
</template>

<style scoped>
.container {
    position: fixed;
    top: 0;
    left: 0;
    min-width: 100vw;
    min-height: 100vh;
    display: flex;
    align-items: center;
    justify-content: center;
    background-color: rgba(255, 240, 240, 0.5);
    z-index: 998;
}

.modal_main {
    background-color: rgb(211, 156, 112);
    border-radius: 25%;
    width: 600px;
    height: 700px;
    z-index: 1000;
    display: flex;
    justify-content: center;
    align-items: center;
}

.back_main {
    position: relative;
    padding: 20px;
    color: white;
    text-align: center;
    word-wrap: break-word;
    overflow-y: auto;
    max-height: calc(100% - 60px);
    /* Учли размер кнопки */
}

.square {
    width: 500px;
    height: 450px;
    color: black;
    border-radius: 25%;
    padding: 20px;
    margin-top: 10px;
}

.btn {
    margin-top: 40px;
    background-color: rgb(218, 172, 140);
    border-color: rgb(44, 0, 0);
}

.btn:hover {
    background-color: rgb(218, 134, 78);
}

/* Общий контейнер мессенджера */
.messenger-container {
    width: 500px;
    margin-bottom: 2px;
    border: 1px solid #ffffff;
    border-radius: 5px;
    /* overflow: hidden;  */
}

/* Контейнер для сообщений */
.messages {
    padding: 10px;
    height: 350px;
    /* Фиксированная высота */
}

.messages::-webkit-scrollbar {
    width: 0px;
    /* Ширина полосы прокрутки */
    background: transparent;
    /*  Цвет фона полосы прокрутки */
}

.messages::-webkit-scrollbar-thumb {
    background: transparent;
    /* Цвет "ползунка" */
}

/* Базовые стили для сообщений */
.message {
    margin-bottom: 10px;
    padding: 8px 12px;
    border-radius: 5px;
    clear: both;
    word-wrap: break-word;
    max-width: 70%;
}

/* Входящие сообщения */
.incoming {
    float: left;
    background-color: #f1f1f1;
    border-bottom-left-radius: 5px;
    margin-right: auto;
}

/* Исходящие сообщения */
.outgoing {
    float: right;
    background-color: #e7ab8b;
    border-bottom-right-radius: 5px;
    margin-left: auto;
}

.messages::after {
    content: "";
    display: table;
    clear: both;
}

/* Стили для поля ввода */
.input-area {
    padding: 10px;
    border-top: 1px solid #ccc;
    display: flex;
    /* Используем flexbox для выравнивания */
}

.input-area input[type="text"] {
    flex-grow: 1;
    background-color: #b17864;
    padding: 8px;
    border: 1px solid #ccc;
    border-radius: 5px;
    margin-right: 10px;
}

.input-area button {
    padding: 8px 12px;
    background-color: #ff4000;
    color: white;
    border: none;
    border-radius: 5px;
    cursor: pointer;
    margin-right: 5px;
}

.input-area button:hover {
    background-color: #b32a00;
}

.message-time {
    font-size: 0.8em;
    opacity: 0.7;
    margin-top: 4px;
}

.message-preview {
    cursor: pointer;
    padding: 8px;
    margin: 4px 0;
    background: rgba(255, 255, 255, 0.2);
    border-radius: 4px;
}

.message-preview:hover {
    background: rgba(255, 255, 255, 0.3);
}

.message-sender {
    font-size: 0.8em;
    opacity: 0.7;
    margin-bottom: 2px;
}

.outgoing .message-sender {
    text-align: right;
}

.incoming .message-sender {
    text-align: left;
}

.file-upload {
    display: flex;
    align-items: center;
    gap: 8px;
    margin-top: 10px;
}

.send-file-btn {
    padding: 4px 8px;
    background-color: #4CAF50;
    color: white;
    border: none;
    border-radius: 4px;
    cursor: pointer;
}

.send-file-btn:hover {
    background-color: #45a049;
}

.message-file {
    margin-top: 5px;
    padding: 8px;
    background-color: #f0f0f0;
    border-radius: 4px;
    word-break: break-all;
}

.file-link {
    color: #2196F3;
    text-decoration: none;
}
</style>
