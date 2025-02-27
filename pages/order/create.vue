<template>
  <div>
    <h1>Make Order as {{ customer?.fullName }}</h1>
    <div><strong>Total Price:</strong>{{ totalPrice }} kr</div>
    <ul v-if="products.length > 0" class="customers-container">
      <li v-for="product in products" :key="product.id" class="customer-container">
        <p>{{ product.name }}</p>
        <button @click="shouldDisplayMoreInfo[product.id] = !shouldDisplayMoreInfo[product.id]" type="button"
          class="display-info-toggle btn btn-primary"><span v-if="shouldDisplayMoreInfo[product.id]">Hide</span><span
            v-else>Show</span>
          additional
          information</button>
        <button @click="addToCart(product)" type="button" class="display-info-toggle btn btn-primary">Add to
          cart</button>
        <button @click="removeFromCart(product)" type="button" class="display-info-toggle btn btn-primary">Remove from
          cart</button>
        <div v-show="shouldDisplayMoreInfo[product.id]" class="more-info">
          <p><strong>Price:</strong> {{ product.price }} kr</p>
          <p><strong>Quantity in Stock:</strong> {{ product.quantityInStock }}</p>
        </div>
      </li>
    </ul>
    <div v-else-if="loading">
      <p>Loading procucts data...</p>
    </div>
    <div v-else-if="error">
      <p>Error loading products: {{ error }}</p>
    </div>
    <div v-else>
      <p>Products not found.</p>
    </div>
    <div><strong>Total Amount:</strong>{{ totalAmount }}</div>
    <div><strong>Total Price:</strong>{{ totalPrice }} kr</div>
    <button type="button" @click="confirmOrder(customer!.id, HARD_CODED_ORDER_ID, totalAmount, totalPrice)">Confirm
      Order</button>
  </div>
</template>

<script setup lang="ts">
import { ref, onMounted } from 'vue';

const shouldDisplayMoreInfo = ref<boolean[]>([]);
const markedItems = ref<Product[]>([]);
const totalAmount = ref(0);
const totalPrice = ref(0);

const addToCart = (product: Product) => {
  product.quantityInStock--;
  markedItems.value.push(product);
  calculateTotalPrice();
  totalAmount.value++;
};

const removeFromCart = (product: Product) => {
  const index = markedItems.value.findIndex(item => item.id === product.id);
  if (index !== -1) {
    product.quantityInStock++;
    markedItems.value.splice(index, 1);
    calculateTotalPrice();
    totalAmount.value--;
  }
};

const calculateTotalPrice = () => {
  totalPrice.value = markedItems.value.reduce((total, item) => total + item.price, 0);
};

const confirmOrder = (customerId: string, orderId: string, totalAmount: number, totalPrice: string) => {

  window.location.href = `/customers/list?customerid=${customerId}&orderid=${orderId}&totalamount=${totalAmount}&totalprice=${totalPrice}`;
};

interface Customer {
  id: number;
  fullName: string;
  registrationDate: string;
  orders?: string[];
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
  quantityInStock: number; // Should deincrement for every order
}

const customer = ref<Customer | null>(null);
const products = ref<Product[]>([]);
const loading = ref(true);
const error = ref(false);
const HARD_CODED_ORDER_ID = 0;

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

async function fetchProducts(): Promise<Product[]> {
  try {
    const response = await fetch('http://localhost:5141/api/products');
    if (!response.ok) {
      throw new Error(`HTTP error! status: ${response.status}`);
    }
    const data: Product[] = await response.json();
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
  products.value = await fetchProducts();

  let customerId: string | null = null;
  let currentUrl: string | null = null;


  currentUrl = window.location.href;
  const urlParams = new URLSearchParams(window.location.search);
  customerId = urlParams.get('customerid');

  if (customerId) {
    console.log("customerId:", customerId);
    customer.value = await fetchCustomer(Number(customerId));
  } else {
    console.log("customerId parameter not found");
  }

  console.log(customer);
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