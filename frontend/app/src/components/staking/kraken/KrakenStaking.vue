<script setup lang="ts">
import { HistoryEventEntryType } from '@rotki/common';
import { useKrakenStakingStore } from '@/store/staking/kraken';
import HistoryEventsView from '@/components/history/events/HistoryEventsView.vue';
import KrakenStakingReceived from '@/components/staking/kraken/KrakenStakingReceived.vue';
import KrakenStakingOverview from '@/components/staking/kraken/KrakenStakingOverview.vue';

const { events } = toRefs(useKrakenStakingStore());
</script>

<template>
  <div class="flex flex-col gap-4">
    <div class="grid md:grid-cols-2 gap-4">
      <KrakenStakingOverview
        :total-usd="events.totalUsdValue"
        :earned="events.received"
      />
      <KrakenStakingReceived :received="events.received" />
    </div>

    <!-- as an exception here we specify event-types to only include staking events  -->
    <!-- if an alternative way becomes possible we can use that -->
    <HistoryEventsView
      use-external-account-filter
      location="kraken"
      :event-types="['staking']"
      :entry-types="[HistoryEventEntryType.HISTORY_EVENT]"
    />
  </div>
</template>
