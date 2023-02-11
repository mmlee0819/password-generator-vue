<script setup lang="ts">
import { ref } from "vue";
import clipboardImg from "./assets/clipboard-green.png";
import copyImg from "./assets/copy.png";

const uppercaseList = [..."ABCDEFGHIJKLMNOPQRSTUVWXYZ"];
const lowercaseList = [..."abcdefghijklmnopqrstuvwxyz"];
const numbersList = [...Array(10).keys()];
const symbolsList = ["!", "@", "#", "$", "%", "^", "&", "*", "~"];
const initialOptions = [
  {
    id: "uppercase",
    text: "uppercase letters",
    isChecked: false,
    range: uppercaseList,
  },
  {
    id: "lowercase",
    text: "lowercase letters",
    isChecked: false,
    range: lowercaseList,
  },
  { id: "numbers", text: "numbers", isChecked: false, range: numbersList },
  { id: "symbols", text: "symbols", isChecked: false, range: symbolsList },
];

const options = ref(initialOptions);
const password = ref("");
const pwLength = ref(8);
const reminder = ref({
  status: "",
  text: "",
});
const getRandomNum = (max: number) => {
  return Math.floor(Math.random() * max);
};

const copyTextToClipboard = async (text: string) => {
  if ("clipboard" in navigator) {
    return await navigator.clipboard.writeText(text);
  } else {
    return document.execCommand("copy", true, text);
  }
};
const handleCheckbox = (e: Event) => {
  const target = e.target as HTMLInputElement;
  const newOptions = options.value.map((item) =>
    item.id === target.id
      ? {
          ...item,
          isChecked: target?.checked,
        }
      : item
  );
  options.value = newOptions;
};
const clickGenerate = () => {
  const selectedOptions = options.value.filter((item) => item.isChecked);

  if (selectedOptions.length === 0) {
    password.value = "";
    reminder.value = {
      status: "isInvalid",
      text: "At least 1 option is required.",
    };
    return;
  }
  if (pwLength.value < 4 || pwLength.value > 20) {
    reminder.value = {
      status: "isInvalid",
      text: "The length should between 4-20.",
    };
    return;
  }
  const newPW: (string | number)[] = [];
  if (selectedOptions.length === 1) {
    for (let i = 1; i <= pwLength.value; i++) {
      const randomNum = getRandomNum(selectedOptions[0].range.length);
      newPW.push(selectedOptions[0].range[randomNum]);
    }
  }
  if (selectedOptions.length === 2) {
    for (let i = 1; i <= pwLength.value; i++) {
      if (i % 2 === 1) {
        const randomNum = getRandomNum(selectedOptions[1].range.length);
        newPW.push(selectedOptions[1].range[randomNum]);
      } else {
        const randomNum = getRandomNum(selectedOptions[0].range.length);
        newPW.push(selectedOptions[0].range[randomNum]);
      }
    }
  }
  if (selectedOptions.length === 3) {
    for (let i = 1; i <= pwLength.value; i++) {
      if (i % 3 === 1) {
        const randomNum = getRandomNum(selectedOptions[1].range.length);
        newPW.push(selectedOptions[1].range[randomNum]);
      } else if (i % 3 === 2) {
        const randomNum = getRandomNum(selectedOptions[0].range.length);
        newPW.push(selectedOptions[0].range[randomNum]);
      } else {
        const randomNum = getRandomNum(selectedOptions[2].range.length);
        newPW.push(selectedOptions[2].range[randomNum]);
      }
    }
  }
  if (selectedOptions.length === 4) {
    for (let i = 1; i <= pwLength.value; i++) {
      if (i % 4 === 1) {
        const randomNum = getRandomNum(selectedOptions[1].range.length);
        newPW.push(selectedOptions[1].range[randomNum]);
      } else if (i % 4 === 2) {
        const randomNum = getRandomNum(selectedOptions[0].range.length);
        newPW.push(selectedOptions[0].range[randomNum]);
      } else if (i % 4 === 3) {
        const randomNum = getRandomNum(selectedOptions[2].range.length);
        newPW.push(selectedOptions[2].range[randomNum]);
      } else {
        const randomNum = getRandomNum(selectedOptions[3].range.length);
        newPW.push(selectedOptions[3].range[randomNum]);
      }
    }
  }
  const stringNewPW = newPW.join("");
  password.value = stringNewPW;
  reminder.value = {
    status: "",
    text: "",
  };
};
const clickCopy = async () => {
  if (password.value.length === 0) {
    reminder.value = {
      status: "isInvalid",
      text: "No password generated yet.",
    };
    return;
  }
  await copyTextToClipboard(password.value);
  reminder.value = {
    status: "isCopied",
    text: "Copied password",
  };
};
</script>

