<script setup lang="ts">


import { Mistral } from '@mistralai/mistralai';
import { mistral } from "~/config/env.json"
import { generation } from "~/prompts/generation.json"

const client = new Mistral({apiKey:mistral.MISTRAL_API_KEY})


type Message = {
	role: "system" | "user" | "assistant";
	content: string
}

const messages = ref<Message[]>([])
const emit = defineEmits(["send"])
const playground = usePlaygroundStore()
const handleSend = async (content:string)=>{
  messages.value.push({role:"user", content})
	const response = await client.chat.stream({model:"codestral-latest",messages:[{role:"system",content:generation.prompt},...messages.value]})
	messages.value.push({role:"assistant",content:""})
	if (playground.fileSelected) {
			if (playground.fileSelected.read()){
        playground.fileSelected.write("")
      }	
	}
	for await (const chunk of response) {
		const output = chunk.data.choices[0].delta.content
    if (!output || (typeof output) !== 'string') continue
		if (output.includes('```')) output.replace('```','')
		if (output.includes('```vue')) output.replace('```vue','')
		emit("send", output)
		console.log(output)
		messages.value[messages.value.length-1].content += output
		if (playground.fileSelected) {
      playground.fileSelected.write(playground.fileSelected.read() + output)
    }
}
}
</script>
<template>
<div>
<ChatInputContent @send="handleSend" />
<ChatMessageList :messages="messages" />
</div>
</template>
<style scoped>

</style>