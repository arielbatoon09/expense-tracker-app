<script setup>
import { uuid } from 'vue-uuid';
import database from '../firebase';
import { ref, onMounted, onBeforeUnmount } from 'vue';
import { useRouter } from 'vue-router';
import { collection, query, serverTimestamp, addDoc, where, doc, deleteDoc, onSnapshot } from "firebase/firestore";

const db = database;
const router = useRouter();
const userDataString = localStorage.getItem('userData');
const userData = JSON.parse(userDataString);
const data = ref([]);
const totalIncome = ref(0);
const totalExpense = ref(0);
const totalBalance = ref(0);
const getDate = ref(null);
const getTime = ref(null);

const FormData = ref({
    transaction: 'Select Option',
    note: null,
    amount: null,
});

const isAuth = () => {
    if (!userData) {
        router.push('/');
    }
};

const renderData = async () => {
    const queryCollection = query(
        collection(db, "tracker"),
        where('uid', '==', userData.uid),
    );

    onSnapshot(queryCollection, (querySnapshot) => {
        const listData = [];
        let incomeTotal = 0;
        let expenseTotal = 0;

        querySnapshot.forEach((item) => {
            const list = {
                id: item.id,
                transaction: item.data().transaction,
                note: item.data().note,
                amount: item.data().amount,
            };
            listData.push(list);

            if (list.transaction === 'Income') {
                incomeTotal += list.amount;
            } else if (list.transaction === 'Expense') {
                expenseTotal += list.amount;
            }
        });
        data.value = listData;

        // Update reactive variables
        totalIncome.value = formatCurrency(incomeTotal);
        totalExpense.value = formatCurrency(expenseTotal);
        totalBalance.value = formatCurrency(incomeTotal - expenseTotal);
    });
};

const handleProcessData = async () => {
    const { transaction, note, amount } = FormData.value;
    const queryCollection = collection(db, 'tracker');

    if (!transaction || !note || !amount) {
        console.error('Error: All fields must be filled!');
        return;
    }

    if (transaction === 'Select Option') {
        return;
    }

    try {
        const response = await addDoc(queryCollection, {
            uid: userData.uid,
            transaction: transaction,
            note: note,
            amount: amount,
            created_at: serverTimestamp()
        });

        if (response.id) {
            FormData.value.transaction = 'Select Option';
            FormData.value.note = null;
            FormData.value.amount = null;

            console.log('Success: Added new data!');
        }
    } catch (error) {
        console.error('Error: Failed to add data!', error);
    }
};

const removeTransaction = async (id) => {
    const querryData = doc(db, 'tracker', id);
    deleteDoc(querryData);
};

const handleLogout = () => {
    localStorage.removeItem('userData');
    router.push('/');
};

const formatCurrency = (amount) => {
    const formattedAmount = amount.toLocaleString();
    return formattedAmount;
};

const getTimeDate = () => {
  const currentDate = new Date();
  getDate.value = currentDate.toLocaleDateString();
  getTime.value = currentDate.toLocaleTimeString([], { hour: '2-digit', minute: '2-digit', second: '2-digit' });

};

onMounted(() => {
    isAuth();
    renderData();
    setInterval(getTimeDate, 50);
});

onBeforeUnmount(() => {
  clearInterval(getTimeDate);
});
</script>

