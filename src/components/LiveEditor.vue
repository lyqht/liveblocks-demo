<script setup lang="ts">
import { createClient } from "@liveblocks/client";
import LiveblocksProvider from "@liveblocks/yjs";
import * as Y from "yjs";
import * as monaco from "monaco-editor";
import { MonacoBinding } from "y-monaco";
import { onMounted, onUnmounted, ref } from "vue";

const element = ref<HTMLElement | null>(null);
const editor = ref<monaco.editor.IStandaloneCodeEditor | null>(null);
const binding = ref<MonacoBinding | null>(null);
const leave = ref<Function | null>(null);

// Set up Liveblocks client
const client = createClient({
  publicApiKey: "pk_dev_vhahfCWbQwH92u95KEgTmEDD",
});

// Enter a multiplayer room
const info = client.enterRoom("my-room", {
  initialPresence: {},
});
const room = info.room;
leave.value = info.leave;

// Set up Yjs document, shared text, and Liveblocks Yjs provider
const yDoc = new Y.Doc();
const yText = yDoc.getText("monaco");
const yProvider = new LiveblocksProvider(room, yDoc);

onMounted(() => {
  if (element.value === null) {
    return;
  }

  // Set up the Monaco editor
  editor.value = monaco.editor.create(element.value, {
    value: "",
    language: "markdown"
  })

  // Attach Yjs to Monaco
  const editorModel = editor.value.getModel();
  if (editorModel != null) {
      binding.value = new MonacoBinding(
        yText,
        editorModel,
        new Set([editor.value]),
        yProvider.awareness
      );
  }

});

onUnmounted(() => {
  binding.value?.destroy();
  editor.value?.dispose();
  leave.value?.();
});
</script>

<template>
  <div ref="element" />
</template>
