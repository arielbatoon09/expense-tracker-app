<script setup>
import { uuid } from 'vue-uuid';
import toastr from 'toastr';
import database from '../firebase';
import { ref, onMounted } from 'vue';
import { useRouter } from 'vue-router';
import { collection, query, onSnapshot, serverTimestamp, getDoc, addDoc, deleteDoc, updateDoc, doc, orderBy, where, getDocs } from "firebase/firestore";

const db = database;
const router = useRouter();

toastr.options = {
  closeButton: false,
  debug: false,
  newestOnTop: false,
  progressBar: true,
  positionClass: 'toast-bottom-right',
  preventDuplicates: true,
  onclick: null,
  showDuration: '300',
  hideDuration: '1000',
  timeOut: '5000',
  extendedTimeOut: '1000',
  showEasing: 'swing',
  hideEasing: 'linear',
  showMethod: 'fadeIn',
  hideMethod: 'fadeOut'
};

// Form State
const FormData = ref({
    fullname: null,
    email: null,
    password: null,
});

const isAuth = () => {
    const userDataString = localStorage.getItem('userData');
    const userData = JSON.parse(userDataString);

    if (userData) {
        router.push('/dashboard');
    }
};

const handleRegisterProcess = async () => {
    const { fullname, email, password } = FormData.value;

    if (!fullname || !email || !password) {
        toastr.error('All fields must be filled.');
        return;
    }

    // Email validation using a regular expression
    const emailRegex = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;
    if (!emailRegex.test(email)) {
        toastr.error('Invalid email address!');
        return;
    }

    const queryCollection = collection(db, 'user');
    const fetchQuery = await getDocs(query(queryCollection, where('email', '==', email)));

    // Check if the email already exists
    if (!isEmpty(fetchQuery.docs)) {
        toastr.error('Email is already in used!');
        return;
    }

    try {
        // Add a new user document
        const response = await addDoc(queryCollection, {
            uid: uuid.v1(),
            fullname: fullname,
            email: email,
            password: password,
            created_at: serverTimestamp()
        });

        if (response.id) {
            FormData.value.fullname = null;
            FormData.value.email = null;
            FormData.value.password = null;

            router.push('/login');
            toastr.success('Registered successfully!');
        }
    } catch (error) {
        console.error('Error: Failed to register!', error);
    }
};

const isEmpty = (obj) => {
    return Object.keys(obj).length === 0;
};

onMounted(() => {
    isAuth();
});

</script>

<template>
    <div class="bg-white pt-28 pb-12">
        <div class="mx-auto max-w-screen-2xl px-4 md:px-8">
            <h2 class="mb-4 text-center text-2xl font-bold text-gray-800 md:mb-8 lg:text-3xl">Create your account</h2>

            <div class="mx-auto max-w-lg rounded-lg border">
                <div class="flex flex-col gap-4 p-4 md:p-8">
                    <div>
                        <label for="fullname" class="mb-2 inline-block text-sm text-gray-800 sm:text-base">Full Name</label>
                        <input v-model="FormData.fullname" name="fullname" type="text"
                            class="w-full rounded border bg-gray-50 px-3 py-2 text-gray-800 outline-none ring-gray-300 transition duration-100 focus:ring" />
                    </div>

                    <div>
                        <label for="email" class="mb-2 inline-block text-sm text-gray-800 sm:text-base">Email</label>
                        <input v-model="FormData.email" name="email" type="email"
                            class="w-full rounded border bg-gray-50 px-3 py-2 text-gray-800 outline-none ring-gray-300 transition duration-100 focus:ring" />
                    </div>

                    <div>
                        <label for="password" class="mb-2 inline-block text-sm text-gray-800 sm:text-base">Password</label>
                        <input v-model="FormData.password" name="password" type="password"
                            class="w-full rounded border bg-gray-50 px-3 py-2 text-gray-800 outline-none ring-gray-300 transition duration-100 focus:ring" />
                    </div>

                    <button @click="handleRegisterProcess" class="block rounded-lg bg-gray-700 px-8 py-3 text-center text-sm font-semibold text-white outline-none ring-gray-300 transition 
                        duration-100 hover:bg-gray-600 focus-visible:ring active:bg-gray-600 md:text-base">
                        Register
                    </button>
                </div>

                <div class="flex items-center justify-center bg-gray-100 p-4">
                    <p class="text-center text-sm text-gray-500">Already have an account?
                        <RouterLink to="/login"
                            class="text-gray-500 transition duration-100 hover:text-gray-600 active:text-gray-700">
                            Login
                        </RouterLink>
                    </p>
                </div>
            </div>
        </div>
    </div>
</template>