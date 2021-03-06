<template>
    <ListDetailLayout class="fav-video"
        :listLoading="listLoading"
        :detailLoading="detailLoading"
        tipText="登录后查看收藏的视频"
        :showTip="!user.loginValid">
        <mu-list slot="list">
            <AvatarListItem v-for="v in user.videos"
                :key="v.id"
                :img="v.picUrl"
                :title="v.name"
                :subTitle="v.creator.map(i => i.name).join(' / ')"
                @click="handleClick(v.id, v.type)">
            </AvatarListItem>
        </mu-list>
        <VideoDetail slot="detail"
            v-if="ui.fav.video"
            :video="ui.fav.video"></VideoDetail>
    </ListDetailLayout>
</template>

<script>
import { mapActions, mapState } from 'vuex';

import { SET_LOGIN_VALID } from '@/store/mutation-types';
import VideoDetail from '@/components/VideoDetail/VideoDetail.vue';
import AvatarListItem from '@/components/AvatarListItem.vue';
import ListDetailLayout from '@/components/ListDetailLayout.vue';

export default {
    data() {
        return {
            listLoading: false,
            detailLoading: false,
            pausedWhenEnter: null
        };
    },
    computed: {
        ...mapState(['ui', 'user']),
    },
    methods: {
        ...mapActions([
            'updateUserVideos',
            'setUiFavVideo',
            'pauseAudio',
            'playAudio'
        ]),
        async loadVideo(id, type) {
            this.detailLoading = true;
            await this.setUiFavVideo({ id, type });
            this.detailLoading = false;
            this.$nextTick(() => {
                const vid = this.$el.querySelector('video');
                if (vid) {
                    vid.addEventListener('play', () => {
                        if (!this.ui.paused) this.pauseAudio();
                    });
                }
            });
        },
        async fetchData() {
            this.listLoading = true;
            await this.updateUserVideos();
            this.listLoading = false;
            const v = this.user.videos[0];
            if (v && v.id) {
                this.loadVideo(v.id, v.type);
            }
        },
        handleClick(id, type) {
            if (this.ui.fav.video && this.ui.fav.video.id == id) return;
            this.loadVideo(id, type);
        }
    },
    mounted() {
        this.pausedWhenEnter = this.ui.paused;
        if (this.user.loginValid) {
            this.fetchData();
        } else {
            this.$store.subscribe(({ type, payload }) => {
                if (type === SET_LOGIN_VALID && payload === true) {
                    this.fetchData();
                }
            });
        }
    },
    activated() {
        this.pausedWhenEnter = this.ui.paused;
    },
    deactivated() {
        if (!this.pausedWhenEnter) {
            this.playAudio();
        }
        this.pausedWhenEnter = null;
    },
    components: {
        VideoDetail,
        AvatarListItem,
        ListDetailLayout
    }
};
</script>
