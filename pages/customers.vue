<template>
    <div>
        <h1>Customer Details</h1>
        <ul v-if="customers.length > 0" class="customers-container">
            <li v-for="customer in customers" :key="customer.id" class="customer-container">
                <p>{{ customer.fullName }}</p>
                <button @click="makeOrder(customer.id)" type="button"
                    class="display-info-toggle btn btn-primary">Make order as</button>
                <button @click="shouldDisplayMoreInfo = !shouldDisplayMoreInfo" type="button"
                    class="display-info-toggle btn btn-primary"><span v-if="shouldDisplayMoreInfo">Hide</span><span
                        v-else>Show</span>
                    additional
                    information</button>
                <div v-show="shouldDisplayMoreInfo" class="more-info">
                    <p><strong>Registration Date:</strong> {{ formatDate(customer.registrationDate) }}</p>
                </div>
                <!-- <h2>Orders</h2>
                <ul v-if="customer.orders && customer.orders.length > 0">
                    <li v-for="order in customer.orders" :key="order.id">
                        Order ID: {{ order.id }}, Total: {{ order.totalAmount }}, Date: {{ formatDate(order.orderDate)
                        }}
                    </li>
                </ul>
                <p v-else>No orders found.</p> -->
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
    orders?: string[];
}

// interface Order {
//     id: number;
//     totalAmount: number;
//     orderDate: string;
// }

const shouldDisplayMoreInfo = ref(false);
const customers = ref<Customer[]>([]);
const loading = ref(true);
const error = ref(false);

const makeOrder = (id: number) => {
    window.location.href = `/makeorder?customerid=${id}`;
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