<script setup>
import { ref, computed } from 'vue'
import Spinner from './components/Spinner.vue'

const rateSelected = ref('USD')
const amount = ref(5)
const exchange = ref(null)
const loading = ref(true)

fetch(`http://data.fixer.io/api/latest?access_key=b2d1a26d1115b3949641c7d42a322403`)
  .then(res => res.json())
  .then(json => exchange.value = json)
  .catch(err => console.log(err))
  .finally(() => loading.value = false)

const rates = computed(() => {
  if (!exchange.value) return null
  return Object.keys(exchange.value.rates)
})

const rateCalculated = computed(() => {
  if (!exchange.value || amount.value < 0) return '0.00'
  return (amount.value * exchange.value.rates[rateSelected.value]).toFixed(2)
})

</script>

<template>
  <main className='bg-slate-200 flex justify-center items-center pb-32 w-full h-screen'>
    <Spinner v-if="loading" />
    <section v-else className='flex gap-3 mx-5 flex-col bg-slate-100 shadow-lg p-4 justify-center rounded-md'>
      <header className='flex gap-2 items-center text-lg justify-center'>
        $
        <input v-model="amount" type='number'
          className='border-0 border-b-2 bg-transparent outline-none w-1/4 border-black/50' />
        <h2 className='text-2xl font-bold'>Euro to </h2>

        <select v-model="rateSelected"
          className='outline-none bg-transparent overflow-hidden max-h-[40px] border-b-2 border-black/50 text-xl'>
          <option class="bg-slate-200" v-if="rates" v-for="rate in rates" :key="rate" :value="rate">{{ rate }}</option>
        </select>

      </header>
      <h2 className='text-center text-3xl font-mono'>${{ rateCalculated }}</h2>
      <p class="text-center">1 {{ rateSelected }} = € {{ exchange?.rates[rateSelected].toFixed(2) }}</p>
      <small class="text-center">{{ exchange?.date }}</small>
    </section>
  </main>
</template>

<style scoped></style>
