<script setup lang="ts">
type Message = {
  role: "user" | "system" | "assistant";
  content: string;
}
defineProps<{messages:Message[]}>()




</script>

<template>
  <div v-for="message in messages" class="p-4">
    <div class="flex flex-col w-full bg-gray-500 rounded-md items-center justify-center px-8" :class="message.role === 'user' ? 'text-right' : 'text-left'">
      <!-- Badge indicating role -->
      <div class="m-2" :class="message.role === 'user' ? 'text-right' : 'text-left'">
        <span 
          class="px-3 py-1 text-sm font-semibold rounded-full" 
          :class="message.role === 'user' ? 'bg-blue-500 text-white' : message.role === 'assistant' ? 'bg-green-500 text-white' : 'bg-gray-700 text-white'"
        >
          {{ message.role.charAt(0).toUpperCase() + message.role.slice(1) }}
        </span>
      </div>
      
      <ChatMarkdownContent :content="message.content" />
    </div>
  </div>
</template>