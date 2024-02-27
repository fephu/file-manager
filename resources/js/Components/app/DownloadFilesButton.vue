<template>
    <PrimaryButton @click="download">
        <ArrowDownTrayIcon class="w-6 h-6 mr-2"/>
        Download
    </PrimaryButton>
</template>

<script setup>
import PrimaryButton from "@/Components/PrimaryButton.vue";
import {ArrowDownTrayIcon} from "@heroicons/vue/24/outline/index.js";
import {usePage} from "@inertiajs/vue3";
import {httpGet} from "@/Helper/http-helper.js";

const page = usePage();

const props = defineProps({
    all: {
        type: Boolean,
        required: false,
        default: false,
    },
    ids: {
        type: Array,
        required: false,
    },
    sharedWithMe: false,
    sharedByMe: false,
});

function download() {
    if (!props.all && props.ids.length === 0) {
        return;
    }

    const p = new URLSearchParams();
    if (page.props.folder?.id) {
        p.append('parent_id', page.props.folder?.id);
    }
    if (props.all) {
        p.append('all', props.all ? '1' : '0');
    } else {
        for (let id of props.ids) {
            p.append('ids[]', id);
        }
    }

    let url = route('file.download');
    if (props.sharedWithMe) {
        url = route('file.downloadSharedWithMe');
    } else if (props.sharedByMe) {
        url = route('file.downloadSharedByMe');
    }
    httpGet(url+'?'+p.toString())
        .then(res => {
            if (!res.url) {
                return;
            }
            const a = document.createElement('a');
            a.download = res.filename;
            a.href = res.url;
            a.click();
        })
}
</script>
