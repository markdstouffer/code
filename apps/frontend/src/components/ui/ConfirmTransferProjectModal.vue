<template>
	<NewModal ref="modal" fade="warning" width="550px">
		<template #title>
			<div class="flex items-center gap-2">
				<span class="text-lg font-extrabold text-contrast">{{ formatMessage(messages.transferTitle) }}</span>
				<Avatar
					:src="project.icon_url"
					:raw-src="project.raw_icon_url"
					:alt="project.name"
					size="xs"
				/>
				<span class="text-lg font-extrabold text-contrast">{{ project.name }}</span>
			</div>
		</template>
		<div class="flex flex-col gap-4">
			<Admonition type="warning" :header="bewareTitle">
				{{ formatMessage(messages.bewarePrefix) }}
				<a
					href="https://support.modrinth.com"
					target="_blank"
					rel="noopener noreferrer"
					class="underline"
					>{{ formatMessage(messages.supportLink) }}</a
				>.
			</Admonition>
			<div
				class="grid grid-cols-[1fr_auto_1fr] items-center justify-center gap-6 rounded-2xl bg-surface-2 p-4"
			>
				<div class="flex items-center gap-2">
					<Avatar :src="currentOwner.avatar_url" :alt="currentOwner.username" size="xs" circle />
					<div class="flex flex-col items-start justify-start gap-1">
						<span class="font-medium text-contrast">{{ currentOwner.username }}</span>
						<span class="text-sm text-secondary">{{ currentOwner.role }}</span>
					</div>
				</div>
				<RightArrowIcon class="h-6 w-6 text-secondary" />
				<div class="flex items-center gap-2">
					<Avatar
						:src="transferTo.avatar_url"
						:alt="transferTo.username || transferTo.name"
						size="xs"
						circle
					/>
					<div class="flex flex-col items-start justify-start gap-1">
						<span class="font-medium text-contrast">
							{{ transferTo.username || transferTo.name }}
						</span>
						<span class="text-sm text-secondary">{{ transferTo.role || 'Member' }}</span>
					</div>
				</div>
			</div>
			<ul class="m-0 flex flex-col gap-1 pl-6 text-secondary">
				<li>{{ formatMessage(messages.ownerAccess) }}</li>
				<li>{{ formatMessage(messages.modifications) }}</li>
				<li>{{ formatMessage(messages.areYouSure) }}</li>
			</ul>
			<div>
				<p class="m-0 mb-2">
					{{ formatMessage(messages.confirmPrefix) }}
					<span class="font-bold text-contrast">{{ project.name }}</span> {{ formatMessage(messages.confirmSuffix) }}
				</p>
				<Input
					v-model="confirmationText"
					:placeholder="`${formatMessage(messages.placeholderPrefix)} ${project.name}`"
					wrapper-class="w-full"
				/>
			</div>
		</div>
		<template #actions>
			<div class="flex justify-end gap-2">
				<Button @click="hide">
					<XIcon />
					{{ formatMessage(messages.cancel) }}
				</Button>
				<Button type="colored" color="red" :disabled="!isConfirmEnabled" @click="onConfirmClick">
					<TransferIcon />
					{{ formatMessage(messages.transferFull) }}
				</Button>
			</div>
		</template>
	</NewModal>
</template>

<script setup lang="ts">
import { RightArrowIcon, TransferIcon, XIcon } from '@modrinth/assets'
import { Admonition, Avatar, Button, defineMessages, Input, NewModal, useVIntl } from '@modrinth/ui'
import { computed, ref } from 'vue'

const props = defineProps<{
	project: { name: string; icon_url: string | null; raw_icon_url?: string | null }
	currentOwner: { avatar_url: string | null; username: string; role: string }
	transferTo: {
		avatar_url?: string | null
		username?: string
		name?: string
		role?: string
	}
	onConfirm: () => void
}>()

const { formatMessage } = useVIntl()

const messages = defineMessages({
	transferTitle: {
		id: 'transfer.title',
		defaultMessage: 'Transfer',
	},
	transferFull: {
		id: 'transfer.full',
		defaultMessage: 'Transfer ownership'
	},
	bewareTitle: {
		id: 'beware.title',
		defaultMessage: 'Beware of scams',
	},
	bewarePrefix: {
		id: 'beware.prefix',
		defaultMessage: 'Do not transfer projects to buyers. This is a common scam and against our TOS. If you encounter a buyer, please',
	},
	supportLink: {
		id: 'beware.supportLink',
		defaultMessage: 'contact support'
	},
	ownerAccess: {
		id: 'warn.ownerAccess',
		defaultMessage: 'You will immediately lose owner access to this project',
	},
	modifications: {
		id: 'warn.modifications',
		defaultMessage: 'The new owner can modify or delete the project at any time',
	},
	areYouSure: {
		id: 'warn.areYouSure',
		defaultMessage: 'This action cannot be undone',
	},
	confirmPrefix: {
		id: 'confirm.prefix',
		defaultMessage: 'To confirm this transfer, type',
	},
	confirmSuffix: {
		id: 'confirm.suffix',
		defaultMessage: 'below',
	},
	placeholderPrefix: {
		id: 'placeholder.prefix',
		defaultMessage: 'Enter',
	},
	cancel: {
		id: 'cancel',
		defaultMessage: 'Cancel',
	},
})

const modal = ref<InstanceType<typeof NewModal>>()
const confirmationText = ref('')

const bewareTitle = computed(() => formatMessage(messages.bewareTitle))

const isConfirmEnabled = computed(
	() =>
		!!props.project.name &&
		confirmationText.value.toLowerCase().trim() === props.project.name.toLowerCase().trim(),
)

function show(e?: MouseEvent) {
	confirmationText.value = ''
	modal.value?.show(e)
}

function hide() {
	modal.value?.hide()
}

function onConfirmClick() {
	hide()
	props.onConfirm()
}

defineExpose({ show, hide })
</script>
