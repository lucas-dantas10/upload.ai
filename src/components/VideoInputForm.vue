<script setup lang="ts">
import { FileVideo } from 'lucide-vue-next';
import { Upload } from 'lucide-vue-next';
import { Textarea } from '../components/ui/textarea';
import { Button } from '../components/ui/button';
import { Label } from '../components/ui/label';
import {  ref, watch } from 'vue';

const videoFile = ref<File | null>(null);
const promptsInputRef = ref<string>('');
const previewUrl = ref<string>('');

function handleFileSelect(event: Event) {
    const  { files } = event.currentTarget as HTMLInputElement;
    
    if (!files) {
        return;
    }

    const selectedFiles = files[0];

    videoFile.value = selectedFiles;
}

function handleUploadVideo() {
    
    if (!videoFile.value) {
        return;
    }

    //converter video em audio
}

watch(videoFile, async(newVideoFile) => {
    if (!newVideoFile) {
        previewUrl.value = '';
        return;
    }

    previewUrl.value = URL.createObjectURL(newVideoFile);
})
</script>

<template>
    <form @submit.prevent="handleUploadVideo" class="space-y-4">
        <label
            for="video"
            class="border flex rounded-md aspect-video cursor-pointer border-dashed text-sm flex-col gap-2 justify-center items-center text-muted-foreground hover:bg-primary/5"
        >
            <div v-if="videoFile">
                <video :src="previewUrl" :controls="false" class="pointer-events-none"></video>
            </div>

            <div class="flex justify-center items-center" v-else>
                <FileVideo class="w-4 h-4 mr-2" />
                Selecione um vídeo
            </div>
            
        </label>

        <input @change="handleFileSelect" type="file" id="video" accept="video/mp4" class="sr-only" />

        <Separator />

        <div class="space-y-2">
            <Label for="transcription_prompt"> Prompt de transcrição </Label>

            <Textarea
                v-model.trim="promptsInputRef"
                id="transcription_prompt"
                class="h-20 leading-relaxed resize-none"
                placeholder="Inclua palavras chaves mencionadas no vídeo separadas por vírgula {,}"
            />
        </div>

        <Button type="submit" class="w-full">
            Carregar vídeo
            <Upload class="w-4 h-4 ml-2" />
        </Button>
    </form>
</template>
