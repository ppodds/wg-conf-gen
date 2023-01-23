<template>
  <v-text-field
    v-for="(item, i) of items"
    :label="`${label}${i + 1}`"
    :model-value="item"
    @update:model-value="updateItem(i, $event)"
    append-icon="mdi-close-circle"
    @click:append="removeItem(i)"
  ></v-text-field>
</template>

<script setup lang="ts">
const props = defineProps<{
  label: string;
  items: string[];
}>();
const emits = defineEmits<{
  (e: "update:items", value: string[]): void;
}>();

function updateItem(index: number, value: string) {
  const newItems = [...props.items];
  newItems[index] = value;
  emits("update:items", newItems);
}

function removeItem(index: number) {
  const newItems = [...props.items];
  newItems.splice(index, 1);
  emits("update:items", newItems);
}
</script>
