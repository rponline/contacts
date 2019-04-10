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
	<div v-if="propModel" class="property__group">
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
			:contact="contact"
			:update-contact="debounceUpdateContact"
			@updatedcontact="debounceUpdateContact" />
	</div>
</template>

<script>

import Contact from 'Models/contact'
import rfcProps from 'Models/rfcProps'

import ContactProperty from 'Components/ContactDetails/ContactDetailsProperty'
import PropertyTitle from 'Components/Properties/PropertyTitle'

export default {
	name: 'ContactDetailsPropertyType',
	components: {
		ContactProperty,
		PropertyTitle
	},
	props: {
		contact: {
			type: Contact,
			default: null
		},
		sortedPropertyGroup: {
			type: Object,
			default() {
				return []
			}
		},
		updateContact: {
			type: Function,
			default: () => {}
		},
		debounceUpdateContact: {
			type: Function,
			default: () => {}
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
		}
	},
	methods: {
	}
}
</script>
