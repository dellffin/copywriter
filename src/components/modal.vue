<script setup>
import { ref } from "vue";

const props = defineProps({
  moduleText: String
});

const emit = defineEmits(['modulYes', 'modulClose']);

const log = ref(false);

function ok() {
  emit('modulYes', log.value);
}

function close() {
  emit('modulClose');
}
</script>

<template>
  <div class="container">
    <div class="modal_main">
      <div class="back_main">
        <slot name="default">
          {{ moduleText }}<br>
        </slot>
        
        <div class="modal-actions" v-if="$slots.actions">
          <slot name="actions"></slot>
        </div>
        <button v-else @click="ok" class="btn">Ясно</button>
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
  height: auto;
  min-height: 200px;
  max-height: 90vh;
  z-index: 1000;
  display: flex;
  justify-content: center;
  align-items: center;
  padding: 20px;
  box-sizing: border-box;
}

.back_main {
  position: relative;
  padding: 20px;
  color: black;
  text-align: center;
  word-wrap: break-word;
  overflow-y: auto;
  max-height: calc(100% - 60px);
  width: 100%;
}

.btn {
  margin-top: 20px;
  background-color: rgb(218, 172, 140);
  border-color: rgb(44, 0, 0);
  padding: 8px 16px;
  border-radius: 4px;
  cursor: pointer;
}

.btn:hover {
  background-color: rgb(218, 134, 78);
}

.modal-actions {
  display: flex;
  justify-content: center;
  gap: 10px;
  margin-top: 20px;
}
</style>