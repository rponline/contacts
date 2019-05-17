<!--
  - @copyright Copyright (c) 2019 xxxxx <xxxxx@xxxxx.xxxx>
  -
  - @author xxxxx <xxxxx@xxxxx.xxxx>
  -
  - @license GNU AGPL version 3 or any later version
  -
  - This program is free software: you can redistribute it and/or modify
  - it under the terms of the GNU Affero General Public License as
  - published by the Free Software Foundation, either version 3 of the
  - License, or (at your option) any later version.
  -
  - This program is distributed in the hope that it will be useful,
  - but WITHOUT ANY WARRANTY; without even the implied warranty of
  - MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE. See the
  - GNU Affero General Public License for more details.
  -
  - You should have received a copy of the GNU Affero General Public License
  - along with this program. If not, see <http://www.gnu.org/licenses/>.
  -
  -->

<template>
	<!-- Only show if propmodel  -->
	<div v-if="propModel" class="property__group"
		:style="height">
		<div ref="grp" class="property__group_inner">
			<!-- title of property group-->
			<property-title :icon="propModel.icon" :readable-name="propModel.readableName"
				:info="propModel.info" />

			<!-- properties iteration -->
			<!-- using contact.key in the key and index as key to avoid conflicts between similar data and exact key -->
			<!-- passing the debounceUpdateContact so that the contact-property component contains the function
			and allow us to use it on the rfcProps since the scope is forwarded to the actions -->
			<contact-property v-for="(property, index) in sortedProperties"
				:key="`${index}-${contact.key}-${property.name}`" :index="index"
				:sorted-properties="sortedProperties"
				:property="property"
				:contact="contact" :local-contact="localContact"
				:update-contact="updateContact"
				@updatedcontact="debounceUpdateContact" />
		</div>
	</div>
	<div v-else-if="special" class="property__group"
		:style="height">
		<div ref="grp" class="property__group_inner">
			<template v-if="special === 'addressbook'">
				<property-title
					:icon="specialModel.addressbook.icon"
					:readable-name="specialModel.addressbook.readableName"
					:info="specialModel.addressbook.info" />
				<property-select
					:prop-model="specialModel.addressbook"
					:value.sync="addressbook"
					:property="{}"
					class="property--addressbooks" />
			</template>
			<template v-else-if="special === 'groups'">
				<property-title
					:icon="specialModel.groups.icon"
					:readable-name="specialModel.groups.readableName"
					:info="specialModel.groups.info" />
				<property-groups
					:prop-model="specialModel.groups"
					:value.sync="groups"
					:contact="contact"
					:is-read-only="isReadOnly"
					class="property--groups" />
			</template>
			<template v-else-if="special === 'add' && !isReadOnly">
				<property-title
					:icon="specialModel.add.icon"
					:readable-name="specialModel.add.readableName"
					:info="specialModel.add.info" />
				<add-new-prop
					:contact="contact"
					class="property--add" />
			</template>
		</div>
	</div>
</template>

<script>

import Contact from 'Models/contact'
import rfcProps from 'Models/rfcProps'

import ContactProperty from 'Components/ContactDetails/ContactDetailsProperty'
import PropertyTitle from 'Components/Properties/PropertyTitle'

import AddNewProp from 'Components/ContactDetails/ContactDetailsAddNewProp'
import PropertySelect from 'Components/Properties/PropertySelect'
import PropertyGroups from 'Components/Properties/PropertyGroups'

