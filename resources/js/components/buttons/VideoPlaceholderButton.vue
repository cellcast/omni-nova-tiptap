<template>
    <span style="z-index: 10">
        <div
            class="fixed top-0 left-0 w-full h-full flex items-center justify-center"
            style="z-index: 50"
            v-show="menuIsActive"
        >
            <div
                class="rounded-lg shadow-lg overflow-hidden w-action-fields max-w-full bg-white"
                style="z-index: 20"
            >
                <div class="px-8 py-8">
                    <p v-text="ttt('select a video') + ':'" class="mb-4" />

                    <input
                        ref="searchInput"
                        v-model="search"
                        type="text"
                        class="form-input form-input-bordered px-2 py-2 w-full text-sm text-90 leading-none mb-4"
                        :placeholder="ttt('search videos')"
                    />

                    <div
                        class="max-h-search overflow-auto border border-gray-200 rounded"
                    >
                        <div
                            v-if="loading"
                            class="p-4 text-center text-sm text-gray-500"
                            v-text="ttt('loading') + '...'"
                        ></div>

                        <div
                            v-else-if="error"
                            class="p-4 text-center text-sm text-red-500"
                            v-text="error"
                        ></div>

                        <div
                            v-else-if="!videos.length"
                            class="p-4 text-center text-sm text-gray-500"
                            v-text="ttt('no videos found')"
                        ></div>

                        <button
                            v-else
                            v-for="video in videos"
                            :key="video.id"
                            type="button"
                            class="block w-full text-left px-4 py-2 text-sm border-b border-gray-100 hover:bg-gray-100 cursor-pointer"
                            @click="insertPlaceholder(video)"
                        >
                            <span v-text="video.title"></span>
                            <span
                                class="text-gray-500 text-xs ml-2"
                                v-text="'(' + video.id + ')'"
                            ></span>
                        </button>
                    </div>
                </div>

                <div class="bg-30 px-6 py-3">
                    <div class="flex items-center justify-end">
                        <button
                            type="button"
                            class="btn h-9 px-3 font-normal text-80"
                            @click="hideMenu"
                            v-text="ttt('cancel')"
                        ></button>
                    </div>
                </div>
            </div>

            <div
                class="absolute top-0 left-0 w-full h-full bg-80 opacity-75"
                style="z-index: 10"
                @click="hideMenu"
            ></div>
        </div>

        <span class="whitespace-nowrap">
            <base-button
                :isDisabled="mode != 'editor'"
                :clickMethod="showMenu"
                :icon="['fas', 'video']"
                :title="ttt('insert video placeholder')"
            >
            </base-button>
        </span>
    </span>
</template>

<script>
import { library } from "@fortawesome/fontawesome-svg-core";
import { faVideo } from "@fortawesome/free-solid-svg-icons";
import BaseButton from "./BaseButton.vue";
import translations from "../../mixins/translations";

library.add(faVideo);

export default {
    mixins: [translations],

    props: ["button", "editor", "field", "mode"],

    data: function () {
        return {
            menuIsActive: false,
            search: "",
            videos: [],
            loading: false,
            error: "",
            searchTimer: null,
            fetchId: 0,
        };
    },

    components: {
        BaseButton,
    },

    computed: {
        endpoint() {
            return this.field && this.field.videoPickerEndpoint
                ? this.field.videoPickerEndpoint
                : null;
        },
    },

    watch: {
        search() {
            clearTimeout(this.searchTimer);
            this.searchTimer = setTimeout(() => {
                this.fetchVideos();
            }, 250);
        },
    },

    methods: {
        showMenu() {
            this.menuIsActive = true;
            this.search = "";
            this.fetchVideos();
            this.$nextTick(() => {
                if (this.$refs.searchInput) {
                    this.$refs.searchInput.focus();
                }
            });
        },

        hideMenu() {
            this.menuIsActive = false;
        },

        fetchVideos() {
            if (!this.endpoint) {
                this.error = this.ttt("video picker endpoint not configured");
                return;
            }

            this.loading = true;
            this.error = "";

            const requestId = ++this.fetchId;

            Nova.request()
                .get(this.endpoint, { params: { search: this.search } })
                .then((response) => {
                    if (requestId !== this.fetchId) {
                        return;
                    }

                    this.videos = Array.isArray(response.data)
                        ? response.data
                        : response.data.data || [];
                    this.loading = false;
                })
                .catch((err) => {
                    if (requestId !== this.fetchId) {
                        return;
                    }

                    this.loading = false;
                    this.error = this.ttt("failed to load videos");
                    console.log(err);
                });
        },

        insertPlaceholder(video) {
            this.editor
                .chain()
                .focus()
                .insertContent("{{$video:" + video.id + "}}")
                .run();
            this.hideMenu();
        },
    },
};
</script>
