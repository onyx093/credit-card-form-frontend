<script setup>
import { ref, watch } from 'vue';

defineProps({
  msg: String,
})

const creditCardNumber = ref('');
const expiryDate = ref('');
const cvv = ref('');

const validationSuccess = ref(false);

watch(creditCardNumber, (newValue) => {
  const cleanNumber = newValue.replace(/[-\s]/g, '');

  // Insert a dash every 4 characters
  let formattedNumber = '';
  for (let i = 0; i < cleanNumber.length; i += 4) {
    formattedNumber += cleanNumber.slice(i, i + 4) + '-';
  }

  // Remove the trailing dash if it exists
  formattedNumber = formattedNumber.replace(/-$/, '');

  creditCardNumber.value = formattedNumber;
});

watch(expiryDate, (newValue) => {
  if (newValue.length >= 2 && newValue[2] !== '/') {
    const month = newValue.slice(0, 2);
    const year = newValue.slice(2);
    expiryDate.value = `${month}/${year}`;
  }
});

const submitForm = async () => {
  try {
    const response = await fetch('http://localhost:8000/api/validate-credit-card', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({
        creditCardNumber: creditCardNumber.value.replaceAll('-', ''),
        expiryDate: expiryDate.value,
        cvv: cvv.value,
      }),
    });
    const data = await response.json();
    if(data.validation_success){
      validationSuccess.value = true;
    }
    console.log(data);
  } catch (error) {
    console.error(error);
  }
};
</script>

<template>

  <div v-show="!validationSuccess">
    <h1 class="text-4xl font-bold mb-4">Kindly fill in your credit card details</h1>
    <form @submit.prevent="submitForm" class="bg-white shadow rounded px-8 pt-6 pb-8 mb-4">
      <h2 class="text-lg font-medium mb-4">Credit Card Information</h2>
      <div class="mb-4">
        <label class="block text-gray-700 font-bold mb-2" for="creditCardNumber">
          Card Number
        </label>
        <input
          class="shadow appearance-none border rounded w-full py-2 px-3 text-gray-700 leading-tight focus:outline-none focus:shadow-outline"
          id="creditCardNumber"
          type="text"
          placeholder="XXXX-XXXX-XXXX-XXXX"
          v-model="creditCardNumber"
        >
      </div>
      <div class="flex flex-wrap -mx-2">
        <div class="w-full md:w-1/2 px-2 mb-4 md:mb-0">
          <label class="block text-gray-700 font-bold mb-2" for="expiryDate">
            Expiry Date
          </label>
          <input
            class="shadow appearance-none border rounded w-full py-2 px-3 text-gray-700 leading-tight focus:outline-none focus:shadow-outline"
            id="expiryDate"
            type="text"
            placeholder="MM/YY"
            v-model="expiryDate"
          >
        </div>
        <div class="w-full md:w-1/2 px-2">
          <label class="block text-gray-700 font-bold mb-2" for="cvv">
            CVV
          </label>
          <input
            class="shadow appearance-none border rounded w-full py-2 px-3 text-gray-700 leading-tight focus:outline-none focus:shadow-outline"
            id="cvv"
            type="text"
            placeholder="XXX"
            v-model="cvv"
          >
        </div>
      </div>
      <button class="bg-blue-500 hover:bg-blue-700 text-white font-bold py-2 px-4 rounded mt-4">
        Submit
      </button>
    </form>
  </div>

  <div v-show="validationSuccess">
    <h1 v-show="validationSuccess" class="text-4xl font-bold mb-4">Your credit card details are valid, thank you!</h1>
    <div v-show="validationSuccess" class="text-green-500">
      <font-awesome-icon class="text-9xl" icon="fa-solid fa-check-circle" size="2xl" />
    </div>
  </div>

</template>

<style scoped>

</style>