export default {
	name: 'ContactDetailsPropertyType',
	components: {
		ContactProperty,
		PropertyTitle,
		PropertySelect,
		PropertyGroups,
		AddNewProp
	},
	props: {
		contact: {
			type: Contact,
			default: null
		},
		localContact: {
			type: Object,
			default: null
		},
		sortedPropertyGroup: {
			type: Object,
			default() {
				return {}
			}
		},
		updateContact: {
			type: Function,
			default: () => {}
		},
		debounceUpdateContact: {
			type: Function,
			default: () => {}
		},
		special: {
			type: String,
			default: null
		},
		isReadOnly: {
			type: Boolean,
			default: null
		}
	},
	data: () => {
		return {
			height: {},
			elementstyles: { gridRowEnd: 'span' }
		}
	},
	computed: {
		sortedProperties() {
			return Object.values(this.sortedPropertyGroup)[0]
		},
		propName() {
			return Object.keys(this.sortedPropertyGroup)[0]
		},
		// add rfc properties array to `this`
		properties() {
			return rfcProps.properties(this)
		},
		propModel() {
			// can be undefined
			return this.properties[this.propName]
		},

		/**
		 * Usable groups object linked to the local contact
		 *
		 * @param {string[]} data An array of groups
		 * @returns {Array}
		 */
		groups: {
			get: function() {
				return this.contact.groups
			},
			set: function(data) {
				this.contact.groups = data
			}
		},

		/**
		 * Fake models to reuse components
		 *
		 * @returns {Object}
		 */
		specialModel() {
			return {
				addressbook: {
					readableName: t('contacts', 'Addressbook'),
					icon: 'icon-address-book',
					options: this.addressbooksOptions
				},
				groups: {
					readableName: t('contacts', 'Groups'),
					icon: 'icon-group'
				},
				add: {
					readableName: t('contacts', 'Add new property'),
					icon: 'icon-add'
				}

			}
		},

		/**
		 * Store getters filtered and mapped to usable object
		 *
		 * @returns {Array}
		 */
		addressbooksOptions() {
			return this.addressbooks
				.filter(addressbook => !addressbook.readOnly && addressbook.enabled)
				.map(addressbook => {
					return {
						id: addressbook.id,
						name: addressbook.displayName
					}
				})
		},

		// store getter
		addressbooks() {
			return this.$store.getters.getAddressbooks
		},

		/**
		 * Usable addressbook object linked to the local contact
		 *
		 * @param {string} [addressbookId] set the addressbook id
		 * @returns {string}
		 */
		addressbook: {
			get: function() {
				return this.contact.addressbook.id
			},
			set: function(addressbookId) {
				this.moveContactToAddressbook(addressbookId)
			}
		}
	},
	mounted() {
		this.setHeight()
	},
	updated: function() {
		// ugly: should just listen on this.$refs.grp.clientHeight
		this.$nextTick(
			this.setHeight()
		)
	},
	methods: {
		// How: Listen on DOM Element Attribute change?
		setHeight() {
			if (this.$refs.grp && this.$refs.grp.clientHeight) {
				let brickHeight = this.$refs.grp.clientHeight
				// ugly: gridRowGap and rowHeight reference to attribute outside this vue component
				let gridRowGap = 40 // ugly: hardcoded grid-row-gap
				let rowHeight = 0 // ugly: hardcoded grid-auto-rows
				let rowSpan = Math.ceil((brickHeight + gridRowGap) / (rowHeight + gridRowGap))
				this.$set(this.height, 'gridRowEnd', 'span ' + rowSpan)
			}
		},

		/**
		 * Move contact to the specified addressbook
		 *
		 * @param {string} addressbookId the desired addressbook ID
		 */
		async moveContactToAddressbook(addressbookId) {
			let addressbook = this.addressbooks.find(search => search.id === addressbookId)
			this.loadingUpdate = true
			if (addressbook) {
				try {
					const contact = await this.$store.dispatch('moveContactToAddressbook', {
						// we need to use the store contact, not the local contact
						// using this.contact and not this.localContact
						contact: this.contact,
						addressbook
					})
					// select the contact again
					this.$router.push({
						name: 'contact',
						params: {
							selectedGroup: this.$route.params.selectedGroup,
							selectedContact: contact.key
						}
					})
				} catch (error) {
					console.error(error)
				} finally {
					this.loadingUpdate = false
				}
			}
		}
	}
}
</script>
