<template>
    <AuthenticatedLayout>
        <nav class="flex items-center justify-between p-1 mb-3">
            <ol class="inline-flex items-center space-x-1 md:space-x-3">
                <li
                    v-for="ans of ancestors.data"
                    :key="ans.id"
                    class="inline-flex items-center"
                >
                    <Link
                        v-if="!ans.parent_id"
                        :href="route('myFiles')"
                        class="inline-flex items-center text-sm font-medium text-gray-700 hover:text-blue-600 dark:text-gray-400 dark:hover:text-white"
                    >
                        <HomeIcon class="w-6 h-6 mr-2" />
                        My Files
                    </Link>
                    <div v-else class="flex items-center">
                        <svg aria-hidden="true" class="w-6 h-6 text-gray-400" fill="currentColor" viewBox="0 0 20 20"
                             xmlns="http://www.w3.org/2000/svg">
                            <path fill-rule="evenodd"
                                  d="M7.293 14.707a1 1 0 010-1.414L10.586 10 7.293 6.707a1 1 0 011.414-1.414l4 4a1 1 0 010 1.414l-4 4a1 1 0 01-1.414 0z"
                                  clip-rule="evenodd"></path>
                        </svg>
                        <Link
                            :href="route('myFiles', {folder: ans.path})"
                            class="ml-1 text-sm font-medium text-gray-700 hover:text-blue-600 md:ml-2 dark:text-gray-400 dark:hover:text-white"
                        >
                            {{ ans.name }}
                        </Link>
                    </div>

                </li>
            </ol>

            <div class="flex ">
                <label class="flex items-center mr-5">
                    Only Favourites
                    <Checkbox @change="showOnlyFavourites" v-model:checked="onlyFavourites" class="ml-3" />
                </label>
                <ShareFilesButton :all-selected="allSelected" :selected-ids="selectedIds"/>
                <DownloadFilesButton :all="allSelected" :ids="selectedIds" class="mr-2"/>
                <DeleteFilesButton
                    :delete-all="allSelected"
                    :delete-ids="selectedIds"
                    @delete="onDelete"
                />
            </div>
        </nav>
        <div class="flex-1 overflow-auto">

            <table class="min-w-full">
                <thead class="bg-gray-100 border-b">
                <tr>
                    <th class="text-sm font-medium text-gray-900 px-6 py-4 text-left w-[30px] max-w-[30px] pr-0">
                        <Checkbox @change="onSelectAllChange" v-model:checked="allSelected" />
                    </th>
                    <th class="text-sm font-medium text-gray-900 px-5 py-4 text-left">

                    </th>
                    <th class="text-sm font-medium text-gray-900 px-0 py-4 text-left">
                        Name
                    </th>
                    <th v-if="search" class="text-sm font-medium text-gray-900 px-0 py-4 text-left">
                        Path
                    </th>
                    <th class="text-sm font-medium text-gray-900 px-6 py-4 text-left">
                        Owner
                    </th>
                    <th class="text-sm font-medium text-gray-900 px-6 py-4 text-left">
                        Last Modified
                    </th>
                    <th class="text-sm font-medium text-gray-900 px-6 py-4 text-left">
                        Size
                    </th>
                </tr>
                </thead>
                <tbody>
                <tr
                    @dblclick="openFolder(file)"
                    @click="$event => toggleFileSelect(file)"
                    v-for="file of allFiles.data" :key="file.id"
                    class="border-b transition duration-300 ease-in-out hover:bg-blue-100 cursor-pointer"
                    :class="(selected[file.id] || allSelected) ? 'bg-blue-50' : 'bg-white'"
                >
                    <td class="px-6 py-4 whitespace-nowrap text-sm font-medium text-gray-900 w-[30px] max-w-[30px] pr-0">
                        <Checkbox
                            @change="$event => onSelectCheckboxChange(file)"
                            v-model="selected[file.id]"
                            :checked="selected[file.id] || allSelected"
                        />
                    </td>
                    <td class="pl-6 py-4 max-w-[30px] text-sm font-medium text-gray-900 text-yellow-500">
                        <div @click.stop.prevent="addRemoveFavourite(file)">
                            <Out v-if="!file.is_favourite" class="w-5 h-5"/>
                            <StarIcon v-else class="w-5 h-5"/>
                        </div>
                    </td>
                    <td class="px-0 py-4 whitespace-nowrap text-sm font-medium text-gray-900 flex items-center">
                        <FileIcon :file="file"/>
                        {{ file.name }}
                    </td>
                    <td v-if="search" class="px-0 py-4 whitespace-nowrap text-sm font-medium text-gray-900">
                        {{ file.path }}
                    </td>
                    <td class="px-6 py-4 whitespace-nowrap text-sm font-medium text-gray-900">
                        {{ file.owner }}
                    </td>
                    <td class="px-6 py-4 whitespace-nowrap text-sm font-medium text-gray-900">
                        {{ file.updated_at }}
                    </td>
                    <td class="px-6 py-4 whitespace-nowrap text-sm font-medium text-gray-900">
                        {{ file.size }}
                    </td>
                </tr>
                </tbody>
            </table>
            <div v-if="!allFiles.data.length" class="py-8 text-center text-sm text-gray-400">
                There is no data in this folder
            </div>
            <div ref="loadMoreIntersect">

            </div>
        </div>
    </AuthenticatedLayout>
