<script setup lang="ts">
import { isBtcChain } from '@/types/blockchain/chains';
import { useAccountLoading } from '@/composables/accounts/loading';
import InputModeSelect from '@/components/accounts/management/inputs/InputModeSelect.vue';
import ChainSelect from '@/components/accounts/blockchain/ChainSelect.vue';
import type { InputMode } from '@/types/input-mode';

const props = defineProps<{
  chain: string;
  inputMode: InputMode;
  editMode: boolean;
}>();

const emit = defineEmits<{
  (e: 'update:chain', selection: string): void;
  (e: 'update:input-mode', mode: InputMode): void;
}>();

const chain = toRef(props, 'chain');

const { loading } = useAccountLoading();

const showInputModeSelector = logicOr(
  computed(() => isBtcChain(get(chain))),
);

function updateModelValue(value?: string) {
  if (!value)
    return;

  emit('update:chain', value);
}
</script>

<template>
  <ChainSelect
    :disabled="loading || editMode"
    :model-value="chain"
    @update:model-value="updateModelValue($event)"
  />

  <InputModeSelect
    v-if="!editMode && showInputModeSelector"
    :input-mode="inputMode"
    :blockchain="chain"
    @update:input-mode="emit('update:input-mode', $event)"
  />
</template>
