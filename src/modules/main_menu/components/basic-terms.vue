<template lang="html">
	<section class="ffz--widget ffz--basic-terms">
		<term-editor
			:term="default_term"
			:colored="item.colored"
			:removable="item.removable"
			:adding="true"
			@save="new_term"
		/>
		<div v-if="! val.length || val.length === 1 && hasInheritance" class="tw-mg-t-05 tw-c-text-alt-2 tw-font-size-4 tw-align-center tw-c-text-alt-2 tw-pd-05">
			{{ t('setting.terms.no-terms', 'no terms are defined in this profile') }}
		</div>
		<ul v-else class="ffz--term-list tw-mg-t-05">
			<term-editor
				v-for="term in val"
				v-if="term.t !== 'inherit'"
				:key="term.id"
				:term="term.v"
				:colored="item.colored"
				:removable="item.removable"
				@remove="remove(term)"
				@save="save(term, $event)"
			/>
		</ul>
	</section>
</template>

<script>

import SettingMixin from '../setting-mixin';
import {deep_copy} from 'utilities/object';

let last_id = 0;

export default {
	mixins: [SettingMixin],
	props: ['item', 'context'],

	data() {
		return {
			default_term: {
				v: '',
				t: 'text',
				c: '',
				remove: false
			}
		}
	},

	computed: {
		hasInheritance() {
			for(const val of this.val)
				if ( val.t === 'inherit' )
					return true;
		},

		val() {
			if ( ! this.has_value )
				return [];

			return this.value.map(x => {
				x.id = x.id || `${Date.now()}-${Math.random()}-${last_id++}`;
				return x;
			})
		}
	},

	methods: {
		new_term(term) {
			if ( ! term.v )
				return;

			const vals = Array.from(this.val);
			vals.push({v: term});
			this.set(deep_copy(vals));
		},

		remove(val) {
			const vals = Array.from(this.val),
				idx = vals.indexOf(val);

			if ( idx !== -1 ) {
				vals.splice(idx, 1);
				if ( vals.length )
					this.set(deep_copy(vals));
				else
					this.clear();
			}
		},

		save(val, new_val) {
			val.v = new_val;
			this.set(deep_copy(this.val));
		}
	}
}

</script>