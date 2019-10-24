<template>
  <page-list
    :list="list"
    :columns="columns"
    :group-actions="groupActions"
    :single-actions="singleActions" />
</template>

<script>
import WindowsMixin from '@/mixins/windows'
import { getNameDescriptionTableColumn, getEnabledTableColumn } from '@/utils/common/tableColumn'
import { ENABLED_OPTS } from '@/constants'

export default {
  name: 'DynamicschedtagList',
  mixins: [WindowsMixin],
  data () {
    return {
      list: this.$list.createList(this, {
        resource: 'dynamicschedtags',
        getParams: { details: true },
        filterOptions: {
          name: {
            label: '名称',
            filter: true,
            formatter: val => {
              return `name.contains(${val})`
            },
          },
          'enabled': {
            label: '启用',
            dropdown: true,
            multiple: true,
            items: ENABLED_OPTS,
          },
        },
      }),
      columns: [
        getNameDescriptionTableColumn({
          vm: this,
          hideField: true,
          slotCallback: row => {
            return (
              <side-page-trigger onTrigger={ () => this.sidePageTriggerHandle(row.id, 'DynamicschedtagSidePage') }>{ row.name }</side-page-trigger>
            )
          },
        }),
        getEnabledTableColumn(),
        {
          field: 'schedtag',
          title: '调度标签',
        },
        {
          field: 'condition',
          title: '条件',
        },
      ],
      groupActions: [
        {
          label: '新建',
          action: () => {
            this.createDialog('CreateDynamicschedtagDialog', {
              title: '创建动态调度标签',
              list: this.list,
            })
          },
          meta: () => {
            return {
              buttonType: 'primary',
            }
          },
        },
        {
          label: '删除',
          action: () => {
            this.createDialog('DeleteResDialog', {
              data: this.list.selectedItems,
              columns: this.columns,
              title: '删除动态调度标签',
              list: this.list,
            })
          },
          meta: () => {
            return {
              validate: this.list.allowDelete(),
            }
          },
        },
      ],
      singleActions: [
        {
          label: '启用',
          action: obj => {
            this.list.singleUpdate(obj.id, { enabled: true })
          },
          meta: obj => {
            return {
              validate: !obj.enabled,
            }
          },
        },
        {
          label: '禁用',
          action: obj => {
            this.list.singleUpdate(obj.id, { enabled: false })
          },
          meta: obj => {
            return {
              validate: obj.enabled,
            }
          },
        },
        {
          label: '更多',
          actions: obj => {
            return [
              {
                label: '调整策略',
                action: () => {
                  this.createDialog('UpdateDynamicschedtagDialog', {
                    data: [obj],
                    columns: this.columns,
                    title: '调整策略',
                    list: this.list,
                  })
                },
              },
              {
                label: '删除',
                action: () => {
                  this.createDialog('DeleteResDialog', {
                    data: [obj],
                    columns: this.columns,
                    title: '删除动态调度标签',
                    list: this.list,
                  })
                },
                meta: () => {
                  return {
                    validate: obj.can_delete,
                  }
                },
              },
            ]
          },
        },
      ],
    }
  },
  created () {
    this.initSidePageTab('dynamicschedtag-detail')
    this.list.fetchData()
  },
}
</script>