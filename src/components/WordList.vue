<template>
  <div class="drag-area" style="height: 400px; overflow-y: auto;" @scroll="handleScroll">
    <VueDraggableNext v-model="displayedWords" @end="handleDragEnd">
      <div
        v-for="(word, index) in displayedWords"
        :key="index"
        class="word-item"
        :class="{ 'is-dragging': isDragging }"
      >
        {{ word }}
        <button class="delete-button" @click="removeWord(index)">Smazat slovíčko</button>
      </div>
    </VueDraggableNext>
    <div v-if="loadingMore" class="loading-indicator">Loading more items...</div>
  </div>
</template>

<script lang="ts">
import { defineComponent, PropType, ref, watch, onMounted, toRefs, Ref } from 'vue';
import { VueDraggableNext } from 'vue-draggable-next';

export default defineComponent({
  name: 'WordList',
  components: {
    VueDraggableNext,
  },
  props: {
    words: { type: Array as PropType<string[]>, required: true },
  },
  setup(props, { emit }) {
    const { words } = toRefs(props);
    const batchSize: number = 30;
    const displayedWords: Ref<string[]> = ref<string[]>([]);
    const isDragging: Ref<boolean> = ref(false);
    const loadingMore: Ref<boolean> = ref(false);
    const currentEndIndex: Ref<number> = ref(batchSize);

    onMounted(() => {
      if (words.value) {
        displayedWords.value = words.value.slice(0, batchSize); 
      }
    });

    watch(words, (newVal: string[]) => {
      if (newVal) {
        displayedWords.value = newVal.slice(0, currentEndIndex.value);
      }
    });

    const loadMoreWords = (): void => {
      if (currentEndIndex.value < words.value.length) {
        const nextEndIndex: number = currentEndIndex.value + batchSize;
        displayedWords.value = [
          ...displayedWords.value,
          ...words.value.slice(currentEndIndex.value, nextEndIndex),
        ];
        currentEndIndex.value = nextEndIndex;
        loadingMore.value = false;
      }
    };

    const handleScroll = async (event: Event): Promise<void> => {
      const target = event.target as HTMLElement; 
      if (target.scrollHeight - target.scrollTop <= target.clientHeight && !loadingMore.value) {
        loadingMore.value = true;
        await new Promise(resolve => setTimeout(resolve, 1000));
        loadMoreWords();
      }
    };

    const removeWord = (index: number): void => {
      // Remove the word from the original array, not the displayed subset
      const actualIndex = index + (currentEndIndex.value - displayedWords.value.length);
      words.value.splice(actualIndex, 1);
      // Update the displayed words.
      displayedWords.value.splice(index, 1);
      emit('update-words', words.value);
    };

    const handleDragEnd = (event: any): void => {
      isDragging.value = false;
      // I had a problem with localStorage and infinitescroll loading with the actual words after a drag
      const oldIndex = event.oldIndex;
      const newIndex = event.newIndex;
      if (typeof oldIndex === 'number' && typeof newIndex === 'number' && oldIndex !== newIndex) {
        const actualOldIndex = oldIndex + (currentEndIndex.value - displayedWords.value.length);
        const actualNewIndex = newIndex + (currentEndIndex.value - displayedWords.value.length);

        // Adjust the original words array
        const [removed] = words.value.splice(actualOldIndex, 1);
        words.value.splice(actualNewIndex, 0, removed);
      }

      emit('update-words', words.value); 
    };

    return {
      displayedWords,
      isDragging,
      handleDragEnd,
      handleScroll,
      loadingMore,
      removeWord,
    };
  },
});
</script>


<style scoped>
.drag-area {
  position: relative;
  border: 1px solid #d9d9d9;
  border-radius: 5px;
  overflow: auto;
}

.word-item {
  padding: 10px;
  border-bottom: 1px solid #eee;
  background-color: #f9f9f9;
  cursor: grab;
  user-select: none;
  color: black;
}

.word-item:last-child {
  border-bottom: none;
}

.is-dragging {
  background-color: #e3e3e3;
}

.loading-indicator {
  text-align: center;
  padding: 10px 0;
  font-style: italic;
  color: #999;
}
</style>
