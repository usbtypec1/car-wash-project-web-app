<template>
  <Dialog
    v-model:visible="visible"
    modal
    class="w-full mx-4"
    :header="carWashService?.name"
  >
    <div class="flex flex-col gap-y-1">
      <label :for="priceInputId" class="font-semibold">Цена</label>
      <InputNumber
        v-model.number="price"
        :min="0"
        :max="1_000_000"
        :input-id="priceInputId"
        fluid
        :invalid="price === null"
      />
      <Message
        v-if="price === null"
        variant="simple"
        severity="error"
        size="small"
      >
        Введите цену
      </Message>
    </div>
    <template #footer>
      <div class="w-full flex gap-x-2">
        <Button
          class="flex-1"
          label="Отменить"
          icon="pi pi-times"
          severity="secondary"
          @click="visible = false"
          autofocus
        />
        <Button
          @click="onSubmit"
          class="flex-1"
          label="Сохранить"
          icon="pi pi-check"
          type="submit"
          autofocus
          :disabled="price === null"
        />
      </div>
    </template>
  </Dialog>
</template>

<script setup lang="ts">
import type { CarWashService } from '~/types/car-wash-services'

const emit = defineEmits(['submit'])

const priceInputId = useId()

const visible = defineModel('visible', { required: true, default: false })

const props = defineProps<{
  carWashService?: CarWashService,
}>()

const price = ref<number | null>(null)

const onSubmit = (): void => {
  visible.value = false
  emit('submit', {
    carWashServiceId: props.carWashService.id,
    price: price.value,
  })
  price.value = null
}
</script>
