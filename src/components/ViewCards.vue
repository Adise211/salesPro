<template>
  <div class="view-card">
    <v-card elevation="5" :max-width="cardMaxWidth" max-height="100%" :height="cardHeight">
      <v-card-title>
        <slot name="card-title"></slot>
      </v-card-title>
      <v-card-subtitle>
        <slot name="card-sub-title"></slot>
      </v-card-subtitle>
      <v-card-text>
        <slot name="card-text"></slot>
      </v-card-text>
      <v-card-actions v-if="withActions">
        <slot name="card-actions"></slot>
      </v-card-actions>
    </v-card>
  </div>
</template>

<script>
export default {
  components: {},
  props: {
    cardMaxWidth: {
      type: [Number, String],
      default: "100%"
    },
    withActions: {
      type: Boolean,
      default: false
    },
    cardHeight: {
      type: [Number, String],
      default: ""
    }
  },
  data: () => ({}),
  created() {},
  mounted() {},
  methods: {},
  computed: {},
  watch: {}
};
</script>

<style scoped></style>
