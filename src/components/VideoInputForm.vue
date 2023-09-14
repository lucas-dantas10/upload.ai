<script setup lang="ts">
import { FileVideo } from 'lucide-vue-next';
import { Upload } from 'lucide-vue-next';
import { Textarea } from '@/components/ui/textarea';
import { Separator } from '../components/ui/separator';
import { Button } from '../components/ui/button';
import { Label } from '../components/ui/label';
import { ref, watch } from 'vue';
import { getFFmpeg } from '@/lib/ffmpeg';
import { fetchFile } from '@ffmpeg/util';
import { api } from '@/lib/axios';

type Status = 'waiting' | 'converting' | 'uploading' | 'generating' | 'success';

const statusMessages = {
    converting: 'Convertendo...',
    generating: 'Transcrevendo...',
    uploading: 'Carregando...',
    success: 'Sucesso'
};

const videoFile = ref<File | null>(null);
const promptsInputRef = ref<string>('');
const previewUrl = ref<string>('');
const status = ref<Status>('waiting');

const emit = defineEmits([
    'onVideoUploaded'
]);

function handleFileSelect(event: Event) {
    const  { files } = event.currentTarget as HTMLInputElement;
    
    if (!files) {
        return;
    }

    const selectedFiles = files[0];

    videoFile.value = selectedFiles;
}

async function convertVideoToAudio(video: File) {
    // debugger;
    console.log('Convert Started')

    const ffmpeg = await getFFmpeg();

    await ffmpeg?.writeFile('input.mp4', await fetchFile(video));

    // ffmpeg?.on('log', log => {
    //     console.log(log);
    // })

    ffmpeg?.on('progress', progress => {
        console.log('Convert progress: ' + Math.round(progress.progress * 100));
    });

    await ffmpeg?.exec([
        '-i',
        'input.mp4',
        '-map',
        '0:a',
        '-b:0',
        '20k',
        '-acodec',
        'libmp3lame',
        'output.mp3'
    ]);

    const data = await ffmpeg?.readFile('output.mp3');

    const audioFileBlob = new Blob([data], { type: 'audio/mpeg' });
    const audioFile = new File([audioFileBlob], 'audio.mp3', 
    { 
        type: 'audio/mpeg'
     });

     console.log('Convert Finished');

     return audioFile;
}   

async function handleUploadVideo() {
    if (!videoFile.value) {
        return;
    }

    status.value = 'converting';

    const audioFile = await convertVideoToAudio(videoFile.value);

    const data = new FormData();

    data.append('file', audioFile);

    status.value = 'uploading';
    
    const response = await api.post('/videos', data)

    const videoId = response.data.video.id;

    status.value = 'generating';

    await api.post(`/videos/${videoId}/transcription`, {
        prompt: promptsInputRef.value
    });

    status.value = 'success';

    emit('onVideoUploaded', videoId);
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
    <form @submit.prevent="handleUploadVideo()" class="space-y-4">
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
                :disabled="status !== 'waiting'"
                id="transcription_prompt"
                class="h-20 leading-relaxed resize-none"
                placeholder="Inclua palavras chaves mencionadas no vídeo separadas por vírgula {,}"
            />
        </div>

        <Button 
            :data-success="status == 'success'"
            :disabled="status !== 'waiting'" 
            type="submit" 
            class="w-full data-[success=true]:bg-emerald-400"
        >
            
            {{ status == 'waiting' ? 'Carregar vídeo': statusMessages[status] }}
            <Upload class="w-4 h-4 ml-2" />
        </Button>
    </form>
</template>
