<script setup lang="ts">
import HistoryEventTypeCounterparty from '@/components/history/events/HistoryEventTypeCounterparty.vue';
import { isEthDepositEventRef, isEvmEventRef, isOnlineHistoryEventRef } from '@/utils/history/events';
import { useHistoryEventMappings } from '@/composables/history/events/mapping';
import HashLink from '@/components/helper/HashLink.vue';
import LocationIcon from '@/components/history/LocationIcon.vue';
import HistoryEventTypeCombination from '@/components/history/events/HistoryEventTypeCombination.vue';
import type { HistoryEventEntry } from '@/types/history/events';
import type { Blockchain } from '@rotki/common';

const props = defineProps<{
  event: HistoryEventEntry;
  chain: Blockchain;
}>();

const { event } = toRefs(props);

const { getEventTypeData } = useHistoryEventMappings();
const attrs = getEventTypeData(event);

const { t } = useI18n();

const onlineEvent = isOnlineHistoryEventRef(event);
const evmOrEthDepositEvent = computed(() => get(isEvmEventRef(event)) || get(isEthDepositEventRef(event)));
</script>

<template>
  <div class="flex items-center text-left">
    <HistoryEventTypeCounterparty
      v-if="evmOrEthDepositEvent"
      :event="evmOrEthDepositEvent"
    >
      <HistoryEventTypeCombination :type="attrs" />
    </HistoryEventTypeCounterparty>
    <HistoryEventTypeCombination
      v-else
      :type="attrs"
    />

    <div class="ml-4">
      <div class="font-bold uppercase">
        {{ attrs.label }}
      </div>
      <div
        v-if="event.locationLabel"
        class="text-rui-text-secondary flex items-center"
      >
        <LocationIcon
          v-if="onlineEvent"
          icon
          :item="onlineEvent.location"
          size="16px"
          class="mr-1"
        />
        <HashLink
          :show-icon="!onlineEvent"
          :no-link="!!onlineEvent"
          :text="event.locationLabel"
          :chain="chain"
          :location="event.location"
          :disable-scramble="!!onlineEvent"
        />
      </div>
      <RuiChip
        v-if="event.customized"
        class="mt-1"
        size="sm"
        color="primary"
      >
        <div class="flex items-center gap-2 text-caption font-bold">
          <RuiIcon
            name="file-edit-line"
            size="14"
          />
          {{ t('transactions.events.customized_event') }}
        </div>
      </RuiChip>
    </div>
  </div>
</template>
