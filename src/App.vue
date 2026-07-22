<script setup>
import { computed, ref, onMounted, watch } from "vue";

const myLink = ref("");
const isCopied = ref(false);
const hasUrlParseError = ref(false);

function isValidURL(link) {
  if (!link?.length) {
    hasUrlParseError.value = false;
    return;
  }
  try {
    new URL(link);
    hasUrlParseError.value = false;
    return true;
  } catch (_) {
    hasUrlParseError.value = true;
    return false;
  }
}

const expressLink = computed(() => {
  const parsedUrl = isValidURL(myLink.value)
    ? new URL(myLink.value)
    : undefined;
  if (parsedUrl) {
    const pathname = parsedUrl?.pathname.replace(/[0-9a-fA-F]{32}/, "default");
    const search = parsedUrl?.search.replace("?", "%3F").replace(/&/gi, "%26");
    return `${parsedUrl.origin}/registration?destination=${pathname}${search}`;
  }
  return "";
});

const hasLink = computed(() => !!expressLink.value);

const copiedValue = computed(() => {
  if (!expressLink.value) {
    return "";
  }
  return isCopied.value ? "Copied" : "Copy";
});

async function handleCopy() {
  try {
    await navigator.clipboard.writeText(expressLink.value);
    isCopied.value = true;
    // Reset status after 2 seconds
    setTimeout(() => {
      isCopied.value = false;
    }, 2000);
  } catch (error) {
    console.error("Failed to copy text: ", error);
  }
}

onMounted(() => {
  document.title = "Express Links";
});
</script>

<template>
  <header class="header">
    <h1>Преобразователь ссылок в expresslink</h1>
  </header>
  <div class="form-container">
    <div class="input-container">
      <label class="label_required" for="link">Вставь ссылку</label>
      <input
        id="link"
        class="input"
        v-model="myLink"
        type="text"
        placeholder="https://..."
      />
    </div>

    <div class="input-container">
      <div>Expresslink</div>
      <div v-if="hasLink" @click="handleCopy">
        <span title="Click to copy" class="pointer">{{ expressLink }}</span>
        <button style="margin-left: 4px" class="pointer button">
          {{ copiedValue }}
        </button>
      </div>
      <div class="error-text" v-if="hasUrlParseError">
        Не удается распарсить URL
      </div>
      <div class="empty-text" v-if="!hasLink && !hasUrlParseError">
        Тут будет ссылка...
      </div>
    </div>
  </div>
</template>

<style scoped>
.header {
  padding: 0 20px;
}
.form-container {
  display: flex;
  flex-direction: column;
  gap: 12px;
  padding: 20px;
  font-family: sans-serif;
}

.input-container {
  display: flex;
  flex-direction: column;
  gap: 7px;
}

.pointer {
  cursor: pointer;
}

.input {
  padding: 8px 12px;
  font-size: 16px;
  min-width: 300px;
}

.error-text {
  color: orange;
}

.empty-text {
  color: grey;
  opacity: 0.5;
}

.button {
  border-radius: 3px;
  opacity: 0.5;
}
</style>
