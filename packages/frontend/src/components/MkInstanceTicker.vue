<!--
SPDX-FileCopyrightText: syuilo and misskey-project
SPDX-License-Identifier: AGPL-3.0-only
-->

<template>
<div v-if="!forceDefault && defaultStore.state.useOriginalInstanceTicker" :class="$style.original" :style="rgbColor">
	<img v-if="faviconUrl" :class="$style.topleftIcon" :src="faviconUrl"/>
</div>
<div v-else :class="$style.root" :style="bg">
	<img v-if="faviconUrl" :class="$style.icon" :src="faviconUrl"/>
	<div :class="$style.name">{{ instance.name }}</div>
</div>
</template>

<script lang="ts" setup>
import { computed } from 'vue';
import { instanceName } from '@/config.js';
import { instance as Instance } from '@/instance.js';
import { getProxiedImageUrlNullable } from '@/scripts/media-proxy.js';
import { defaultStore } from '@/store.js';

const props = defineProps<{
	instance?: {
		faviconUrl?: string | null
		name?: string | null
		themeColor?: string | null
	}
	forceDefault?: boolean
}>();

// if no instance data is given, this is for the local instance
const instance = props.instance ?? {
	name: instanceName,
	themeColor: document.querySelector<HTMLMetaElement>('meta[name="theme-color-orig"]')?.content,
};

const faviconUrl = computed(() => props.instance ? getProxiedImageUrlNullable(props.instance.faviconUrl, 'preview') : getProxiedImageUrlNullable(Instance.iconUrl, 'preview') ?? '/favicon.ico');

const bgColor = instance.themeColor ?? '#777777';

const bg = {
	background: `linear-gradient(90deg, ${bgColor}, ${bgColor}00)`,
};

const rgbColor = {
	'--rgb': bgColor.slice(1).match(/.{2}/g)?.map(hex => parseInt(hex, 16)).join(', ') ?? '119, 119, 119',
};

</script>

<style lang="scss" module>
$height: 2ex;

.original {
	position: absolute;
	z-index: -1;
	inset: 0;
	background: linear-gradient(to top, rgba(var(--rgb), 0.1), rgba(var(--rgb), 0));
}

.topleftIcon {
	position: absolute;
	height: 2em;
	border-radius: 20%;
	opacity: 0.7;
	top: 5px;
	left: 5px;
}

.root {
	display: flex;
	align-items: center;
	height: $height;
	border-radius: 4px 0 0 4px;
	overflow: clip;
	color: #fff;
	text-shadow: /* .866 ≈ sin(60deg) */
		1px 0 1px #000,
		.866px .5px 1px #000,
		.5px .866px 1px #000,
		0 1px 1px #000,
		-.5px .866px 1px #000,
		-.866px .5px 1px #000,
		-1px 0 1px #000,
		-.866px -.5px 1px #000,
		-.5px -.866px 1px #000,
		0 -1px 1px #000,
		.5px -.866px 1px #000,
		.866px -.5px 1px #000;
	mask-image: linear-gradient(90deg,
		rgb(0,0,0),
		rgb(0,0,0) calc(100% - 16px),
		rgba(0,0,0,0) 100%
	);
}

.icon {
	height: $height;
	flex-shrink: 0;
}

.name {
	margin-left: 4px;
	line-height: 1;
	font-size: 0.9em;
	font-weight: bold;
	white-space: nowrap;
	overflow: visible;
}
</style>
