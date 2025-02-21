<template>
  <detail
    :onManager="onManager"
    :data="data"
    :base-info="baseInfo"
    :extra-info="extraInfo"
    resource="storages"
    statusModule="blockstorage" />
</template>

<script>
// import BrandIcon from '@/sections/BrandIcon'
import { STORAGE_TYPES, MEDIUM_TYPES } from '@Storage/constants/index.js'
import { sizestr } from '@/utils/utils'
import WindowsMixin from '@/mixins/windows'
import {
  getBrandTableColumn,
  getEnabledTableColumn,
  getPublicScopeTableColumn,
} from '@/utils/common/tableColumn'
import {
  getUserTagColumn,
  getExtTagColumn,
} from '@/utils/common/detailColumn'
import ColumnsMixin from '../mixins/columns'

export default {
  name: 'BlockStorageDetail',
  mixins: [WindowsMixin, ColumnsMixin],
  props: {
    data: {
      type: Object,
      required: true,
    },
    onManager: {
      type: Function,
      required: true,
    },
    refresh: {
      type: Function,
      required: true,
    },
    columns: Array,
  },
  data () {
    return {
      baseInfo: [
        getUserTagColumn({
          onManager: this.onManager,
          resource: 'storage',
          columns: () => this.columns,
          tipName: this.$t('storage.text_37'),
          editCheck: (row) => (row.provider || '').toLowerCase() !== 'bingocloud',
        }),
        getExtTagColumn({
          onManager: this.onManager,
          resource: 'storage',
          columns: () => this.columns,
          tipName: this.$t('storage.text_37'),
          editCheck: (row) => (row.provider || '').toLowerCase() !== 'bingocloud',
        }),
        getPublicScopeTableColumn({ vm: this, resource: 'storages' }),
        getBrandTableColumn(),
        getEnabledTableColumn(),
        {
          field: 'storage_type',
          title: this.$t('storage.text_38'),
          slots: {
            default: ({ row }) => {
              return STORAGE_TYPES[row.storage_type] || '-'
            },
          },
        },
        {
          field: 'medium_type',
          title: this.$t('storage.text_39'),
          slots: {
            default: ({ row }) => {
              return MEDIUM_TYPES[row.medium_type] || '-'
            },
          },
        },
      ],
    }
  },
  computed: {
    extraInfo () {
      const _extraInfo = [
        {
          title: this.$t('storage.text_76'),
          items: [
            {
              field: 'capacity',
              title: this.$t('storage.text_177'),
              formatter: ({ row }) => {
                const capacity = this._sizestr(row.capacity)
                const allowedBrands = ['VMware', 'OneCloud']
                const actual_capacity_used = sizestr(row.actual_capacity_used, 'M', 1024)
                return allowedBrands.includes(row.brand) ? this.$t('storage.text_179', [actual_capacity_used, capacity]) : capacity
              },
            },
            {
              field: 'commit_bound',
              title: this.$t('storage.text_60'),
              slots: {
                default: ({ row }) => {
                  return [
                    <a class="edit-item" onClick={this.updateCommitBound}>{row.commit_bound} <a class="edit-icon"><a-icon type='edit' /></a></a>,
                  ]
                },
              },
            },
            {
              field: 'virtual_capacity',
              title: this.$t('storage.text_43'),
              slots: {
                default: ({ row }) => {
                  return this._sizestr(row.virtual_capacity)
                },
              },
            },
            {
              field: 'commit_rate',
              title: this.$t('storage.text_77'),
            },
            {
              field: 'used_capacity',
              title: this.$t('storage.text_44'),
              slots: {
                default: ({ row }) => {
                  return this._sizestr(row.used_capacity)
                },
              },
            },
            {
              field: 'reserved',
              title: this.$t('storage.text_78'),
              slots: {
                default: ({ row }) => {
                  return this._sizestr(row.reserved)
                },
              },
            },
            {
              field: 'waste_capacity',
              title: this.$t('storage.text_79'),
              slots: {
                default: ({ row }) => {
                  return this._sizestr(row.waste_capacity)
                },
              },
            },
          ],
        },
      ]
      if (this.data.storage_type === 'rbd') {
        return _extraInfo.concat({
          title: this.$t('storage.text_80'),
          items: [
            {
              field: 'storage_conf.mon_host',
              title: 'Ceph Mon Host',
            },
            {
              field: 'storage_conf.pool',
              title: 'Ceph Pool',
            },
          ],
        })
      }
      if (this.data.storage_type === 'nfs') {
        return _extraInfo.concat({
          title: this.$t('storage.text_80'),
          items: [
            {
              field: 'storage_conf.nfs_host',
              title: 'NFS Host',
            },
            {
              field: 'storage_conf.nfs_shared_dir',
              title: 'NFS Shared Dir',
            },
          ],
        })
      }
      return _extraInfo
    },
  },
  methods: {
    _sizestr (value) {
      if (!value) return '-'
      return sizestr(value, 'M', 1024)
    },
    updateCommitBound () {
      this.createDialog('BlockStorageUpdateCommitBoundDialog', {
        data: [this.data],
        columns: this.columns,
        title: this.$t('storage.text_34'),
        onManager: this.onManager,
        refresh: this.refresh,
      })
    },
  },
}
</script>
<style lang="less" scoped>
  .edit-item {
    .edit-icon {
      display: none;
    }
    &:hover .edit-icon {
      display: inline;
    }
  }
</style>
