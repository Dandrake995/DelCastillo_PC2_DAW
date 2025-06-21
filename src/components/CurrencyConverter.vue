<template>
  <q-card class="q-pa-lg q-mx-auto" style="max-width: 400px">
    <q-card-section>
      <div class="text-h6">Conversión de Monedas</div>
    </q-card-section>
    <q-card-section>
      <q-input
        v-model="amount"
        label="Monto a convertir"
        type="number"
        min="0"
        dense
        outlined
        :error="amountError"
        error-message="Ingrese un monto válido"
      />
      <div class="row q-mt-md">
        <div class="col">
          <q-select
            v-model="from"
            :options="currencyOptions"
            label="Desde"
            dense
            outlined
            emit-value
            map-options
            :error="fromError"
            error-message="Seleccione una moneda"
          />
        </div>
        <div class="col-auto flex flex-center">
          <q-btn
            icon="swap_horiz"
            flat
            round
            @click="swapCurrencies"
            :disable="!from || !to"
            class="q-mx-sm"
          />
        </div>
        <div class="col">
          <q-select
            v-model="to"
            :options="currencyOptions"
            label="Hacia"
            dense
            outlined
            emit-value
            map-options
            :error="toError"
            error-message="Seleccione una moneda"
          />
        </div>
      </div>
      <q-btn
        label="Convertir"
        color="primary"
        class="full-width q-mt-md"
        @click="convert"
        :loading="loading"
      />
      <q-banner v-if="error" class="bg-red-2 text-red q-mt-md">
        {{ error }}
      </q-banner>
      <q-banner v-if="result" class="bg-green-2 text-green q-mt-md">
        {{ result }}
      </q-banner>
    </q-card-section>
  </q-card>
</template>

<script setup>
import { ref, onMounted } from 'vue'
import { api } from 'boot/axios'

const amount = ref('')
const from = ref(null)
const to = ref(null)
const currencyOptions = ref([])
const loading = ref(false)
const result = ref('')
const error = ref('')

const amountError = ref(false)
const fromError = ref(false)
const toError = ref(false)

onMounted(async () => {
  try {
    const res = await api.get('https://api.frankfurter.app/currencies')
    currencyOptions.value = Object.entries(res.data).map(([code, name]) => ({
      label: `${code} - ${name}`,
      value: code,
    }))
  } catch {
    error.value = 'No se pudieron cargar las monedas.'
  }
})

function swapCurrencies() {
  const temp = from.value
  from.value = to.value
  to.value = temp
}

async function convert() {
  error.value = ''
  result.value = ''
  amountError.value = false
  fromError.value = false
  toError.value = false

  if (!amount.value || isNaN(amount.value) || Number(amount.value) <= 0) {
    amountError.value = true
    error.value = 'Ingrese un monto válido'
    return
  }
  if (!from.value) {
    fromError.value = true
    error.value = 'Seleccione la moneda de origen'
    return
  }
  if (!to.value) {
    toError.value = true
    error.value = 'Seleccione la moneda de destino'
    return
  }
  if (from.value === to.value) {
    error.value = 'Las monedas deben ser diferentes'
    return
  }

  loading.value = true
  try {
    const res = await api.get(
      `https://api.frankfurter.app/latest?amount=${amount.value}&from=${from.value}&to=${to.value}`,
    )
    const converted = res.data.rates[to.value]
    result.value = `${amount.value} ${from.value} equivalen a ${converted} ${to.value}`
  } catch {
    error.value = 'No se pudo realizar la conversión'
  }
  loading.value = false
}
</script>
