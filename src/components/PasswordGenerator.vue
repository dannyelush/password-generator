<template>
  <div class="container">
    <h2>Password Generator</h2>
    <div class="wrapper">
      <div class="input-box">
        <input type="text" spellcheck="false" readonly v-model="password"/>
        <span class="material-symbols-rounded" ref="copyBtn" @click.prevent="copy">copy_all</span>
      </div>
      <div class="pass-indicator" :id="passIndicatorId"></div>
      <div class="pass-length">
        <div class="details">
          <label>Password Length</label>
          <span>{{ passwordLength }}</span>
        </div>
        <input type="range" min="1" :max="sliderMax" v-model="passwordLength" step="1" ref="lengthSlider" :style="{ background: `linear-gradient(to right, #9b59b6 ${percent}%, #DFDFDF ${percent}%)`}"/>
      </div>
      <div class="pass-settings">
        <label>Password Settings</label>
        <ul class="options">
          <li class="option">
            <input type="checkbox" id="lowercase" value="lowercase" v-model="options"/>
            <label for="lowercase">Lowercase (a-z)</label>
          </li>
          <li class="option">
            <input type="checkbox" id="uppercase" value="uppercase" v-model="options"/>
            <label for="uppercase">Uppercase (A-Z)</label>
          </li>
          <li class="option">
            <input type="checkbox" id="numbers" value="numbers" v-model="options"/>
            <label for="numbers">Numbers (0-9)</label>
          </li>
          <li class="option">
            <input type="checkbox" id="symbols" value="symbols" v-model="options"/>
            <label for="symbols">Symbols (!-$^+)</label>
          </li>
          <li class="option">
            <input type="checkbox" id="exc-duplicate" value="exc-duplicate" v-model="options"/>
            <label for="exc-duplicate">Exclude Duplicate</label>
          </li>
          <li class="option">
            <input type="checkbox" id="spaces" value="spaces" v-model="options"/>
            <label for="spaces">Include Spaces</label>
          </li>
        </ul>
      </div>
      <button class="generate-btn" type="button" @click.prevent="generatePassword">Generate Password</button>
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref, computed, watch, onMounted } from "vue";

interface charactersInfo {
  lowercase: string;    
  uppercase: string;
  numbers: string;
  symbols: string;
}

const characters: charactersInfo = {
    lowercase: "abcdefghijklmnopqrstuvwxyz",
    uppercase: "ABCDEFGHIJKLMNOPQRSTUVWXYZ",
    numbers: "0123456789",
    symbols: "^!$%&|[](){}:;.,*+-#@<>~"
}

const password = ref('');
const passwordLength = ref(15);
const sliderMax = 30;
const options = ref(['lowercase'])
const percent = computed(() => (passwordLength.value / sliderMax) * 100);
const copyBtn = ref<HTMLInputElement | null>(null);

const passIndicatorId = computed(() => {
  return passwordLength.value <= 8 ? "weak" : passwordLength.value <= 16 ? "medium" : "strong";
});

function generatePassword() {
  let staticPassword = "",
  randomPassword = "",
  excludeDuplicate = false,
  passLength = passwordLength.value;

  options.value.forEach((option: string) => {
      if(option !== "exc-duplicate" && option !== "spaces") {
        staticPassword += characters[option];
      } else if(option === "spaces") {
        staticPassword = `  ${staticPassword}  `;
      } else {
        excludeDuplicate = true;
      }
  });

  for (let i = 0; i < passLength; i++) {
      let randomChar = staticPassword[Math.floor(Math.random() * staticPassword.length)];
      if(excludeDuplicate) {
        !randomPassword.includes(randomChar) || randomChar == " " ? randomPassword += randomChar : i--;
      } else {
        randomPassword += randomChar;
      }
  }
  password.value = randomPassword;
}

watch(passwordLength, () => {
  generatePassword();
});

function copy() {
  navigator.clipboard.writeText(password.value).then(() => {
    if (copyBtn && copyBtn.value) {
      copyBtn.value.innerText = "check";
      copyBtn.value.style.color = "#9b59b6";
      setTimeout(() => {
        if (copyBtn && copyBtn.value) {
          copyBtn.value.innerText = "copy_all";
          copyBtn.value.style.color = "#707070";
        }
      }, 1500);
    }
  }).catch(err => {
      alert(`Couldn't copy the password: ${err}`);
  });
}