<template>
  <div class="App">
    <div class="container">
      <h1>Password Generator</h1>
      <div class="result-wrapper">
        <div
          class="tooltip"
          :style="{
            visibility: reminder.status === '' ? 'hidden' : 'visible',
            opacity: reminder.status === '' ? '0' : '1',
            transition: 'opacity 0.5s',
          }"
        >
          {{ reminder.text }}
          <div class="tooltip-angle"></div>
        </div>
        <span class="result">{{ password }}</span>
        <img
          :src="reminder.status === 'isCopied' ? clipboardImg : copyImg"
          alt="copy"
          @click="clickCopy"
        />
      </div>
      <div class="optionsWrapper">
        <div class="pw-length-wrapper">
          Password length
          <input
            v-model="pwLength"
            type="number"
            min="4"
            max="20"
            defaultValue="8"
          />
        </div>
        <label v-for="item in options" :key="item.id">
          {{ `Include ${item.text}` }}
          <input type="checkbox" :id="item.id" @change="handleCheckbox" />
        </label>
      </div>
      <div class="btn-generate" @click="clickGenerate">Generate password</div>
    </div>
  </div>
</template>

<style scoped>
* {
  box-sizing: border-box;
}

.App {
  display: flex;
  justify-content: center;
  font-family: sans-serif;
  text-align: center;
}
.container {
  display: flex;
  flex-flow: column wrap;
  align-items: center;
  padding: 20px;
  width: 350px;
  max-width: 100%;
  color: #fff;
  background-color: #272525;
  box-shadow: 0px 2px 10px rgb(255 255 255 / 20%);
}
.result-wrapper {
  position: relative;
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding-left: 5px;
  width: 310px;
  background-color: #120f0f;
  border: 1px solid #cccccc;
}
.result {
  display: flex;
  max-width: calc(100% - 40px);
  text-align: start;
  word-break: break-word;
  background: none;
}
.optionsWrapper {
  display: flex;
  flex-flow: column wrap;
  align-items: start;
  margin: 15px 0;
  gap: 15px;
}

.pw-length-wrapper,
label {
  display: flex;
  justify-content: space-between;
  width: 100%;
}
img {
  width: 40px;
  height: 40px;
  cursor: pointer;
}

h1 {
  margin: 10px 0 20px 0;
  font-size: 24px;
}
input[type="checkbox"] {
  margin: 3px 3px 3px 4px;
}
input[type="number"] {
  background-color: #fff;
}

.btn-generate {
  padding: 8px 12px;
  color: #80ffad;
  background-color: #2d2d2d;
  cursor: pointer;
}

.tooltip {
  position: absolute;
  right: 45px;
  padding: 0 5px;
  line-height: 30px;
  color: #fff;
  background-color: #00bd7e;
  opacity: "0";
  transition: "opacity 1s";
  z-index: 1;
}
.tooltip-angle {
  position: absolute;
  top: 10px;
  right: -5px;
  width: 10px;
  height: 10px;
  background-color: #00bd7e;
  transform: rotate(45deg);
  z-index: -1;
}
</style>
