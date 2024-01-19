<template>
  <div>
    <input v-model="text" type="text" placeholder="Text to encrypt or encrypted text" />
    <input v-model="password" type="password" placeholder="Password" />
    <button @click="performEncryption">Encrypt</button>
    <button @click="performDecryption">Decrypt</button>
    <button @click="generateRandomPassword">Generate Random Password</button>
    <p>Result: {{ result }}</p>
  </div>
</template>

<script setup>
import { ref } from 'vue';
import CryptoJS from 'crypto-js';

const text = ref('');
const password = ref('');
const result = ref('');

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

const generateRandomPassword = () => {
  password.value = CryptoJS.lib.WordArray.random(16).toString();
};
</script>