onMounted(() => {
  generatePassword()
})

</script>

<style scoped>
.container {
  width: 450px;
  background: #fff;
  border-radius: 8px;
  box-shadow: 0 10px 20px rgba(0, 0, 0, 0.05);
}
.container h2 {
  font-weight: 600;
  font-size: 1.31rem;
  padding: 1rem 1.75rem;
  border-bottom: 1px solid #d4dbe5;
}
.wrapper {
  margin: 1.25rem 1.75rem;
}
.wrapper .input-box {
  position: relative;
}
.input-box input {
  width: 100%;
  height: 53px;
  outline: none;
  font-weight: 500;
  border-radius: 4px;
  font-size: 1.06rem;
  pointer-events: none;
  letter-spacing: 1.4px;
  border: 1px solid #aaa;
  padding: 0 2.85rem 0 1rem;
}
.input-box span {
  right: 13px;
  z-index: 999;
  user-select: none;
  position: absolute;
  line-height: 53px;
  cursor: pointer;
  color: #707070;
}
.input-box span:hover {
  color: #9b59b6 !important;
}
.wrapper .pass-indicator {
  display: flex;
  height: 4px;
  width: 100%;
  position: relative;
  background: #dfdfdf;
  margin-top: 0.75rem;
  border-radius: 25px;
}
.pass-indicator::before {
  position: absolute;
  content: "";
  height: 100%;
  width: 0%;
  border-radius: inherit;
  transition: width 0.3s ease;
}
.pass-indicator#weak::before {
  width: 20%;
  background: #e64a4a;
}
.pass-indicator#medium::before {
  width: 50%;
  background: #f1c80b;
}
.pass-indicator#strong::before {
  width: 100%;
  background: #2a9241;
}
.wrapper .pass-length {
  margin: 1.56rem 0 1.25rem;
}
.pass-length .details {
  display: flex;
  justify-content: space-between;
}
.pass-length input {
  width: 100%;
  height: 5px;
  margin-top: 0.63rem;
  -webkit-appearance: none;
  background: #dfdfdf;
  border-radius: 15px;
  background: linear-gradient(
    to right,
    #9b59b6 50%,
    rgb(223, 223, 223) 50%
  );
}
.pass-length input::-webkit-slider-thumb {
  appearance: none;
  height: 20px;
  width: 20px;
  cursor: pointer;
  border-radius: 50%;
  background: #fff;
  border: 3px solid #9b59b6;
  box-shadow: 0 0 4px rgba(0, 0, 0, 0.1);
}
.pass-length input::-moz-range-thumb {
  height: 13px;
  width: 13px;
  cursor: pointer;
  border-radius: 50%;
  background: #fff;
  border: 3px solid #9b59b6;
  box-shadow: 0 0 4px rgba(0, 0, 0, 0.1);
}
.pass-settings .options {
  display: flex;
  list-style: none;
  flex-wrap: wrap;
  margin-top: 1rem;
}
.options .option {
  display: flex;
  align-items: center;
  width: calc(100% / 2);
  margin-bottom: 1rem;
}
.options .option:nth-child(even) {
  width: auto;
  margin-left: 0.6rem;
}
.options .option:nth-child(1) input {
  opacity: 0.8;
}
.options .option:nth-child(1) {
  pointer-events: none;
}
.options .option input {
  height: 16px;
  width: 16px;
  cursor: pointer;
  accent-color: #9b59b6;
}
.options .option label {
  color: #4f4f4f;
  cursor: pointer;
  padding-left: 0.625rem;
}
.wrapper .generate-btn {
  width: 100%;
  color: #fff;
  cursor: pointer;
  border: none;
  padding: 0.94rem 0;
  font-size: 1.06rem;
  border-radius: 5px;
  background: #9b59b6;
  text-transform: uppercase;
  margin: 0.94rem 0 1.3rem;
  transition: all 0.3s ease;
}
.generate-btn:hover {
  background: #7d4892;
}

@media screen and (max-width: 600px) {
  .container h2 {
    padding: 1rem 1.5rem;
  }
  .wrapper {
    margin: 1.25rem 1.5rem;
  }
  .options .option {
    margin-bottom: 1.1rem;
  }
  .options .option label {
    padding-left: 0.42rem;
  }
  .wrapper .generate-btn {
    font-size: 1rem;
    margin: 0.8rem 0 1rem;
  }
}
</style>
