<template>
    <span class="w-5 h-5 inline-flex items-center justify-center mr-5">
        <FolderIcon v-if="file.is_folder" />
        <template v-else>
            <img class="max-w-full" v-if="isImage(file)" :src="`/images/icons/image.png`" alt=""/>
            <img class="max-w-full" v-else-if="isPDF(file)" :src="`/images/icons/pdf.png`" alt="">
            <img class="max-w-full" v-else-if="isAudio(file)" :src="`/images/icons/audio.png`" alt="">
            <img class="max-w-full" v-else-if="isVideo(file)" :src="`/images/icons/video.png`" alt="">
            <img class="max-w-full" v-else-if="isWord(file)" :src="`/images/icons/word.png`" alt="">
            <img class="max-w-full" v-else-if="isExcel(file)" :src="`/images/icons/excel.png`" alt="">
            <img class="max-w-full" v-else-if="isZip(file)" :src="`/images/icons/zip.png`" alt="">
            <img class="max-w-full" v-else-if="isText(file)" :src="`/images/icons/txt-file.png`" alt="">
            <img class="max-w-full" v-else :src="`/images/icons/attach-file.png`" alt="">
        </template>
    </span>
</template>

<script setup>
import { FolderIcon } from "@heroicons/vue/24/outline";
import {isAudio, isExcel, isImage, isPDF, isText, isVideo, isWord, isZip} from "@/Helper/file-helper.js";

const {file} = defineProps({
    file: Object,
});

</script>

