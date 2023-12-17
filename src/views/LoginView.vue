<script setup>
import { ref, onMounted } from 'vue';
import toastr from 'toastr';
import database from '../firebase';
import { useRouter } from 'vue-router';
import { collection, query, where, getDocs } from "firebase/firestore";

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

const FormData = ref({
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

const handleLoginProcess = async () => {
    const { email, password } = FormData.value;

    if (!email || !password) {
        toastr.error('All fields must be filled.');
        return;
    }

    const queryCollection = collection(db, 'user');
    const fetchQuery = await getDocs(query(queryCollection, where('email', '==', email)));

    try {
        if (fetchQuery.empty) {
            toastr.error('Incorrect email or password.');
            return;
        }

        const userDoc = fetchQuery.docs[0];
        const userData = userDoc.data();

        if (userData.password === password) {
            const userDataToStore = {
                uid: userData.uid,
                fullname: userData.fullname,
                email: userData.email,
            };

            localStorage.setItem('userData', JSON.stringify(userDataToStore));

            router.push('/dashboard');
            toastr.success('Login successfully!');
        } else {
            toastr.error('Incorrect email or password.');
        }
    } catch (error) {
        console.error('Error during login:', error.message);
    }
};

onMounted(() => {
    isAuth();
});

</script>

<template>
    <div class="bg-white pt-28 pb-12">
        <div class="mx-auto max-w-screen-2xl px-4 md:px-8">
            <h2 class="mb-4 text-center text-2xl font-bold text-gray-800 md:mb-8 lg:text-3xl">Sign In</h2>

            <div class="mx-auto max-w-lg rounded-lg border">
                <div class="flex flex-col gap-4 p-4 md:p-8">

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

                    <button @click="handleLoginProcess"
                        class="block rounded-lg bg-gray-700 px-8 py-3 text-center text-sm font-semibold text-white outline-none ring-gray-300 transition 
                        duration-100 hover:bg-gray-600 focus-visible:ring active:bg-gray-600 md:text-base">
                        Login
                    </button>
                </div>

                <div class="flex items-center justify-center bg-gray-100 p-4">
                    <p class="text-center text-sm text-gray-500">Don't have an account?
                        <RouterLink to="/register"
                            class="text-gray-500 transition duration-100 hover:text-gray-600 active:text-gray-700">
                            Register
                        </RouterLink>
                    </p>
                </div>
            </div>
        </div>
    </div>
</template>