<template>
    <div class="bg-white">
        <div class="grid grid-cols-12 gap-0 h-screen">
            <div class="bg-fixed relative col-span-12 sm:col-span-12 md:col-span-7 lg:col-span-8 xxl:col-span-8 hidden md:block"
                style="
            background-image:url('https://i.pinimg.com/originals/74/79/b0/7479b0e55f61567d40b0709a4d1b95d9.gif');
            background-size: cover;
            ">
                <div class="absolute inset-0 z-20 flex items-center justify-center h-full bg-gray-900 bg-opacity-50">
                    <div class="flex flex-col justify-center h-full items-center fixed top-0">
                        <p class="text-white text-5xl lg:text-7xl">
                            <span class="border-2 p-4 rounded-md">{{ getTime }}</span>
                        </p>
                        <p class="text-white">{{ getDate }}</p>
                    </div>
                </div>
            </div>
            <div class="col-span-12 sm:col-span-12 md:col-span-5 lg:col-span-4 xxl:col-span-4">
                <div>
                    <div class="border-b flex justify-between items-center">
                        <div class="my-4 px-6">
                            <h2 class="font-semibold text-2xl">Expense Tracker</h2>
                        </div>
                        <button @click="handleLogout"
                            class="my-4 px-6 text-gray-600 hover:text-gray-700 flex items-center gap-2">
                            <svg class="w-[20px] h-[20px] text-gray-800 dark:text-white" aria-hidden="true"
                                xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 18 15">
                                <path stroke="currentColor" stroke-linecap="round" stroke-linejoin="round"
                                    stroke-width="1.5"
                                    d="M1 7.5h11m0 0L8 3.786M12 7.5l-4 3.714M12 1h3c.53 0 1.04.196 1.414.544.375.348.586.82.586 1.313v9.286c0 .492-.21.965-.586 1.313A2.081 2.081 0 0 1 15 14h-3" />
                            </svg>
                            Logout
                        </button>
                    </div>
                    <div class="px-8 py-2">
                        <h3 class="text-2xl my-4 font-bold text-gray-700">Hi, {{ userData.fullname }} 👋</h3>
                        <h4 class="text-lg text-gray-500 font-thin">Your Balance</h4>
                        <h4 class="text-2xl font-semibold">PHP {{ totalBalance }}</h4>
                    </div>
                    <div class="flex space-x-0 flex-col lg:flex-row lg:space-x-2 my-2 px-6">
                        <div class="bg-green-500 p-4 border-2 rounded-md shadow-lg  w-full text-white text-center">
                            <h1 class="text-xl font-light">INCOME</h1>
                            <h1 class="text-2xl text-green-100 font-semibold">PHP {{ totalIncome }}</h1>
                        </div>
                        <div class="bg-red-500 p-4 border-2 rounded-md shadow-lg  w-full text-white text-center">
                            <h1 class="text-xl font-light">EXPENSE</h1>
                            <h1 class="text-2xl text-red-100 font-semibold">PHP {{ totalExpense }}</h1>
                        </div>
                    </div>
                    <div class="px-8 my-6">
                        <div class="my-4 border-b w-full">
                            <h2 class="font-semibold text-lg">History <small
                                    class="text-green-500 font-normal">(Scrollable)</small></h2>
                        </div>

                        <!-- List of Data -->
                        <div class="h-[200px] overflow-y-auto">
                            <div v-if="data.length > 0" v-for="list in data" :key="list.id"
                                class="ml-6 relative bg-white p-4 border-r-8 shadow-md my-4 flex justify-between"
                                :class="list.transaction === 'Income' ? 'border-green-300' : 'border-red-300'">
                                <div class="absolute -left-6">
                                    <div @click="removeTransaction(list.id)"
                                        class="cursor-pointer bg-red-400 p-2 w-6 h-6 flex items-center text-xs text-white justify-center rounded-full ml-2">
                                        <svg class="w-[12px] h-[12px] text-white" aria-hidden="true"
                                            xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 14 14">
                                            <path stroke="currentColor" stroke-linecap="round" stroke-linejoin="round"
                                                stroke-width="1.5" d="m1 1 6 6m0 0 6 6M7 7l6-6M7 7l-6 6" />
                                        </svg>
                                    </div>
                                </div>
                                <div>
                                    <p>{{ list.note }}</p>
                                </div>
                                <div>
                                    <p>PHP {{ list.transaction === 'Income' ? '+' : '-' }} {{ Math.abs(list.amount) }}</p>
                                </div>
                            </div>
                            <div v-else>No History</div>
                        </div>
                    </div>
                    <div class="px-8 my-6">
                        <div class="my-4 border-b w-full">
                            <h2 class="font-semibold text-lg">Add new transaction</h2>
                        </div>
                        <div class="bg-white p-4 border-2 rounded-md">
                            <div class="mt-4">
                                <div class="my-5 text-sm">
                                    <label for="transaction"
                                        class="block mb-2 text-sm font-medium text-gray-900 dark:text-white">Transaction
                                        Type</label>
                                    <select id="transaction" v-model="FormData.transaction"
                                        class="bg-gray-50 border border-gray-300 text-gray-900 text-sm rounded-lg focus:ring-blue-500 focus:border-blue-500 block w-full p-2.5 dark:bg-gray-700 dark:border-gray-600 dark:placeholder-gray-400 dark:text-white dark:focus:ring-blue-500 dark:focus:border-blue-500">
                                        <option selected>Select Option</option>
                                        <option value="Income">Income</option>
                                        <option value="Expense">Expense</option>
                                    </select>
                                </div>
                                <div class="my-5 text-sm">
                                    <label for="text" class="block text-black">Note</label>
                                    <input v-model="FormData.note" type="text"
                                        class="rounded-sm px-4 py-3 mt-1 focus:outline-none bg-gray-100 w-full"
                                        placeholder="Enter Note" />
                                </div>
                                <div class="my-5 text-sm">
                                    <label for="amount" class="block text-black">
                                        Amount
                                    </label>
                                    <input v-model="FormData.amount" type="number"
                                        class="rounded-sm px-4 py-3 mt-1 focus:outline-none bg-gray-100 w-full"
                                        placeholder="Enter Amount" />
                                </div>
                                <div class="my-5">
                                    <button @click="handleProcessData"
                                        class="rounded block text-center text-white bg-gray-700 hover:bg-gray-600 p-3 duration-300 w-full">
                                        Add Transaction
                                    </button>
                                </div>
                            </div>
                        </div>
                    </div>
                </div>
            </div>
        </div>
    </div>
</template>