<template>
  <div class="flex flex-col gap-y-4">
    <template
      v-if="carWashPenaltiesStatus !== 'pending' && carWashStatus !== 'pending'"
    >
      <h1 class="font-semibold text-xl mb-2">
        Оштрафовать мойку {{ carWash?.name }}
      </h1>
      <CarWashPenaltyListDataView
        @delete-penalty="onDeletePenalty"
        :car-wash-penalties="carWashPenalties!"
      />
      <Button
        @click="isCarWashPenaltyCreateDialogVisible = true"
        label="Добавить штраф"
        size="large"
        icon="pi pi-plus"
        severity="danger"
        fluid
        rounded
      />
      <CarWashPenaltyCreateDialog
        v-model:visible="isCarWashPenaltyCreateDialogVisible"
        @create-car-wash-penalty="onCreateCarWashPenalty"
      />
    </template>
    <ProgressSpinner v-else />
    <BackButton @click="navigateTo({ name: 'penalties-car-wash' })" />
  </div>
</template>

<script setup lang="ts">
import type {
  CarWashPenalty,
  CarWashPenaltyCreateEvent,
} from "~/types/penalties"
import { useWebAppPopup, BackButton, useWebAppHapticFeedback } from "vue-tg"
import type { CarWashWithServices } from "~/types/car-washes"
import { formatDate } from "date-fns"

const { notificationOccurred } = useWebAppHapticFeedback()
const { showAlert, showConfirm } = useWebAppPopup()

const runtimeConfig = useRuntimeConfig()

const route = useRoute()

const carWashId = Number(route.params.id)

const isCarWashPenaltyCreateDialogVisible = ref<boolean>(false)

const {
  data: carWashPenalties,
  status: carWashPenaltiesStatus,
  refresh: refreshCarWashPenalties,
} = useFetch("/economics/car-washes/penalties", {
  baseURL: runtimeConfig.public.apiBaseUrl,
  query: { car_wash_ids: [carWashId] },
  transform(data: { penalties: CarWashPenalty[] }) {
    return data.penalties
  },
})
const { data: carWash, status: carWashStatus } = useFetch<CarWashWithServices>(
  `/car-washes/${carWashId}/`,
  {
    baseURL: runtimeConfig.public.apiBaseUrl,
  },
)

const onCreateCarWashPenalty = async ({
  amount,
  reason,
  date,
}: CarWashPenaltyCreateEvent): Promise<void> => {
  console.log({ amount, reason, date })
  await $fetch("/economics/car-washes/penalties/", {
    method: "POST",
    baseURL: runtimeConfig.public.apiBaseUrl,
    body: {
      car_wash_id: carWashId,
      amount,
      reason,
      date: formatDate(date, "yyyy-MM-dd"),
    },
    async onResponse({ response }) {
      if (!response.ok) return
      notificationOccurred("success")
      showAlert(
        `❗️ Мойка ${
          carWash.value!.name
        } успешно оштрафована на сумму ${amount} рублей`,
      )
      await refreshCarWashPenalties()
    },
    onResponseError() {
      notificationOccurred("error")
      showAlert(`❌ Не удалось оштрафовать мойку ${carWash.value?.name}`)
    },
  })
}

const onDeletePenalty = async (penaltyId: number): Promise<void> => {
  notificationOccurred("warning")
  showConfirm(
    "❗️ Вы уверены что хотите удалить штраф?",
    async (ok: boolean): Promise<void> => {
      if (!ok) return
      await $fetch(`/economics/car-washes/penalties/${penaltyId}/`, {
        method: "DELETE",
        baseURL: runtimeConfig.public.apiBaseUrl,
        async onResponse({ response }) {
          if (!response.ok) return
          notificationOccurred("success")
          showAlert(`❗️ Штраф успешно удален`)
          await refreshCarWashPenalties()
        },
        onResponseError() {
          notificationOccurred("error")
          showAlert(`❌ Не удалось удалить штраф`)
        },
      })
    },
  )
}
</script>
