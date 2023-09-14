<script setup lang="ts">
import { Button } from './components/ui/button';
import { Label } from './components/ui/label';
import { Slider } from './components/ui/slider';
import { Separator } from './components/ui/separator';
import { Textarea } from './components/ui/textarea';
import { Github, Wand2 } from 'lucide-vue-next';
import { Select, SelectContent, SelectItem, SelectTrigger, SelectGroup, SelectValue } from './components/ui/select';
import VideoInputForm from './components/VideoInputForm.vue';
import PromptSelect from './components/PromptSelect.vue';
import { useCompletion } from 'ai/vue';
import {  ref } from 'vue';

const temperature = ref([0.5]);
const videoId = ref('');
const {
    input,
    completion,
    handleSubmit,
    isLoading,
} = useCompletion({
  api: 'http://localhost:3333/ai/complete',
  body: {
    videoId: videoId.value,
    temperature: temperature.value[0],
  },
  headers: {
    'Content-Type': 'application/json',
  }
});


function handlePromptsSelected(template: string) {
    input.value = template;
}

function setVideoId(idVideo: string) {
    videoId.value = idVideo;
}
</script>

<template>
    <div class="min-h-screen flex flex-col">
        <header class="px-4 py-3 flex items-center justify-between border-b">
            <h1 class="text-xl font-bold">upload.io</h1>

            <div class="flex items-center gap-3">
                <span class="text-sm text-muted-foreground">
                    Desenvolvido com 💗 no NLW
                </span>

                <Separator orientation="vertical" class="h-6" />

                <Button variant="outline">
                    <Github class="w-4 h-4 mr-2" />
                    Github
                </Button>
            </div>
        </header>

        <main class="flex-1 p-6 flex gap-6">
            <section class="flex flex-col flex-1 gap-4">
                <div class="grid grid-rows-2 gap-4 flex-1">
                    <Textarea 
                        class="resize-none p-4 leading-relaxed"
                        placeholder="Inclua o prompt para a IA..." 
                        v-model="input"
                    /> 
                    <Textarea 
                        class="resize-none p-4 leading-relaxed"
                        placeholder="Resultado gerado pela IA..." 
                        readonly
                        v-model="completion"
                    />
                </div>

                <p class="text-sm text-muted-foreground">
                    Lembre-se: você pode utilizar a variável <code class="text-violet-400">{transcription}</code> no seu prompt para adicionar o conteúdo da transcrição do vídeo selecionado.
                </p>
            </section>

            <aside class="w-80 space-y-6">
                <VideoInputForm @onVideoUploaded="setVideoId" />

                <Separator />

                <form @submit="handleSubmit" class="space-y-4">
                    <div class="space-y-2">
                        <Label>Prompt</Label>
                        <PromptSelect 
                            @onPromptSelected="handlePromptsSelected"
                        />
                    </div>

                    <div class="space-y-2">
                        <Label>Modelo</Label>
                        <Select disabled>
                            <SelectTrigger>
                                <SelectValue placeholder="GPT 3.5-turbo 16k" />
                            </SelectTrigger>
                            <SelectContent>
                                <SelectGroup>
                                    <SelectItem value="gpt3.5">
                                        GPT 3.5-turbo 16k
                                    </SelectItem>
                                </SelectGroup>
                                
                            </SelectContent>
                        </Select>
                        <span class="block text-sm text-muted-foreground">Você poderá customizar essa opção em preve</span>
                    </div>

                    <Separator /> 

                    <div class="space-y-4">
                        <Label>Temperatura</Label>

                        <Slider 
                            :min="0"
                            :max="1"
                            :step="0.1"
                            v-model="temperature"
                        />

                        <span class="block text-sm text-muted-foreground leading-relaxed">Valores mais altos tendem a deixar os resultados mais criativos e com possíveis erros</span>
                    </div>

                    <Separator />

                    <Button :disabled="isLoading" type="submit" class="w-full">
                        Executar
                        <Wand2 class="w-4 h-4 ml-2" />
                    </Button>
                </form>
            </aside>
        </main>
    </div>
</template>
