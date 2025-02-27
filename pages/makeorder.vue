<template>
    <div>
        <h1>Make Order as Customer {{ customer?.fullName }}</h1>
        <div><strong>Total Price:</strong>{{ totalPrice }} kr</div>
        <ul v-if="products.length > 0" class="customers-container">
            <li v-for="product in products" :key="product.id" class="customer-container">
                <p>{{ product.name }}</p>
                <button @click="shouldDisplayMoreInfo = !shouldDisplayMoreInfo" type="button"
                    class="display-info-toggle btn btn-primary"><span v-if="shouldDisplayMoreInfo">Hide</span><span
                        v-else>Show</span>
                    additional
                    information</button>
                <button @click="addToCart(product)" type="button" class="display-info-toggle btn btn-primary">Add to
                    cart</button>
                <button @click="removeFromCart(product)" type="button"
                    class="display-info-toggle btn btn-primary">Remove from
                    cart</button>
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
        <div><strong>Total Amount:</strong>{{ totalAmount }}</div>
        <div><strong>Total Price:</strong>{{ totalPrice }} kr</div>
        <button type="button" @click="confirmOrder()">Confirm
            Order</button>
    </div>
</template>

<script setup lang="ts">
import { ref, onMounted } from 'vue';

const shouldDisplayMoreInfo = ref(false);
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

const confirmOrder = () => {

    window.location.href = `/customers`;
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
    products: Product[];
    totalAmount: number;
    totalPrice: number;
    orderDate: string;
}

interface Product {
    id: number;
    name: string;
    price: number;
    quantityInStock: number; // Should deincrement for every order
}

const customer = ref<Customer | null>(null);
const customers = ref<Customer[]>([]);
const products = ref<Product[]>([]);
const loading = ref(true);
const error = ref(false);

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
    customers.value = await fetchCustomers();

    let customerId: string | null = null;
    let currentUrl: string | null = null;


    currentUrl = window.location.href;
    console.log(currentUrl);

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