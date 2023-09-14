<script setup lang="ts">
import {
    Select,
    SelectContent,
    SelectItem,
    SelectTrigger,
    SelectGroup,
    SelectValue,
} from "@/components/ui/select";
import { api } from "@/lib/axios";
import { onMounted, ref } from "vue";

interface Prompt {
    id: String,
    title: String,
    template: String
}

const prompts = ref<Prompt[] | string>('');
const emit = defineEmits([
    'onPromptSelected'
]);

onMounted(async () => {
    try {
        const responsePrompts = await api.get("/prompts");
        prompts.value = responsePrompts.data;
    } catch (err) {
        console.log(err);
    }
})

function handlePromptsSelected(promptId: string) {
    const selectedPrompt = prompts.value.find(prompt => prompt.id == promptId);

    if (!selectedPrompt) {
        return;
    }

    emit('onPromptSelected', selectedPrompt.template);
}
</script>

<template>
    <div>
        <Select @update:model-value="handlePromptsSelected">
            <SelectTrigger>
                <SelectValue placeholder="Selecione um prompt..." />
            </SelectTrigger>
            <SelectContent>
                <SelectGroup>
                    <SelectItem
                        :value="prompt.id"
                        v-for="(prompt, index) in prompts" 
                        :key="index"
                    >
                        {{ prompt.title }}
                    </SelectItem>
                </SelectGroup>
            </SelectContent>
        </Select>
    </div>
</template>
