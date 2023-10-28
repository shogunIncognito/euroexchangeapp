<script setup>
import { ref, computed, onMounted } from 'vue'
import Spinner from './components/Spinner.vue'

const rateSelected = ref('USD')
const amount = ref(5)
const exchange = ref(null)
const loading = ref(false)

const exchangeState = ref({
  rateSelected: 'USD',
  amount: 5,
  exchange: null,
})

const getExchangeRates = async () => {
  try {
    loading.value = true
    const res = await fetch(`http://data.fixer.io/api/latest?access_key=b2d1a26d1115b3949641c7d42a322403`)
    const json = await res.json()

    exchangeState.value.exchange = json
  } catch (err) {
    console.error(err)
  } finally {
    loading.value = false
  }
}

const rates = computed(() => {
  if (!exchangeState.value.exchange) return null
  return Object.keys(exchangeState.value.exchange.rates)
})

const rateCalculated = computed(() => {
  const { amount, exchange, rateSelected } = exchangeState.value

  if (!exchange || amount < 0) return '0.00'
  return (amount * exchange.rates[rateSelected]).toFixed(2)
})

onMounted(() => {
  getExchangeRates()
})

const equivalentExchange = computed(() => {
  const { exchange, rateSelected } = exchangeState.value
  return {
    euroToSelected: exchange?.rates[rateSelected].toFixed(2),
    selectedToEuro: (1 / exchange?.rates[rateSelected]).toFixed(7),
  }
})

</script>

<template>
  <main className='bg-slate-200 flex-col flex justify-center items-center pb-32 w-full h-screen'>
    <Spinner v-if="loading" />
    <template v-else>
      <img width="160" class="mb-7 shadow-xl rounded-full select-none pointer-events-none"
        src="https://static.vecteezy.com/system/resources/previews/001/197/992/non_2x/world-png.png" alt="worldimage">
      <section className='flex gap-3 mx-5 flex-col bg-slate-100 shadow-lg p-4 justify-center rounded-md'>
        <header className='flex gap-2 items-center text-lg justify-center'>
          $
          <input v-model="exchangeState.amount" type='number'
            className='border-0 border-b-2 bg-transparent outline-none w-1/4 border-black/50' />
          <h2 className='text-2xl font-bold'>Euro to </h2>

          <select v-model="exchangeState.rateSelected"
            className='outline-none bg-transparent overflow-hidden max-h-[40px] border-b-2 border-black/50 text-xl'>
            <option class="bg-slate-200" v-if="rates" v-for="rate in rates" :key="rate" :value="rate">{{ rate }}</option>
          </select>

        </header>
        <h2 className='text-center text-3xl font-mono'>${{ rateCalculated }}</h2>
        <p class="text-center">1 EUR = $ {{ equivalentExchange.euroToSelected }} {{ exchangeState.rateSelected }}</p>
        <p class="text-center">1 {{ exchangeState.rateSelected }} = € {{ equivalentExchange.selectedToEuro }} EUR</p>
        <small class="text-center">{{ exchangeState.exchange?.date }}</small>
      </section>
    </template>
  </main>
</template>

<style scoped></style>
