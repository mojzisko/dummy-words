<template>
  <div class="drag-area" style="height: 400px; overflow-y: auto;" @scroll="handleScroll">
    <VueDraggableNext v-model="displayedWords" @end="handleDragEnd" :disabled="isButtonDisabled">
      <div
        v-for="(word, index) in displayedWords"
        :key="index"
        class="word-item"
        :class="{ 'is-dragging': isDragging }"
      >
        {{ word }}
        <button class="delete-button" @click="removeWord(index)" :disabled="isButtonDisabled">Smazat slovíčko</button>
      </div>
    </VueDraggableNext>
    <div v-if="loadingMore" class="loading-indicator">Loading more items...</div>
  </div>
</template>

<script lang="ts">
import { defineComponent, PropType, Ref, ref, computed, watch, onMounted, toRefs } from 'vue';
import { VueDraggableNext } from 'vue-draggable-next';

interface WordListProps {
  words: string[];
}

export default defineComponent({
  name: 'WordList',
  components: {
    VueDraggableNext,
  },
  props: {
    words: { type: Array as PropType<string[]>, required: true },
  },
  setup(props: WordListProps, { emit }) {
    const { words } = toRefs(props);
    const batchSize: number = 30;
    const isDragging: Ref<boolean> = ref(false);
    const loadingMore: Ref<boolean> = ref(false);
    const currentEndIndex: Ref<number> = ref(batchSize);
    const isDeleting: Ref<boolean> = ref(false);

    const displayedWords = computed<string[]>(() => words.value.slice(0, currentEndIndex.value));
    const hasMoreWords = computed<boolean>(() => currentEndIndex.value < words.value.length);
    const isButtonDisabled = computed<boolean>(() => isDeleting.value);

    watch(words, (newVal: string[] | undefined) => {
      if (newVal) {
        currentEndIndex.value = Math.min(newVal.length, currentEndIndex.value);
      }
    });

    watch(isButtonDisabled, (newVal: boolean) => {
      console.log('watched deleting', newVal)
      emit('deleting-status-changed', newVal);  // Notifying parent component about deleting state
    });

    onMounted(() => {
      if (words.value.length > 0) {
        currentEndIndex.value = Math.min(words.value.length, batchSize);
      }
    });

    const loadMoreWords = (): void => {
      if (hasMoreWords.value) {
        loadingMore.value = true;
        setTimeout(() => {
          const nextIndex = currentEndIndex.value + batchSize;
          currentEndIndex.value = nextIndex <= words.value.length ? nextIndex : words.value.length;
          loadingMore.value = false;
        }, 1000); 
      }
    };

    const handleScroll = (event: Event): void => {
      const target = event.target as HTMLElement;
      if (target.scrollHeight - target.scrollTop <= target.clientHeight && !loadingMore.value && hasMoreWords.value) {
        loadMoreWords();
      }
    };

    const removeWord = (index: number): void => {
      isDeleting.value = true;
      // Fake delay to simulate server response time.
      setTimeout(() => {
        const actualIndex = index + (currentEndIndex.value - displayedWords.value.length);
        words.value.splice(actualIndex, 1);
        emit('update-words', words.value); // Notifying the parent component.
        isDeleting.value = false; 
      }, 1000);
    };

    const handleDragEnd = (event: { oldIndex: number; newIndex: number }): void => {
      isDragging.value = false;
      const { oldIndex, newIndex } = event;

      if (oldIndex !== newIndex) {
        const actualOldIndex = oldIndex + (currentEndIndex.value - displayedWords.value.length);
        const actualNewIndex = newIndex + (currentEndIndex.value - displayedWords.value.length);

        const movedItem = words.value.splice(actualOldIndex, 1)[0];
        words.value.splice(actualNewIndex, 0, movedItem);
        emit('update-words', words.value); 
      }
    };

    return {
      displayedWords,
      isDragging,
      handleDragEnd,
      handleScroll,
      loadingMore,
      removeWord,
      isButtonDisabled,
      hasMoreWords,
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
  border-bottom: none.
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