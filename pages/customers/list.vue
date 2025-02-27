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
          <p><strong>Registration Date:</strong> {{ formatDate(customer.registrationDate) }}</p>
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

async function fetchCustomer(customerId: number): Promise<Customer | null> {
  try {
    const response = await fetch(`http://localhost:5141/api/customers`);
    if (!response.ok) {
      if (response.status === 404) {
        return null;
      }
      throw new Error(`HTTP error! status: ${response.status}`);
    }
    const dataArray: Customer[] = await response.json();
    try {
      return dataArray.find(customer => customer.id === customerId);
    } catch {
      return null;
    }
  } catch (err) {
    error.value = true;
    console.error('Error fetching customer:', err);
    return null;
  } finally {
    loading.value = false;
  }
}

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

  let customerId: string | null = null;
  let currentUrl: string | null = null;
  let orderId: number | null = null;
  let totalAmount: number | null = null;
  let totalPrice: number | null = null;

  currentUrl = window.location.href;
  const urlParams = new URLSearchParams(window.location.search);
  customerId = urlParams.get('customerid');
  orderId = Number(urlParams.get('orderid'));
  totalAmount = Number(urlParams.get('totalamount'));
  totalPrice = Number(urlParams.get('totalprice'));

  if (customerId) {
    console.log("customerId:", customerId);
    console.log("orderId:", orderId);
    console.log("totalAmount:", totalAmount);
    console.log("totalPrice:", totalPrice);
    customer.value = await fetchCustomer(Number(customerId));
    customers.value![Number(customerId)].orders!.push({ "id": orderId, "totalAmount": totalAmount, "totalPrice": totalPrice });
    console.log(customer.value);
  } else {
    console.log("customerId parameter not found");
  }
});

const formatDate = (dateString: string) => {
  const options: Intl.DateTimeFormatOptions = { year: 'numeric', month: 'long', day: 'numeric', hour: '2-digit', minute: '2-digit', second: '2-digit' };
  return new Date(dateString).toLocaleDateString(undefined, options);
};
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