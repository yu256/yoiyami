<template>
<div v-if="user" class="_gaps_m" :class="$style.container">
	<VrchatUser :user="user"/>
	<div v-if="instance" class="_gaps_m">
		<div v-if="user.location === 'traveling'">移動中</div>
		<MkA :to="`/world/${user.location.replace('traveling', user.travelingToLocation!).split(':')[0]}`" style="font-size: 1.5em">{{ instance.name }} ({{ instance.userCount }})</MkA>
		<div style="display: flex; gap: 0.5em;">
			<MkButton :class="$style.button" @click="fetchVrcWithAuth('invite/myself', { instance_id: location()! }).then(ok => ok && toast('✅'))">Invite myself</MkButton>
			<MkButton :class="$style.button" @click="copyToClipboard(location())">locationをコピー</MkButton>
		</div>
		<MkA v-if="instance.ownerId?.startsWith('usr')" :to="`/vrchat/${instance.ownerId}`">
			<div v-if="owner">
				<VrcAvatar :user="owner" :class="$style.avatar_host"/>{{ owner.displayName }}
			</div>
			<div v-else-if="instance.ownerId === props.id">
				<VrcAvatar :user="user" :class="$style.avatar_host"/>{{ user.displayName }}
			</div>
		</MkA>
		<VrcGroup v-else-if="instance.ownerId" :id="instance.ownerId"/>
		<div :class="[$style.content, $style.instance]">
			<div class="_gaps_s">
				<span :class="$style.users" class="_gaps_s">
					<span v-for="[img, name] in Object.entries(instance.users)" :key="name" :class="$style.user">
						<VrcAvatar :user="{ currentAvatarThumbnailImageUrl: img }" :class="$style.avatar_host"/>{{ name }}
					</span>
				</span>
				<div>{{ instance.description }}</div>
			</div>
			<img :class="$style.img" :src="instance.thumbnailImageUrl" decoding="async"/>
		</div>
	</div>
	<div v-else-if="user.location === 'private' || user.location === 'offline'">
		{{ user.location }}
	</div>
	<MkLoading v-else-if="user.location !== 'traveling'"/>
</div>
</template>

<script lang="ts" setup>
import { ref } from 'vue';
import VrchatUser from '@/components/VrcUser.user.vue';
import VrcAvatar from '@/components/VrcAvatar.vue';
import VrcGroup from '@/components/VrcGroup.vue';
import { definePageMetadata } from '@/scripts/page-metadata.js';
import { Instance, User, fetchVrcWithAuth } from '@/scripts/vrchat-api.js';
import { defaultStore } from '@/store.js';
import { toast } from '@/os.js';
import MkButton from '@/components/MkButton.vue';
import copyToClipboard from '@/scripts/copy-to-clipboard.js';

const props = defineProps<{
	id: string;
}>();

const user = ref<User>();
const instance = ref<Instance>();
const owner = ref<User>();

// eslint-disable-next-line vue/no-setup-props-destructure
fetchVrcWithAuth('user', {
	user_id: props.id === defaultStore.state.VRChatId ? undefined : props.id,
}).then(async usr => {
	if (!usr) return;
	user.value = usr;
	if (usr.location.startsWith('wrld')) {
		instance.value = await fetchVrcWithAuth('instance', { instance_id: usr.location });
	} else if (usr.location === 'traveling') {
		if (usr.travelingToLocation) instance.value = await fetchVrcWithAuth('instance', { instance_id: usr.travelingToLocation });
	}

	if (!instance.value || instance.value.ownerId === props.id || !(instance.value.ownerId?.startsWith('usr'))) {
		return;
	}

	owner.value = await fetchVrcWithAuth('user', { user_id: instance.value.ownerId });
});

const location = (): string | undefined => user.value && (user.value.location === 'traveling' ? user.value.travelingToLocation : user.value.location);

definePageMetadata({
	title: 'VRChat',
	icon: 'ti ti-badge-vr',
});
</script>

<style lang="scss" module>
@container (min-width: 400px) {
	.instance {
		display: grid;
		grid-template-columns: 1fr 1fr;
		.img {
			margin: .5em;
		}
	}
}

.container {
	background: var(--navBg);
	border-radius: 2em;
	padding: 1em;
	a {
		color: var(--link);
	}
}

.avatar_host {
	width: 20px;
	height: 20px;
	margin-right: 1em;
}

.content {
	background: var(--bg);
	border-radius: 1.5em;
	padding: 1em;
}

.img {
	border-radius: 10%;
	width: 100%;
	margin: 1em auto;
}

.users {
	display: grid;
	grid-template-columns: 1fr 1fr;
	.user {
		background: var(--navBg);
		border-radius: 2em;
		padding: .5em;
	}
}
</style>
