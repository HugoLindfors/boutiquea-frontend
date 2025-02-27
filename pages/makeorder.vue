<template>
    <div>
        <h1>Customer Details</h1>
        <ul v-if="products.length > 0" class="customers-container">
            <li v-for="product in products" :key="product.id" class="customer-container">
                <p>{{ product.name }}</p>
                <button @click="shouldDisplayMoreInfo = !shouldDisplayMoreInfo" type="button"
                    class="display-info-toggle btn btn-primary"><span v-if="shouldDisplayMoreInfo">Hide</span><span
                        v-else>Show</span>
                    additional
                    information</button>
                <button @click="addToCart(customer.id, product.id)" type="button"
                    class="display-info-toggle btn btn-primary">Add to cart</button>
                <div v-show="shouldDisplayMoreInfo" class="more-info">
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
    </div>
</template>

<script setup lang="ts">
import { ref, onMounted } from 'vue';

const shouldDisplayMoreInfo = ref(false);

interface Customer {
    id: number;
    fullName: string;
    registrationDate: string;
    orders?: string[];
}

// interface Order {
//     id: number;
//     totalAmount: number;
//     orderDate: string;
// }

interface Product {
    id: number;
    name: string;
    price: number;
    quantityInStock: number;
}

const customer = ref<Customer>();
const products = ref<Product[]>([]);
const loading = ref(true);
const error = ref(false);

const urlParams = new URLSearchParams(window.location.search);
const customerId = urlParams.get('customerid');

const addToCart = (customerId: number, productId: number) => {

};

async function fetchCustomer(customerId: number): Promise<Customer | null> {
    try {
        const response = await fetch(`http://localhost:5141/api/customers/${customerId}`);
        if (!response.ok) {
            if (response.status === 404) {
                return null;
            }
            throw new Error(`HTTP error! status: ${response.status}`);
        }
        const data: Customer = await response.json();
        return data;
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
    customer.value = await fetchCustomer(Number(customerId));
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