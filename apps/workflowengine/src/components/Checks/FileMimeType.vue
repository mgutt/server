<!--
  - @copyright Copyright (c) 2019 Julius Härtl <jus@bitgrid.net>
  -
  - @author Julius Härtl <jus@bitgrid.net>
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
	<div>
		<NcMultiselect :value="currentValue"
			:placeholder="t('workflowengine', 'Select a file type')"
			label="label"
			track-by="pattern"
			:options="options"
			:multiple="false"
			:tagging="false"
			@input="setValue">
			<template slot="singleLabel" slot-scope="props">
				<span v-if="props.option.icon" class="option__icon" :class="props.option.icon" />
				<img v-else
					class="option__icon-img"
					:src="props.option.iconUrl"
					alt="">
				<span class="option__title option__title_single">{{ props.option.label }}</span>
			</template>
			<template slot="option" slot-scope="props">
				<span v-if="props.option.icon" class="option__icon" :class="props.option.icon" />
				<img v-else
					class="option__icon-img"
					:src="props.option.iconUrl"
					alt="">
				<span class="option__title">{{ props.option.label }}</span>
			</template>
		</NcMultiselect>
		<input v-if="!isPredefined"
			type="text"
			:value="currentValue.pattern"
			:placeholder="t('workflowengine', 'e.g. httpd/unix-directory')"
			@input="updateCustom">
	</div>
</template>

<script>
import NcMultiselect from '@nextcloud/vue/dist/Components/NcMultiselect.js'
import valueMixin from './../../mixins/valueMixin.js'
import { imagePath } from '@nextcloud/router'

export default {
	name: 'FileMimeType',
	components: {
		NcMultiselect,
	},
	mixins: [
		valueMixin,
	],
	data() {
		return {
			predefinedTypes: [
				{
					icon: 'icon-folder',
					label: t('workflowengine', 'Folder'),
					pattern: 'httpd/unix-directory',
				},
				{
					icon: 'icon-picture',
					label: t('workflowengine', 'Images'),
					pattern: '/image\\/.*/',
				},
				{
					iconUrl: imagePath('core', 'filetypes/x-office-document'),
					label: t('workflowengine', 'Office documents'),
					pattern: '/(vnd\\.(ms-|openxmlformats-|oasis\\.opendocument).*)$/',
				},
				{
					iconUrl: imagePath('core', 'filetypes/application-pdf'),
					label: t('workflowengine', 'PDF documents'),
					pattern: 'application/pdf',
				},
			],
		}
	},
	computed: {
		options() {
			return [...this.predefinedTypes, this.customValue]
		},
		isPredefined() {
			const matchingPredefined = this.predefinedTypes.find((type) => this.newValue === type.pattern)
			if (matchingPredefined) {
				return true
			}
			return false
		},
		customValue() {
			return {
				icon: 'icon-settings-dark',
				label: t('workflowengine', 'Custom MIME type'),
				pattern: '',
			}
		},
		currentValue() {
			const matchingPredefined = this.predefinedTypes.find((type) => this.newValue === type.pattern)
			if (matchingPredefined) {
				return matchingPredefined
			}
			return {
				icon: 'icon-settings-dark',
				label: t('workflowengine', 'Custom mimetype'),
				pattern: this.newValue,
			}
		},
	},
	methods: {
		validateRegex(string) {
			const regexRegex = /^\/(.*)\/([gui]{0,3})$/
			const result = regexRegex.exec(string)
			return result !== null
		},
		setValue(value) {
			if (value !== null) {
				this.newValue = value.pattern
				this.$emit('input', this.newValue)
			}
		},
		updateCustom(event) {
			this.newValue = event.target.value
			this.$emit('input', this.newValue)
		},
	},
}
</script>
<style scoped lang="scss">
	.multiselect, input[type='text'] {
		width: 100%;
	}
	.multiselect >>> .multiselect__content-wrapper li>span,
	.multiselect >>> .multiselect__single {
		display: flex;
		white-space: nowrap;
		overflow: hidden;
		text-overflow: ellipsis;
	}

	.option__icon {
		display: inline-block;
		min-width: 30px;
		background-position: left;
	}

	.option__icon-img {
		margin-right: 14px;
	}
</style>
