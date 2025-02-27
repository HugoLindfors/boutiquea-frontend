<template>
  <div>
    <button @click="switchView('/customers/list')">Customers</button>
    <button @click="switchView('/products/list')">Products</button>
    <h1>List of customers</h1>
    <ul v-if="customers.length > 0" class="customers-container">
      <li v-for="customer in customers" :key="customer.id" class="customer-container">
        <p>{{ customer.fullName }}</p>
        <button @click="makeOrder(customer.id)" type="button" class="display-info-toggle btn btn-primary">Make
          order as</button>
        <button @click="shouldDisplayMoreInfo[customer.id] = !shouldDisplayMoreInfo[customer.id]" type="button"
          class="display-info-toggle btn btn-primary"><span v-if="shouldDisplayMoreInfo[customer.id]">Hide</span><span
            v-else>Show</span>
          additional
          information</button>
        <div v-show="shouldDisplayMoreInfo[customer.id]" class="more-info">
          <p><strong>Registration Date:</strong> {{ customer.registrationDate.substring(0, 12) }}</p>
        </div>
        <h2>Orders</h2>
        <ul v-if="customer.orders && customer.orders.length > 0">
          <li v-for="order in customer.orders" :key="order.id">
            <p>Total: {{ order.totalAmount }} Items, Total Cost: {{ order.totalPrice }} kr</p>
          </li>
        </ul>
        <p v-else>No orders found.</p>
      </li>
    </ul>
    <div v-else-if="loading">
      <p>Loading customers data...</p>
    </div>
    <div v-else-if="error">
      <p>Error loading customers: {{ error }}</p>
    </div>
    <div v-else>
      <p>Customers not found.</p>
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref, onMounted } from 'vue';

interface Customer {
  id: number;
  fullName: string;
  registrationDate: string;
  orders: Order[];
}

// This should be filled up
interface Order {
  id: number;
  totalAmount: number;
  totalPrice: number;
}

interface Product {
  id: number;
  name: string;
  price: number;
  quantityInStock: number;
}


const shouldDisplayMoreInfo = ref<boolean[]>([]);
const customer = ref<Customer | null>(null);
const customers = ref<Customer[]>([]);
const loading = ref(true);
const error = ref(false);

const makeOrder = (id: number) => {
  window.location.href = `/order/create?customerid=${id}`;
};

const switchView = (newView: string) => {
  window.location.href = `${newView}`;
};

async function fetchCustomers(): Promise<Customer[]> {
  try {
    const response = await fetch('http://localhost:5141/api/customers');
    if (!response.ok) {
      throw new Error(`HTTP error! status: ${response.status}`);
    }
    const data: Customer[] = await response.json();
    return data;
  } catch (err) {
    error.value = true;
    console.error('Error fetching customers:', err);
    return [];
  } finally {
    loading.value = false;
  }
}

onMounted(async () => {
  customers.value = await fetchCustomers();

  const urlParams = new URLSearchParams(window.location.search);

  let customerId: string | null = urlParams.get('customerid');
  let orderId: number | null = Number(urlParams.get('orderid'));
  let totalAmount: number | null = Number(urlParams.get('totalamount'));
  let totalPrice: number | null = Number(urlParams.get('totalprice'));

  customers.value![Number(customerId)].orders!.push({ "id": orderId, "totalAmount": totalAmount, "totalPrice": totalPrice });
  console.log(customer.value);
});
</script>

<style>
body {
  background-color: black;
  color: white
}

ul {
  list-style-type: none;
}

.customers-container {
  display: grid;
  grid-template-columns: repeat(4, 1fr);
}

.customer-container {
  border: 1px solid white;
  border-radius: 15px;
  margin: 15px;
  padding: 10px;
}

button {
  background-color: black;
  color: white;
}
</style>