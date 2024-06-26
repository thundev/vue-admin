<template>
    <div v-if="!items.length" class="font-light text-2xl text-center py-5">
        {{ $vueAdmin.t('THERE_IS_NOTHING_HERE') }}
    </div>
    <table v-else class="min-w-full leading-normal">
        <thead>
            <tr>
                <th
                    v-for="(field, key) in fields"
                    :key="key"
                    scope="col"
                    class="px-5 py-3 bg-white border-b border-gray-200 text-gray-500 text-left text-sm uppercase font-normal"
                >
                    <div
                        class="flex items-center space-x-2"
                        :class="{
                            'hover:text-gray-400 transition cursor-pointer': sortEnabled && isSortableField(field.name),
                        }"
                        @click="sortEnabled && isSortableField(field.name) && onColumnSort(field.name)"
                    >
                        {{ field.title }}
                        <div v-if="sortEnabled && isSortableField(field.name) && isSortedField(field.name)">
                            <angle-up v-if="localSort.direction === 'asc'" width="20px" height="20px" />
                            <angle-down v-else width="20px" height="20px" />
                        </div>
                    </div>
                </th>
                <th
                    v-if="canEdit || canDelete"
                    scope="col"
                    class="px-5 py-3 bg-white border-b border-gray-200 text-gray-500 text-left text-sm uppercase font-normal"
                />
            </tr>
        </thead>
        <tbody>
            <tr
                v-for="item in items"
                :key="item.id"
                class="hover:bg-gray-100 transition"
                :class="{
                    'item-created': !!item._created,
                    'item-updated': !!item._updated,
                    'item-deleted': !!item._deleted,
                }"
            >
                <td v-for="(field, key) in fields" :key="key" class="px-5 py-5 border-b border-gray-200 text-sm">
                    <component
                        :is="field.component ?? 'entity-table-field'"
                        v-bind="getProps(item, field)"
                        v-if="getVisibility(item, field)"
                        @event="field.listener"
                    />
                </td>
                <td v-if="canEdit" class="px-5 py-5 border-b border-gray-200 text-sm text-right">
                    <button
                        class="inline-block pointer text-indigo-500 hover:text-indigo-600"
                        type="button"
                        @click="$emit('edit', item.id)"
                    >
                        {{ $vueAdmin.t('EDIT') }}
                    </button>
                </td>
                <td v-if="canDelete" class="px-5 py-5 border-b border-t border-gray-200 text-sm">
                    <entity-delete-item :item="item" :controller="controller" />
                </td>
            </tr>
        </tbody>
    </table>
</template>

<script lang="ts">
import { defineComponent, PropType } from 'vue';
import EntityTableField from './VEntityTableField.vue';
import EntityForm from './VEntityForm.vue';
import EntityDeleteItem from './VEntityDeleteItem.vue';
import AngleUp from '../svg/AngleUp.vue';
import AngleDown from '../svg/AngleDown.vue';
import { IEntityController } from '@/types';

export default defineComponent({
    name: 'VEntityTable',
    components: {
        AngleDown,
        AngleUp,
        EntityTableField,
        EntityDeleteItem,
        EntityForm,
    },
    props: {
        fields: { type: Array<any>, default: undefined },
        sort: { type: Object as any, default: () => ({}) },
        canEdit: { type: Boolean, default: false },
        canDelete: { type: Boolean, default: false },
        controller: { type: Object as PropType<IEntityController>, required: true },
    },
    emits: ['edit', 'sort'],
    data() {
        return {
            loading: false,
            selectedDeleteItem: null,
            deleteItemConfirmVisible: false,
            localSort: this.sort,
        };
    },
    computed: {
        items(): Array<any> {
            return this.controller.items();
        },
        sortEnabled() {
            return this.localSort.type === 'table';
        },
    },
    methods: {
        getProps(item: any, field: any) {
            const name = field.name;
            const value = item[name];

            let extra = field.props || {};
            if (typeof extra === 'function') {
                extra = extra(value, item);
            }

            return {
                value: value,
                ...extra,
                controller: this.controller,
                entity: item,
                field: name,
            };
        },
        getVisibility(item: any, field: any): boolean {
            if (typeof field.visibility === 'function') {
                return field.visibility(item[field.name], item);
            }

            return field.visibility ?? true;
        },
        getSortableField(field: string) {
            return this.localSort.fields?.find((item: any) => item.name === field);
        },
        isSortableField(field: string): boolean {
            return this.getSortableField(field) !== undefined;
        },
        isSortedField(field: string): boolean {
            const active = this.getSortableField(field).value;
            return this.localSort.active === active;
        },
        onColumnSort(field: string) {
            const active = this.getSortableField(field).value;
            if (this.localSort.active !== active) {
                this.localSort.active = active;
                this.localSort.direction = 'asc';
            } else {
                this.localSort.direction = this.localSort.direction === 'asc' ? 'desc' : 'asc';
            }
            this.$emit('sort', this.localSort);
        },
    },
});
</script>
