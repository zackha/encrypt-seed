<template>
  <div class="content">
    <div class="main">
      <div class="program">
        <div class="left">
          <textarea v-model="text" rows="6" type="text" placeholder="Text to encrypt or encrypted text"></textarea>
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
            <button @click="performDecryption" style="border-left: 1px solid">Decrypt</button>
          </div>
          <div class="result-container">
            {{ result }}
          </div>
        </div>
      </div>
      <div class="footer">
        <div>
          <div v-if="state.copyResultSuccess">Copied!</div>
          <div v-if="state.copyPassSuccess">Password copied!</div>
        </div>
        <div class="footer-right">
          <span @click="isDark = !isDark" style="display: flex">
            <PixelarticonsMoon v-if="isDark" />
            <PixelarticonsSunAlt v-else />
          </span>
          <a class="link" href="https://github.com/zackha/encrypt-seed" target="_blank" rel="noopener noreferrer"> github </a>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import CryptoJS from 'crypto-js';
import PixelarticonsEye from '@/components/PixelarticonsEye.vue';
import PixelarticonsEyeClosed from '@/components/PixelarticonsEyeClosed.vue';
import PixelarticonsReload from '@/components/PixelarticonsReload.vue';
import PixelarticonsMoon from './components/PixelarticonsMoon.vue';
import PixelarticonsSunAlt from './components/PixelarticonsSunAlt.vue';

const colorMode = useColorMode();
const text = ref('');
const password = ref('');
const result = ref('');
const state = reactive({ copyPassSuccess: false, copyResultSuccess: false });
const isPasswordVisible = ref(true);
const passwordFieldType = computed(() => (isPasswordVisible.value ? 'text' : 'password'));
const passwordIconComponent = computed(() => (isPasswordVisible.value ? PixelarticonsEyeClosed : PixelarticonsEye));

const isDark = computed({
  get() {
    return colorMode.value === 'dark';
  },
  set() {
    colorMode.preference = colorMode.value === 'dark' ? 'light' : 'dark';
  },
});

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
* {
  font-family: 'VT323', monospace;
  word-break: break-word;
}
body {
  background-color: #fff;
  color: #000;
}
.dark-mode body {
  background-color: #000;
  color: #fff;
}
.content {
  display: flex;
  justify-content: center;
}
.main {
  position: absolute;
  top: 50%;
  transform: translateY(-50%);
  display: flex;
  flex-direction: column;
  gap: 12px;
}
.footer {
  display: flex;
  align-items: center;
  justify-content: space-between;
}
.program {
  border: 1px solid;
  height: 200px;
  display: flex;
  justify-content: center;
}
.left {
  display: flex;
  border-right: 1px solid;
  flex-direction: column;
}
textarea,
.result-container {
  font-size: 18px;
  resize: none;
  border: 0px;
  margin: 10px 10px 10px 10px;
  padding: 0px 7px 0px 0px;
  flex: 1;
  overflow-x: hidden;
}
.dark-mode textarea,
.dark-mode .result-container,
.dark-mode input,
.dark-mode .password,
.dark-mode button,
.dark-mode .link {
  background-color: #000;
  color: #fff;
}
textarea:focus,
input:focus {
  outline: none;
}
input {
  font-size: 18px;
  border: 0px;
  padding: 10px;
  flex: 1;
  width: 280px;
}
.password {
  font-size: 18px;
  display: flex;
  align-items: center;
  border-top: 1px solid;
}
.comp {
  display: flex;
  padding: 10px;
}
.comp:active {
  scale: 0.8;
  color: grey;
}
.passwordcomp {
  display: flex;
}
.right {
  display: flex;
  flex-direction: column;
}
.buttons {
  display: flex;
  border-bottom: 1px solid;
}
button {
  font-size: 18px;
  padding: 10px;
  flex: 1;
  border: 0px;
}
.result-container {
  overflow: auto;
  width: 200px;
}
.link {
  padding-bottom: 2px;
  text-decoration-line: none;
  color: #000;
}
.footer-right {
  display: flex;
  align-items: center;
  gap: 12px;
}
::-webkit-scrollbar {
  width: 2px;
}
::-webkit-scrollbar-thumb {
  background-color: #fff;
}
</style>
