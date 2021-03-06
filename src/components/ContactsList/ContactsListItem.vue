<template>
	<transition name="delete-slide-left">
		<div v-if="!deleteTimeout" :id="id"
			:class="{active: selectedContact === contact.key}"
			tabindex="0" class="app-content-list-item"
			@click.prevent.stop="selectContact" @keypress.enter.prevent.stop="selectContact">
			<!-- keyboard accessibility will focus the input and not the label -->
			<!--
			<input ref="selected" :id="contact.key" type="checkbox"
				class="app-content-list-item-checkbox checkbox" @keypress.enter.space.prevent.stop="toggleSelect">
			<label :for="contact.key" @click.prevent.stop="toggleSelect" @keypress.enter.space.prevent.stop="toggleSelect" />
			-->
			<div :style="{ 'backgroundColor': colorAvatar }" class="app-content-list-item-icon">
				{{ contact.displayName | firstLetter }}
				<!-- try to fetch the avatar only if the contact exists on the server -->
				<div v-if="hasPhoto" :style="{ 'backgroundImage': avatarUrl }" class="app-content-list-item-icon__avatar" />
			</div>

			<!-- contact data -->
			<div class="app-content-list-item-line-one">
				{{ contact.displayName }}
			</div>
			<div v-if="contact.email" class="app-content-list-item-line-two">
				{{ contact.email }}
			</div>

			<!-- undo actions -->
			<div v-if="!contact.addressbook.readOnly && !deleteTimeout" class="icon-delete" tabindex="0"
				@click.prevent.stop="deleteContact" @keypress.enter.prevent.stop="deleteContact" />
		</div>

		<!-- Deleted contact (pending) -->
		<div v-else :id="id" key="deleted"
			class="deleted app-content-list-item">
			<div :style="{ backgroundColor: 'grey' }" class="app-content-list-item-icon">
				{{ contact.displayName | firstLetter }}
				<!-- try to fetch the avatar only if the contact exists on the server -->
				<div v-if="hasPhoto" :style="{ 'backgroundImage': avatarUrl }" class="app-content-list-item-icon__avatar" />
			</div>
			<!-- contact data -->
			<div class="app-content-list-item-line-one">
				{{ contact.displayName }}
			</div>
			<div v-tooltip.auto="t('contacts', 'Deleting the contact in {countdown} seconds', { countdown })" class="icon-history"
				tabindex="0"
				@click.prevent.stop="cancelDeletion" @keypress.enter.prevent.stop="cancelDeletion" />
		</div>
	</transition>
</template>

<script>
export default {
	name: 'ContactsListItem',
	filters: {
		firstLetter(value) {
			return value.charAt(0)
		}
	},
	props: {
		index: {
			type: Number,
			required: true
		},
		contact: {
			type: Object,
			required: true
		}
	},
	data() {
		return {
			deleteInterval: null,
			deleteTimeout: null,
			countdown: 7
		}
	},
	computed: {
		selectedGroup() {
			return this.$route.params.selectedGroup
		},
		selectedContact() {
			return this.$route.params.selectedContact
		},

		// usable and valid html id for scrollTo
		id() {
			return window.btoa(this.contact.key).slice(0, -2)
		},

		hasPhoto() {
			return this.contact.dav && (this.contact.dav.hasphoto || this.contact.photo)
		},

		/**
		 * avatar color based on server toRgb method and the displayName
		 * @returns {string} the color in css format
		 */
		colorAvatar() {
			try {
				let color = this.contact.uid.toRgb()
				return `rgb(${color.r}, ${color.g}, ${color.b})`
			} catch (e) {
				return 'grey'
			}
		},
		avatarUrl() {
			if (this.contact.photo) {
				const type = this.contact.vCard.getFirstProperty('photo').type
				if (!this.contact.photo.startsWith('data') && type === 'binary') {
					// split on coma in case of any leftover base64 data and retrieve last part
					// usually we come to this part when the base64 image type is unknown
					return `url(data:image;base64,${this.contact.photo.split(',').pop()})`
				}
				return `url(${this.contact.photo})`
			}
			return `url(${this.contact.url}?photo)`
		}
	},
	methods: {
		/**
		 * Checkbox management method
		 */
		toggleSelect() {
			// toggle checkbox here because we stop the propagation to not trigger selectContact
			// therefore the selectContact prevent the checkbox label+input propagation
			this.$refs.selected.checked = !this.$refs.selected.checked
		},

		/**
		 * Dispatch contact deletion request
		 */
		deleteContact() {
			this.deleteInterval = setInterval(() => {
				this.countdown--
			}, 1000)
			this.deleteTimeout = setTimeout(() => {
				this.$store.dispatch('deleteContact', { contact: this.contact })
				this.$emit('deleted', this.index)
				// reset
				clearInterval(this.deleteInterval)
				this.countdown = 7
			}, 7000)
		},

		cancelDeletion() {
			clearTimeout(this.deleteTimeout)
			clearInterval(this.deleteInterval)
			this.deleteTimeout = null
			this.deleteInterval = null
			this.countdown = 7
		},

		/**
		 * Select this contact within the list
		 */
		selectContact() {
			// change url with router
			this.$router.push({ name: 'contact', params: { selectedGroup: this.selectedGroup, selectedContact: this.contact.key } })
		}
	}
}
</script>
