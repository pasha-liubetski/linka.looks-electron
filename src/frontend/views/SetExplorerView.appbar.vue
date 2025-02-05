<template>
  <v-app-bar>
    <v-btn
      flat
      icon
      @click="back"
    >
      <v-icon>mdi-arrow-left</v-icon>
    </v-btn>
    <v-app-bar-title>
      {{ title }}
    </v-app-bar-title>
    <v-spacer />
    <notes-button
      v-if="config?.description"
      :config="config"
    />
    <v-btn
      flat
      icon
      :color="interfaceOutputLine ? 'primary' : ''"
      :title="(interfaceOutputLine ? 'Скрыть' : 'Показать') + ' строку вывода'"
      @click="switchInterfaceOutputLine"
    >
      <v-icon>mdi-form-textbox</v-icon>
    </v-btn>
    <v-btn
      flat
      icon
      :color="buttonEnabled ? 'primary' : ''"
      :title="
        (buttonEnabled ? 'Выключить' : 'Включить') + ' управление глазами'
      "
      @click="switchButtonEnabled"
    >
      <v-icon>{{ buttonEnabled ? 'mdi-eye' : 'mdi-eye-off' }}</v-icon>
    </v-btn>
    <v-btn
      flat
      icon
      :color="animation ? 'primary' : ''"
      :title="(animation ? 'Выключить' : 'Включить') + ' анимацию изображений'"
    >
      <v-icon @click="switchAnimation">
        {{ animation ? 'mdi-pause' : 'mdi-play' }}
      </v-icon>
    </v-btn>
    <v-spacer />
    <share-button />
    <v-btn
      flat
      icon
      :to="editLink"
    >
      <v-icon>mdi-pencil</v-icon>
    </v-btn>
    <folder-button
      :file="file"
      @move="move"
    />
    <delete-button
      :file="title"
      @delete="del"
    />
  </v-app-bar>
</template>

<script lang="ts" setup>
import { computed } from "vue";
import DeleteButton from "@/frontend/components/SetExplorer/DeleteButton.vue";
import FolderButton from "@/frontend/components/SetExplorer/FolderButton.vue";
import NotesButton from "@/frontend/components/SetExplorer/NotesButton.vue";

import { useStore } from "vuex";
import { useRoute, useRouter } from "vue-router";

import { storageService } from "@/frontend/services/card-storage-service";
import ShareButton from "@/frontend/components/ShareButton.vue";
import { Metric } from "@/frontend/utils/Metric";
import pathModule from "path";

const route = useRoute();
const router = useRouter();
const store = useStore();

const file = computed(() => route.params.path.toString());
const config = computed(() => store.state.explorer.config);

const title = computed(() => {
  const arr = file.value.split("§");
  return pathModule.basename(arr[arr.length - 1]);
});

const editLink = computed(() => {
  return route.fullPath.replace("set", "edit");
});

const interfaceOutputLine = computed(() => {
  return store.state.ui.outputLine;
});

const buttonEnabled = computed(() => {
  return store.state.button.enabled;
});

const animation = computed(() => {
  return store.state.button.animation;
});

function switchButtonEnabled () {
  store.dispatch("button_enabled");
}

function switchInterfaceOutputLine () {
  store.dispatch("interface_outputLine");
}

function back () {
  if (file.value.includes(":")) {
    router.push("/");
    return;
  }
  router.push("/" + file.value.split("§").slice(0, -1).join("§"));
}

function switchAnimation () {
  store.dispatch("button_animation_toggle");
}

async function del () {
  await storageService.moveToTrash(file.value);
  back();
  Metric.registerEvent(store.state.pcHash, "trash");
}

async function move (location: string) {
  const target = await storageService.moveSet(file.value, location);
  const url = target
    .slice(target.lastIndexOf("LINKa") + 5)
    .replaceAll("/", "§")
    .replace("\\", "§");

  router.push("/set/" + url);
  Metric.registerEvent(store.state.pcHash, "move");
}
</script>
