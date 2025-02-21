<template>
  <page-list
    :list="list"
    :columns="columns"
    :group-actions="groupActions"
    :single-actions="singleActions"
    :export-data-options="exportDataOptions" />
</template>

<script>
import { getStatusFilter, getDomainFilter, getTenantFilter, getDescriptionFilter } from '@/utils/common/tableFilter'
import WindowsMixin from '@/mixins/windows'
import ListMixin from '@/mixins/list'
import expectStatus from '@/constants/expectStatus'
import SingleActionsMixin from '../mixins/singleActions'
import ColumnsMixin from '../mixins/columns'

export default {
  name: 'InstanceGroupList',
  mixins: [WindowsMixin, ListMixin, ColumnsMixin, SingleActionsMixin],
  props: {
    id: String,
    getParams: {
      type: Object,
      default: () => ({}),
    },
  },
  data () {
    return {
      list: this.$list.createList(this, {
        id: this.id,
        resource: 'instancegroups',
        steadyStatus: Object.values(expectStatus.instanceGroup).flat(),
        getParams: this.getParam,
        filterOptions: {
          id: {
            label: this.$t('table.title.id'),
          },
          name: {
            label: this.$t('table.title.name'),
            filter: true,
            formatter: val => {
              return `name.contains(${val})`
            },
          },
          description: getDescriptionFilter(),
          status: getStatusFilter('instanceGroup'),
          force_dispersion: {
            label: this.$t('table.title.strategy'),
            dropdown: true,
            items: [
              { label: this.$t('compute.text_695'), key: true },
              { label: this.$t('compute.text_696'), key: false },
            ],
          },
          enabled: {
            label: this.$t('table.title.enable_status'),
            dropdown: true,
            items: [
              { label: this.$t('compute.text_656'), key: true },
              { label: this.$t('compute.text_569'), key: false },
            ],
          },
          projects: getTenantFilter(),
          project_domains: getDomainFilter(),
        },
        hiddenColumns: ['created_at'],
      }),
      exportDataOptions: {
        items: [
          { label: 'ID', key: 'id' },
          { label: this.$t('table.title.enable_status'), key: 'enabled' },
          { label: this.$t('table.title.name'), key: 'name' },
          { label: this.$t('table.title.strategy'), key: 'force_dispersion' },
          { label: this.$t('table.title.granularity'), key: 'granularity' },
          { label: this.$t('table.title.associated_server'), key: 'guest_count' },
          { label: this.$t('res.project'), key: 'tenant' },
          { label: this.$t('table.title.create_time'), key: 'created_at' },
        ],
      },
      groupActions: [
        {
          label: this.$t('compute.perform_create'),
          permission: 'instancegroups_create',
          action: () => {
            this.createDialog('InstanceGroupCreateDialog', {
              onManager: this.onManager,
              refresh: this.refresh,
            })
          },
          meta: () => {
            return {
              buttonType: 'primary',
            }
          },
        },
        {
          label: this.$t('compute.perform_delete'),
          permission: 'instancegroups_delete',
          action: () => {
            this.createDialog('DeleteResDialog', {
              vm: this,
              data: this.list.selectedItems,
              columns: this.columns,
              title: this.$t('compute.text_700', [this.$t('dictionary.instancegroup')]),
              name: this.$t('dictionary.instancegroup'),
              onManager: this.onManager,
            })
          },
          meta: () => this.$getDeleteResult(this.list.selectedItems),
        },
      ],
    }
  },
  created () {
    this.initSidePageTab('instance-group-detail')
    this.list.fetchData()
    this.$bus.$on('InstanceGroupListRefresh', (id) => {
      this.list.singleRefresh(id, Object.values(expectStatus.instanceGroup).flat())
    })
  },
  methods: {
    getParam () {
      const ret = {
        details: true,
        ...this.getParams,
      }
      return ret
    },
    handleOpenSidepage (row) {
      this.sidePageTriggerHandle(this, 'InstanceGroupSidePage', {
        id: row.id,
        resource: 'instancegroups',
        getParams: this.getParam,
      }, {
        list: this.list,
      })
    },
  },
}
</script>
