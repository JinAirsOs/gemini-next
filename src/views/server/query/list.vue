<template>
  <PageHeader
    v-if="!props.isRecord"
    :title="title.title"
    :sub-title="title.subTitle"
  ></PageHeader>
  <a-card>
    <QuerySearch
      @search="
        (exp) => {
          tblRef.expr = exp;
          tbl.manual();
        }
      "
    ></QuerySearch>
    <c-table ref="tbl" :tbl-ref="tblRef" :size="props.size">
      <template #bodyCell="{ column, text, record }">
        <template v-if="column.dataIndex === 'action'">
          <a-space>
            <template v-if="record.status === 2 && !props.isRecord">
              <a-popconfirm
                :title="$t('order.query.audit.end.tips')"
                @confirm="
                  () => queryStopOrder(record.work_id).then(() => tbl.manual())
                "
              >
                <a-button size="small" danger>{{ $t('order.end') }}</a-button>
              </a-popconfirm>
            </template>

            <a-button
              size="small"
              type="primary"
              @click="
                () =>
                  $router.push({
                    path: '/server/order/profile',
                    query: { order: JSON.stringify(record) },
                  })
              "
            >
              {{ $t('common.profile') }}</a-button
            >
          </a-space>
        </template>
        <template v-if="column.dataIndex === 'status'">
          <a-tag :color="StateQueryUsage(text).color">
            <template #icon>
              <component
                :is="StateQueryUsage(text).icon"
                :spin="StateQueryUsage(text).color === '#408B9B'"
              />
            </template>
            {{ StateQueryUsage(text).title }}
          </a-tag>
        </template>
        <template v-if="column.dataIndex === 'export'">
          <span>{{ text === 0 ? $t('common.no') : $t('common.yes') }}</span>
        </template>
      </template>
    </c-table>
  </a-card>
</template>

<script lang="ts" setup>
  import {
    IQueryListResp,
    QueryExpr,
    queryList,
    QueryParams,
    queryStopOrder,
  } from '@/apis/query';
  import PageHeader from '@/components/pageHeader/pageHeader.vue';
  import { StateQueryUsage } from '@/lib';
  import QuerySearch from './querySearch.vue';
  import { reactive, ref } from 'vue';
  import { useI18n } from 'vue-i18n';
  import { tableRef } from '@/components/table';

  interface propsAttr {
    isRecord?: boolean;
    size?: string;
  }

  const props = withDefaults(defineProps<propsAttr>(), {
    isRecord: false,
    size: 'default',
  });

  const { t } = useI18n();

  const tbl = ref();

  const tblRef = reactive<tableRef>({
    col: [
      {
        title: t('order.query.table.work_id'),
        dataIndex: 'work_id',
      },
      {
        title: t('order.query.table.username'),
        dataIndex: 'username',
      },
      {
        title: t('order.query.table.real_name'),
        dataIndex: 'real_name',
      },
      {
        title: t('order.query.table.time'),
        dataIndex: 'date',
      },
      {
        title: t('order.query.table.export'),
        dataIndex: 'export',
      },
      {
        title: t('order.query.table.status'),
        dataIndex: 'status',
      },
      {
        title: t('common.action'),
        dataIndex: 'action',
      },
    ],
    data: [] as IQueryListResp[],
    pageCount: 0,
    defaultPageSize: 20,
    expr: {
      work_id: '',
      username: '',
      status: 7,
    } as QueryExpr,
    fn: async (expr: QueryParams) => {
      const { data } = await queryList(
        expr,
        props.isRecord ? 'record' : 'order'
      );
      tblRef.data = data.payload.data;
      tblRef.pageCount = data.payload.page;
    },
  });

  const title = {
    title: t('order.query.title'),
    subTitle: t('order.query.desc'),
  };
</script>
