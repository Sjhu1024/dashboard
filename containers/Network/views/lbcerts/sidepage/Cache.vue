<template>
  <page-list
    :list="list"
    :columns="columns"
    :single-actions="singleActions" />
</template>

<script>
import * as R from 'ramda'
import {
  getCopyWithContentTableColumn,
  getBrandTableColumn,
  getRegionTableColumn,
  getAccountTableColumn,
  getTimeTableColumn,
} from '@/utils/common/tableColumn'
import ListMixin from '@/mixins/list'
import WindowsMixin from '@/mixins/windows'

export default {
  name: 'LbcertCacheList',
  mixins: [WindowsMixin, ListMixin],
  props: {
    id: String,
    getParams: {
      type: [Function, Object],
    },
    cloudEnv: String,
  },
  data () {
    return {
      list: this.$list.createList(this, {
        id: this.id,
        resource: 'cachedloadbalancercertificates',
        getParams: this.getParam,
        filterOptions: {
          name: {
            label: this.$t('network.text_317'),
            filter: true,
            formatter: val => {
              return `name.contains(${val})`
            },
          },
        },
      }),
      columns: [
        getCopyWithContentTableColumn({ title: this.$t('network.text_317') }),
        {
          field: 'common_name',
          title: this.$t('network.text_318'),
          width: 70,
          formatter: ({ cellValue, row, column }) => {
            return this.$attrs.data.common_name || '-'
          },
        },
        getTimeTableColumn(),
        {
          field: 'subject_alternative_names',
          title: this.$t('network.text_320'),
          width: 100,
          formatter: ({ cellValue, row, column }) => {
            return this.$attrs.data.subject_alternative_names || '-'
          },
        },
        getBrandTableColumn(),
        getRegionTableColumn(),
        getAccountTableColumn(),
      ],
      singleActions: [
        {
          label: this.$t('network.text_131'),
          permission: 'cachedloadbalancercertificates_delete',
          action: (obj) => {
            this.createDialog('DeleteResDialog', {
              vm: this,
              title: this.$t('network.text_131'),
              name: this.$t('network.text_742'),
              data: [obj],
              columns: this.columns,
              onManager: this.onManager,
              refresh: this.refresh,
              alert: this.$t('network.text_752'),
            })
          },
          meta: (obj) => this.$getDeleteResult(obj),
        },
      ],
    }
  },
  created () {
    this.list.fetchData()
  },
  methods: {
    getParam () {
      const ret = {
        ...(R.is(Function, this.getParams) ? this.getParams() : this.getParams),
      }
      return ret
    },
  },
}
</script>
