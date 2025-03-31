<script setup lang="ts">
const content = ref("")
defineEmits(["send"])
</script>
<template>
<div class="flex flex-row items-center mx-auto w-full">
<textarea
class="w-full p-3 shadow-md shadow-green-300" placeholder="Message..."
 :rows="3" v-model="content" @keydown.enter="$emit('send',content);content=''"></textarea>

</div>
</template>