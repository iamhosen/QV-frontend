<script setup lang="ts">
import axios from "axios";
import { ref } from "vue";

interface ResponseInterface {
  message: string;
  status: number;
  errors?: string[];
  isValid?: boolean;
}

interface HistoryInterface {
  request: string;
  response?: ResponseInterface;
}

const isLoading = ref<boolean>(false);
const input = ref<string>("");
const history = ref<HistoryInterface[]>([]);

const handleSubmit = async () => {
  isLoading.value = true;

  const query: HistoryInterface = {
    request: input.value,
  };

  input.value = "";

  await axios
    .post("http://127.0.0.1:8080/check", {
      query: query.request,
    },
    {
      timeout: 10000,
    })
    .then((res: { data: ResponseInterface }) => {
      query.response = res.data;
      query.response.isValid = true;
      isLoading.value = false;
    })
    .catch((err: { response: { data: ResponseInterface }, code: string }) => {
      if(err.code === 'ECONNABORTED') {
        query.response = {
          message: 'Request timeout',
          status: 408,
          isValid: false,
        };
        isLoading.value = false;
        return;
      }

      query.response = err.response.data;
      query.response.isValid = false;
      isLoading.value = false;
    });

  query.response && history.value.push(query);
};
</script>

<template>
  <div class="terminal">
    <div class="output" id="output">
      <div v-for="(item, index) in history" :key="index" class="request">
        <p>$ {{ item.request }}</p>
        <p
          :class="
            item.response?.isValid ? 'response--valid' : 'response--invalid'
          "
        >
          [{{ item.response?.status }}] {{ item.response?.message }}
        </p>
        <p>{{ item.response?.errors?.join(", ") }}</p>
      </div>
      <p v-if="isLoading">Loading...</p>
    </div>
    <div class="command-examples">Enter the sql command...</div>
    <div class="input">
      <span id="input-prefix">$</span>
      <input
        v-model="input"
        @keydown.enter="handleSubmit"
        type="text"
        id="input"
        placeholder="Type your query..."
        autofocus
      />
    </div>
  </div>
</template>


<style>
body {
  margin: 0;
  display: flex;
  align-items: center;
  justify-content: center;
  height: 100vh;
  background: #000;
  color: #fff;
  font: 14px "Courier New", monospace;
}

.terminal {
  width: 80vw;
  height: 80vh;
  border: 2px solid #0f0;
  background: black;
  overflow: hidden;
  border-radius: 5px;
  box-shadow: 0 0 10px rgba(0, 255, 0, 0.5);
  display: flex;
  flex-direction: column;
}

/* @media (max-width: 600px) {
  .terminal {
  }
} */

.output {
  flex-grow: 1;
  padding: 10px;
  overflow-y: scroll;
  line-height: 1.4;
  color: #0f0;
  padding-left: 10px;
}

.input {
  display: flex;
  background: #000;
  align-items: center;
}

#input {
  flex-grow: 1;
  padding: 10px;
  border: none;
  background: #000;
  color: #fff;
  outline: none;
  caret-color: #0f0;
}

#input::placeholder {
  color: #555;
}

.command-examples {
  align-self: flex-start;
  font-size: 12px;
  color: #555;
  padding: 15px 10px 5px 10px;
  padding-left: 10px;
}

#input-prefix {
  padding-left: 10px;
  padding-right: 5px;
}

.output .request {
  margin-bottom: 3em !important;
}

.output .response--valid {
  color: #0f0 !important;
  font-weight: 700 !important;
}

.output .response--invalid {
  color: #f00 !important;
}
</style>