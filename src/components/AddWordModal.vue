<template>
  <div class="modal-mask" @click.self="closeModal">
    <div class="modal-wrapper">
      <div class="modal-container">

        <div class="modal-header">
          <slot name="header">
            Add a new word
          </slot>
        </div>

        <div class="modal-body">
          <slot name="body">
            <input type="text" v-model="newWord" placeholder="Enter a new word" class="input-field" @keyup.enter="emitNewWord"/>
          </slot>
        </div>

        <div class="modal-footer">
          <slot name="footer">
            <button class="modal-default-button" @click="emitNewWord">
              Add Word
            </button>
          </slot>
        </div>

      </div>
    </div>
  </div>
</template>

<script lang="ts">
import { defineComponent, ref, Ref } from 'vue';

export default defineComponent({
  name: 'AddWordModal',
  setup(_, { emit }) {
    const newWord: Ref<string> = ref(''); 

    const emitNewWord = (): void => {
      if (newWord.value.trim()) {
        emit('word-added', newWord.value.trim()); 
        newWord.value = ''; 
        closeModal();
      }
    };

    const closeModal = (): void => { 
      emit('close-modal'); 
    };

    return {
      newWord,
      emitNewWord,
      closeModal,
    };
  },
});
</script>

<style>
.modal-mask {
  position: fixed;
  z-index: 9998;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background-color: rgba(0, 0, 0, .5);
  display: flex;
  justify-content: center;
  align-items: center;
}

.modal-wrapper {
  width: 300px;
}

.modal-container {
  background-color: #fff;
  border-radius: 5px;
  box-shadow: 0 2px 8px rgba(0, 0, 0, .33);
  transition: all .3s ease;
}

.modal-header {
  padding: 15px;
  border-bottom: 1px solid #e6e6e6;
  color: black;
}

.modal-body {
  padding: 20px;
}

.input-field {
  width: 100%;
  padding: 10px;
  margin-top: 5px;
  box-sizing: border-box;
  border-radius: 5px;
  border: 1px solid #d0d0d0;
}

.modal-footer {
  padding: 10px;
  border-top: 1px solid #e6e6e6;
  text-align: right;
}

.modal-default-button {
  background-color: #41b883;
  color: white;
  padding: 10px 15px;
  border: none;
  border-radius: 5px;
  cursor: pointer;
}

.modal-default-button:hover {
  background-color: #369671;
}
</style>
