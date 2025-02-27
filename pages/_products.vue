<template>
    <div>
        <button @click="switchView('/customers')">Customers</button>
        <button @click="switchView('/pendingorders')">Pending Orders</button>
        <button @click="switchView('/products')">Products</button>
        <h1>Product Details</h1>
        <ul v-if="products.length > 0" class="customers-container">
            <li v-for="product in products" :key="product.id" class="customer-container">
                <p>{{ product.name }}</p>
                <button @click="shouldDisplayMoreInfo = !shouldDisplayMoreInfo" type="button"
                    class="display-info-toggle btn btn-primary"><span v-if="shouldDisplayMoreInfo">Hide</span><span
                        v-else>Show</span>
                    additional
                    information</button>
                <div v-show="shouldDisplayMoreInfo" class="more-info">
                    <p><strong>Price:</strong> {{ product.price }} kr</p>
                    <p><strong>Quantity in Stock:</strong> {{ product.quantityInStock }}</p>
                </div>
                <button @click="addToCart(product)" type="button" class="display-info-toggle btn btn-primary">Add to
                    cart</button>
                <button @click="removeFromCart(product)" type="button"
                    class="display-info-toggle btn btn-primary">Remove from
                    cart</button>
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
        <div><strong>Total Price:</strong>{{ totalPrice }} kr</div>
    </div>
</template>

<script setup lang="ts">
import { ref, onMounted } from 'vue';

const shouldDisplayMoreInfo = ref(false);
const markedItems = ref<Product[]>([]);
const totalPrice = ref(0);

const switchView = (newView: string) => {
    window.location.href = `${newView}`;
};

const addToCart = (product: Product) => {
    markedItems.value.push(product);
    calculateTotalPrice();
};

const removeFromCart = (product: Product) => {
    const index = markedItems.value.findIndex(item => item.id === product.id);
    if (index !== -1) {
        markedItems.value.splice(index, 1);
        calculateTotalPrice();
    }
};

const calculateTotalPrice = () => {
    totalPrice.value = markedItems.value.reduce((total, item) => total + item.price, 0);
};

interface Product {
    id: number;
    name: string;
    price: number;
    quantityInStock: number;
}

const products = ref<Product[]>([]);
const loading = ref(true);
const error = ref(false);


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