</template>

<script setup>
// Imports
import AuthenticatedLayout from "@/Layouts/AuthenticatedLayout.vue";
import {router, useForm, usePage} from "@inertiajs/vue3";
import { Link } from "@inertiajs/vue3";
import { HomeIcon } from '@heroicons/vue/24/outline';
import FileIcon from "@/Components/app/FileIcon.vue";
import {computed, onMounted, onUpdated, ref} from "vue";
import {httpGet, httpPost} from "@/Helper/http-helper.js";
import Checkbox from "@/Components/Checkbox.vue";
import DeleteFilesButton from "@/Components/app/DeleteFilesButton.vue";
import DownloadFilesButton from "@/Components/app/DownloadFilesButton.vue";
import { StarIcon } from "@heroicons/vue/20/solid";
import { StarIcon as Out } from "@heroicons/vue/24/outline";
import {emitter, ON_SEARCH, showSuccessNotification} from "@/event-bus.js";
import ShareFilesButton from "@/Components/app/ShareFilesButton.vue";

const search = ref('');

const page = usePage();

const allSelected = ref(false);
const onlyFavourites = ref(false);
const selected = ref({});
const loadMoreIntersect = ref(null);

const allFiles = ref({
    data: props.files.data,
    next: props.files.links.next,
})

let params = null;

// Props & Emits
const props = defineProps({
    files: Object,
    folder: Object,
    ancestors: Object,
})

// Computed
const selectedIds = computed(() => Object.entries(selected.value).filter(a => a[1]).map(a => a[0]));

// Functions
function openFolder(file) {
    if (!file.is_folder) {
        return;
    }

    router.visit(route('myFiles',{folder: file.path}))
}
function loadMore() {
    if (allFiles.value.next === null) {
        return;
    }

    httpGet(allFiles.value.next).then(res => {
        allFiles.value.data = [...allFiles.value.data, ...res.data];
        allFiles.value.next = res.links.next;
    })
}
function onSelectAllChange() {
    allFiles.value.data.forEach(f => {
        selected.value[f.id] = allSelected.value;
    })
}
function toggleFileSelect(file) {
    selected.value[file.id] = !selected.value[file.id];
    onSelectCheckboxChange(file);
}
function onSelectCheckboxChange(file) {
    if (!selected.value[file.id]) {
        allSelected.value = false;
    } else {
        let checked = true;

        for (let file of allFiles.value.data) {
            if (!selected.value[file.id]) {
                checked = false;
                break;
            }
        }

        allSelected.value = checked;
    }
}
function onDelete() {
    allSelected.value = false;
    selected.value = {};
}

function addRemoveFavourite(file) {

    httpPost(route('file.addToFavourites'), {id: file.id})
        .then(() => {
            file.is_favourite = !file.is_favourite;
            showSuccessNotification('Selected files have been added to favourites');
        })
        .catch((err) => {
            console.log(err.error.message)
        })

}

function showOnlyFavourites() {
    if (onlyFavourites.value) {
        params.set('favourites', 1);
    } else {
        params.delete('favourites');
    }
    router.get(window.location.pathname + '?' + params.toString());
}

onUpdated(() => {
    allFiles.value = {
        data: props.files.data,
        next: props.files.links.next,
    }
})
onMounted(() => {

    params = new URLSearchParams(window.location.search);
    onlyFavourites.value = params.get('favourites') === '1'
    search.value = params.get('search');
    emitter.on(ON_SEARCH, (value) => {
        search.value = value;
    });

    const observer = new IntersectionObserver((entries) => entries.forEach(entry => entry.isIntersecting && loadMore()), {
        rootMargin: '-250px 0px 0px 0px'
    });

    observer.observe(loadMoreIntersect.value)
})

</script>
