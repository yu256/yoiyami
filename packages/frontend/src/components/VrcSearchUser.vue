<template>
<MkInput v-model="query" placeholder="ユーザー検索"/>
<MkButton class="ti ti-search" rounded inline @click="search"/>
<div v-if="users" class="_gaps_s">
	<VrcUser v-for="user in users" :key="user.id" :user="user"/>
</div>
</template>

<script lang="ts" setup>
import { ref } from 'vue';
import VrcUser from './VrcUser.vue';
import MkInput from '@/components/MkInput.vue';
import MkButton from '@/components/MkButton.vue';
import { HitUsers, fetchVrcWithAuth } from '@/scripts/vrchat-api.js';

let query = '';
const users = ref<HitUsers>();

async function search(): Promise<void> {
	users.value = await fetchVrcWithAuth('search/user', { username: query });
}
</script>
