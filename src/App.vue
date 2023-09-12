<script setup lang="ts">
import { Button } from './components/ui/button';
import { Separator } from './components/ui/separator';
import { Github } from 'lucide-vue-next';
</script>

<template>
    <div>
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
    </div>
</template>
