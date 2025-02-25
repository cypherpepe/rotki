<script setup lang="ts">
import { omit } from 'lodash-es';
import { useMessageStore } from '@/store/message';
import { useConfirmStore } from '@/store/confirm';
import { usePaginationFilters } from '@/composables/use-pagination-filter';
import { useCexMappingForm } from '@/composables/assets/forms/cex-mapping-form';
import { useCommonTableProps } from '@/composables/use-common-table-props';
import { useAssetCexMappingApi } from '@/composables/api/assets/cex-mapping';
import ManageCexMappingFormDialog from '@/components/asset-manager/cex-mapping/ManageCexMappingFormDialog.vue';
import ManageCexMappingTable from '@/components/asset-manager/cex-mapping/ManageCexMappingTable.vue';
import TablePageLayout from '@/components/layout/TablePageLayout.vue';
import type { CexMapping, CexMappingRequestPayload } from '@/types/asset';

const { t } = useI18n();
const router = useRouter();
const route = useRoute();

const { deleteCexMapping, fetchAllCexMapping } = useAssetCexMappingApi();

const selectedLocation = ref<string>('');
const selectedSymbol = ref<string>('');
const editMode = ref<boolean>(false);

const { editableItem } = useCommonTableProps<CexMapping>();

const extraParams = computed(() => {
  const location = get(selectedLocation);
  const symbol = get(selectedSymbol);
  const data: { location?: string; locationSymbol?: string } = {};
  if (location)
    data.location = location;
  if (symbol)
    data.locationSymbol = symbol;
  return data;
});

const {
  fetchData,
  isLoading: loading,
  pagination,
  state,
} = usePaginationFilters<
  CexMapping,
  CexMappingRequestPayload
>(fetchAllCexMapping, {
  extraParams,
  history: 'router',
  onUpdateFilters(query) {
    set(selectedLocation, query.location || '');
    set(selectedSymbol, query.locationSymbol || '');
  },
});

onMounted(async () => {
  const { query } = get(route);
  if (query.add) {
    await router.replace({ query: {} });
    add({
      location: (query.location as string) || '',
      locationSymbol: (query.locationSymbol as string) || '',
    });
  }

  await fetchData();
});

const { setOpenDialog, setPostSubmitFunc } = useCexMappingForm();
setPostSubmitFunc(fetchData);

function add(payload?: Partial<CexMapping>) {
  set(editableItem, payload || null);
  set(editMode, false);
  setOpenDialog(true);
}

function edit(editMapping: CexMapping) {
  set(editableItem, editMapping);
  set(editMode, true);
  setOpenDialog(true);
}

const { show } = useConfirmStore();
const { setMessage } = useMessageStore();

async function confirmDelete(mapping: CexMapping) {
  try {
    const success = await deleteCexMapping(omit(mapping, 'asset'));
    if (success)
      await fetchData();
  }
  catch (error: any) {
    setMessage({
      description: t('asset_management.cex_mapping.delete_error', {
        message: error.message,
      }),
    });
  }
}

function showDeleteConfirmation(item: CexMapping) {
  show(
    {
      message: t('asset_management.cex_mapping.confirm_delete.message', {
        asset: item.locationSymbol,
        location: item.location || t('asset_management.cex_mapping.all_exchanges'),
      }),
      title: t('asset_management.cex_mapping.confirm_delete.title'),
    },
    async () => await confirmDelete(item),
  );
}

const dialogTitle = computed<string>(() =>
  get(editMode) ? t('asset_management.cex_mapping.edit_title') : t('asset_management.cex_mapping.add_title'),
);
</script>

<template>
  <TablePageLayout
    child
    class="md:-mt-[4.5rem]"
  >
    <template #buttons>
      <RuiButton
        color="primary"
        variant="outlined"
        :loading="loading"
        @click="fetchData()"
      >
        <template #prepend>
          <RuiIcon name="refresh-line" />
        </template>
        {{ t('common.refresh') }}
      </RuiButton>

      <RuiButton
        data-cy="managed-cex-mapping-add-btn"
        color="primary"
        @click="add()"
      >
        <template #prepend>
          <RuiIcon name="add-line" />
        </template>
        {{ t('asset_management.cex_mapping.add_mapping') }}
      </RuiButton>
    </template>
    <RuiCard>
      <ManageCexMappingTable
        v-model:location="selectedLocation"
        v-model:symbol="selectedSymbol"
        v-model:pagination="pagination"
        :collection="state"
        :loading="loading"
        @refresh="fetchData()"
        @edit="edit($event)"
        @delete="showDeleteConfirmation($event)"
      />
      <ManageCexMappingFormDialog
        :title="dialogTitle"
        :edit-mode="editMode"
        :form="editableItem"
        :selected-location="selectedLocation"
      />
    </RuiCard>
  </TablePageLayout>
</template>
