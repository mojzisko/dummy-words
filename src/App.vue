<template>
  <div id="app">
    <button :disabled="isDeletingComputed" @click="isAddWordModalOpen = true">Přidat fajné slovíčko</button>
    <word-list :words="words" @word-removed="handleWordRemoved" @deleting-status-changed="handleDeletingStatusChange" @update-words="handleWordsUpdate"></word-list>
    <add-word-modal v-if="isAddWordModalOpen" @close-modal="isAddWordModalOpen = false" @word-added="handleWordAdded"></add-word-modal>
  </div>
</template>

<script lang="ts">
import { defineComponent, onMounted, computed, ref, Ref } from 'vue'; 
import WordList from './components/WordList.vue';
import AddWordModal from './components/AddWordModal.vue';
import { nextTick } from 'vue';

// This app took me probably 5 hours, because I wanted to sort out all the problems that I faced. 
// I had few issues with Vue npm packages such as VueDraggable and also with infinite scroll for better app performance. 

export default defineComponent({
  name: 'App',
  components: {
    WordList,
    AddWordModal,
  },
  setup() {
    const words: Ref<string[]> = ref<string[]>([]);
    const isAddWordModalOpen: Ref<boolean> = ref(false);
    const isDeleting: Ref<boolean> = ref(false);
    
    const handleDeletingStatusChange = (status: boolean) => {
      console.log('ahoj', status)
      isDeleting.value = status;
    };

    const isDeletingComputed = computed(() => isDeleting.value);

    const generateDummyWords = (count: number): string[] => {
      const wordsArray: string[] = [];
      for (let i = 0; i < count; i++) {
        wordsArray.push(`fajné_slovíčko-${i}`);
      }
      return wordsArray;
    };

    const fetchWords = (): void => { 
      const savedWords = localStorage.getItem('wordListState');

      if (savedWords) {
        words.value = JSON.parse(savedWords);
      } else {
        words.value = generateDummyWords(10000); // Generate 10,000 dummy cool words
        localStorage.setItem('wordListState', JSON.stringify(words.value));
      }
    };

    const handleWordsUpdate = (newWords: string[]): void => {
      words.value = newWords;
      localStorage.setItem('wordListState', JSON.stringify(newWords));
    };

    const handleWordAdded = (newWord: string): void => {
      if (newWord.trim()) {
        // Add the new word at the beginning of the list because there are 10k words and we want to see new ones
        words.value = [newWord, ...words.value];
        localStorage.setItem('wordListState', JSON.stringify(words.value));
        nextTick(() => {
          isAddWordModalOpen.value = false; 
        });
      }
    };

    const handleWordRemoved = (index: number): void => { 
      // Remove the word from the original array based on the index
      words.value.splice(index, 1);
      localStorage.setItem('wordListState', JSON.stringify(words.value));
    };

    onMounted(fetchWords);

    return {
      words,
      isAddWordModalOpen,
      handleWordsUpdate,
      handleWordAdded,
      handleWordRemoved,
      isDeletingComputed,
      handleDeletingStatusChange,
    };
  },
});
</script>