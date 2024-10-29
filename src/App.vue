<script setup lang="ts">
  import IconClipboard from '~icons/mdi/clipboard'
  import { useClipboard } from '@vueuse/core'
  import { ref } from "vue"

  const url = ref("")
  const result = ref("")
  const errorMessage = ref("")
  let timerId: number | null = null

  const { copy, copied } = useClipboard({ copiedDuring: 10000 })

  const allowedHost = [
    "amazon.co.jp",
    "amazon.com",
    "www.amazon.co.jp",
    "www.amazon.com"
  ]

  const handleInput = () => {
    if (timerId !== null) {
      clearTimeout(timerId)
      timerId = null
    }

    setTimeout(() => {
      errorMessage.value = ""

      try {
        result.value = ""
        const parsed = new URL(url.value)

        if (!allowedHost.includes(parsed.host)) {
          errorMessage.value = "このURLは対応していません"
        }

        const filtered = parsed.pathname
          .split("/")
          .filter(value => 
            value.length < 20 &&
            !value.includes("=")
          )
          .join("/")

        result.value = parsed.origin + filtered
      } catch (e) {
        if (e instanceof TypeError) {
          errorMessage.value = "URLを入力してください"
        } else {
          errorMessage.value = "エラーが発生しました"
        }
      }

      timerId = null
    }, 500)
  }
</script>

<template>
  <section>
    <h1>
      <img src="./assets/torimochi.png" alt="Torimochi" class="logo">
    </h1>
    <form @input="handleInput" class="input-field">
      <input v-model="url" name="url" type="url" placeholder="https://amazon.com/xxx/too-long-text/xxx" />
      <div v-if="errorMessage" class="output-field">
        <output class="output-field__output output-field__output--error">{{ errorMessage }}</output>
      </div>
      <div v-else-if="result" class="output-field">
        <output class="output-field__output">{{ result }}</output>
        <button type="button" @click="copy(result)" aria-label="Copy to clipboard"><IconClipboard /></button>
      </div>
      <p v-if="copied" class="status status--succeeded" role="status">Copied!</p>
    </form>
  </section>
</template>

<style scoped>
.input-field {
  width: 100%;
  max-width: 900px;
  display: flex;
  flex-direction: column;
}

.input-field > * + * {
  margin-top: 1em;
}

.output-field {
  background: #333;
  border: 1px solid #888;
  box-shadow: 5px 5px 5px black;
  display: flex;
  justify-content: center;
  overflow-wrap: anywhere;
  padding: 1em;
}

.output-field > * + * {
  margin-left: 1em;
}

.output-field > button {
  font-size: 1em;
  cursor: pointer;
  padding: 1em;
  line-height: 1.1em;
}

.output-field__output {
  padding: 1em;
}

.logo {
  width: 50%;
}

.status {
  padding: 1em;
  border: solid 1px;
  box-shadow: 5px 5px 5px black;
}

.status--succeeded {
  background: #9cfc9a;
  color: #0f210e;
}
</style>
