<script setup lang="ts">
import { Blockchain } from '@rotki/common';
import { isEthBlockEventRef, isEthDepositEventRef, isEvmEventRef, isWithdrawalEventRef } from '@/utils/history/events';
import { useSupportedChains } from '@/composables/info/chains';
import HashLink from '@/components/helper/HashLink.vue';
import type { HistoryEventEntry } from '@/types/history/events';

const props = defineProps<{
  event: HistoryEventEntry;
}>();

const { event } = toRefs(props);

const { getChain } = useSupportedChains();
const { is2xlAndUp } = useBreakpoint();

const translationKey = computed<string>(() => `transactions.events.headers.${toSnakeCase(get(event).entryType)}`);

const evmOrDepositEvent = computed(() => get(isEvmEventRef(event)) || get(isEthDepositEventRef(event)));
const blockEvent = isEthBlockEventRef(event);
const withdrawEvent = isWithdrawalEventRef(event);

/**
 * The key is used to avoid an issue where the block event identifier would be reused
 * to display a hash event identifier resulting in a numerical display instead.
 */
const key = computed(() => {
  if (get(evmOrDepositEvent))
    return 'tx_hash';
  else if (get(blockEvent))
    return 'block';
  else if (get(withdrawEvent))
    return 'withdraw';
  else
    return undefined;
});
</script>

<template>
  <i18n-t
    :key="key"
    :keypath="translationKey"
    tag="span"
    class="flex items-center gap-2"
  >
    <template #location>
      {{ toSentenceCase(event.location) }}
    </template>

    <template
      v-if="blockEvent"
      #blockNumber
    >
      <HashLink
        :class="$style.wrapper"
        :text="blockEvent.blockNumber.toString()"
        :show-icon="false"
        type="block"
      />
    </template>

    <template
      v-if="withdrawEvent"
      #validatorIndex
    >
      <HashLink
        :class="$style.wrapper"
        :text="withdrawEvent.validatorIndex.toString()"
        :show-icon="false"
        :chain="Blockchain.ETH2"
        type="address"
      />
    </template>

    <template
      v-if="evmOrDepositEvent"
      #txHash
    >
      <HashLink
        :class="$style.wrapper"
        :text="evmOrDepositEvent.txHash"
        :show-icon="false"
        type="transaction"
        :chain="getChain(evmOrDepositEvent.location)"
        :truncate-length="8"
        :full-address="is2xlAndUp"
      />
    </template>
  </i18n-t>
</template>

<style lang="scss" module>
.wrapper {
  @apply bg-rui-grey-300 pr-1 pl-2 rounded-full m-0.5;
}

:global(.dark) {
  .wrapper {
    @apply bg-rui-grey-800;
  }
}
</style>
