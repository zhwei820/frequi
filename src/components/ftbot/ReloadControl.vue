<script setup lang="ts">
const botStore = useBotStore();
const autoRefreshLoc = computed({
  get() {
    return botStore.globalAutoRefresh;
  },
  set(newValue: boolean) {
    botStore.setGlobalAutoRefresh(newValue);
  },
});

// Refreshes usually finish in well under a second; keep spinning for at least one
// full rotation (animate-spin period) so the feedback is visible.
const MIN_SPIN_MS = 1000;
const spinning = ref(false);
let spinStart = 0;
let stopTimer: number | undefined;

watch(
  () => botStore.isRefreshing,
  (active) => {
    if (active) {
      window.clearTimeout(stopTimer);
      if (!spinning.value) {
        spinning.value = true;
        spinStart = Date.now();
      }
      return;
    }
    const elapsed = Date.now() - spinStart;
    const remaining = MIN_SPIN_MS - (elapsed % MIN_SPIN_MS);
    stopTimer = window.setTimeout(() => {
      spinning.value = false;
    }, remaining);
  },
  { immediate: true },
);

onBeforeUnmount(() => window.clearTimeout(stopTimer));
</script>

<template>
  <div class="flex items-center ms-2">
    <BaseCheckbox v-model="autoRefreshLoc" size="sm" title="Auto Refresh for all bots" />
    <UButton
      class="m-1"
      color="neutral"
      variant="outline"
      title="Auto Refresh all bots now"
      icon="mdi:refresh"
      :ui="{ leadingIcon: spinning ? 'animate-spin' : '' }"
      @click="botStore.allRefreshFull"
    />
  </div>
</template>
