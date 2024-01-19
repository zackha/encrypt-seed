<template>
  <div>
    <input v-model="text" type="text" placeholder="Text to encrypt or encrypted text" />
    <div>
      <input :type="passwordFieldType" v-model="password" placeholder="Password" />
      <button @click="togglePasswordVisibility">{{ passwordIcon }}</button>
    </div>
    <button @click="performEncryption">Encrypt</button>
    <button @click="performDecryption">Decrypt</button>
    <button @click="generatePassword">Generate Password</button>
    <button v-if="result" @click="copyResult">Copy Result</button>
    <p v-if="state.copyPassSuccess">Password copied!</p>
    <p v-if="state.copyResultSuccess">Copied!</p>
    <p>Result: {{ result }}</p>
  </div>
</template>

<script setup>
import { ref, reactive } from 'vue';
import CryptoJS from 'crypto-js';

const text = ref('');
const password = ref('');
const result = ref('');
const state = reactive({ copyPassSuccess: false, copyResultSuccess: false });
const isPasswordVisible = ref(false);

const performEncryption = () => {
  const salt = CryptoJS.lib.WordArray.random(128 / 8);
  const key = CryptoJS.PBKDF2(password.value, salt, {
    keySize: 256 / 32,
    iterations: 1000,
  });

  const iv = CryptoJS.lib.WordArray.random(128 / 8);
  const encrypted = CryptoJS.AES.encrypt(text.value, key, { iv: iv });
  result.value = CryptoJS.enc.Base64.stringify(salt.concat(iv).concat(encrypted.ciphertext));
};

const performDecryption = () => {
  try {
    const cipherParams = CryptoJS.enc.Base64.parse(text.value);
    const salt = CryptoJS.lib.WordArray.create(cipherParams.words.slice(0, 4));
    const iv = CryptoJS.lib.WordArray.create(cipherParams.words.slice(4, 8));
    const ciphertext = CryptoJS.lib.WordArray.create(cipherParams.words.slice(8));

    const key = CryptoJS.PBKDF2(password.value, salt, {
      keySize: 256 / 32,
      iterations: 1000,
    });

    const decrypted = CryptoJS.AES.decrypt({ ciphertext: ciphertext }, key, { iv: iv });
    result.value = decrypted.toString(CryptoJS.enc.Utf8);
  } catch (e) {
    result.value = 'Decryption error!';
  }
};

const generatePassword = async () => {
  password.value = CryptoJS.lib.WordArray.random(16).toString(); // 128-bit key
  try {
    await navigator.clipboard.writeText(password.value);
    state.copyPassSuccess = true;
    setTimeout(() => {
      state.copyPassSuccess = false;
    }, 1900);
  } catch (err) {
    console.error('Failed to copy password', err);
  }
};

const togglePasswordVisibility = () => {
  isPasswordVisible.value = !isPasswordVisible.value;
};

const passwordFieldType = computed(() => (isPasswordVisible.value ? 'text' : 'password'));
const passwordIcon = computed(() => (isPasswordVisible.value ? 'hide' : 'show'));

const copyResult = async () => {
  try {
    await navigator.clipboard.writeText(result.value);
    state.copyResultSuccess = true;
    setTimeout(() => {
      state.copyResultSuccess = false;
    }, 1900);
  } catch (err) {
    console.error('Failed to copy result', err);
  }
};
</script>
