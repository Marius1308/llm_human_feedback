<script setup lang="ts">
defineProps<{
  currentChat: string[];
  extended?: boolean;
}>();

defineEmits(["extend"]);
</script>

<template>
  <div
    class="chat"
    :data-large="extended ? true : undefined"
    @click="$emit('extend')"
  >
    <div class="title">Chat:</div>
    <div
      class="chat-message"
      v-for="(message, index) in currentChat"
      :key="index"
    >
      {{ message }}
    </div>
  </div>
</template>

<style scoped>
.chat {
  background-color: #f0f0f0;
  padding: 20px;
  padding-top: 0;
  font-size: 16px;
  height: 20%;
  overflow: auto;

  &[data-large] {
    flex: 1;
  }
}

.title {
  padding-top: 20px;
  background-color: #f0f0f0;
  position: --webkit-sticky; /* Safari */
  position: sticky;
  top: 0;
  font-weight: bold;
  padding-bottom: 10px;
  margin-bottom: 10px;
  border-bottom: 1px solid #d0d0d0;
}

.chat-message {
  margin-bottom: 10px;
}
</style>
