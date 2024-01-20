<template>
  <div class="body">
    <div class="content">
      <div class="main">
        <div class="texts">
          <textarea v-model="text" rows="12" type="text" placeholder="Text to encrypt or encrypted text"></textarea>
          <div class="password">
            <input :type="passwordFieldType" v-model="password" placeholder="Password" />
            <div class="passwordcomp">
              <span class="comp" @click="generatePassword">
                <component :is="PixelarticonsReload" />
              </span>
              <span class="comp" @click="togglePasswordVisibility">
                <component :is="passwordIconComponent" />
              </span>
            </div>
          </div>
        </div>
        <div class="right">
          <div class="buttons">
            <button @click="performEncryption">Encrypt</button>
            <button @click="performDecryption">Decrypt</button>
          </div>
          <div class="result-container">
            <div>{{ result }}</div>
          </div>
        </div>
      </div>
      <p v-if="state.copyResultSuccess">Copied!</p>
      <p v-if="state.copyPassSuccess">Password copied!</p>
    </div>
    <select v-model="colorMode.preference">
      <option value="system">System</option>
      <option value="light">Light</option>
      <option value="dark">Dark</option>
    </select>
  </div>
</template>

<script setup>
import CryptoJS from 'crypto-js';
import PixelarticonsEye from '@/components/PixelarticonsEye.vue';
import PixelarticonsEyeClosed from '@/components/PixelarticonsEyeClosed.vue';
import PixelarticonsReload from '@/components/PixelarticonsReload.vue';

const colorMode = useColorMode();
const text = ref('');
const password = ref('');
const result = ref('');
const state = reactive({ copyPassSuccess: false, copyResultSuccess: false });
const isPasswordVisible = ref(false);
const passwordFieldType = computed(() => (isPasswordVisible.value ? 'text' : 'password'));
const passwordIconComponent = computed(() => (isPasswordVisible.value ? PixelarticonsEyeClosed : PixelarticonsEye));

const performEncryption = () => {
  const salt = CryptoJS.lib.WordArray.random(128 / 8);
  const key = CryptoJS.PBKDF2(password.value, salt, {
    keySize: 256 / 32,
    iterations: 1000,
  });

  const iv = CryptoJS.lib.WordArray.random(128 / 8);
  const encrypted = CryptoJS.AES.encrypt(text.value, key, { iv: iv });
  result.value = CryptoJS.enc.Base64.stringify(salt.concat(iv).concat(encrypted.ciphertext));
  copyResultToClipboard();
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
    copyResultToClipboard();
  } catch (e) {
    result.value = 'Decryption error!';
  }
};

const generatePassword = async () => {
  password.value = CryptoJS.lib.WordArray.random(19).toString();
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

const copyResultToClipboard = async () => {
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

<style>
body {
  background-color: #fff;
  color: #000;
  font-family: 'VT323', monospace;
  margin: 0;
  overflow: hidden;
}
.dark-mode body {
  background-color: #000;
  color: #fff;
}
.content {
  display: flex;
  flex-wrap: wrap;
  justify-content: center;
}
.main {
  position: absolute;
  top: 50%;
  transform: translateY(-50%);
  padding: 20px;
  border: 2px solid #2b2b2b;
  display: grid;
  justify-content: space-between;
  gap: 16px;
  grid-template-columns: 1fr 1fr;
  max-width: 90%;
  max-height: 90%;
}
.texts {
  gap: 16px;
  display: grid;
}
textarea,
.result-container {
  border: 2px solid #2b2b2b;
  font-size: 18px;
  font-family: 'VT323', monospace;
  background-color: #000;
  color: #fff;
  padding: 10px;
  word-wrap: break-word;
}
textarea:focus,
input:focus {
  outline: none;
}
input {
  font-size: 18px;
  font-family: 'VT323', monospace;
  background-color: #000;
  color: #fff;
  border: 0px;
  padding: 12px 10px;
  width: 100%;
}
.password {
  border: 2px solid #2b2b2b;
  font-size: 18px;
  font-family: 'VT323', monospace;
  background-color: #000;
  color: #fff;
  display: flex;
  align-items: center;
}
.comp {
  display: flex;
  padding: 12px 10px;
}
.comp:active {
  scale: 0.8;
  color: #fff;
}
.passwordcomp {
  display: flex;
}
.right {
  display: flex;
  flex-direction: column;
  gap: 16px;
}
.buttons {
  gap: 16px;
  display: flex;
}
button {
  background-color: black;
  color: white;
  border: 2px solid #2b2b2b;
  font-family: 'VT323', monospace;
  font-size: 18px;
  padding: 18px;
  width: 100%;
}
.result-container {
  height: 100%;
}
</style>
