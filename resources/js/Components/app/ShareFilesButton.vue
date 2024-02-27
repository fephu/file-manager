<template>
    <button
        @click="onClick"
        class="mr-2 inline-flex items-center px-4 py-2 text-sm font-medium text-gray-900 bg-white border border-gray-200 rounded-lg hover:bg-gray-100 hover:text-blue-700 focus:z-10 focus:ring-2 focus:ring-blue-700 focus:text-blue-700 dark:bg-gray-700 dark:border-gray-600 dark:text-white dark:hover:text-white dark:hover:bg-gray-600 dark:focus:ring-blue-500 dark:focus:text-white"
    >
        <ShareIcon class="w-6 h-6 mr-2"/>
        Share
    </button>

    <ShareFilesModal v-model="showSharingModal" :all-selected="allSelected" :selected-ids="selectedIds"/>
</template>

<script setup>
import { ShareIcon } from "@heroicons/vue/24/outline";
import ConfirmationDialog from "@/Components/ConfirmationDialog.vue";
import {ref} from "vue";
import {useForm, usePage} from "@inertiajs/vue3";
import {showErrorDialog, showSuccessNotification} from "@/event-bus.js";
import ShareFilesModal from "@/Components/app/ShareFilesModal.vue";

const page = usePage();
const form = useForm({
    all: null,
    ids: [],
    parent_id: null
})

const showSharingModal = ref(false);

const props = defineProps({
    allSelected: {
        type: Boolean,
        required: false,
        default: false,
    },
    selectedIds: {
        type: Array,
        required: false,
    }
});
const emit = defineEmits(['restore'])

function onClick() {
    if (!props.allSelected && !props.selectedIds.length) {
        showErrorDialog('Please select at least one file to share!');
    } else {
        showSharingModal.value = true;
    }
}
</script>

