<template>
  <q-dialog v-model="alert">
    <q-card>
      <q-card-section class="row items-center q-pb-none">
        <div class="text-h6">{{ title }}</div>
        <q-space />
        <q-btn icon="close" flat round dense v-close-popup />
      </q-card-section>
      <q-separator color="transparent" spaced="lg" inset />
      <q-card-section class="q-pt-none" v-for="item in content" :key="item">
        {{ item }}
      </q-card-section>
    </q-card>
  </q-dialog>
</template>

<script>
import { ref } from "vue";

export default {
  data() {
    return {
      alert: ref(false),
    };
  },
  methods: {
    // Usage: show("Title", ["Content line 1", "Contnent line 2"])
    show(title, content) {
      this.title = title;
      this.content = content;
      this.alert = true;
    },
    hide() {
      this.alert = false;
    },
  },
};
</script>
